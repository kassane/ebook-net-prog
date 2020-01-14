# io_context

O conceito é baseado na API de rede do `Unix`, o `Boost.Asio` também possui o conceito "socket", mas isso não é suficiente, um objeto `io_context` (a classe `io_service` está obsoleta agora) é necessário para se comunicar com os serviços da E/S do sistema operacional. A  imagem abaixo mostrará a estrutura da Arquitetura `Asio`:

![image](https://raw.githubusercontent.com/kassane/Livro-Programacao-de-Redes/gh-pages/images/architecture.jpg) 

`io_context` deriva de `execution_context`:  

```cpp
	class io_context
	  : public execution_context
	{
	......
	}
```
Enquanto `execution_context` deriva de `noncopyable`:  

```cpp
	class execution_context
	  : private noncopyable
	{
	......
	}
```

Observe a classe `noncopyable`:

```cpp
	class noncopyable
	{
	protected:
	  noncopyable() {}
	  ~noncopyable() {}
	private:
	  noncopyable(const noncopyable&);
	  const noncopyable& operator=(const noncopyable&);
	};
```

Isso significa que o objeto `io_context` não pode ser utilizado como _copy constructed/copy assignment/move constructed/move assignment_. Portanto, durante a inicialização do socket, ou seja, associar o socket ao` io_context`, o `io_context` deve ser passado como referência.

Ex.:

```cpp
	template <typename Protocol
	    BOOST_ASIO_SVC_TPARAM_DEF1(= datagram_socket_service<Protocol>)>
	class basic_datagram_socket
	  : public basic_socket<Protocol BOOST_ASIO_SVC_TARG>
	{
	public:
	......
	  explicit basic_datagram_socket(boost::asio::io_context& io_context)
	    : basic_socket<Protocol BOOST_ASIO_SVC_TARG>(io_context)
	  {
	  }
	......
	}
```