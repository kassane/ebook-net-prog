# Operações Assíncronas E/S

Diferentemente da API sockets do `UNIX`, o Asio possui habilidades de leitura & gravação (read/write) assíncronas inclusas. Usando `basic_stream_socket` como exemplo, as assinaturas públicas das funções assíncronas são:

```cpp
// Envio assíncrono — retorna imediatamente, handler chamado ao completar
template <typename ConstBufferSequence, typename WriteHandler>
void async_send(const ConstBufferSequence& buffers, WriteHandler&& handler);

// Recebimento assíncrono — retorna imediatamente, handler chamado ao completar
template <typename MutableBufferSequence, typename ReadHandler>
void async_receive(const MutableBufferSequence& buffers, ReadHandler&& handler);
```

Como as funções `async_send` e `async_receive` retornam imediatamente e não bloqueiam a thread atual, você deve passar uma função de retorno de chamada (callback) ou completion token como parâmetro. O callback recebe o resultado das operações:

```cpp
void handler(
    const asio::error_code& error,  // resultado da operação
    std::size_t bytes_transferred   // bytes processados
)
```

> **Dica:** Em código moderno, prefira lambdas a `std::bind` para callbacks — o código fica mais legível e o compilador pode otimizar melhor.

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
	        io_context.run();
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
	            io_context.run();
	            io_context.restart();
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
