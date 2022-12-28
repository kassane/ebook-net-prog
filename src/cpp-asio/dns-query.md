# DNS Query

O objeto `asio::ip::tcp::resolver` é usado para resolver nomes de domínio em endereços IP. Ele é útil em situações em que o programa precisa se conectar a um host remoto ou a um servidor usando um nome de domínio, mas precisa do endereço IP para estabelecer a conexão de rede.

Para usar o objeto `asio::ip::tcp::resolver`, é preciso incluir o cabeçalho `<asio/ip/tcp.hpp>` no seu código e criar um objeto da classe passando um objeto `asio::io_context` para o construtor. Em seguida, é possível usar o método resolve para resolver o nome de domínio em um endereço IP. O método resolve retorna um iterador que pode ser usado para percorrer a lista de endereços IP retornados.

O objeto `asio::ip::tcp::resolver` também fornece uma série de outras funcionalidades úteis. Por exemplo, é possível usar o método async_resolve para resolver o nome de domínio de forma assíncrona, permitindo que o programa continue rodando enquanto a resolução é realizada em segundo plano. Além disso, é possível usar o método cancel para cancelar a resolução de um nome de domínio em andamento.

Veja `boost::asio::ip::tcp::resolver` no exemplo abaixo:

```cpp

	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::resolver resolver{io_context};
	        boost::asio::ip::tcp::resolver::results_type endpoints =
	                resolver.resolve("google.com", "https");
	
	        for (auto it = endpoints.cbegin(); it != endpoints.cend(); it++)
	        {
	            boost::asio::ip::tcp::endpoint endpoint = *it;
	            std::cout << endpoint << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	    }
	
	    return 0;
	}
```

O resultado da execução será:   

	74.125.24.101:443
	74.125.24.139:443
	74.125.24.138:443
	74.125.24.102:443
	74.125.24.100:443
	74.125.24.113:443

O elemento `boost::asio::ip::tcp::resolver::results_type` é o iterador de `basic_resolver_entry`: 

```cpp
	template <typename InternetProtocol>
	class basic_resolver_entry
	{
	......
	public:
	  /// The protocol type associated with the endpoint entry.
	  typedef InternetProtocol protocol_type;
	
	  /// The endpoint type associated with the endpoint entry.
	  typedef typename InternetProtocol::endpoint endpoint_type;
	......
	  /// Convert to the endpoint associated with the entry.
	  operator endpoint_type() const
	  {
	    return endpoint_;
	  }
	......
	}
```
Como ele possui o operador `endpoint_type()`, ele pode ser convertido diretamente no endpoint:

```cpp
	boost::asio::ip::tcp::endpoint endpoint = *it;
```

Em resumo, o objeto `asio::ip::tcp::resolver` do Asio é usado para resolver nomes de domínio em endereços IP. Ele fornece uma série de funcionalidades úteis, como a possibilidade de resolver o nome de domínio de forma assíncrona e de cancelar a resolução de um nome de domínio em andamento.