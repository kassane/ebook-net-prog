## Conexão Serial

A conexão serial é um tipo de conexão de comunicação que permite que dois dispositivos se comuniquem por meio de uma porta serial. A porta serial é uma interface física que permite que um dispositivo envie e receba dados por meio de um par de fios. Ela é comumente usada para se comunicar com dispositivos externos, como Arduinos, dispositivos de comunicação industrial e dispositivos de automação.

### Asio

A biblioteca Asio inclui suporte para trabalhar com portas seriais, o que permite aos programadores escrever aplicativos que se comunicam com dispositivos seriais, como Arduino e dispositivos de comunicação industrial. Isso é útil quando é preciso enviar ou receber dados de um dispositivo por meio de uma porta serial, como em aplicativos de automação industrial ou em projetos de robótica.

Para usar a biblioteca Asio para trabalhar com portas seriais, é necessário incluir o cabeçalho `#include <boost/asio.hpp>` no início do seu código. Em seguida, é preciso criar uma instância de um objeto `serial_port`, que representa a porta serial a ser usada, e passar a ela as informações sobre a porta, como o nome da porta (por exemplo, `"COM1"` no **Windows** ou `"/dev/ttyUSB0"` no **Linux**) e a taxa de transmissão (baud rate).

Aqui está um exemplo de código que abre uma porta serial e envia uma mensagem pelo Arduino:

```c++
#include <boost/asio.hpp>
#include <iostream>

namespace asio = boost::asio;

int main() {
  // Cria um objeto boost::asio::io_context para gerenciar as operações de entrada/saída.
  asio::io_context io;

  // Cria um objeto boost::asio::serial_port para representar a porta serial.
  asio::serial_port serial(io, "/dev/ttyUSB0");

  // Configura a porta serial com a taxa de transmissão desejada.
  serial.set_option(asio::serial_port_base::baud_rate(9600));

  // Envia uma mensagem pelo Arduino.
  asio::write(serial, asio::buffer("Hello, Arduino!\n"));

  return 0;
}
```

Além de enviar e receber dados, você também pode configurar várias opções da porta serial usando a biblioteca Asio. Por exemplo, você pode definir o número de bits de dados, a paridade e o número de bits de parada usando as opções `serial_port_base::character_size`, `serial_port_base::parity` e `serial_port_base::stop_bits`, respectivamente. Aqui está um exemplo de como fazer isso:

```c++
serial.set_option(boost::asio::serial_port_base::character_size(8));
serial.set_option(boost::asio::serial_port_base::parity(boost::asio::serial_port_base::parity::none));
serial.set_option(boost::asio::serial_port_base::stop_bits(boost::asio::serial_port_base::stop_bits::one));
```

Você também pode definir o modo de fluxo de dados da porta serial usando as opções `serial_port_base::flow_control`. Por exemplo, para habilitar o controle de fluxo hardware (RTS/CTS), você pode usar o seguinte código:

```c++
serial.set_option(boost::asio::serial_port_base::flow_control(boost::asio::serial_port_base::flow_control::hardware));
```

Além disso, a biblioteca permite que você trate eventos de interrupção da porta serial usando a classe `serial_port_service`. Isso é útil quando você precisa ser notificado quando a porta serial for interrompida, por exemplo, quando um dispositivo conectado à porta envia um sinal de interrupção. Para usar essa funcionalidade, você precisa criar um objeto `serial_port_service` e registrar um manipulador de eventos de interrupção usando a função `async_wait_for_interrupt()`. Aqui está um exemplo de como fazer isso:

```c++
#include <boost/asio/serial_port_service.hpp>

// Define uma função de callback para ser chamada quando a interrupção ocorrer.
void interrupt_callback(const boost::system::error_code& error) {
  if (!error) {
    // A interrupção ocorreu. Faça alguma coisa aqui.
    std::cout << "Interrupt received!" << std::endl;
  }
}

int main() {
    // Cria um objeto boost::asio::io_context para gerenciar as operações de entrada/saída.
    boost::asio::io_context io;

    // Cria um objeto boost::asio::serial_port para representar a porta serial.
    boost::asio::serial_port serial(io, "/dev/ttyUSB0");

    // Cria um objeto boost::asio::serial_port_service para tratar eventos de interrupção da porta serial
    boost::asio::serial_port_service serial_service(io);

    // Registra o manipulador de eventos de interrupção.
    serial_service.async_wait_for_interrupt(serial, interrupt_callback);

    // Executa o loop de eventos da biblioteca Asio. Isso fará com que o manipulador de interrupção seja chamado quando a interrupção ocorrer.
    io.run();
}
```

Essa é uma maneira de tratar eventos de interrupção da porta serial usando a biblioteca Asio. Note que você precisa executar o loop de eventos (chamando a função `io_context::run()`) para que os manipuladores de eventos sejam chamados quando os eventos ocorrerem.
