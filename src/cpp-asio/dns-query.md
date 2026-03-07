# DNS Query

O objeto `asio::ip::tcp::resolver` é usado para resolver nomes de domínio em endereços IP. Ele é útil em situações em que o programa precisa se conectar a um host remoto usando um nome de domínio, mas precisa do endereço IP para estabelecer a conexão de rede.

Para usar o objeto `asio::ip::tcp::resolver`, inclua o cabeçalho `<asio/ip/tcp.hpp>` no seu código e crie um objeto da classe passando um `asio::io_context` para o construtor. Em seguida, use o método `resolve` para resolver o nome de domínio em um endereço IP.

## Resolução Síncrona

O método `resolve` retorna um range de resultados que pode ser percorrido com range-for moderno:

```cpp
#include <asio.hpp>
#include <iostream>

int main()
{
    try
    {
        asio::io_context io_context;

        asio::ip::tcp::resolver resolver{io_context};

        // resolve(host, serviço) — serviço pode ser nome ("https") ou porta ("443")
        auto endpoints = resolver.resolve("google.com", "https");

        // Iteração moderna com range-for
        for (const auto& entry : endpoints)
        {
            std::cout << entry.endpoint() << '\n';
        }
    }
    catch (std::exception& e)
    {
        std::cerr << e.what() << '\n';
    }

    return 0;
}
```

O resultado da execução será algo como:

```
142.250.184.206:443
[2800:3f0:4004:811::200e]:443
```

O tipo `asio::ip::tcp::resolver::results_type` é um range de `basic_resolver_entry`:

```cpp
template <typename InternetProtocol>
class basic_resolver_entry
{
public:
    typedef InternetProtocol protocol_type;
    typedef typename InternetProtocol::endpoint endpoint_type;

    // Conversão implícita para endpoint — permite uso direto com connect()
    operator endpoint_type() const { return endpoint_; }
    ......
}
```

Como ele possui o operador de conversão `endpoint_type()`, pode ser passado diretamente para funções como `asio::connect()`:

```cpp
// O resolver results pode ser passado diretamente para connect
asio::ip::tcp::socket socket{io_context};
asio::connect(socket, endpoints); // itera automaticamente todos os endpoints
```

## Resolução Assíncrona

O método `async_resolve` resolve o nome de domínio de forma assíncrona, sem bloquear a thread. Isso é essencial em servidores que precisam resolver múltiplos nomes simultaneamente:

```cpp
#include <asio.hpp>
#include <iostream>

int main()
{
    asio::io_context io_context;
    asio::ip::tcp::resolver resolver{io_context};

    // async_resolve aceita um callback (ou use_awaitable em corrotinas)
    resolver.async_resolve("google.com", "https",
        [](const asio::error_code& ec,
           asio::ip::tcp::resolver::results_type results)
        {
            if (ec) {
                std::cerr << "Erro na resolução: " << ec.message() << '\n';
                return;
            }
            for (const auto& entry : results)
                std::cout << entry.endpoint() << '\n';
        });

    io_context.run(); // processa a operação assíncrona
    return 0;
}
```

### Resolução Assíncrona com Corrotinas

Com corrotinas C++20, a resolução assíncrona fica ainda mais legível:

```cpp
#include <asio.hpp>
#include <asio/co_spawn.hpp>
#include <asio/detached.hpp>
#include <iostream>

using asio::awaitable;
using asio::use_awaitable;
using asio::ip::tcp;

awaitable<void> resolver_dns(asio::io_context& ctx,
                              const std::string& host)
{
    tcp::resolver resolver{ctx};

    // co_await suspende a corrotina até a resolução completar
    auto endpoints = co_await resolver.async_resolve(host, "https",
                                                     use_awaitable);

    for (const auto& entry : endpoints)
        std::cout << entry.endpoint() << '\n';
}

int main()
{
    asio::io_context ctx;
    asio::co_spawn(ctx,
        resolver_dns(ctx, "google.com"),
        asio::detached);
    ctx.run();
    return 0;
}
```

## Cancelamento

O objeto `resolver` também permite cancelar uma resolução em andamento com o método `cancel()`:

```cpp
resolver.cancel(); // cancela todas as operações pendentes no resolver
```

Isso é útil em aplicações com timeout, onde a resolução não deve bloquear indefinidamente.

> **Atenção:** A classe `resolver::query` foi **depreciada** no Asio moderno. Use `resolver.resolve(host, serviço)` diretamente, sem criar um objeto `query` intermediário.
