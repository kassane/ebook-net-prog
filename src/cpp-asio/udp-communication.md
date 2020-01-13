# Protocolo UDP

Nós discutimos como se comunicar através do `TCP` o suficiente, então é hora de mudar para o` UDP` agora. O `UDP` é um protocolo de conexão não confiável e é mais fácil de usar que o `TCP`.

Há um exemplo de Cliente/Servidor. 

### *Cliente*:

```cpp
	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::udp::socket socket{io_context};
	        socket.open(boost::asio::ip::udp::v4());
	
	        socket.send_to(
	                boost::asio::buffer("Welcome to C++ Networking."),
	                boost::asio::ip::udp::endpoint{boost::asio::ip::make_address("192.168.35.145"), 3303});
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}
```

Embora não seja necessário chamar a função `socket.connect`, você precisa chamar explicitamente o `socket.open`. Além disso, o `endpoint` do servidor precisa ser especificado ao chamar `socket.send_to`.

### *Servidor*:

```cpp
	#include <ctime>
	#include <functional>
	#include <iostream>
	#include <string>
	#include <boost/asio.hpp>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        for (;;)
	        {
	            boost::asio::ip::udp::socket socket(
	                    io_context,
	                    boost::asio::ip::udp::endpoint{boost::asio::ip::udp::v4(), 3303});
	
	            boost::asio::ip::udp::endpoint client;
	            char recv_str[1024] = {};
	
	            socket.receive_from(
	                    boost::asio::buffer(recv_str),
	                    client);
	            std::cout << client << ": " << recv_str << '\n';
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << std::endl;
	    }
	
	    return 0;
	}
```

<!-- Very easy, isn't it? Build and run client and server. The following log will be printed on server side:   -->
Muito fácil, não é? Crie e execute o cliente e servidor. Então o seguinte log será impresso no lado do servidor:

	$ ./servidor
	10.217.242.21:63838: Welcome to C++ Networking.
	10.217.242.21:61259: Welcome to C++ Networking.
