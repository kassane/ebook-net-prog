# Corrotina

Corrotinas são uma técnica de programação que permite que uma função seja dividida em várias partes, cada uma delas sendo executada de forma separada. Elas são usadas para criar funções assíncronas, ou seja, funções que podem ser "pausadas" e retomadas posteriormente, permitindo que o programa execute outras tarefas enquanto aguarda a conclusão de uma operação.

Por exemplo, suponha que você tenha uma função que faz uma chamada de rede para obter os dados de um determinado recurso. Usando corrotinas, você pode escrever essa função de forma síncrona, como se a chamada de rede fosse imediatamente concluída, e depois usar as corrotinas para "pausar" a execução da função enquanto aguarda a resposta da chamada de rede. Isso permite que o programa execute outras tarefas enquanto aguarda a resposta, em vez de ficar "bloqueado" aguardando a conclusão da chamada de rede.

## Como usar corrotinas no C++

No C++, existem duas maneiras de criar corrotinas: a primeira é usando a biblioteca Asio, e a segunda é usando o recurso `std::coroutine`, que foi adicionado ao C++20. As corrotinas `<coroutine>` são baseadas em uma nova sintaxe de linguagem do C++ e são mais fáceis de usar do que as corrotinas Asio, mas elas ainda são um recurso relativamente novo e podem não estar disponíveis em todas as implementações do C++.

Em resumo, as corrotinas Asio são uma maneira de criar e executar corrotinas no C++ usando a biblioteca Asio, enquanto as corrotinas `<coroutine>` são uma maneira de criar corrotinas usando uma nova sintaxe de linguagem adicionada ao C++20. Ambas as opções permitem que você crie funções assíncronas de forma fácil e eficiente.

Uma vantagem das corrotinas Asio é que elas podem ser usadas com qualquer biblioteca ou sistema que suporte a biblioteca Asio, o que significa que elas são compatíveis com uma ampla variedade de plataformas de rede. Além disso, as corrotinas Asio fornecem uma maneira de escrever código assíncrono de forma mais clara e legível, pois permitem que você escreva código síncrono que é "convertido" em código assíncrono pelo próprio Asio.

Por outro lado, as corrotinas são baseadas em uma nova sintaxe de linguagem e, por esse motivo, podem ser mais fáceis de usar do que as corrotinas Asio. Além disso, elas podem ser mais eficientes em termos de desempenho, pois elas são implementadas diretamente na linguagem C++ e não dependem de uma biblioteca externa. No entanto, elas ainda são um recurso relativamente novo e podem não estar disponíveis em todas as implementações do C++.

Em resumo, as corrotinas Asio e `std::coroutine` são duas maneiras diferentes de criar e executar corrotinas no C++. As corrotinas Asio são compatíveis com uma ampla variedade de plataformas e fornecem uma maneira de escrever código assíncrono de forma mais clara, enquanto as corrotinas `std::coroutine` são baseadas em uma nova sintaxe de linguagem e podem ser mais eficientes em termos de desempenho. Qual das duas opções é a melhor para você depende das suas necessidades e da plataforma em que está trabalhando.

A seguir, um exemplo de como usar corrotinas Asio para fazer uma chamada HTTP GET assíncrona usando a biblioteca Asio:

```c++
#include <iostream>
#include <boost/asio/co_spawn.hpp>
#include <boost/asio/detached.hpp>
#include <boost/asio/io_context.hpp>
#include <boost/asio/ip/tcp.hpp>

#include <boost/beast/core.hpp>
#include <boost/beast/http.hpp>
#include <boost/beast/version.hpp>
#include <boost/asio/connect.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <cstdlib>

namespace beast = boost::beast;         // from <boost/beast.hpp>
namespace http = beast::http;           // from <boost/beast/http.hpp>
namespace net = boost::asio;            // from <boost/asio.hpp>
using tcp = net::ip::tcp;               // from <boost/asio/ip/tcp.hpp>

// Realiza uma chamada HTTP GET assíncrona e imprime o corpo da resposta
void async_http_get(net::io_context& ioc, const std::string& host, const std::string& target)
{
    // Cria um socket TCP
    tcp::resolver resolver{ioc};
    beast::tcp_stream stream{ioc};

    // Realiza a resolução do nome do host e conecta ao servidor
    co_await resolver.async_resolve(host, "http", net::use_awaitable);
    co_await stream.async_connect(resolver.results(), net::use_awaitable);

    // Cria uma solicitação HTTP e envia-a para o servidor
    http::request<http::string_body> req{http::verb::get, target, 11};
    req.set(http::field::host, host);
    req.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);
    co_await http::async_write(stream, req, net::use_awaitable);

    // Recebe a resposta do servidor
    beast::flat_buffer buffer;
    http::response<http::string_body> res;
    co_await http::async_read(stream, buffer, res, net::use_awaitable);

    // Imprime o corpo da resposta
    std::cout << res << std::endl;
}

int main()
{
    net::io_context ioc;

    // Inicia a chamada HTTP GET assíncrona em uma corrotina
    asio::co_spawn(ioc, [&] {
        co_return async_http_get(ioc, "www.example.com", "/");
    }, asio::detached);

    // Executa o event loop
    ioc.run();

    return EXIT_SUCCESS;
}
```

Ao ter o primeiro contato com corrotinas em C++ (com asio) suponho que conhecerá novas palavras-chave que necessitará compreender, que são:

- `co_spawn`: é uma função da biblioteca Asio que permite criar e iniciar uma corrotina de forma assíncrona. Ela é usada para "lançar" uma corrotina em uma determinada contexto de I/O, permitindo que a corrotina execute tarefas assíncronas como fazer chamadas de rede ou ler e escrever em arquivos.

- `co_yield`: é uma palavra-chave do C++ que permite que uma corrotina seja "pausada" e permita que outras corrotinas sejam executadas. Quando uma corrotina é "pausada" com `co_yield`, ela é suspensa temporariamente e permite que outras corrotinas sejam executadas. Quando outra corrotina termina sua execução, a corrotina "pausada" é retomada a partir do ponto onde foi interrompida.

- `co_await`: é uma palavra-chave do C++ que permite que uma corrotina aguarde a conclusão de uma operação assíncrona. Quando uma corrotina encontra um `co_await`, ela é "pausada" até que a operação assíncrona seja concluída, permitindo que outras corrotinas sejam executadas enquanto aguarda.

- `co_return`: é uma palavra-chave do C++ que permite que uma corrotina retorne um valor quando ela é concluída. É usado para encerrar a execução de uma corrotina e retornar o valor especificado para quem a chamou.

- `detached`: é um parâmetro opcional que pode ser usado com a função co_spawn da biblioteca Asio. Ele indica que a corrotina deve ser executada de forma assíncrona e não precisa ser aguardada para concluir sua execução. Isso é útil quando você deseja que a corrotina execute uma tarefa de forma independente e não precisa saber quando ela termina.

As corrotinas Asio são uma extensão da biblioteca Asio que fornece suporte nativo para a criação e execução de corrotinas no C++. Elas permitem que você escreva código síncrono que é "convertido" em código assíncrono pelo próprio Asio, o que torna mais fácil criar funções assíncronas de forma clara e legível.

Para usar corrotinas Asio, você precisa incluir o cabeçalho `<asio/co_spawn.hpp>` e usar a palavra-chave co_await para "pausar" a execução da corrotina enquanto aguarda a conclusão de uma operação assíncrona. Você também pode usar a palavra-chave `co_yield` para "pausar" a execução da corrotina e permitir que outras corrotinas sejam executadas.

Além disso, as corrotinas Asio podem ser "lançadas" em um contexto de I/O usando a função `co_spawn`, que permite que a corrotina execute tarefas assíncronas como fazer chamadas de rede ou ler e escrever em arquivos. Você também pode usar a função `async_write` e `async_read` da biblioteca Asio para escrever e ler dados de forma assíncrona, respectivamente.

No entanto, é importante lembrar que as corrotinas Asio dependem da biblioteca Asio para funcionar, o que pode afetar o desempenho em comparação com outras opções, como as corrotinas `std::coroutine`, que são implementadas diretamente na linguagem C++. Qual das duas opções é a melhor para você depende das suas necessidades e da plataforma em que está trabalhando.

#### Completion Tokens

Em C++ ASIO, um [completion token](https://think-async.com/Asio/asio-1.24.0/doc/asio/overview/model/completion_tokens.html) é um tipo de dado usado para especificar como uma operação assíncrona deve ser completada. Ele é passado como um parâmetro para uma função assíncrona e é usado para determinar como a função deve notificar o chamador quando a operação for concluída.

Existem vários tipos de completion token disponíveis no C++ ASIO, como `asio::use_future` e `asio::use_awaitable`. Cada um desses tipos de completion token especifica uma forma diferente de completar a operação assíncrona.

Por exemplo, o completion token `asio::use_future` é usado para completar a operação assíncrona retornando um objeto `std::future` que pode ser usado para obter o resultado da operação. Isso permite que o chamador da função assíncrona aguarde o término da operação de forma síncrona, usando a sintaxe de await do C++20.

O completion token `asio::use_awaitable`, por outro lado, é usado para completar a operação assíncrona retornando um objeto awaitable que pode ser usado para aguardar o término da operação de forma assíncrona. Isso permite que o chamador da função assíncrona aguarde o término da operação de forma assíncrona, usando a sintaxe de await do C++20.

Em resumo, os completion tokens são usados ​​no C++ ASIO para especificar como uma operação assíncrona deve ser completada. Eles são passados como parâmetros para funções assíncronas e são usados ​​para determinar como a função deve notificar o chamador quando a operação for concluída. Existem vários tipos de completion token disponíveis, cada um com suas próprias características e usos específicos.
