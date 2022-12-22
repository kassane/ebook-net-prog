# `asio::io_context`

O C++ ASIO possui uma classe chamada `asio::io_context` que é usada para gerenciar operações de entrada e saída assíncronas em um programa de computador. Ela é usada para monitorar e gerenciar operações de entrada e saída em vários descritores de arquivo e sockets, permitindo que o programa seja notificado quando os dados estão disponíveis para leitura ou quando os dados podem ser escritos sem bloquear o processador.

A `asio::io_context` é geralmente usada em conjunto com um objeto de work, que é responsável por manter o loop de eventos da `asio::io_context` rodando. O loop de eventos monitora os descritores de arquivo e sockets gerenciados pela `asio::io_context` e notifica o programa quando os dados estão disponíveis para leitura ou quando os dados podem ser escritos.

O conceito é baseado na API de rede do `Unix`, o `Asio` também possui o conceito "socket", mas isso não é suficiente, um objeto `io_context` (a classe [`io_service`](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0112r0.html) está obsoleta agora) é necessário para se comunicar com os serviços da E/S do sistema operacional. A  imagem abaixo mostrará a estrutura da Arquitetura `Asio`:

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

Observe a classe `noncopyable`:

```cpp
	class noncopyable
	{
	protected:
	  noncopyable() {}
	  ~noncopyable() {}
	private:
	  noncopyable(const noncopyable&);
	  const noncopyable& operator=(const noncopyable&);
	};
```

Isso significa que o objeto `io_context` não pode ser utilizado como _copy constructed/copy assignment/move constructed/move assignment_. Portanto, durante a inicialização do socket, ou seja, associar o socket ao` io_context`, o `io_context` deve ser passado como referência.

Ex.:

```cpp
	template <typename Protocol
	    BOOST_ASIO_SVC_TPARAM_DEF1(= datagram_socket_service<Protocol>)>
	class basic_datagram_socket
	  : public basic_socket<Protocol BOOST_ASIO_SVC_TARG>
	{
	public:
	......
	  explicit basic_datagram_socket(boost::asio::io_context& io_context)
	    : basic_socket<Protocol BOOST_ASIO_SVC_TARG>(io_context)
	  {
	  }
	......
	}
```
Além de gerenciar operações de entrada e saída assíncronas, a `asio::io_context` também fornece uma série de outras funcionalidades úteis. Por exemplo, ela permite que o programa agende operações para serem executadas em um momento futuro, permitindo que o programa execute tarefas de forma assíncrona de acordo com um cronograma. Ela também permite que o programa cancele operações que estão em andamento, permitindo que o programa interrompa tarefas que não são mais necessárias.

Outra funcionalidade útil da `asio::io_context` é a capacidade de escalonar operações em vários threads. Isso é útil em situações em que é necessário realizar várias operações de entrada e saída ao mesmo tempo e é importante aproveitar ao máximo o poder de processamento do computador. A `asio::io_context` pode ser configurada para escalonar operações em vários threads, permitindo que elas sejam executadas em paralelo e aproveitando ao máximo o poder de processamento do computador.

Em resumo, a `asio::io_context` é uma classe fundamental do C++ ASIO que é usada para gerenciar operações de entrada e saída assíncronas em um programa de computador. Ela monitora e gerencia descritores de arquivo e sockets, permitindo que o programa seja notificado quando os dados estão disponíveis para leitura ou quando os dados podem ser escritos sem bloquear o processador. Além disso, a `asio::io_context` fornece uma série de outras funcionalidades úteis, como agendamento de operações para serem executadas em um momento futuro, cancelamento de operações em andamento e escalonamento de operações em vários threads.

A seguir, estão descritas todas as funções comuns do `asio::io_context`:

- `run`: A função run é usada para iniciar o loop de eventos do `asio::io_context`. Ela bloqueia o thread atual até que todas as operações agendadas tenham sido concluídas ou o `asio::io_context` seja interrompido.

- `poll`: A função poll é similar à função run, mas não bloqueia o thread atual. Em vez disso, ela processa todas as operações pendentes no `asio::io_context` e retorna imediatamente.

- `run_one`: A função run_one é similar à função run, mas processa apenas uma operação pendente no `asio::io_context` antes de retornar.

- `stop`: A função stop é usada para interromper o loop de eventos do `asio::io_context`. Isso é útil em situações em que o programa precisa sair do loop de eventos antes que todas as operações pendentes tenham sido concluídas.

- `reset`: A função reset é usada para reiniciar o `asio::io_context`. Isso é útil em situações em que o programa precisa começar a processar operações pendentes novamente após ter sido interrompido.

- `poll_one`: A função poll_one é similar à função poll, mas processa apenas uma operação pendente no asio::io_context antes de retornar.

- `poll_one_at`: A função poll_one_at é similar à função poll_one, mas retorna após o tempo especificado, mesmo se não houver operações pendentes para processar.

- `stopped`: A função stopped retorna true se o `asio::io_context` foi interrompido e false caso contrário.

- `get_executor`: A função get_executor retorna um objeto executor que pode ser usado para agendar operações para serem executadas no `asio::io_context`.

- `dispatch`: A função dispatch é usada para agendar uma operação para ser executada de forma síncrona no `asio::io_context`. Isso garante que a operação seja executada imediatamente, sem esperar por outras operações pendentes.

- `post`: A função post é usada para agendar uma operação para ser executada de forma assíncrona no `asio::io_context`. Isso permite que o programa continue rodando enquanto a operação é executada em segundo plano.

- `work`: Isso informa ao `asio::io_context` que há operações pendentes e garante que o loop de eventos continue rodando, mesmo quando não há operações pendentes.

- `work_guard`: A classe `asio::work_guard` é usada para gerenciar um objeto `asio::work` adicionado ao `asio::io_context`. Ela garante que o objeto `asio::work` não seja removido do `asio::io_context` enquanto o objeto asio::work_guard estiver em uso. Quando o objeto `asio::work_guard` é destruído, o objeto `asio::work` é removido do `asio::io_context` e o loop de eventos pode parar, se não houver mais operações pendentes para processar.
Além de gerenciar o objeto `asio::work`, `asio::work_guard` também fornece uma série de outras funcionalidades úteis. Por exemplo, é possível usar a função reset para adicionar um novo objeto `asio::work` ao `asio::io_context`, mesmo se o objeto asio::work_guard já estiver gerenciando um objeto `asio::work`. Além disso, é possível usar a função get_executor para obter um objeto executor que pode ser usado para agendar operações para serem executadas no `asio::io_context`.

Em resumo, `asio::work_guard` é uma classe do C++ ASIO que é usada para gerenciar um objeto `asio::work` adicionado ao `asio::io_context`. Ela garante que o objeto `asio::work` não seja removido do `asio::io_context` enquanto o objeto `asio::work_guard` estiver em uso, garantindo assim que o loop de eventos continue rodando, mesmo quando não há operações pendentes para processar. Além disso, a `asio::work_guard` fornece uma série de outras funcionalidades úteis, como a possibilidade de adicionar um novo objeto `asio::work` ao `asio::io_context` e de obter um objeto executor para agendar operações para serem executadas no `asio::io_context`. `asio::work_guard` é especialmente útil em situações em que o programa precisa garantir que o asio::io_context continue rodando por um período prolongado de tempo, mesmo quando não há operações pendentes para processar. Isso é comum em programas que usam o C++ ASIO para implementar serviços de rede, como servidores web ou servidores de banco de dados.