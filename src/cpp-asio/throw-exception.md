# Exceção

O Asio fornece duas formas de lidar com erros nas operações de rede: **por exceção** (comportamento padrão) e **por código de erro** (sem exceção, usando `error_code`). Conhecer as duas formas é fundamental para escrever código robusto.

## Tratamento por Exceção

Por padrão, as funções síncronas do Asio lançam `asio::system_error` (standalone) ou `boost::system::system_error` (Boost.Asio) quando ocorre um erro. Ambas derivam de `std::runtime_error`.

O Asio fornece uma série de exceções de erro que incluem:

- `asio::error::address_family_not_supported`: o tipo de endereço especificado não é suportado pela plataforma.
- `asio::error::address_in_use`: o endereço especificado já está em uso por outro processo.
- `asio::error::connection_aborted`: a conexão foi abortada pelo host remoto.
- `asio::error::connection_refused`: a conexão foi recusada pelo host remoto.
- `asio::error::connection_reset`: a conexão foi reiniciada pelo host remoto.
- `asio::error::eof`: o par encerrou a conexão (fim de arquivo/stream).
- `asio::error::timed_out`: a operação excedeu o tempo limite.
- `asio::error::host_not_found`: o nome do host não pôde ser resolvido.

Essas exceções são lançadas pelos métodos do Asio que realizam operações de rede síncronas e podem ser capturadas e tratadas de acordo com o erro específico:

```cpp
#include <asio.hpp>
#include <iostream>

int main()
{
    try {
        asio::io_context io_context;
        asio::ip::tcp::resolver resolver{io_context};

        // Lança asio::system_error se a resolução falhar
        auto endpoints = resolver.resolve("host.invalido.exemplo", "https");

        for (const auto& ep : endpoints)
            std::cout << ep.endpoint() << '\n';

    } catch (const asio::system_error& e) {
        // e.code() retorna o error_code específico
        std::cerr << "Erro Asio [" << e.code() << "]: " << e.what() << '\n';
    } catch (const std::exception& e) {
        std::cerr << "Exceção: " << e.what() << '\n';
    }

    return 0;
}
```

A função membro `what()` retorna as informações detalhadas da exceção, enquanto `code()` retorna o `error_code` correspondente.

## Tratamento por Código de Erro (sem exceção)

A maioria das funções síncronas do Asio possui uma sobrecarga que recebe um `asio::error_code` por referência. Nesse caso, **nenhuma exceção é lançada** — o erro é armazenado no `error_code` e a função retorna normalmente:

```cpp
#include <asio.hpp>
#include <iostream>

int main()
{
    asio::io_context io_context;
    asio::ip::tcp::socket socket{io_context};

    asio::ip::tcp::endpoint endpoint{
        asio::ip::make_address("192.168.1.1"), 8080};

    // Sobrecarga sem exceção: erro armazenado em 'ec'
    asio::error_code ec;
    socket.connect(endpoint, ec);

    if (ec) {
        if (ec == asio::error::connection_refused)
            std::cerr << "Conexão recusada!\n";
        else
            std::cerr << "Erro ao conectar: " << ec.message() << '\n';
        return 1;
    }

    std::cout << "Conectado com sucesso!\n";
    return 0;
}
```

### Verificando erros em operações assíncronas

Nos callbacks de operações assíncronas, o primeiro parâmetro é sempre um `error_code`:

```cpp
socket.async_receive(asio::buffer(buffer),
    [](const asio::error_code& ec, std::size_t bytes) {
        if (ec) {
            if (ec == asio::error::eof) {
                std::cout << "Conexão encerrada pelo par\n";
            } else {
                std::cerr << "Erro: " << ec.message() << '\n';
            }
            return;
        }
        // Processar bytes recebidos...
        std::cout << "Recebidos " << bytes << " bytes\n";
    });
```

## Tabela de Erros Comuns

| Código de Erro | Significado | Causa Comum |
|---|---|---|
| `asio::error::eof` | Fim da conexão | Par encerrou a conexão normalmente |
| `asio::error::connection_refused` | Conexão recusada | Porta fechada no servidor |
| `asio::error::connection_reset` | Conexão reiniciada | Desconexão abrupta |
| `asio::error::connection_aborted` | Conexão abortada | Timeout ou erro de rede |
| `asio::error::address_in_use` | Endereço em uso | Porta já ocupada |
| `asio::error::host_not_found` | Host não encontrado | Falha de DNS |
| `asio::error::timed_out` | Tempo esgotado | Servidor não respondeu |
| `asio::error::operation_aborted` | Operação cancelada | `cancel()` ou `stop()` chamado |

## Standalone vs Boost.Asio

| Aspecto | Standalone Asio | Boost.Asio |
|---|---|---|
| Tipo de erro | `asio::error_code` | `boost::system::error_code` |
| Exceção | `asio::system_error` | `boost::system::system_error` |
| Categoria | `asio::error::get_system_category()` | `boost::system::system_category()` |

> **Dica:** Em código de produção, prefira a sobrecarga com `error_code` para operações críticas, pois o tratamento explícito de erros é mais eficiente e previsível do que o mecanismo de exceções.
