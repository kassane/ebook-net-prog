# Conectar Servidor

O cliente pode usar os endpoints retornados por [DNS](dns-query.md) para conectar o aplicativo servidor. Veja o código abaixo:

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
	
	        boost::asio::ip::tcp::tcp::socket socket{io_context};
	        auto endpoint = boost::asio::connect(socket, endpoints);
	
	        std::cout << "Connect to " << endpoint << " successfully!\n";
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

	Connect to 172.217.194.101:443 successfully!

Observe que o `boost::asio::connect` requer o iterador de endpoints. Se você quiser apenas um endpoint específico, poderá usar a função membro `connect` do socket. Verifique o código abaixo:

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
	
	        boost::asio::ip::tcp::tcp::socket socket{io_context};
	        auto eit = endpoints.cbegin();
	        for (; eit != endpoints.cend(); eit++)
	        {
	            boost::system::error_code ec;
	            boost::asio::ip::tcp::endpoint endpoint = *eit;
	            socket.connect(endpoint, ec);
	            if (!ec)
	            {
	                std::cout << "Connect to " << endpoint << " successfully!\n";
	                break;
	            }
	        }
	
	        if (eit == endpoints.cend())
	        {
	            std::cout << "Connect failed!\n";
	            return  -1;
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

	Connect to 172.217.194.139:443 successfully!
