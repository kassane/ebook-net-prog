# Soquetes (Sockets)

Existem `4` tipos de soquetes:

(1) `basic_stream_socket`:  
Este soquete fornece fluxos de bytes baseados em conexão bidirecional, confiável e sequencial. `tcp::socket` é uma instância deste soquete:

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
Este soquete fornece serviço de datagrama sem garatias de conexão. `udp::socket` é uma instância deste soquete:

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
<!-- This socket provides access to internal network protocols and interfaces. `icmp::socket` is an instance of this socket:   -->
Este soquete fornece acesso a protocolos e interfaces de rede interno. O `icmp::socket` é uma instância deste soquete:

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
Este soquete combina fluxo e datagrama: fornece um serviço de datagramas com conexão bidirecional, confiável e bidirecional. [SCTP](https://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol) é um exemplo deste tipo de serviço.  

<!-- All these `4` sockets derive from `basic_socket` class, and need to associate with an `io_context` during initialization. Take `tcp::socket` as an example:   -->
Todos esses `4` soquetes derivam da classe `basic_socket` e precisam ser associados a um `io_context` durante a inicialização. Veja `tcp::socket` como exemplo:

```cpp
		boost::asio::io_context io_context;
		boost::asio::ip::tcp::socket socket{io_context};
```

Observe que o `io_context` deve ser uma referência no construtor do` socket` (consulte [io_context](io_context.md)). Ainda use `basic_socket` e uma instância, um de seus construtores é o seguinte:

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