# Corrotina

Corrotinas são uma técnica de programação que permite que uma função seja dividida em várias partes, cada uma delas sendo executada de forma separada. Elas são usadas para criar funções assíncronas, ou seja, funções que podem ser "pausadas" e retomadas posteriormente, permitindo que o programa execute outras tarefas enquanto aguarda a conclusão de uma operação.

Por exemplo, suponha que você tenha uma função que faz uma chamada de rede para obter os dados de um determinado recurso. Usando corrotinas, você pode escrever essa função de forma síncrona, como se a chamada de rede fosse imediatamente concluída, e depois usar as corrotinas para "pausar" a execução da função enquanto aguarda a resposta. Isso permite que o programa execute outras tarefas enquanto aguarda, em vez de ficar "bloqueado".

## Como usar corrotinas no C++

No C++20, o suporte a corrotinas foi adicionado diretamente à linguagem com o cabeçalho `<coroutine>`. Ele é amplamente suportado pelos compiladores modernos (GCC 10+, Clang 12+, MSVC 19.25+) e é a base sobre a qual o Asio constrói sua integração com corrotinas.

A biblioteca Asio aproveita o mecanismo de corrotinas do C++20 para permitir que você escreva código assíncrono de forma sequencial e legível, usando `co_await` para suspender a execução enquanto aguarda operações de E/S.

A seguir, um exemplo de servidor TCP usando corrotinas Asio com standalone Asio:

```c++
#include <asio.hpp>
#include <asio/co_spawn.hpp>
#include <asio/detached.hpp>
#include <iostream>

using asio::awaitable;
using asio::co_spawn;
using asio::detached;
using asio::use_awaitable;
namespace ip = asio::ip;
using ip::tcp;

// Corrotina que trata uma conexão de cliente
awaitable<void> tratar_cliente(tcp::socket socket)
{
    try {
        char dados[1024];
        for (;;) {
            // co_await suspende a corrotina até que haja dados disponíveis
            std::size_t n = co_await socket.async_read_some(
                asio::buffer(dados), use_awaitable);

            // Eco: envia de volta o que foi recebido
            co_await asio::async_write(socket,
                asio::buffer(dados, n), use_awaitable);
        }
    } catch (std::exception& e) {
        std::cout << "Conexão encerrada: " << e.what() << '\n';
    }
}

// Corrotina do servidor que aceita conexões
awaitable<void> servidor(asio::io_context& ctx, unsigned short porta)
{
    tcp::acceptor acceptor(ctx, {tcp::v4(), porta});
    std::cout << "Servidor ouvindo na porta " << porta << '\n';

    for (;;) {
        tcp::socket socket = co_await acceptor.async_accept(use_awaitable);
        // Lança uma corrotina independente para cada cliente
        co_spawn(ctx, tratar_cliente(std::move(socket)), detached);
    }
}

int main()
{
    asio::io_context ctx;

    co_spawn(ctx, servidor(ctx, 8080), detached);

    ctx.run();
    return 0;
}
```

Ao ter o primeiro contato com corrotinas em C++ (com Asio) você encontrará novas palavras-chave que precisará compreender:

- `co_spawn`: é uma função da biblioteca Asio que permite criar e iniciar uma corrotina de forma assíncrona. Ela "lança" uma corrotina em um dado executor, permitindo que a corrotina execute tarefas assíncronas de forma independente.

- `co_yield`: é uma palavra-chave do C++ que permite que uma corrotina seja "pausada" e permita que outras corrotinas sejam executadas. Quando uma corrotina é pausada com `co_yield`, ela é suspensa temporariamente e permite que outras tarefas sejam processadas.

- `co_await`: é uma palavra-chave do C++ que permite que uma corrotina aguarde a conclusão de uma operação assíncrona. Quando uma corrotina encontra um `co_await`, ela é suspensa até que a operação termine, liberando a thread para executar outras tarefas.

- `co_return`: é uma palavra-chave do C++ que permite que uma corrotina retorne um valor ao encerrar. É usado para finalizar a execução de uma corrotina e retornar o resultado para quem a aguarda.

- `detached`: é um completion token especial que indica que a corrotina deve ser executada de forma independente — ninguém irá aguardar seu resultado. Útil para tarefas de servidor do tipo "disparar e esquecer".

- `awaitable<T>`: é o tipo de retorno de uma corrotina Asio. Uma função que retorna `awaitable<void>` é uma corrotina que pode ser pausada com `co_await` e não produz valor ao concluir.

> **Atenção:** Uma função que usa `co_await` internamente **deve** retornar `awaitable<T>` (ou outro tipo de corrotina). Declarar a função como `void` e usar `co_await` dentro dela é um **erro de compilação**.

#### Completion Tokens

Em Asio, um [completion token](https://think-async.com/Asio/asio-1.24.0/doc/asio/overview/model/completion_tokens.html) é um tipo de dado usado para especificar como uma operação assíncrona deve ser completada. Ele é passado como último parâmetro para funções assíncronas e determina como o resultado é entregue ao chamador.

Existem vários tipos de completion token disponíveis no Asio:

- `asio::use_awaitable`: transforma a operação em uma expressão que pode ser usada com `co_await` dentro de uma corrotina. É a opção recomendada para código moderno com C++20.
- `asio::use_future`: transforma a operação em um `std::future`, permitindo obter o resultado de forma síncrona ou em outra thread.
- `asio::detached`: indica que o resultado da operação não será observado (disparar e esquecer).
- `asio::redirect_error(token, ec)`: captura erros no `error_code` fornecido em vez de lançar exceção.

### Adaptadores de Token de Completude

Ao usar `use_awaitable`, por padrão os erros são convertidos em exceções `asio::system_error`. Em código de produção, é comum preferir tratamento explícito de erros sem exceções. O Asio oferece adaptadores para isso:

#### `redirect_error` — captura o erro sem exceção

```c++
#include <asio.hpp>
#include <asio/co_spawn.hpp>
#include <asio/redirect_error.hpp>
#include <iostream>

using asio::awaitable;
using asio::use_awaitable;

awaitable<void> ler_com_tratamento(asio::ip::tcp::socket& socket)
{
    char buffer[1024];
    asio::error_code ec;

    // O erro é capturado em 'ec' em vez de ser lançado como exceção
    std::size_t n = co_await socket.async_read_some(
        asio::buffer(buffer),
        asio::redirect_error(use_awaitable, ec));

    if (ec) {
        if (ec == asio::error::eof)
            std::cout << "Conexão encerrada pelo par\n";
        else
            std::cout << "Erro: " << ec.message() << '\n';
        co_return;
    }

    std::cout << "Recebidos " << n << " bytes\n";
}
```

#### `as_tuple` — retorna `(error_code, resultado)` como tupla

```c++
#include <asio.hpp>
#include <asio/experimental/as_tuple.hpp>
#include <iostream>

using asio::awaitable;
using asio::use_awaitable;
using asio::experimental::as_tuple;

awaitable<void> ler_com_tupla(asio::ip::tcp::socket& socket)
{
    char buffer[1024];

    // Desestrutura automaticamente o resultado em (ec, bytes)
    auto [ec, n] = co_await socket.async_read_some(
        asio::buffer(buffer),
        as_tuple(use_awaitable));

    if (ec) {
        std::cout << "Erro: " << ec.message() << '\n';
        co_return;
    }

    std::cout << "Recebidos " << n << " bytes\n";
}
```

> **Dica:** Prefira `redirect_error` ou `as_tuple` em código de produção para ter controle explícito sobre os erros. Reserve o tratamento por exceção para cenários onde a corrotina deve abortar completamente ao encontrar um erro.

### Composição de Corrotinas

O Asio suporta a composição de corrotinas usando os operadores `&&` e `||` sobre objetos `awaitable<>`. Isso permite executar múltiplas operações em paralelo de forma elegante:

```c++
#include <asio.hpp>
#include <asio/experimental/awaitable_operators.hpp>
#include <iostream>

using namespace asio::experimental::awaitable_operators;
using asio::awaitable;
using asio::use_awaitable;

awaitable<void> aguardar_timeout(asio::steady_timer& timer,
                                  std::chrono::seconds segundos)
{
    timer.expires_after(segundos);
    co_await timer.async_wait(use_awaitable);
}

awaitable<void> ler_com_timeout(asio::ip::tcp::socket& socket,
                                 asio::io_context& ctx)
{
    asio::steady_timer timer(ctx);
    char buffer[1024];

    // Executa as duas corrotinas em paralelo:
    // retorna quando a PRIMEIRA terminar (leitura ou timeout)
    co_await (
        socket.async_read_some(asio::buffer(buffer), use_awaitable)
        || aguardar_timeout(timer, std::chrono::seconds(5))
    );
}
```

- Operador `||`: aguarda a **primeira** corrotina a terminar e cancela as demais.
- Operador `&&`: aguarda **todas** as corrotinas terminarem antes de prosseguir.

### Asio Corrotina comparado com Cppcoro

O [cppcoro](https://github.com/lewissbaker/cppcoro) foi uma biblioteca pioneira de corrotinas para C++ que fornecia primitivas para escrever código assíncrono de forma legível. Ela foi projetada para funcionar com o padrão de corrotinas do C++20 e serviu como referência importante para o design moderno de corrotinas em C++.

> **Atenção:** O cppcoro está **largamente sem manutenção desde 2021**, quando Lewis Baker saiu da Microsoft. Para novos projetos, recomenda-se usar diretamente as corrotinas do Asio com `use_awaitable`, ou considerar alternativas ativas como [libcoro](https://github.com/jbaldwin/libcoro).

Principais diferenças:

- O **Asio** é uma biblioteca completa de E/S de rede com suporte a corrotinas integradas ao seu modelo de executores — ideal para programação de redes.
- O **cppcoro** era focado exclusivamente em primitivas de corrotinas (sem integração de E/S de rede direta) e não recebe mais atualizações.
- Ambos seguem como referência a proposta técnica [P1056R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1056r0.html), que descreve o suporte a corrotinas no C++.
