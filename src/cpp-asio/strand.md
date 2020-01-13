# ``Strands``: Usar threads sem bloqueio explícito

Uma thread é definida como uma chamada estritamente sequencial de manipuladores de eventos[event handler] (ou seja, nenhuma chamada simultânea). O uso de `strand` permite a execução de código em um programa multithread sem a necessidade de bloqueio explícito (por exemplo, usando `mutex`[exclusão mútua]). E pode ser usado para sincronizar mais cenários.

Strand é mais genérico e pode ser usado para sincronizar mais cenários.

* Primeiro, que `defer()` só consegue enfileirar as tarefas de uma única cadeia de operações. Se você tem um canal duplex (ex.: `read()` e `write()` num único socket), então `defer()` já não oferece garantias o suficiente pra sincronizar o acesso aos dados compartilhados, e ainda tem que usar `strands`. Por esse motivo não posso fazer uso da otimização de `defer()` na lib de fibras, porque tem sempre um segundo canal assíncrono de notificações que representa a cadeia de cancelamento da fibra.

* Segundo, `strands` podem ser usados no cenário onde há objetos trafegando entre múltiplos io_executors, mas `defer()` falha se você agenda, a partir de um contexto de execução, uma tarefa em outro contexto esperando que isso vá sincronizar/enfileirar/serializar algum trabalho.

`Strand` agrupa tarefas. Tarefas do mesmo `strand` não podem rodar em paralelo, mas, quando suspensas porque não estão fazendo nada, podem ser executadas em outra thread quando acordarem da próxima vez.

As ``strands`` podem ser implícitas ou explícitas, conforme ilustrado pelas seguintes abordagens alternativas:

* Utilizando `io_context::run()` em apenas uma thread significa que todos os manipuladores de eventos são executados em uma thread implícita, devido à garantia do `io_context` de que os manipuladores[handlers] são invocados somente de dentro da função `run()`.

* Onde existe uma única cadeia de operações assíncronas associadas a uma conexão (por exemplo, em uma implementação de protocolo half-duplex como HTTP), não há possibilidade de execução simultânea dos manipuladores. Neste caso seria um `strand` implícito.

* Um `strand` explícito é uma instância `strand<>` ou `io_context::strand`. Todos os objetos de função do manipulador[handler] de eventos precisam ser vinculados ao strand usando `boost::asio::bind_executor()` ou de outra forma postados/despachados através do objeto strand.

O executor associado deve atender aos requisitos do executor. Ele será usado pela operação assíncrona para enviar manipuladores intermediários e finais para execução.

O executor pode ser customizado para um tipo de manipulador específico, especificando um tipo aninhado `executor_type` e a função de membro `get_executor()`.

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