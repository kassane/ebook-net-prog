# Instalando Asio

Existem duas versões principais disponíveis:

- **[Asio Standalone](https://think-async.com/Asio)** (sem Boost) — Versão mais recente: **1.38.0** (dez/2025). Requer C++11 ou posterior. **Recomendado** para projetos novos que não dependem do Boost.

- **[Boost.Asio](https://www.boost.org)** — Faz parte do ecossistema Boost. Versão atual do Boost: **1.87.0** (2025). Mais comum em projetos que já usam Boost.

- **[Networking-TS](https://timsong-cpp.github.io/cppwp/networking-ts/)** — Baseado no Asio/Boost.Asio, proposto para a biblioteca padrão como `std::net` (ainda em discussão no ISO).

O Asio é uma biblioteca **header-only** — não requer compilação prévia. Basta adicionar o diretório de headers ao seu projeto.

---

## Asio Standalone

### Linux

**Ubuntu/Debian:**

```sh
sudo apt install libasio-dev
```

**Arch Linux:**

```sh
sudo pacman -S asio
```

Para compilar sem Boost:

```sh
g++ -std=c++20 client.cpp -o client -pthread -DASIO_STANDALONE
```

### macOS

```sh
brew install asio
```

### Windows (vcpkg)

```sh
vcpkg install asio
```

### CMake com FetchContent (sem Boost, qualquer plataforma)

Esta é a forma recomendada para projetos CMake — garante sempre a versão correta:

```cmake
cmake_minimum_required(VERSION 3.14)
project(meu_projeto)

include(FetchContent)

FetchContent_Declare(
    asio
    GIT_REPOSITORY https://github.com/chriskohlhoff/asio.git
    GIT_TAG        asio-1-38-0
    SOURCE_SUBDIR  asio
)
FetchContent_MakeAvailable(asio)

add_executable(meu_programa main.cpp)
target_include_directories(meu_programa PRIVATE ${asio_SOURCE_DIR}/asio/include)
target_compile_definitions(meu_programa PRIVATE ASIO_STANDALONE)
target_link_libraries(meu_programa PRIVATE pthread)
```

No código, use:

```cpp
#include <asio.hpp>          // Standalone Asio
// em vez de:
// #include <boost/asio.hpp> // Boost.Asio
```

---

## Boost.Asio

### Linux

**Ubuntu/Debian:**

```sh
sudo apt install libboost-dev
```

**Arch Linux:**

```sh
sudo pacman -S boost
```

Para compilar:

```sh
g++ -std=c++20 client.cpp -o client -pthread -lboost_system
```

### BSD

**OpenBSD:**

```sh
pkg_add boost
```

Para compilar:

```sh
c++ -std=c++20 -L/usr/local/lib client.cpp -o client -lboost_system
```

### Windows

**vcpkg** (recomendado):

```sh
# x64
vcpkg install boost-asio:x64-windows

# x86
vcpkg install boost-asio:x86-windows
```

**MinGW (MSYS2):**

```sh
# x86_64
pacman -S mingw-w64-x86_64-boost

# i686
pacman -S mingw-w64-i686-boost
```

> **Nota:** O ambiente MSYS2 não requer `sudo` para instalação de pacotes.

**Conan 2.x:**

```sh
conan install --requires="boost/1.87.0" --build=missing
```

> **Atenção:** A sintaxe antiga `Boost/1.72.0@bincrafters/stable` pertence ao Conan 1.x e ao Bintray, que **encerrou em maio de 2021**. Use sempre a sintaxe do Conan 2.x mostrada acima.

**MSVC (linha de comando):**

```sh
cl /EHsc /std:c++20 /I "C:\path\to\boost" example.cpp
```

---

## Diferença entre Standalone e Boost.Asio

| Aspecto | Asio Standalone | Boost.Asio |
|---|---|---|
| Dependência | Nenhuma | Boost |
| Header principal | `<asio.hpp>` | `<boost/asio.hpp>` |
| Namespace | `asio::` | `boost::asio::` |
| Error codes | `asio::error_code` | `boost::system::error_code` |
| Macro necessária | `ASIO_STANDALONE` | — |
| Versão atual | 1.38.0 (dez/2025) | Parte do Boost 1.87.0 |

> **Dica:** As duas versões compartilham praticamente a mesma API. Migrar entre elas é simples: basta trocar os includes, o namespace e a macro de compilação.
