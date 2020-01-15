# Operações síncronas E/S

Depois que a conexão é estabelecida, o cliente e o servidor podem se comunicar. Como na API sockets do `UNIX`, o `boost::asio` também fornece as funções `send` e` receive` Use `basic_stream_socket` como exemplo e um par de implementações assim:

```cpp
	template <typename ConstBufferSequence>
  	std::size_t send(const ConstBufferSequence& buffers)
	{
		......
	}
	......
	template <typename MutableBufferSequence>
  	std::size_t receive(const MutableBufferSequence& buffers)
	{
		......
	}
```

Observe que os tipos de buffer de `send/receive` são `ConstBufferSequence/MutableBufferSequence`, e podemos usar a função `boost::asio::buffer` para construir tipos relacionados.

Abaixo está um programa cliente que envia "`Hello world!`" Para o servidor após o estabelecimento da conexão:

```cpp
	#include <boost/asio.hpp>
	#include <iostream>
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::endpoint endpoint{
	                boost::asio::ip::make_address("10.217.242.61"),
	                3303};
	        boost::asio::ip::tcp::tcp::socket socket{io_context};
	        socket.connect(endpoint);
	
	        std::cout << "Connect to " << endpoint << " successfully!\n";
	
	        socket.send(boost::asio::buffer("Hello world!"));
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}
```

O programa servidor que aguarda o recebimento da saudação do cliente:  

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
	                boost::asio::ip::tcp::endpoint{boost::asio::ip::tcp::v4(), 3303}};
	
	        while (1)
	        {
	            boost::asio::ip::tcp::socket socket{io_context};
	            acceptor.accept(socket);
	
	            std::cout << socket.remote_endpoint() << " connects to " << socket.local_endpoint() << '\n';
	
	            char recv_str[1024] = {};
	            socket.receive(boost::asio::buffer(recv_str));
	
	            std::cout << "Receive string: " << recv_str << '\n';
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

Compile e execute os programas. O cliente exibirá o seguinte:  

	$ ./client
	Connect to 10.217.242.61:3303 successfully!

Servidor exibirá  seguinte:  

	$ ./server
	10.217.242.21:64776 connects to 10.217.242.61:3303
	Receive string: Hello world!

Se nenhum erro ocorrer, o `send` pode garantir que pelo menos um byte seja enviado com sucesso, e você deve verificar o valor de retorno para ver se todos os bytes foram enviados com sucesso ou não. `receive` é semelhante a `send`. O `boost::asio::basic_stream_socket` também fornece` read_some` e `write_some`, que têm as mesmas funções que `receive` e `send`.

Se não nos preocuparmos em verificar o estado do meio (bytes parciais são enviados com sucesso), e apenas nos importarmos se todos os bytes serão enviados com sucesso ou não, podemos usar o `boost::asio::write` que usa o `write_some` por baixo. Da mesma forma, não é difícil adivinhar o que `boost::asio::read` faz.