# ``Strands``: Usar threads sem bloqueio explícito

A classe `asio::strand` é usada para garantir que operações de entrada e saída sejam executadas de forma serial em um objeto de io_context. Isso é útil em situações em que é importante que as operações de entrada e saída sejam realizadas em uma determinada ordem ou em que é importante evitar conflitos entre operações simultâneas.

Para usar a `asio::strand`, é preciso criar um objeto da classe e passar um objeto de io_context para o construtor. Em seguida, é possível chamar as funções de leitura e escrita assíncronas fornecidas pela `asio::strand` e passar um objeto de completion token para elas. Quando uma operação de entrada e saída é agendada através de uma `asio::strand`, ela é adicionada a uma fila de operações e é garantido que as operações na fila sejam executadas de forma serial.

Além de garantir que as operações de entrada e saída sejam executadas de forma serial, a `asio::strand` também fornece uma série de outras funcionalidades úteis. Por exemplo, ela permite que o programa cancele operações que estão em andamento e permite que o programa obtenha informações sobre o número de operações pendentes na fila.

Em resumo, `asio::strand` é usada para garantir que operações de entrada e saída sejam executadas de forma serial em um objeto de io_context. Ela oferece uma série de funcionalidades úteis, como cancelamento de operações em andamento e obtenção de informações sobre o número de operações pendentes na fila.

Uma thread é definida como uma chamada estritamente sequencial de manipuladores de eventos[event handler] (ou seja, nenhuma chamada simultânea). O uso de `asio::strand` permite a execução de código em um programa multithread sem a necessidade de bloqueio explícito (por exemplo, usando `mutex`[exclusão mútua]). E pode ser usado para sincronizar mais cenários.


`Strand` agrupa tarefas. Tarefas do mesmo `asio::strand` não podem rodar em paralelo, mas quando suspensas, podem ser executadas em outra thread quando acordarem da próxima vez.

As `strands` podem ser implícitas ou explícitas, conforme ilustrado pelas seguintes abordagens alternativas:

* Utilizando `asio::io_context::run()` em apenas uma thread significa que todos os manipuladores de eventos são executados em uma thread implícita, devido à garantia do `asio::io_context` de que os manipuladores[handlers] são invocados somente de dentro da função `run()`.

* Onde existe uma única cadeia de operações assíncronas associadas a uma conexão (por exemplo, em uma implementação de protocolo half-duplex como HTTP), não há possibilidade de execução simultânea dos manipuladores. Neste caso seria um `strand` implícito.

* Um `strand` explícito é uma instância `strand<>` ou `asio::io_context::strand`. Todos os objetos de função do manipulador[handler] de eventos precisam ser vinculados ao strand usando `asio::bind_executor()` ou de outra forma postados/despachados através do objeto strand.

Strand é genérico e pode ser usado para sincronizar mais cenários.

* Primeiro, que `defer()` só consegue enfileirar as tarefas de uma única cadeia de operações. Se você tem um canal duplex (ex.: `read()` e `write()` num único socket), então `defer()` já não oferece garantias o suficiente pra sincronizar o acesso aos dados compartilhados, e ainda tem que usar `strands`. Por esse motivo não posso fazer uso da otimização de `defer()` na lib de fibras, porque tem sempre um segundo canal assíncrono de notificações que representa a cadeia de cancelamento da fibra.

* Segundo, `strands` podem ser usados no cenário onde há objetos trafegando entre múltiplos io_executors, mas `defer()` falha se você agenda, a partir de um contexto de execução, uma tarefa em outro contexto esperando que isso vá sincronizar/enfileirar/serializar algum trabalho.

O executor associado deve atender aos requisitos do executor. Ele será usado pela operação assíncrona para enviar manipuladores intermediários e finais para execução.

O executor pode ser customizado para um tipo de manipulador específico, especificando um tipo aninhado `executor_type` e a função membro `get_executor()`.

Veja o exemplo abaixo:

```cpp
    class my_handler
    {
    public:
    // Custom implementation of Executor type requirements.
    typedef my_executor executor_type;

    // Return a custom executor implementation.
    executor_type get_executor() const noexcept
    {
        return my_executor();
    }

    void operator()() { ... }
    };
```