# Concorrência & Paralelismo

## Threads:

É um fluxo seqüencial de controle dentro de um programa. Basicamente, consiste em uma unidade básica de utilização da CPU, compreendendo um ID, um contador de programa, um conjunto de registradores e uma pilha. Um processo tradicional tem uma única thread de controle. Se o processo possui múltiplas threads de controle, ele pode realizar mais do que uma tarefa a cada momento. Essa possibilidade abre portas para um novo modelo de programação.

### Green Threads:

Green threads resolvem um problema comum na programação. Você não deseja que seu código bloqueie a CPU, impedindo que ela faça um trabalho significativo. Resolvemos isso usando multitarefa, o que nos permite suspender a execução de um pedaço de código enquanto retomamos outro e alternamos entre 'contextos'.

Isso não deve ser confundido com paralelismo, embora fácil confundir, são duas coisas diferentes. Pense dessa maneira: green threads nos permite trabalhar de maneira mais inteligente e eficiente e, assim, usar nossos recursos com mais eficiência, e o paralelismo é como jogar mais recursos no problema.

Geralmente, existem duas maneiras de fazer isso:

* Multitarefa preemptivo;
* Multitarefa não preemptivo (ou multitarefa cooperativa).

## MultiTasking (MultiTarefa):

É o processo de executar várias tarefas ao mesmo tempo em um único processador. Ele é usado para permitir que várias tarefas sejam executadas de forma simultânea e para aumentar a eficiência do processador. Existem várias técnicas de multitasking, como multitasking baseado em tempo, multitasking baseado em eventos e multitasking baseado em processos.

### Tipos de tarefas:

- **Preemptivo:** Ocorre quando uma tarefa é interrompida por algum agendador externo e executa outra antes de voltar. A tarefa não tem nada a haver sobre esse assunto, a decisão é tomada pelo agendador. O kernel usa isso em sistemas operacionais, ou seja, para permitir que você use a interface do usuário enquanto executa a CPU para fazer cálculos em sistemas de thread único.

- **Não-Preemptivo:** Uma tarefa decide por si mesma quando é melhor a CPU fazer outra coisa do que esperar que algo aconteça na tarefa atual. Geralmente isso ocorre quando `yield` repassa o controle ao agendador. Um caso de uso normal para isso é gerar controle quando algo que irá bloquear a execução ocorre. Um exemplo disso são as operações de E/S. Quando o controle é gerado, um agendador central direciona a CPU para retomar o trabalho em outra tarefa que está pronta para realmente fazer outra coisa além de apenas bloquear.

## Síncrono (sync) e assíncrono (async):

São dois termos que se referem ao modo como as operações são executadas em um programa de computador.

`Sync`: são aquelas que são executadas de forma sequencial, ou seja, uma operação é executada após a conclusão da operação anterior. Isso significa que o programa é bloqueado enquanto a operação está sendo executada e não pode continuar até que a operação seja concluída.

`Async`: são aquelas que são executadas de forma independente, ou seja, uma operação é iniciada e o programa continua a executar outras operações enquanto a operação assíncrona está sendo executada. Isso significa que o programa não é bloqueado enquanto a operação assíncrona está sendo executada e pode continuar a executar outras operações enquanto aguarda o término da operação assíncrona.

Operações síncronas são geralmente mais fáceis de programar e entender, mas podem ser menos eficientes em situações em que é necessário aguardar o término de uma operação para continuar a execução. Operações assíncronas, por outro lado, são geralmente mais eficientes, mas podem ser mais complexas de programar e entender.


## Bloqueante e não-bloqueante:

Blocking e non-blocking são termos que se referem ao modo como as operações são executadas em um programa de computador.

Operações blocking são aquelas que bloqueiam o programa enquanto aguardam o término de uma operação. Isso significa que o programa não pode continuar a executar outras operações enquanto aguarda o término da operação blocking.

Operações non-blocking, por outro lado, são aquelas que não bloqueiam o programa enquanto aguardam o término de uma operação. Isso significa que o programa pode continuar a executar outras operações enquanto aguarda o término da operação non-blocking.

Operações blocking são geralmente mais fáceis de programar e entender, mas podem ser menos eficientes em situações em que é necessário aguardar o término de uma operação para continuar a execução. Operações non-blocking, por outro lado, são geralmente mais eficientes, mas podem ser mais complexas de programar e entender.

## Exclusão Mútua (Mutex)

`Mutex` (mutual exclusion, exclusão mútua em inglês): é um mecanismo de sincronização que é usado para garantir que apenas uma thread (rotina) de um programa de computador tenha acesso a uma região crítica de código por vez. Ele é usado para evitar conflitos de acesso entre threads que podem ocorrer quando várias threads tentam acessar e modificar o mesmo dado ao mesmo tempo. O mutex bloqueia a região crítica de código enquanto uma thread está acessando-a, garantindo que outras threads aguardem o término da operação antes de tentarem acessar a região crítica de código. 

## Epoll/Kqueue/IOCP/IO_uring:

Epoll, Kqueue, IOCP e IO_uring são todos mecanismos de E/S event-driven (E/S com base em eventos) que são usados ​​para monitorar e gerenciar operações de entrada e saída assíncronas em sistemas operacionais diferentes. Eles são usados ​​para permitir que as aplicações sejam notificadas quando os dados estão disponíveis para leitura ou quando os dados podem ser escritos sem bloquear o processador.

`Epoll`: é um mecanismo de E/S com base em eventos disponível no Linux. Ele permite que as aplicações monitorem vários descritores de arquivo para verificar se eles estão prontos para leitura ou escrita.

`Kqueue`: é um mecanismo de E/S com base em eventos disponível no FreeBSD, NetBSD e OpenBSD. Ele funciona de maneira semelhante ao Epoll, permitindo que as aplicações monitorem descritores de arquivo para verificar se eles estão prontos para leitura ou escrita.

`IOCP`: é um mecanismo de E/S com base em eventos disponível no Windows. Ele permite que as aplicações monitorem vários handles de arquivo e sockets para verificar se eles estão prontos para leitura ou escrita.

`IO_uring`: é um mecanismo de E/S com base em eventos disponível no Linux. Ele foi projetado para ser mais rápido e eficiente do que o Epoll e oferece uma interface mais simples para as aplicações.

## Coroutine (Corrotina):

Na ciência da computação, as rotinas são definidas como uma sequência de operações. A execução de rotinas forma um relacionamento pai-filho e o filho termina sempre antes do pai. Corrotinas (o termo foi introduzido por [Melvin Conway](https://en.wikipedia.org/wiki/Melvin_Conway)) são uma generalização de rotinas ([Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)). A principal diferença entre corrotinas e rotinas é que uma corrotina permite suspender e retomar explicitamente seu progresso por meio de operações adicionais, preservando o estado de execução e, portanto, fornecendo um fluxo de controle aprimorado (mantendo o contexto de execução).

As características de uma corrotina são:

* Os valores dos dados locais persistem entre chamadas sucessivas (alternância de contexto);
* A execução é suspensa quando o controle sai da rotina e é retomada em algum momento posterior;
* Mecanismo de controle-transferência;
* Simétrico ou assimétrico;
* Objeto de primeira classe (pode ser passado como argumento, retornado por procedimentos, armazenado em uma estrutura de dados para ser usado posteriormente ou livremente manipulado pelo desenvolvedor);
* Stackful (com pilha) ou Stackless (sem pilha).

### **Stackfulness (com pilha ou sem pilha)**

- Stackful (com pilha): pode ser suspensa de dentro de um stackframe (quadro de pilha) aninhado. A execução continua exatamente no mesmo ponto no código em que foi suspensa antes.

- Stackless (sem pilha): apenas a rotina de nível superior pode ser suspensa. Qualquer rotina chamada por essa rotina de nível superior pode não ser suspensa. Isso proíbe o fornecimento de operações de suspensão ou retomada de rotinas dentro de uma biblioteca de uso geral.

As coroutines stackless são implementadas sem usar uma pilha de chamadas de função, enquanto as coroutines stackful são implementadas com uma pilha de chamadas de função. As coroutines stackless são geralmente mais eficientes em termos de uso de memória, mas são mais difíceis de implementar e menos flexíveis do que as coroutines stackful.

## Fiber (Fibra):

Uma fibra pode salvar o estado de execução atual, incluindo todos os registradores e sinalizadores da CPU, o ponteiro de instruções e o ponteiro de pilha e, posteriormente, restaurar esse estado. A idéia é ter vários caminhos de execução em execução em uma única thread usando o planejamento cooperativo (ao contrario das threads, que são agendados preventivamente). A fibra em execução decide explicitamente quando deve permitir que outra fibra seja executada (alternância de contexto).

O controle é passado cooperativamente entre as fibras lançadas em um determinado segmento. Em um determinado momento, em um determinada thread, no máximo uma fibra está em execução.

A geração de fibras adicionais em um determinada thread não distribui seu programa por mais núcleos de hardware, embora possa fazer um uso mais eficaz do núcleo no qual está sendo executado.

Por outro lado, uma fibra pode acessar com segurança qualquer recurso pertencente exclusivamente a seu thread pai, sem precisar explicitamente defender esse recurso contra o acesso simultâneo de outras fibras na mesma thread. Você já está garantido que nenhuma outra fibra nessa thread está acessando simultaneamente neste mesmo recurso. Isso pode ser particularmente importante ao introduzir a concorrência no código legado. Você pode gerar fibras com segurança executando código antigo, usando E/S assíncrona para intercalar a execução.

Com efeito, as fibras fornecem uma maneira natural de organizar o código simultâneo com base em E/S assíncronas. Em vez de encadear manipuladores de conclusão, o código executado em uma fibra pode fazer o que parece ser uma chamada de função de bloqueio normal. Essa chamada pode suspender de forma barata a fibra de chamada, permitindo que outras fibras no mesma thread sejam executadas. Quando a operação é concluída, a fibra suspensa é retomada, sem a necessidade de salvar ou restaurar explicitamente seu estado. Suas variáveis ​​de pilha local persistem na chamada.
