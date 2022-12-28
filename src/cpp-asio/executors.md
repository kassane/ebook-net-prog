## O que é um `Executor`

O executor é um objeto que pode executar unidades de trabalho encapsuladas. A partir do C++11,
podemos implementar um executor que gerencie o agendamento de operações relacionadas a espera de alguma duração de tempo.
Há o recurso global RTC e, em vez de criar várias e várias threads para a execução de operações bloqueantes como a operação `sleep_for`, vamos usar um executor, que irá agendar e tratar todos os eventos de tempo que se façam necessário.

### Surgimento do assunto em torno do C++ STL

Em 6 de julho de 2021, a proposta dos Executores foi atualizada com mais um bilhão de pontos. O novo documento, [P2300R1](http://open-std.org/JTC1/SC22/WG21/docs/papers/2021/p2300r1.html),
oficialmente denominado `std::execution`, em comparação com The Unified Executor for C++, [P0443R14](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0443r14),
expõe mais sistematicamente as ideias de design dos Executors; dá mais instruções sobre implementação.
A biblioteca Executors praticada pelo autor em seu tempo livre acaba de concluir o conteúdo do [P1879R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1897r3.html).

`Unified Executors` propõe que o namespace `std::execution` do C++ Standard Library que visa fornecer uma forma mais flexível e genérica de trabalhar com Executors. A proposta foi apresentada no Grupo de Trabalho 21 (WG21) do Comitê de Padrões do C++ como a Proposta de Padrão P1907R0.

Atualmente, o namespace std::execution fornece vários tipos de Executors, como `std::execution::sequenced_policy` e `std::execution::parallel_policy`, que podem ser usados ​​para controlar como as tarefas são agendadas e executadas. No entanto, esses Executors são bastante rígidos e não permitem muita flexibilidade na customização da forma como as tarefas são agendadas e executadas.

A proposta de universal executors visa fornecer uma forma mais flexível de trabalhar com Executors, permitindo que os programadores criem seus próprios Executors personalizados de acordo com suas necessidades específicas. Isso seria feito através da introdução de novos tipos e funções no namespace `std::execution`, como `std::execution::uniform_invocable` e `std::execution::execute`, que permitiriam a criação de Executors personalizados de forma mais fácil e rápida.

A proposta de universal executors ainda está em fase de discussão no Grupo de Trabalho 21 (WG21) e ainda não foi adotada como parte do C++ Standard. No entanto, se aprovada, ela pode ser uma adição importante ao C++ Standard

### Por quê Executores?

C++ sempre careceu de infraestrutura de programação simultânea disponível, e a infraestrutura recém-introduzida desde C++11, bem como a melhoria de bibliotecas de terceiros, como boost e folly, têm mais ou menos problemas e certas limitações.

### `std::async` não é assíncrono

`std::async` é uma função do C++ Standard Library que é usada para iniciar uma tarefa assíncrona em um ponto específico no tempo. Ela é usada para criar uma tarefa assíncrona que será executada em uma thread separada e retorna um objeto `std::future` que pode ser usado para obter o resultado da tarefa quando ela for concluída.

Apesar de seu nome, `std::async` não é uma função assíncrona no sentido tradicional da palavra. Ela não é capaz de retornar imediatamente para o chamador enquanto a tarefa assíncrona é executada, mas simplesmente inicia a tarefa em uma thread separada e retorna um objeto `std::future`. Isso significa que o código que chama `std::async` não pode ser escrito de forma assíncrona usando a sintaxe de await do C++20.

Apesar disso, `std::async` pode ser útil em situações em que é necessário iniciar uma tarefa assíncrona de forma fácil e rápida. Ele é especialmente útil quando é necessário obter o resultado da tarefa assíncrona de forma síncrona, usando a sintaxe de await do C++20 ou esperando pelo objeto `std::future` retornado por `std::async`.

A seguir, um exemplo de uso da função `std::async` para iniciar uma tarefa assíncrona e obter o resultado da tarefa de forma síncrona:

```c++
#include <iostream>
#include <future>

  // Função que será executada assincronamente
  int long_running_task(int x, int y) {
  // Simulando um processamento demorado
  std::this_thread::sleep_for(std::chrono::seconds(2));
  return x + y;
}

int main() {
  // Iniciando a tarefa assíncrona com std::async
  std::future<int> result = std::async(long_running_task, 10, 20);

  // Obtendo o resultado da tarefa síncronamente com std::future::get
  int sum = result.get();

  std::cout << "Resultado da tarefa assíncrona: " << sum << std::endl;

  return 0;
}
```

Neste exemplo, a função long_running_task é iniciada de forma assíncrona com `std::async` e o resultado da tarefa é obtido síncronamente com `std::future::get`. Isso significa que o código que chama `std::async` será bloqueado até que a tarefa seja concluída e o resultado esteja disponível.

Observe que, apesar de usarmos `std::async` para iniciar a tarefa assíncrona, o código não pode ser escrito de forma assíncrona usando a sintaxe de await do C++20. Para escrever código assíncrono de forma mais simples e clara, é recomendável usar outras bibliotecas de tempo de execução, como Asio ou Libunifex.

Em resumo, `std::async` é uma função do C++ Standard Library que é usada para iniciar uma tarefa assíncrona em uma thread separada. Ela não é uma função assíncrona no sentido tradicional da palavra e não pode ser usada com a sintaxe de await do C++20, mas pode ser útil em situações em que é necessário iniciar uma tarefa assíncrona de forma fácil e rápida.


### Modelo de Evolução do Future/Promise

No C++11, o modelo future/promise é um meio de permitir que uma thread espere por um valor a ser produzido por outra thread de maneira assíncrona. Ele é composto pelos seguintes elementos:

- `Promise`: um objeto que permite que um valor seja definido em algum momento no futuro. A promessa possui um método setValue para definir o valor e um método `setException` para definir uma exceção a ser lançada quando o valor for solicitado.

- `Future`: um objeto que permite que uma thread espere por um valor produzido por outra thread. O futuro possui um método wait que faz a thread que o chama esperar até que o valor esteja disponível. Além disso, o futuro possui métodos como `then` e `onError` que permitem que ações sejam executadas quando o valor estiver disponível ou uma exceção for lançada.

A implementação de um Future/Promise típico em C++ é mostrada na figura abaixo:
![img](https://user-images.githubusercontent.com/6756180/208737071-abe8d31c-fe0d-4023-8a1a-64083099c4f6.jpg)

Para usar o modelo de futuros e promessas, é preciso criar um objeto promessa e obter um objeto futuro a partir dele. Em seguida, a thread que produzirá o valor deve definir o valor na promessa usando o método `setValue` ou `setException`. A thread que estiver esperando pelo valor pode usar o método wait do futuro para esperar até que o valor esteja disponível.

Aqui está um exemplo de como usar o modelo de futuros e promessas em C++:

```c++
#include <future>
#include <iostream>

int main() {
  // Cria uma promessa e um futuro
  std::promise<int> promise;
  std::future<int> future = promise.get_future();

  // Define o valor da promessa em uma thread separada
  std::thread([&promise] {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    promise.set_value(42);
  }).detach();

  // Espera pelo valor a ser definido e imprime-o
  std::cout << future.get() << std::endl;

  return 0;
}
```

Este código cria um objeto `std::promise` e um objeto `std::future`, e define o valor da `std::promise` em uma thread separada usando um `std::thread`. O objeto `std::future` é então usado para esperar pelo valor a ser definido, e o valor é impresso no console.

Essas classes são mais básicas do que as oferecidas pelas bibliotecas `folly` e `asio`, mas são parte da Biblioteca Padrão de C++ e, portanto, estão disponíveis em qualquer compilador C++ padrão.
As classes `std::future` e `std::promise` fornecem um conjunto similar de funcionalidades às classes `folly::Future` e `folly::Promise` da biblioteca [folly](https://github.com/facebook/folly), mas são parte da Biblioteca Padrão de C++ e não exigem dependências adicionais.

  
### Executores do Asio


Os executores são componentes do asio que definem o contexto de execução de uma função ou um bloco de código. Eles podem ser usados para controlar como e quando uma função ou um bloco de código é executado, e permitem que você aproveite os recursos de concorrência fornecidos pelo asio para executar tarefas de forma assíncrona e concorrente.

O asio fornece várias formas de se trabalhar com executores, incluindo a possibilidade de especificar o contexto de execução de uma função ou um bloco de código usando o template `asio::execution`, ou usando funções como `asio::post`, que permitem agendar a execução de uma função ou um bloco de código em um determinado contexto de execução.

`asio::execution` é um modelo C++ que representa um contexto de execução, ou um objeto que define como uma função ou um bloco de código deve ser executado. É usado como um parâmetro de tipo em vários componentes asio, como `asio::strand` e `asio::spawn`, para especificar o contexto de execução no qual uma função ou um bloco de código deve ser executado.

Existem vários tipos de executores que podem ser usados com `asio::execution`. Estes incluem:

- `asio::io_context::executor_type`: Este é o tipo de executor padrão para `asio::io_context` e representa o contexto de execução fornecido por um objeto `asio::io_context`. As funções ou blocos de código executados usando este executor serão executados no contexto do loop de eventos do io_context.

- `asio::strand<Executor>`: Este é um executor decorador que envolve outro executor, Executor, e garante que as funções ou blocos de código executados com ele sejam executados de forma serializada, ou seja, apenas um de cada vez. Isso pode ser útil para sincronizar o acesso a recursos compartilhados.

- `asio::system_executor`: Este é um tipo de executor especial que representa o contexto de execução fornecido pelo sistema operacional. As funções ou blocos de código executados usando este executor serão executados no contexto da thread pool do sistema operacional.

- `asio::thread_pool`: Este é um tipo de executor que representa um pool (grupo) de threads que podem serem usados para executar funções ou blocos de código, ou também para executar tarefas de forma concorrente em múltiplas threads.

- `asio::post`: Esta é uma função que leva uma função ou um bloco de código e um executor e agenda a função ou o bloco de código para ser executado no contexto do executor especificado.

Em geral, `asio::execution` é um conceito poderoso que permite especificar o contexto de execução no qual uma função ou um bloco de código deve ser executado e aproveitar os vários tipos de executores fornecidos por asio para controlar a execução do seu código.


### Asio executores em comparação com outras alternativas


#### `std::execution`


`std::execution` é um namespace do C++ Standard Library que fornece tipos e funções relacionados à execução de tarefas assíncronas. Ele foi introduzido no C++17 e ampliado no C++20 para fornecer uma interface padronizada para a execução de tarefas assíncronas em diferentes plataformas e bibliotecas de tempo de execução.

O conceito de Executor é uma parte importante do namespace `std::execution`. Ele é um tipo de modelo de classe que define uma interface para a execução de tarefas assíncronas. Um Executor é responsável por agendar tarefas para serem executadas em um determinado ponto no tempo, permitindo que o código assíncrono seja escrito de forma mais simples e clara.

O conceito de Executor é importante porque ele permite que você escreva código assíncrono de forma mais genérica, pois você pode usar a mesma interface para trabalhar com diferentes bibliotecas de tempo de execução sem precisar se preocupar com as diferenças entre elas. Isso torna o código mais portável e facilita a manutenção e a expansão do código no futuro.

O namespace `std::execution` fornece vários tipos de Executor, como `std::execution::sequenced_policy` e `std::execution::parallel_policy`, que podem ser usados ​​para controlar como as tarefas são agendadas e executadas. Além disso, ele fornece funções como `std::execution::execute` e `std::execution::bulk_execute`, que podem ser usadas para executar tarefas de forma assíncrona de acordo com o Executor especificado.

Em resumo, `std::execution` é um namespace do C++ Standard Library que fornece uma interface padronizada para a execução de tarefas assíncronas em diferentes plataformas e bibliotecas de tempo de execução, enquanto que Asio é uma biblioteca de tempo de execução que oferece recursos para criar aplicações de rede de forma assíncrona. Asio pode ser usado com o namespace `std::execution`, mas também pode ser usado de forma independente. A escolha da biblioteca a ser usada depende das necessidades específicas de sua aplicação e de suas preferências de programação.

#### Libunifex

[Libunifex](https://github.com/facebookexperimental/libunifex) e Asio são duas bibliotecas diferentes que são utilizadas para criar aplicações de redes de forma assíncrona em C++.

ASIO é uma biblioteca de tempo de execução que oferece suporte para a comunicação assíncrona entre sistemas de computador. Ela é amplamente utilizada para a criação de aplicações de redes, como servidores de rede e clientes de rede. Asio fornece uma série de recursos, como sockets de rede, temporizadores e sinais de interrupção, que podem ser usados ​​para criar aplicações de rede de forma assíncrona.

Por outro lado, Libunifex é uma biblioteca de executores para C++ que oferece uma interface uniforme para a execução de tarefas assíncronas em diferentes plataformas e bibliotecas de tempo de execução, como Asio Standalone e Boost.ASIO. Ela permite que você escreva código assíncrono de forma mais portável, pois você pode usar a mesma interface para trabalhar com diferentes bibliotecas de tempo de execução sem precisar se preocupar com as diferenças entre elas.

#### Cppcoro

[Cppcoro](https://github.com/lewissbaker/cppcoro) é uma alternativa a outras bibliotecas de tempo de execução para C++, como Asio e Libunifex, que também oferecem suporte para a programação assíncrona, porém com o uso de corrotinas ao invés de executores. Ela permite que você escreva código assíncrono de forma mais simples e clara, usando a sintaxe de corrotinas do C++20 STL.

Cppcoro usa a funcionalidade de coroutinas introduzida no C++20 para permitir que você escreva código assíncrono de forma mais fácil e natural. Ele fornece uma série de funções e tipos de dados que permitem que você crie, gerencie e execute coroutinas de forma mais eficiente. Além disso, ele fornece suporte para a execução de coroutinas em paralelo, o que pode ser útil em aplicações de alta performance.

Em resumo, Asio é uma biblioteca de tempo de execução que fornece recursos para criar aplicações de rede de forma assíncrona, enquanto que Libunifex é uma biblioteca de executores que oferece uma interface uniforme para trabalhar com diferentes bibliotecas de tempo de execução de forma mais portável.