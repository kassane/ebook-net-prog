# Protocolo Raw

Uma das funcionalidades que a ASIO oferece é a possibilidade de enviar e receber pacotes de dados usando o protocolo de rede a baixo nível. Isso é conhecido como "envio/recebimento de pacotes raw", ou simplesmente "raw sockets".

A classe `asio::ip::raw` é usada para implementar a comunicação de pacotes e também fornece uma interface para criar e gerenciar sockets de pacotes raw. Ela também fornece métodos para enviar e receber pacotes raw através de um socket, bem como para gerenciar a conexão e desconexão de clientes.
Para usar raw sockets com a ASIO, é necessário incluir o cabeçalho `<asio/ip/raw_socket.hpp>` e criar uma instância de `asio::ip::raw_socket`, que é a classe responsável por gerenciar a conexão de rede.

Os pacotes raw são pacotes de rede que são enviados e recebidos diretamente, sem qualquer tipo de encapsulamento ou formatação adicional. Eles são usados para implementar protocolos de nível inferior, como o protocolo ICMP, ou para fazer debug de aplicações de rede.

Exemplo de código para enviar um pacote raw:

```c++
#include <iostream>
#include <asio/io_context.hpp>
#include <asio/ip/raw_socket.hpp>

int main() {
  asio::io_context io_context;
  asio::ip::raw_socket socket(io_context);

  // Conecta o socket ao endereço IP e porta especificados
  socket.connect({asio::ip::make_address("127.0.0.1"), 1234});

  // Cria um buffer com os dados a serem enviados
  std::vector<std::uint8_t> data = {0x01, 0x02, 0x03, 0x04};
  asio::const_buffer buffer(data.data(), data.size());

  // Envia o pacote
  socket.send(buffer);

  return 0;
}
```
O exemplo acima mostra como enviar um pacote raw para o endereço IP "127.0.0.1" na porta 1234. O pacote é criado como um buffer de dados e enviado através da chamada ao método `send()` do socket.

Para receber pacotes raw, basta chamar o método `receive()` do socket, passando um buffer para armazenar os dados recebidos.

```c++
#include <iostream>
#include <asio/io_context.hpp>
#include <asio/ip/raw_socket.hpp>

int main() {
  asio::io_context io_context;
  asio::ip::raw_socket socket(io_context);

  // Conecta o socket ao endereço IP e porta especificados
  socket.bind({asio::ip::make_address("127.0.0.1"), 1234});

  // Cria um buffer para armazenar os dados recebidos
  std::vector<std::uint8_t> data(1024);
  asio::mutable_buffer buffer(data.data(), data.size());

  // Recebe o pacote
  std::size_t bytes_received = socket.receive(buffer);

  // Imprime os dados recebidos
  std::cout << "Pacote recebido com " << bytes_received << " bytes: ";
  for (std::size_t i = 0; i < bytes_received; ++i) {
      std::cout << std::hex << std::setw(2) << std::setfill('0') << static_cast<int>(data[i]) << " ";
  }
  std::cout << std::endl;

  return 0;
}
```
No exemplo acima, o socket é conectado ao endereço IP "127.0.0.1" na porta 1234 e cria um buffer para armazenar os dados recebidos. Em seguida, é chamado o método receive() do socket, que bloqueia a execução do programa até que um pacote seja recebido. Quando o pacote é recebido, o número de bytes recebidos é armazenado em bytes_received e os dados são impressos na tela em formato hexadecimal.

É importante notar que, ao trabalhar com pacotes raw, é necessário se preocupar com os detalhes da camada de rede (como cabeçalhos de protocolo, endereçamento, etc.), o que pode ser complexo e trabalhoso. Além disso, o envio/recebimento de pacotes raw geralmente só é necessário em casos especiais, como quando é preciso implementar um protocolo de rede customizado ou realizar testes de baixo nível. Em muitos casos, é mais conveniente usar um protocolo de rede mais alto nível, como o TCP ou o UDP, que já fornecem muitas das funcionalidades necessárias para a comunicação de rede.