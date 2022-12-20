## O que seria o tal `Executor`

O executor é um objeto que pode executar unidades de trabalho encapsuladas. A partir do C++11,
podemos implementar um executor que gerencie o agendamento de operações relacionadas a espera de alguma duração de tempo.
Há o recurso global RTC e, em vez de criar várias e várias threads para a execução de operações bloqueantes como a operação `sleep_for`,
vamos usar um executor, que irá agendar e tratar todos os eventos de tempo que se façam necessário.

### Surgimento do assunto em torno do C++ STD

Em 6 de julho de 2021, a proposta dos Executores foi atualizada com mais um bilhão de pontos. O novo documento,
[P2300R1](http://open-std.org/JTC1/SC22/WG21/docs/papers/2021/p2300r1.html),
oficialmente denominado `std::execution`, em comparação com The Unified Executor for C++, [P0443R14](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0443r14),
expõe mais sistematicamente as ideias de design dos Executors; dá mais instruções sobre implementação; exclui o Executor Concept,
mantém e estabelece o modelo `Sender/Receiver/Scheduler` é introduzido;
o conjunto inicial de algoritmos que deveria estar na biblioteca é fornecido e não há pequenas alterações no design do algoritmo anterior;
há semântica mais clara, como tarefa multi-shot (multi-shot) e Single-shot, lazy e entrega ansiosa de tarefas, etc.
A biblioteca Executors praticada pelo autor em seu tempo livre acaba de concluir o conteúdo do P1879R3.

### 1. Por que Executores?

C++ sempre careceu de infraestrutura de programação simultânea disponível, e a infraestrutura recém-introduzida desde C++11,
bem como a melhoria de bibliotecas de terceiros, como boost e folly, têm mais ou menos problemas e certas limitações.

### 1.1 `std::async` não é assíncrono

Vamos voltar no tempo para os tempos modernos do padrão C++11. O padrão C++11 introduziu oficialmente um recurso multithreading unificado, como,
<thread>e outros blocos de construção de baixo nível. Ele também introduziu uma interface para iniciar chamadas de função assíncronas.
  Mas não é assíncrono. Vamos usar um exemplo de cppreference ilustrar:

- `<atomic>`
- `<mutex>`
- `<conditional_variable>`
- `std::asyncstd::async`

```c++
// temporariamente std::future<void> é construido
std::async([]{ f(); });
// bloqueado pelo destrutor de std::future<void>
std::async([]{ g(); });
```

Geralmente falando, a tarefa de `g` tenta executar uma função que **pode** iniciar `f` o agendamento quando a função é executada.
No entanto, da forma como os códigos listados acima usam , o escalonamento da tarefa que `std::async` inicia a função de execução deve ocorrer
após o retorno da tarefa que executa a função. A razão é que:
  `g` e `f`

A primeira linha std::asynccria uma `std::future<void>` variável temporária do tipo Temp;A variável temporária Temp é destruída antes de
iniciar a execução da segunda linha;
`std::future<void>` O destruidor aguardará o retorno da operação de forma síncrona e bloqueará o thread atual.

std::asyncInicialmente, uma nova thread de execução é criada para cada tarefa iniciada. Daí std::asynca notoriedade. Agora que é mencionado
aqui std::future, também tem muitos problemas.

### 1.2 Modelo de Evolução do Futuro/Promessa

O modelo Futuro/Promessa é um modelo clássico de programação concorrente, que fornece aos programadores um mecanismo completo para controlar
a sincronização e a assincronia do programa. O mecanismo Future/Promise também foi introduzido no C++11.
Future é essencialmente uma operação simultânea que iniciamos e Promise é essencialmente um retorno de chamada para operações simultâneas.
Podemos aguardar a operação e obter o resultado da operação através do objeto Future, e o objeto Promise é responsável por escrever o valor
de retorno e nos notificar.
  
A implementação de um Future/Promise típico em C++ é mostrada na figura abaixo:
![img](https://user-images.githubusercontent.com/6756180/208737071-abe8d31c-fe0d-4023-8a1a-64083099c4f6.jpg)
  
... Continua
