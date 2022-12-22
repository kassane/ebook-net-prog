# Protocolo UDP

Nós discutimos como se comunicar através do [TCP](https://pt.wikipedia.org/wiki/Transmission_Control_Protocol) o suficiente, então é hora de mudar para o [UDP](https://pt.wikipedia.org/wiki/User_Datagram_Protocol) agora. O [UDP](https://pt.wikipedia.org/wiki/User_Datagram_Protocol) é um protocolo sem conexão e não confiável, mas é mais fácil de usar que o [TCP](https://pt.wikipedia.org/wiki/Transmission_Control_Protocol).

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
Muito fácil, não é? Crie e execute o cliente e servidor. Então o seguinte log será impresso no lado do servidor:

	$ ./servidor
	10.217.242.21:63838: Welcome to C++ Networking.
	10.217.242.21:61259: Welcome to C++ Networking.


Outro exemplo é um Client NTP:

https://gist.github.com/kassane/9f3b8d8fcd5a1a0adfca6c159d61dc92

```c++
// Usado asio standalone (sem boost)
#include <array>
#include <asio.hpp>
#include <iostream>

using asio::ip::udp;

int main(int argc, char *argv[]) {
  try {
    if (argc != 2) {
      std::cerr << "Usage: ntp_client <host>" << std::endl;
      return 1;
    }

    asio::io_context io_context;

    udp::resolver resolver(io_context);
    udp::endpoint receiver_endpoint =
        *resolver.resolve(udp::v4(), argv[1], "123").begin();

    udp::socket socket(io_context);
    socket.open(udp::v4());

    std::array<char, 48> send_buf = {0x1b, 0, 0, 0, 0, 0, 0, 0, 0};
    socket.send_to(asio::buffer(send_buf), receiver_endpoint);

    std::array<char, 48> recv_buf;
    udp::endpoint sender_endpoint;
    size_t len = socket.receive_from(asio::buffer(recv_buf), sender_endpoint);

    std::cout << "received " << len << " bytes from " << sender_endpoint
              << std::endl;

    // Extract the NTP timestamp from the response
    unsigned long long int ntp_timestamp =
      (unsigned long long int)(recv_buf[40]) << 24 |
      (unsigned long long int)(recv_buf[41]) << 16 |
      (unsigned long long int)(recv_buf[42]) << 8 |
      (unsigned long long int)(recv_buf[43]);

   // Convert the timestamp to a std::chrono::system_clock::time_point
    std::chrono::system_clock::time_point time_point =
        std::chrono::system_clock::time_point(
            std::chrono::seconds(ntp_timestamp - 2208988800ull));

    // Convert the time_point to a std::time_t and print it
    std::time_t time = std::chrono::system_clock::to_time_t(time_point);
    std::cout << std::ctime(&time) << std::endl;

  } catch (std::exception &e) {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```