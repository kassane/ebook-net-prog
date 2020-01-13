# Exceção

As funções do `Boost.Asio` podem gerar a exceção `boost::system::system_error`. Veja o [`resolve`](dns-query.md) no exemplo abaixo:  

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
