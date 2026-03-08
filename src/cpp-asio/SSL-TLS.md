## SSL/TLS

SSL (Secure Sockets Layer) e TLS (Transport Layer Security) são dois protocolos de segurança amplamente utilizados na internet para proteger as comunicações entre dispositivos. Ambos são baseados em certificados de segurança e chaves criptográficas, que são usados para criptografar e decifrar os dados transmitidos.

A principal diferença entre SSL e TLS é que TLS é uma versão atualizada e aprimorada do SSL. O SSL foi originalmente criado pela Netscape nos anos 1990 como uma maneira de proteger as comunicações na internet. No entanto, com o tempo, problemas de segurança foram encontrados no SSL, levando ao desenvolvimento do TLS como substituto.

Atualmente, **SSLv2 e SSLv3 são considerados inseguros** e devem ser desabilitados. A maioria dos sistemas usa **TLS 1.2** como mínimo, com **TLS 1.3** como versão preferida por ser mais rápida e segura. O Asio suporta TLS 1.3 via OpenSSL 1.1.1+.

### Asio SSL

A biblioteca Asio SSL fornece uma API que permite aos desenvolvedores criar e gerenciar conexões seguras usando TLS. Ela inclui funções para realizar handshakes SSL/TLS, criptografar e decifrar dados e verificar a validade de certificados de segurança.

Para usar SSL/TLS com Asio, inclua `<asio/ssl.hpp>` (standalone) ou `<boost/asio/ssl.hpp>` (Boost.Asio) e vincule com a biblioteca OpenSSL.

#### Exemplo: cliente SSL/TLS assíncrono (abordagem moderna)

O padrão moderno usa `async_handshake()` em vez do handshake síncrono:

```c++
#include <asio.hpp>
#include <asio/ssl.hpp>
#include <iostream>
#include <string>

namespace ssl = asio::ssl;
using tcp = asio::ip::tcp;

int main()
{
    asio::io_context io_context;

    // Use 'tls_client' para conexões de cliente (ou 'tls' para uso genérico)
    // Evite 'sslv23' — está depreciado
    ssl::context ctx(ssl::context::tls_client);

    // Opções de segurança: desabilitar protocolos antigos e vulneráveis
    ctx.set_options(
        ssl::context::default_workarounds
        | ssl::context::no_sslv2
        | ssl::context::no_sslv3
        | ssl::context::single_dh_use);

    // Verificação de certificado do servidor (recomendado em produção)
    ctx.set_verify_mode(ssl::verify_peer);
    ctx.set_default_verify_paths();

    // Socket SSL sobre TCP
    ssl::stream<tcp::socket> socket(io_context, ctx);

    // Resolver e conectar ao host
    tcp::resolver resolver(io_context);
    auto endpoints = resolver.resolve("exemplo.com", "443");
    asio::connect(socket.lowest_layer(), endpoints);

    // SNI (Server Name Indication) — necessário para servidores que hospedam
    // múltiplos certificados no mesmo IP
    SSL_set_tlsext_host_name(socket.native_handle(), "exemplo.com");

    // Handshake assíncrono (abordagem recomendada)
    socket.async_handshake(ssl::stream_base::client,
        [&socket](const asio::error_code& ec) {
            if (ec) {
                std::cerr << "Erro no handshake: " << ec.message() << '\n';
                return;
            }
            std::cout << "Handshake TLS bem-sucedido!\n";

            // Enviar dados após o handshake
            std::string request = "GET / HTTP/1.1\r\nHost: exemplo.com\r\n\r\n";
            asio::async_write(socket, asio::buffer(request),
                [](const asio::error_code& ec, std::size_t bytes) {
                    if (!ec)
                        std::cout << "Enviados " << bytes << " bytes\n";
                });
        });

    io_context.run();
    return 0;
}
```

> **Atenção:** `asio::ssl::context::sslv23` está **depreciado**. Use `ssl::context::tls` (genérico), `ssl::context::tls_client` (para clientes) ou `ssl::context::tls_server` (para servidores).

#### Exemplo: servidor SSL/TLS assíncrono

```c++
#include <asio.hpp>
#include <asio/ssl.hpp>
#include <iostream>
#include <memory>

namespace ssl = asio::ssl;
using tcp = asio::ip::tcp;

class SessaoSSL : public std::enable_shared_from_this<SessaoSSL>
{
public:
    SessaoSSL(tcp::socket socket, ssl::context& ctx)
        : socket_(std::move(socket), ctx) {}

    void iniciar()
    {
        auto self = shared_from_this();
        // Handshake assíncrono como servidor
        socket_.async_handshake(ssl::stream_base::server,
            [self](const asio::error_code& ec) {
                if (!ec) self->ler();
            });
    }

private:
    void ler()
    {
        auto self = shared_from_this();
        socket_.async_read_some(asio::buffer(dados_),
            [self](const asio::error_code& ec, std::size_t n) {
                if (!ec) self->escrever(n);
            });
    }

    void escrever(std::size_t n)
    {
        auto self = shared_from_this();
        asio::async_write(socket_, asio::buffer(dados_, n),
            [self](const asio::error_code& ec, std::size_t) {
                if (!ec) self->ler();
            });
    }

    ssl::stream<tcp::socket> socket_;
    char dados_[1024];
};

int main()
{
    asio::io_context ctx_io;
    ssl::context ctx_ssl(ssl::context::tls_server);

    ctx_ssl.set_options(
        ssl::context::default_workarounds
        | ssl::context::no_sslv2
        | ssl::context::no_sslv3
        | ssl::context::single_dh_use);

    ctx_ssl.use_certificate_chain_file("server.crt");
    ctx_ssl.use_private_key_file("server.key", ssl::context::pem);
    ctx_ssl.use_tmp_dh_file("dh2048.pem");

    tcp::acceptor acceptor(ctx_io, {tcp::v4(), 443});

    std::function<void()> aceitar;
    aceitar = [&]() {
        acceptor.async_accept(
            [&](const asio::error_code& ec, tcp::socket socket) {
                if (!ec)
                    std::make_shared<SessaoSSL>(std::move(socket), ctx_ssl)->iniciar();
                aceitar(); // aceita a próxima conexão
            });
    };
    aceitar();

    ctx_io.run();
    return 0;
}
```

#### TLS 1.3

O TLS 1.3 está disponível no Asio através do OpenSSL 1.1.1 ou superior. Para restringir a versões modernas:

```c++
ssl::context ctx(ssl::context::tls);

// Permitir apenas TLS 1.2 e 1.3 (desabilitar versões antigas)
ctx.set_options(
    ssl::context::no_sslv2
    | ssl::context::no_sslv3
    | ssl::context::no_tlsv1
    | ssl::context::no_tlsv1_1);
```

> **Dica:** TLS 1.3 oferece handshake mais rápido (1-RTT e até 0-RTT para reconexões), forward secrecy obrigatório e remoção de algoritmos criptográficos obsoletos. Prefira TLS 1.3 sempre que possível.

#### Compilando com suporte a SSL

```sh
# Standalone Asio com OpenSSL
g++ -std=c++20 servidor.cpp -o servidor -pthread -lssl -lcrypto -DASIO_STANDALONE

# Boost.Asio com OpenSSL
g++ -std=c++20 servidor.cpp -o servidor -pthread -lssl -lcrypto -lboost_system
```
