# Protocolo Raw

Uma das funcionalidades que aAsio oferece é a possibilidade de enviar e receber pacotes de dados usando o protocolo de rede a baixo nível. Isso é conhecido como "envio/recebimento de pacotes raw", ou simplesmente "raw sockets".

A classe `asio::generic::raw_protocol` é usada para implementar a comunicação de pacotes e também fornece uma interface para criar e gerenciar sockets de pacotes raw. Ela também fornece métodos para enviar e receber pacotes raw através de um socket, bem como para gerenciar a conexão e desconexão de clientes.
Para usar raw sockets com aAsio, é necessário incluir o cabeçalho `<asio/generic/raw_socket.hpp>` e criar uma instância de `asio::generic::raw_protocol`, que é a classe responsável por gerenciar a conexão de rede.

Os pacotes raw são pacotes de rede que são enviados e recebidos diretamente, sem qualquer tipo de encapsulamento ou formatação adicional. Eles são usados para implementar protocolos de nível inferior, como o protocolo ICMP, ou para fazer debug de aplicações de rede.

Exemplo de código para enviar um pacote raw:

```c++
#include <iostream>
#include <boost/asio/io_context.hpp>
#include <boost/asio/generic/raw_socket.hpp>

namespace asio = boost::asio;

int main() {
  asio::io_context io_context;
  asio::basic_raw_socket<asio::generic::raw_protocol> socket(io_context);

  // Cria um buffer com os dados a serem enviados
  std::vector<std::uint8_t> data = {0x01, 0x02, 0x03, 0x04};
  asio::const_buffer buffer(data.data(), data.size());

  // Envia o pacote para o endereço IP e porta especificados
  socket.send_to(buffer, asio::ip::udp::endpoint(asio::ip::make_address("127.0.0.1"), 1234));

  return 0;
}
```
O exemplo acima mostra como enviar um pacote raw para o endereço IP "127.0.0.1" na porta 1234. O pacote é criado como um buffer de dados e enviado através da chamada ao método `send_to()` do socket para enviar o buffer de dados para o endereço IP e porta especificados usando o tipo `asio::ip::udp::endpoint`.

Para receber pacotes raw, basta chamar o método `receive_from()` do socket, passando um buffer para armazenar os dados recebidos.

```c++
#include <iostream>
#include <boost/asio/io_context.hpp>
#include <boost/asio/generic/raw_socket.hpp>

namespace asio = boost::asio;

int main() {
  asio::io_context io_context;
  asio::basic_raw_socket<asio::generic::raw_protocol> socket(io_context);

  // Liga o soquete a uma porta específica
  socket.bind(asio::ip::udp::endpoint(asio::ip::make_address("127.0.0.1"), 1234));

  // Recebe os dados através do soquete raw
  char recv_buf[1024];
  asio::ip::udp::endpoint sender_endpoint;
  size_t bytes_received = socket.receive_from(
  asio::buffer(recv_buf, sizeof(recv_buf)), sender_endpoint);

  // Imprime os dados do endpoint
  std::cout << "Received " << bytes_received << " bytes from ";
  std::cout.write(reinterpret_cast<char*>(sender_endpoint.data()), sender_endpoint.size());
  std::cout << std::endl;
  std::cout << "Data: " << recv_buf << std::endl;

  return 0;
}
```
No exemplo acima, o socket é conectado ao endereço IP "127.0.0.1" na porta 1234 e cria um buffer para armazenar os dados recebidos. Em seguida, é chamado o método `receive_from()` do socket, que bloqueia a execução do programa até que um pacote seja recebido. Quando o pacote é recebido, usamos a função data para obter um ponteiro para os dados do endpoint e a função size para obter o tamanho dos dados. Em seguida, usamos a função write da classe ostream para escrever os dados na tela. Porém, o tipo `basic_endpoint<raw_protocol>::data_type` é um ponteiro para um `sockaddr`, enquanto o tipo esperado pela função write é um ponteiro para o tipo caractere (`char*`). Para corrigir isso, basta usar o operador `reinterpret_cast` para converter o ponteiro para `sockaddr` em um ponteiro para caractere. Isso permite que a função write seja chamada com os dados do endpoint.

É importante notar que, ao trabalhar com pacotes raw, é necessário se preocupar com os detalhes da camada de rede (como cabeçalhos de protocolo, endereçamento, etc.), o que pode ser complexo e trabalhoso. Além disso, o envio/recebimento de pacotes raw geralmente só é necessário em casos especiais, como quando é preciso implementar um protocolo de rede customizado ou realizar testes de baixo nível. Em muitos casos, é mais conveniente usar um protocolo de rede mais alto nível, como o TCP ou o UDP, que já fornecem muitas das funcionalidades necessárias para a comunicação de rede.