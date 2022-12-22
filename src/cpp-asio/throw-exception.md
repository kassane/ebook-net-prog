# Exceção

O C++ ASIO fornece uma série de exceções de erro que podem ser lançadas em situações em que ocorrem erros durante a execução de operações de rede. Essas exceções são derivadas da classe `boost::system::system_error` e incluem:

- `boost::asio::error::address_family_not_supported`: lançada quando o tipo de endereço especificado não é suportado pela plataforma.
- `boost::asio::error::address_in_use`: lançada quando o endereço especificado já está em uso por outro processo.
- `boost::asio::error::connection_aborted`: lançada quando a conexão é abortada pelo host remoto.
- `boost::asio::error::connection_refused`: lançada quando a conexão é recusada pelo host remoto.
- `boost::asio::error::connection_reset`: lançada quando a conexão é reiniciada pelo host remoto.

Essas exceções são lançadas pelos métodos do C++ ASIO que realizam operações de rede, como `boost::asio::ip::tcp::acceptor::accept` ou `boost::asio::ip::tcp::socket::connect`. Elas podem ser capturadas pelo programa e tratadas de acordo com o erro específico que ocorreu.

As funções do `Asio` podem gerar a exceção `boost::system::system_error`. Veja o [`resolve`](dns-query.md) no exemplo abaixo:  

```cpp
	results_type resolve(BOOST_ASIO_STRING_VIEW_PARAM host,
		BOOST_ASIO_STRING_VIEW_PARAM service, resolver_base::flags resolve_flags)
	{
	  boost::system::error_code ec;
	  ......
	  boost::asio::detail::throw_error(ec, "resolve");
	  return r;
	}
```

Há duas sobrecargas de funções `boost::asio::detail::throw_error`:  

```cpp
	inline void throw_error(const boost::system::error_code& err)
	{
	  if (err)
	    do_throw_error(err);
	}
	
	inline void throw_error(const boost::system::error_code& err,
	    const char* location)
	{
	  if (err)
	    do_throw_error(err, location);
	}
```
As diferenças dessas duas funções é que estão apenas incluindo a string "location" ("`resolve`" no nosso exemplo) ou não. Assim, o `do_throw_error` também tem duas sobrecargas, veja uma como exemplo:

```cpp
	void do_throw_error(const boost::system::error_code& err, const char* location)
	{
	  boost::system::system_error e(err, location);
	  boost::asio::detail::throw_exception(e);
	}
```
`boost::system::system_error` deriva de `std::runtime_error`:  

```cpp
	class BOOST_SYMBOL_VISIBLE system_error : public std::runtime_error
	{
	......
	public:
	      system_error( error_code ec )
	          : std::runtime_error(""), m_error_code(ec) {}
	
	      system_error( error_code ec, const std::string & what_arg )
	          : std::runtime_error(what_arg), m_error_code(ec) {}
	......
	      const error_code &  code() const BOOST_NOEXCEPT_OR_NOTHROW { return m_error_code; }
	      const char *        what() const BOOST_NOEXCEPT_OR_NOTHROW;
	......
	}
```

A função membro chamado `what()` retorna as informações detalhadas da exceção.

Em resumo, o C++ ASIO fornece uma série de exceções de erro que podem ser lançadas em situações em que ocorrem erros durante a execução de operações de rede. Essas exceções são derivadas da classe `boost::system::system_error` ou `asio::system_error` (**standalone**) e incluem erros comuns de rede, como conexão abortada, conexão recusada ou endereço em uso. Elas podem ser capturadas pelo programa e tratadas de acordo com o erro específico que ocorreu.