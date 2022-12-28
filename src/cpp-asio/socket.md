# Sockets (Soquetes)

A classe `asio::basic_stream_socket` do Asio é uma classe genérica que é usada como base para a criação de classes de socket específicas para diferentes protocolos de rede. Ela fornece uma interface comum para realizar operações de entrada e saída em sockets e é a classe base para a criação de classes de socket para protocolos como TCP, UDP, ICMP e SCTP.

A `asio::basic_stream_socket` fornece uma série de funções de leitura e escrita assíncronas que podem ser usadas para enviar e receber dados através de um socket de forma assíncrona. Ela também oferece funções para estabelecer conexões com outros hosts na rede e para fechar conexões existentes.

Para usar a `asio::basic_stream_socket`, é preciso criar uma classe derivada que especifique o tipo de socket que deseja criar, como um socket TCP, UDP, ICMP ou SCTP. Em seguida, é possível criar um objeto da classe derivada e passar um objeto de resolução de endereço e um objeto de io_context para o construtor. Em seguida, é possível chamar as funções de leitura e escrita fornecidas pela `asio::basic_stream_socket` para enviar e receber dados através do socket.

Existem `4` tipos de sockets:

(1) `basic_stream_socket`:  
Este socket fornece fluxos de bytes baseados em conexão bidirecional, confiável e sequencial. `tcp::socket` é uma instância deste socket:

```cpp
	class tcp
	{
	......
	  /// The TCP socket type.
	  typedef basic_stream_socket<tcp> socket;
	......
	}
```

(2) `basic_datagram_socket`:  
Este socket fornece serviço de datagrama sem garantias de conexão e não confiável. `udp::socket` é uma instância deste socket:

```cpp
	class udp
	{
	......
	  /// The UDP socket type.
  	  typedef basic_datagram_socket<udp> socket;
	......
	}
```
(3) `basic_raw_socket`:  
Este socket fornece acesso a protocolos e interfaces de rede interno. O `icmp::socket` é uma instância deste socket:

```cpp
	class icmp
	{
	......
	  /// The ICMP socket type.
  	  typedef basic_raw_socket<icmp> socket;
	......
	}
```
(4) `basic_seq_packet_socket`:  
Este socket combina fluxo(stream) e datagrama: fornece um serviço de datagramas com conexão bidirecional, confiável e bidirecional. [SCTP](https://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol) é um exemplo deste tipo de serviço.  

Todos esses `4` sockets derivam da classe `basic_socket` e precisam ser associados a um `io_context` durante a inicialização. Veja `tcp::socket` como exemplo:

```cpp
		boost::asio::io_context io_context;
		boost::asio::ip::tcp::socket socket{io_context};
```

Observe que o `io_context` deve ser uma referência no construtor do` socket` (consulte [io_context](cpp-asio/io_context.md)). Ainda use `basic_socket` e uma instância, um de seus construtores é o seguinte:

```cpp
	  explicit basic_socket(boost::asio::io_context& io_context)
	    : basic_io_object<BOOST_ASIO_SVC_T>(io_context)
	  {
	  }
```

Para a classe `basic_io_object`, ele não suporta copy constructed/copy assignment:  

```cpp
	......
	private:
	  basic_io_object(const basic_io_object&);
	  void operator=(const basic_io_object&);
	......
```

mas suporta move constructed/move assignment:  

```cpp	
	......
	protected:  
	  basic_io_object(basic_io_object&& other)
	  {
	    ......
	  }
	  basic_io_object& operator=(basic_io_object&& other)
	  {
	    ......
	  }
```

Além das funções de leitura e escrita assíncronas, a `asio::basic_stream_socket` também oferece uma série de outras funcionalidades úteis. Por exemplo, ela permite que o programa configure opções de socket, como o timeout de leitura e escrita, o tamanho do buffer de leitura e escrita e o uso de Keepalives. Ela também permite que o programa obtenha informações sobre o socket, como o endereço local e remoto, o estado da conexão e o número de bytes enviados e recebidos.

Em resumo, a asio::`basic_stream_socket` é uma classe genérica do Asio que é usada como base para a criação de classes de socket específicas para diferentes protocolos de rede. Ela fornece uma interface comum para realizar operações de entrada e saída em sockets e oferece uma série de funcionalidades úteis, como configuração de opções de socket e obtenção de informações sobre o socket.