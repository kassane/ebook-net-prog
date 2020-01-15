# Operações Assíncronas E/S

Diferentemente da API de sockets do `UNIX`, o` boost.asio` possui habilidades de leitura & gravação(read/write) assíncronas inclusas. Ainda pode usar `basic_stream_socket` como exemplo, e um par de implementações assim:

```cpp
	template <typename ConstBufferSequence, typename WriteHandler>
	  BOOST_ASIO_INITFN_RESULT_TYPE(WriteHandler,
	      void (boost::system::error_code, std::size_t))
	  async_send(const ConstBufferSequence& buffers,
	      BOOST_ASIO_MOVE_ARG(WriteHandler) handler)
	{
		.......
	}
	template <typename MutableBufferSequence, typename ReadHandler>
	  BOOST_ASIO_INITFN_RESULT_TYPE(ReadHandler,
	      void (boost::system::error_code, std::size_t))
	  async_receive(const MutableBufferSequence& buffers,
	      BOOST_ASIO_MOVE_ARG(ReadHandler) handler)
	{
		.......
	}
```
Como as funções `async_send` e `async_receive` retornam imediatamente, e não bloqueiam a thread atual, você deve passar uma função de retorno de chamada como o parâmetro que recebe o resultado das operações de leitura & gravação:

```cpp
	void handler(
		const boost::system::error_code& error, // Result of operation.
		std::size_t bytes_transferred           // Number of bytes processed.
	)
```

Há um exemplo simples de cliente/servidor. Abaixo está o código do cliente:  

```cpp
	#include <boost/asio.hpp>
	#include <functional>
	#include <iostream>
	#include <memory>
	
	void callback(
	        const boost::system::error_code& error,
	        std::size_t bytes_transferred,
	        std::shared_ptr<boost::asio::ip::tcp::socket> socket,
	        std::string str)
	{
	    if (error)
	    {
	        std::cout << error.message() << '\n';
	    }
	    else if (bytes_transferred == str.length())
	    {
	        std::cout << "Message is sent successfully!" << '\n';
	    }
	    else
	    {
	        socket->async_send(
	                boost::asio::buffer(str.c_str() + bytes_transferred, str.length() - bytes_transferred),
	                std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str));
	    }
	}	
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::endpoint endpoint{
	                boost::asio::ip::make_address("192.168.35.145"),
	                3303};
	
	        std::shared_ptr<boost::asio::ip::tcp::socket> socket{new boost::asio::ip::tcp::socket{io_context}};
	        socket->connect(endpoint);
	
	        std::cout << "Connect to " << endpoint << " successfully!\n";
	
	        std::string str{"Hello world!"};
	        socket->async_send(
	                boost::asio::buffer(str),
	                std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str));
	        socket->get_executor().context().run();
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << '\n';
	        return -1;
	    }
	
	    return 0;
	}
```

Vamos analisar o código:  

(1) Como o objeto sockets é non-copyable ([sockets](socket.md)), sockets é criado como um ponteiro inteligente de memória compartilhada (shared_pointer):  

```cpp
	......
	std::shared_ptr<boost::asio::ip::tcp::socket> socket{new boost::asio::ip::tcp::socket{io_context}};
	......
```

(2) Como o `callback` possui apenas dois parâmetros, ele precisa usar `std::bind` para passar parâmetros adicionais:

```cpp
	......
	std::bind(callback, std::placeholders::_1, std::placeholders::_2, socket, str)
	......
```

(3) `async_send` não garante que todos os bytes sejam enviados (`boost::asio::async_write` retorna todos os bytes enviados com sucesso ou ocorre um erro), portanto, é necessário reemitir `async_send` no `callback`:  

```cpp
	......
	if (error)
	{
	    ......
	}
	else if (bytes_transferred == str.length())
	{
	    ......
	}
	else
	{
	    socket->async_send(......);
	}
```
(4) A função `io_context.run` será bloqueada até que todo o trabalho termine e não haja mais handlers(manipuladores) a serem despachados, ou até que o` io_context` seja interrompido:

```cpp
	socket->get_executor().context().run();
```
Se não houver a função `io_context.run`, o programa será encerrado imediatamente.  

Verifique o código do servidor que usa `async_receive`:  

```cpp
	#include <ctime>
	#include <functional>
	#include <iostream>
	#include <string>
	#include <boost/asio.hpp>
	
	void callback(
	        const boost::system::error_code& error,
	        std::size_t,
	        char recv_str[]) {
	    if (error)
	    {
	        std::cout << error.message() << '\n';
	    }
	    else
	    {
	        std::cout << recv_str << '\n';
	    }
	}
	
	int main()
	{
	    try
	    {
	        boost::asio::io_context io_context;
	
	        boost::asio::ip::tcp::acceptor acceptor(
	                                        io_context,
	                                        boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(), 3303));
	
	        for (;;)
	        {
	            boost::asio::ip::tcp::socket socket(io_context);
	            acceptor.accept(socket);
	
	            char recv_str[1024] = {};
	            socket.async_receive(
	                    boost::asio::buffer(recv_str),
	                    std::bind(callback, std::placeholders::_1, std::placeholders::_2, recv_str));
	            socket.get_executor().context().run();
	            socket.get_executor().context().restart();
	        }
	    }
	    catch (std::exception& e)
	    {
	        std::cerr << e.what() << std::endl;
	    }
	
	    return 0;
	}
```
	
Há duas advertências às quais você precisa prestar atenção:  

(1) Apenas para fins de demonstração: para cada cliente, o `callback` é chamado apenas uma vez;  
(2) O `io_context.restart` deve ser chamado para chamar outro` io_context.run`.  

Da mesma forma, você também pode verificar como usar o `boost::asio::async_read`.

Compile e execute os programas.

O cliente exibirá o seguinte:  

	$ ./client
	Connect to 192.168.35.145:3303 successfully!
	Message is sent successfully!

Servidor emitirar o seguinte:  

	$ ./server
	Hello world!
