# Endpoint (Ponto de Extremidade)

O C++ ASIO fornece outros tipos de objetos endpoint para representar pontos finais de conexões de rede de outros protocolos, além do TCP. Por exemplo, o objeto `asio::ip::udp::endpoint` é usado para representar um ponto final de uma conexão de rede UDP. Ele é composto por um endereço IP e uma porta, e funciona de maneira similar ao objeto `asio::ip::tcp::endpoint`, mas é usado para protocolos UDP em vez de TCP.

O objeto `asio::ip::tcp::endpoint` é usado para representar um ponto final de uma conexão de rede TCP. Ele é composto por um endereço IP e uma porta, que são usados para identificar o host remoto ou o servidor que o programa deseja se conectar ou se comunicar.

Para usar o objeto `asio::ip::tcp::endpoint`, é preciso incluir o cabeçalho `<boost/asio/ip/tcp.hpp>` no seu código e criar um objeto da classe passando um endereço IP e uma porta para o construtor. Em seguida, é possível usar o objeto `asio::ip::tcp::endpoint` para identificar o host remoto ou o servidor que o programa deseja se conectar ou se comunicar.

O objeto `asio::ip::tcp::endpoint` também fornece uma série de outras funcionalidades úteis. Por exemplo, é possível usar as funções address e port para obter o endereço IP e a porta do objeto, respectivamente. Além disso, é possível usar a função `to_string` para obter uma string que representa o objeto `asio::ip::tcp::endpoint`.

Além disso, o C++ ASIO fornece outros tipos de objetos endpoint para representar pontos finais de conexões de outros protocolos, como o objeto `asio::ip::icmp::endpoint` para o protocolo ICMP e o objeto `asio::local::stream_protocol::endpoint` para o protocolo Unix local. Cada um desses objetos endpoint é composto por um endereço e uma porta, e fornece uma série de funcionalidades úteis para trabalhar com os respectivos protocolos de rede.

```cpp
	basic_endpoint(const boost::asio::ip::address& addr, unsigned short port_num)
	    : impl_(addr, port_num)
	{
		//TODO
	}
```

O cliente usa o `endpoint` para designar o endereço do servidor, e o aplicativo do servidor usa o `endpoint` para identificar qual endereço será usado para escutar e aceitar conexões. Um exemplo de [TCP](https://pt.wikipedia.org/wiki/Transmission_Control_Protocol) `endpoint` abaixo:

```cpp
	boost::asio::ip::tcp::endpoint endpoint{
		boost::asio::ip::make_address("127.0.0.1"), 3303};
```

Normalmente, o servidor precisa escutar(listen) todos os endereços da máquina atual e pode recorrer a outro construtor de `basic_endpoint`:

```cpp
	basic_endpoint(const InternetProtocol& internet_protocol,
	      unsigned short port_num)
	    : impl_(internet_protocol.family(), port_num)
	{
	}
```

Um exemplo de servidor [UDP](https://pt.wikipedia.org/wiki/User_Datagram_Protocol) que escuta todos os endereços `IPv4` & `IPv6`:

```cpp
	//IPv6
	boost::asio::ip::udp::endpoint endpoint{
            boost::asio::ip::udp::v6(),
            3303};
	...
	//IPv4
	boost::asio::ip::udp::endpoint endpoint{
            boost::asio::ip::udp::v4(),
            3306};
```

Em resumo, o C++ ASIO fornece uma série de objetos endpoint para representar pontos finais de conexões de rede de diferentes protocolos. Cada um desses objetos endpoint é composto por um endereço e uma porta, e fornece uma série de funcionalidades úteis para trabalhar com os respectivos protocolos de rede.