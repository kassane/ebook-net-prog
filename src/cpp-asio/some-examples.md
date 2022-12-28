# Exemplos

## Cliente NTP

**Nota:** Nos formatos da data e o timestamp, a base da era 0, resultando inicialmente no seguinte horário: 0h 1 de janeiro de 1900 UTC, quando todos os bits são zero. [RFC 5902](https://www.rfc-editor.org/rfc/rfc5905#page-12)
Para exibir o horário atual, precisará alterar o timestamp!

```c++
#include <array>
#include <boost/asio.hpp>
#include <iostream>
#include <chrono>

namespace asio = boost::asio;
using asio::ip::udp;

int main(int argc, char *argv[]) {
  try {
    if (argc != 2) {
      std::cerr << "Usage: ntp_client <host>" << std::endl;
      return 1;
    }

    asio::io_context io_context;

    udp::resolver resolver(io_context);
    udp::endpoint receiver_endpoint =
        *resolver.resolve(udp::v4(), argv[1], "123").begin();

    udp::socket socket(io_context);
    socket.open(udp::v4());

    std::array<char, 48> send_buf = {0x1b, 0, 0, 0, 0, 0, 0, 0, 0};
    socket.send_to(asio::buffer(send_buf), receiver_endpoint);

    std::array<char, 48> recv_buf;
    udp::endpoint sender_endpoint;
    size_t len = socket.receive_from(asio::buffer(recv_buf), sender_endpoint);

    std::cout << "received " << len << " bytes from " << sender_endpoint
              << std::endl;

    // Extrair o NTP timestamp da resposta (do servidor)
    unsigned long long int ntp_timestamp =
      (unsigned long long int)(recv_buf[40]) << 24 |
      (unsigned long long int)(recv_buf[41]) << 16 |
      (unsigned long long int)(recv_buf[42]) << 8 |
      (unsigned long long int)(recv_buf[43]);

   // Converter o timestamp para std::chrono::system_clock::time_point
    std::chrono::system_clock::time_point time_point =
        std::chrono::system_clock::time_point(
            std::chrono::seconds(ntp_timestamp - 2208988800ull));
                              // ntp_timestamp - unix_timestamp

    // Converter o time_point para std::time_t e exibir na tela
    std::time_t time = std::chrono::system_clock::to_time_t(time_point);
    std::cout << std::ctime(&time) << std::endl;

  } catch (std::exception &e) {
    std::cerr << e.what() << std::endl;
  }

  return 0;
}
```

## DNS resolver (ShowMeIP)

```c++
#include <iostream>
#include <string>
#include <asio.hpp>

int main(int argc, char* argv[])
{
    // Verificar o número de argumentos
    if (argc != 2) {
        std::cerr << "Usage: showip hostname" << std::endl;
        return 1;
    }

    // Descobrir o endereço IP por baixo do link mencionado no argv[1]
    asio::io_context io_context;
    asio::ip::tcp::resolver resolver(io_context);
    asio::ip::tcp::resolver::query query(argv[1], "");
    auto results = resolver.resolve(query);

    // Iterar todos os IPs detectados e exibi-los na tela.
    std::cout << "IP addresses for " << argv[1] << ":" << std::endl << std::endl;
    for (auto result : results) {
        std::cout << "  " << result.endpoint().address().to_string() << std::endl;
    }

    return 0;
}
```

## QuickSort com Corrotinas

Referência: [Zap/Cpp benchmark](https://github.com/kprotty/zap/pull/8) - An asynchronous runtime with a focus on performance and resource efficiency. 

```c++
#include <algorithm>
#include <asio.hpp>
#include <chrono>
#include <iostream>
#include <random>
#include <vector>

using namespace std::chrono;
using asio::awaitable;
using asio::co_spawn;
using asio::detached;

awaitable<void> quickSort(asio::io_context &ctx,
                          std::vector<int>::iterator begin,
                          std::vector<int>::iterator end) {
  if (std::distance(begin, end) <= 32) {
    // Use std::sort for small inputs
    std::sort(begin, end);
  } else {
    auto pivot = begin + std::distance(begin, end) - 1;
    auto i = std::partition(begin, pivot, [=](int x) { return x <= *pivot; });
    std::swap(*i, *pivot);

    co_await quickSort(ctx, begin, i);
    co_await quickSort(ctx, i + 1, end);
  }
  co_return;
}

void shuffle(std::vector<int> &arr) {
  std::mt19937 rng(std::random_device{}());
  std::shuffle(std::begin(arr), std::end(arr), rng);
}

int main() {
  std::vector<int> arr(10'000'000);

  std::cout << "filling" << std::endl;
  std::iota(std::begin(arr), std::end(arr), 0);

  std::cout << "shuffling" << std::endl;
  shuffle(arr);

  std::cout << "running" << std::endl;

  const int num_threads = std::thread::hardware_concurrency();
  asio::io_context ctx{num_threads};
  const auto start = high_resolution_clock::now();

  co_spawn(
      ctx,
      [&]() -> awaitable<void> {
        co_await quickSort(ctx, std::begin(arr), std::end(arr));
      },
      detached);

  // Run the io_context to process the posted tasks
  ctx.run();

  const auto elapsed =
      duration_cast<milliseconds>(high_resolution_clock::now() - start);
  std::cout << "took " << elapsed.count() << "ms" << std::endl;

  if (!is_sorted(std::begin(arr), std::end(arr))) {
    throw std::runtime_error("array not sorted");
  }
}
```