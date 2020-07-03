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

- **Preemptivo:** Ocorre quando uma tarefa é interrompida por algum agendador externo e executa outra antes de voltar. A tarefa não tem nada a haver sobre esse assunto, a decisão é tomada pelo agendador. O kernel usa isso em sistemas operacionais, ou seja, para permitir que você use a interface do usuário enquanto executa a CPU para fazer cálculos em sistemas de thread único.

- **Não-Preemptivo:** Uma tarefa decide por si mesma quando é melhor a CPU fazer outra coisa do que esperar que algo aconteça na tarefa atual. Geralmente isso ocorre quando `yield` repassa o controle ao agendador. Um caso de uso normal para isso é gerar controle quando algo que irá bloquear a execução ocorre. Um exemplo disso são as operações de E/S. Quando o controle é gerado, um agendador central direciona a CPU para retomar o trabalho em outra tarefa que está pronta para realmente fazer outra coisa além de apenas bloquear.

## Síncrono (sync) e assíncrono (async):

Síncrono e assíncrono refere-se à interação entre o aplicativo e o kernel.
- **Síncrono**: refere-se ao processo do usuário acionando operações de E/S e aguarda ou verifica se as operações de E/S estão prontas.

- **Assíncrono**: refere-se ao processo do usuário que aciona operações de E/S. Após a operação de saída, ele começa a fazer suas próprias coisas e, quando a operação de E/S for concluída, ela será notificada da conclusão de E/S.

## Bloqueante e não-bloqueante:

Bloqueante e não-bloqueante são maneiras diferentes que os processos seguem de acordo com a prontidão das operações de E/S ao acessar dados. Para colocar claramente, é uma implementação de funções de operação de leitura ou gravação. A função de gravação aguardará para sempre. No modo sem bloqueio, a função de leitura ou gravação retornará imediatamente um valor de status. 

## Epoll/Kqueue/IOCP/IO_uring:

Esses métodos nos permitem conectar-se ao sistema operacional de uma maneira em que podemos esperar por muitos eventos, em vez de ficarmos limitados a esperar um evento por thread, podemos esperar por muitos eventos em uma única thread. Isso nos permite evitar uma das maiores desvantagens do uso de um thread por evento, que é toda a memória ocupada e a sobrecarga de gerar novos threads continuamente. Agora, temos apenas um segmento aguardando muitas tarefas.

Esses métodos têm em comum que eles são uma espécie de bloqueio da E/S. Se registrarmos apenas um evento na fila de eventos `epoll/kqueue/iocp/io_uring` e aguardarmos, não será diferente do uso de bloqueio da E/S. A vantagem é que podemos ter uma fila que aguarda centenas de milhares de eventos com pouquíssimos recursos desperdiçados.

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

## Fiber (Fibra):

Uma fibra pode salvar o estado de execução atual, incluindo todos os registradores e sinalizadores da CPU, o ponteiro de instruções e o ponteiro de pilha e, posteriormente, restaurar esse estado. A idéia é ter vários caminhos de execução em execução em uma única thread usando o planejamento cooperativo (ao contrario das threads, que são agendados preventivamente). A fibra em execução decide explicitamente quando deve permitir que outra fibra seja executada (alternância de contexto).

O controle é passado cooperativamente entre as fibras lançadas em um determinado segmento. Em um determinado momento, em um determinada thread, no máximo uma fibra está em execução.

A geração de fibras adicionais em um determinada thread não distribui seu programa por mais núcleos de hardware, embora possa fazer um uso mais eficaz do núcleo no qual está sendo executado.

Por outro lado, uma fibra pode acessar com segurança qualquer recurso pertencente exclusivamente a seu thread pai, sem precisar explicitamente defender esse recurso contra o acesso simultâneo de outras fibras na mesma thread. Você já está garantido que nenhuma outra fibra nessa thread está acessando simultaneamente neste mesmo recurso. Isso pode ser particularmente importante ao introduzir a concorrência no código legado. Você pode gerar fibras com segurança executando código antigo, usando E/S assíncrona para intercalar a execução.

Com efeito, as fibras fornecem uma maneira natural de organizar o código simultâneo com base em E/S assíncronas. Em vez de encadear manipuladores de conclusão, o código executado em uma fibra pode fazer o que parece ser uma chamada de função de bloqueio normal. Essa chamada pode suspender de forma barata a fibra de chamada, permitindo que outras fibras no mesma thread sejam executadas. Quando a operação é concluída, a fibra suspensa é retomada, sem a necessidade de salvar ou restaurar explicitamente seu estado. Suas variáveis ​​de pilha local persistem na chamada.
