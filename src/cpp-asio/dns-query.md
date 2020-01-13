# DNS Query

A classe `resolver` é usada para fazer consultas [`DNS`](https://pt.wikipedia.org/wiki/Sistema_de_Nomes_de_Dom%C3%ADnio), ou seja, converter um serviço host + em `IP` + porta. Veja `boost::asio::ip::tcp::resolver` no exemplo abaixo:

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