# Accept connections

O servidor precisa aceitar as solicitações dos clientes. O servidor cria um `acceptor`:  

```cpp
	......
	boost::asio::io_context io_context;
        boost::asio::ip::tcp::acceptor acceptor{
            io_context,
            boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v6(), 3303}};
	......
```
`boost::asio::ip::tcp::acceptor` é uma instância de `basic_socket_acceptor`:  

```cpp
	class tcp
	{
	......
	  /// The TCP acceptor type.
	  typedef basic_socket_acceptor<tcp> acceptor;
	......
	}
```

O construtor `basic_socket_acceptor` combinarar criação de soquete, configuração de endereço de reutilização, funções binding & listening:

```cpp
	basic_socket_acceptor(boost::asio::io_context& io_context,
	    const endpoint_type& endpoint, bool reuse_addr = true)
	  : basic_io_object<BOOST_ASIO_SVC_T>(io_context)
	{
	......
	}
```

Então o `acceptor` aceitará as conexões dos clientes. O código abaixo mostra o endereço do cliente e fecha a conexão:

```cpp
	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	        boost::asio::ip::tcp::acceptor acceptor{
	            io_context,
	            boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v6(), 3303}};
	
	        while (1)
	        {
	            boost::asio::ip::tcp::socket socket{io_context};
	            acceptor.accept(socket);
	
	            std::cout << socket.remote_endpoint() << " connects to " << socket.local_endpoint() << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}
```

O resultado da execução será:  

	[::ffff:10.217.242.61]:39290 connects to [::ffff:192.168.35.145]:3303
	......
