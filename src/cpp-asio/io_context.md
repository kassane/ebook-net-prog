# `asio::io_context`

O Asio possui uma classe chamada `asio::io_context` que é usada para gerenciar operações de entrada e saída assíncronas em um programa de computador. Ela é usada para monitorar e gerenciar operações de entrada e saída em vários descritores de arquivo e sockets, permitindo que o programa seja notificado quando os dados estão disponíveis para leitura ou quando os dados podem ser escritos sem bloquear o processador.

A `asio::io_context` é geralmente usada em conjunto com um objeto `executor_work_guard`, que é responsável por manter o loop de eventos da `asio::io_context` rodando mesmo quando não há operações pendentes.

O conceito é baseado na API de rede do `Unix`. O `Asio` também possui o conceito "socket", mas isso não é suficiente: um objeto `io_context` é necessário para se comunicar com os serviços de E/S do sistema operacional.

> **Nota histórica:** A classe `io_service` foi o nome antigo de `io_context`. Ela foi renomeada e está **obsoleta** — use sempre `io_context` em código novo.

A imagem abaixo mostrará a estrutura da Arquitetura `Asio`:

![image](https://raw.githubusercontent.com/kassane/Livro-Programacao-de-Redes/gh-pages/images/architecture.jpg)

`io_context` deriva de `execution_context`:

```cpp
class io_context
  : public execution_context
{
  ......
}
```

Enquanto `execution_context` deriva de `noncopyable`:

```cpp
class execution_context
  : private noncopyable
{
  ......
}
```

Isso significa que o objeto `io_context` não pode ser copiado. Portanto, durante a inicialização do socket, ou seja, associar o socket ao `io_context`, o `io_context` deve ser passado como referência.

Ex.:

```cpp
asio::io_context io_context;
asio::ip::tcp::socket socket{io_context}; // io_context passado por referência
```

Além de gerenciar operações de entrada e saída assíncronas, a `asio::io_context` também fornece uma série de outras funcionalidades úteis. Por exemplo, ela permite que o programa agende operações para serem executadas em um momento futuro, permitindo que o programa execute tarefas de forma assíncrona de acordo com um cronograma. Ela também permite que o programa cancele operações que estão em andamento.

A `asio::io_context` pode ser configurada para escalonar operações em vários threads, permitindo que elas sejam executadas em paralelo e aproveitando ao máximo o poder de processamento do computador:

```cpp
// Sugestão de concorrência para o io_context — pode otimizar internamente
asio::io_context ctx{std::thread::hardware_concurrency()};
```

A seguir, estão descritas todas as funções comuns do `asio::io_context`:

- `run`: Inicia o loop de eventos do `asio::io_context`. Bloqueia o thread atual até que todas as operações agendadas tenham sido concluídas ou o `io_context` seja interrompido.

- `poll`: Similar à função `run`, mas não bloqueia o thread atual. Em vez disso, processa todas as operações prontas no `asio::io_context` e retorna imediatamente.

- `run_one`: Similar à função `run`, mas processa apenas uma operação pendente antes de retornar.

- `poll_one`: Similar à função `poll`, mas processa apenas uma operação pronta antes de retornar.

- `stop`: Interrompe o loop de eventos do `asio::io_context`. Útil quando o programa precisa sair antes que todas as operações pendentes sejam concluídas.

- `restart`: Reinicia o `asio::io_context` após ter sido interrompido com `stop()`, permitindo que ele aceite novas operações. Deve ser chamado antes de uma nova chamada a `run()`.

> **Atenção:** Em versões antigas do Asio, esta função se chamava `reset()`. O nome `reset()` está **depreciado** — use `restart()` em código novo.

- `stopped`: Retorna `true` se o `asio::io_context` foi interrompido com `stop()`.

- `get_executor`: Retorna um objeto executor que pode ser usado para agendar operações para serem executadas no `asio::io_context`.

### `asio::post` e `asio::dispatch`

Em versões modernas do Asio, as operações de agendamento são realizadas através de funções livres que recebem um executor como primeiro parâmetro:

- `asio::post(executor, fn)`: Agenda a função `fn` para ser executada de forma assíncrona no contexto do executor. A função nunca é executada diretamente na chamada — sempre é enfileirada.

- `asio::dispatch(executor, fn)`: Agenda a função `fn` para ser executada no contexto do executor. Se o chamador já estiver dentro do contexto do executor, a função pode ser executada imediatamente (in-line).

```cpp
asio::io_context ctx;

// Agenda uma tarefa para execução assíncrona
asio::post(ctx, [] {
    std::cout << "Executando de forma assíncrona\n";
});

ctx.run();
```

> **Atenção:** As formas antigas `io_context.post(fn)` e `io_context.dispatch(fn)` estão **depreciadas**. Use `asio::post(ctx, fn)` e `asio::dispatch(ctx, fn)`.

### `executor_work_guard`

A classe `asio::executor_work_guard` é usada para manter o loop de eventos da `asio::io_context` rodando mesmo quando não há operações pendentes. Isso é essencial quando você quer que a thread do `io_context` continue viva aguardando trabalho futuro (por exemplo, em um servidor que usa múltiplas threads).

```cpp
#include <asio.hpp>
#include <thread>
#include <iostream>

int main()
{
    asio::io_context ctx;

    // Mantém o io_context vivo mesmo sem operações pendentes
    auto guard = asio::make_work_guard(ctx);

    std::thread worker([&ctx] {
        ctx.run(); // bloqueia aqui até o guard ser liberado
    });

    // Faz algum trabalho...
    asio::post(ctx, [] { std::cout << "Tarefa executada!\n"; });

    // Libera o guard para permitir que ctx.run() retorne
    guard.reset();
    worker.join();

    return 0;
}
```

> **Atenção:** A classe `asio::io_context::work` e a macro-based work guard estão **depreciadas**. Use `asio::make_work_guard(ctx)` que retorna um `executor_work_guard<io_context::executor_type>`.
