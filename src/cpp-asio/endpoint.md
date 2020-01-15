# Endpoint (Ponto de Extremidade)

Um `endpoint` é o nome para uma entidade em um terminal de uma conexão da [camada de transporte](https://pt.wikipedia.org/wiki/Camada_de_transporte). Ele é composto por: "Endereço [IP](https://pt.wikipedia.org/wiki/Endere%C3%A7o_IP) + porta de conexão":  

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