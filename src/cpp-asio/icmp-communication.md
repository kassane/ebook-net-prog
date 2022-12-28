# Protocolo ICMP

A classe `asio::ip::icmp` é uma classe em C++ que faz parte da biblioteca Asio (Asynchronous Input/Output) e é usada para implementar a comunicação usando o protocolo ICMP (Internet Control Message Protocol).

O protocolo ICMP é um protocolo de nível de rede que é usado para enviar mensagens de erro e de controle entre dispositivos de rede. Ele é comumente usado para testar a conectividade entre dispositivos de rede, como o comando ping em sistemas operacionais.

A classe `asio::ip::icmp` fornece uma interface para criar e gerenciar sockets ICMP. Ela é usada para criar sockets ICMP, que são usados para enviar e receber mensagens ICMP. Ela também fornece métodos para enviar e receber mensagens ICMP através de um socket, bem como para gerenciar a conexão e desconexão de clientes.

A classe `asio::ip::icmp` é derivada da classe `asio::basic_socket<Protocol>`, que é uma classe genérica que representa um socket de rede. Ela fornece uma interface para criar e gerenciar sockets de rede usando qualquer protocolo de rede suportado pelo Asio. A classe `asio::ip::icmp` é uma especialização da classe `asio::basic_socket<Protocol>` para o protocolo ICMP.

A classe `asio::ip::icmp` fornece os seguintes métodos e funcionalidades:

- `connect`: estabelece uma conexão com um host especificado através de um endpoint ICMP.
- `close`: fecha o socket e interrompe qualquer conexão existente.
- `read`: lê dados de um socket e armazena os dados em um buffer de saída.
- `write`: escreve dados de um buffer em um socket.

Além disso, a classe `asio::ip::icmp` fornece várias configurações de socket, como opções de buffer de entrada e saída, opções de tempo de espera e opções de recurso. Essas opções podem ser ajustadas usando os métodos set_option e get_option da classe `asio::ip::icmp`.

Aqui está um exemplo de como a classe `asio::ip::icmp` pode ser usada para implementar um programa em C++ que envia uma mensagem ICMP para um host especificado e aguarda por uma resposta:

```c++
#include <iostream>
#include <chrono>
#include <asio.hpp>


// Constantes para os campos protocol e type
const unsigned char IPPROTO_ICMP = 1;
const unsigned char ICMP_ECHO = 8;
const unsigned char ICMP_ECHOREPLY = 0;
const unsigned char ICMP_DEST_UNREACH = 3;
const unsigned char ICMP_TIME_EXCEEDED = 11;

struct iphdr
{
    unsigned char  ihl:4;
    unsigned char  version:4;
    unsigned char  tos;
    unsigned short tot_len;
    unsigned short id;
    unsigned short frag_off;
    unsigned char  ttl;
    unsigned char  protocol;
    unsigned short check;
    unsigned int   saddr;
    unsigned int   daddr;
};

struct icmphdr
{
    unsigned char  type;
    unsigned char  code;
    unsigned short checksum;
    union un
    {
        struct echo
        {
            unsigned short id;
            unsigned short sequence;
        };
        unsigned int   gateway;
        struct frag
        {
            unsigned short __unused;
            unsigned short mtu;
        };
    };
};

int main()
{
    // Cria um objeto io_context doAsio
    asio::io_context io_context;

    // Cria um socket ICMP
    asio::ip::icmp::socket socket(io_context);

    // Configura um endpoint ICMP para o host
    asio::ip::icmp::endpoint host_endpoint("www.example.com", 0);

    // Conecta o socket ao host
    socket.connect(host_endpoint);

    // Envia um ping para o host
    std::vector<unsigned char> ping(sizeof(icmphdr) + sizeof(iphdr) + 8);
    iphdr* ip_header = reinterpret_cast<iphdr*>(ping.data());
    ip_header->ihl = 5;
    ip_header->version = 4;
    ip_header->tot_len = htons(ping.size());
    ip_header->protocol = IPPROTO_ICMP;
    ip_header->saddr = inet_addr("127.0.0.1");
    ip_header->daddr = inet_addr("www.example.com");
    icmphdr* icmp_header = reinterpret_cast<icmphdr*>(ping.data() + sizeof(iphdr));
    icmp_header->type = ICMP_ECHO;
    icmp_header->code = 0;
    icmp_header->un.echo.id = htons(1234);
    icmp_header->un.echo.sequence = htons(1);
    *reinterpret_cast<unsigned long*>(ping.data() + sizeof(icmphdr) + sizeof(iphdr)) = htonl(time(nullptr));

    // Calcula o checksum do ping
    icmp_header->checksum = 0;
    icmp_header->checksum = asio::ip::icmp::checksum(ping.data(), ping.size());

    // Armazena o tempo de envio do ping
    auto send_time = std::chrono::high_resolution_clock::now();

    // Envia o ping para o host
    asio::write(socket,asio::buffer(ping));

    // Aguarda por uma resposta de ping do host
    std::vector<unsigned char> reply(1024);
    size_t bytes_received = asio::read(socket, asio::buffer(reply));

    // Verifica se o tipo de mensagem recebida é um ICMP_ECHOREPLY
    iphdr* reply_ip_header = reinterpret_cast<iphdr*>(reply.data());
    icmphdr* reply_icmp_header = reinterpret_cast<icmphdr*>(reply.data() + (reply_ip_header->ihl * 4));
    if (reply_icmp_header->type == ICMP_ECHOREPLY)
    {
        // Calcula o tempo de viagem do ping
        auto trip_time = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::high_resolution_clock::now() - send_time).count();

        // Imprime o resultado do ping
        std::cout << "Recebida resposta de ping do host em " << trip_time << " ms" << std::endl;
    }
    else
    {
        std::cout << "Tipo de mensagem não reconhecido" << std::endl;
    }

    return 0;
}
```