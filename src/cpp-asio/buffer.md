# Buffers

Fundamentalmente, E/S envolve a transferência de dados para e de regiões contíguas da memória, chamadas buffers. Esses buffers podem ser simplesmente expressos como uma tupla que consiste em um ponteiro e um tamanho em bytes. No entanto, para permitir o desenvolvimento de aplicativos de rede eficientes, o `Boost.Asio` inclui suporte para operações de coleta de dispersão. Essas operações envolvem um ou mais buffers:

* Uma scatter-read recebe dados em vários buffers.
* Uma gather-write transmite vários buffers.

Portanto, exigimos uma abstração para representar uma coleção de buffers. A abordagem usada no `Boost.Asio` é definir um tipo (na verdade, dois tipos) para representar um único buffer. Eles podem ser armazenados em um contêiner, que pode ser passado para as operações de coleta de dispersão.

Além de especificar buffers como ponteiro e medir o tamanho em bytes, o `Boost.Asio` faz uma distinção entre memória modificável (mutável) e memória não modificável (onde a última é criada a partir do armazenamento para uma variável qualificada de const). Esses dois tipos podem, portanto, ser definidos da seguinte maneira:

```cpp
    typedef std::pair<void*, std::size_t> mutable_buffer;
    typedef std::pair<const void*, std::size_t> const_buffer;
```

Um `mutable_buffer` seria conversível em um `const_buffer`, mas a conversão na direção oposta não é válida.

No entanto, o `Boost.Asio` não usa as definições acima como estão, mas define duas classes: `mutable_buffer` e `const_buffer`. O objetivo deles é fornecer uma representação opaca da memória contígua, onde:

* Os tipos se comportam como `std::pair` nas conversões. Ou seja, um `mutable_buffer` é conversível em um `const_buffer`, mas a conversão oposta é desabilitada.

* There is protection against buffer overruns. Given a buffer instance, a user can only create another buffer representing the same range of memory or a sub-range of it. To provide further safety, the library also includes mechanisms for automatically determining the size of a buffer from an array, boost::array or std::vector of POD elements, or from a std::string.

* A memória subjacente é acessada explicitamente usando a função membro `data()`. Em geral, um aplicativo nunca deve precisar fazer isso, mas é necessário que a implementação da biblioteca passe a memória não processada para as funções subjacentes do sistema operacional.

Finalmente, vários buffers podem ser passados para operações (como `read()` ou `write()`) colocando os objetos do buffer em um contêiner. Os conceitos `MutableBufferSequence` e `ConstBufferSequence` foram definidos para que contêineres como `std::vector`, `std::list`, `std::array` ou `boost::array` possam ser usados.

## Streambuf para integração com Iostreams

A classe `boost::asio::basic_streambuf` é derivada de `std::basic_streambuf` para associar a sequência de entrada e a saída a um ou mais objetos de algum tipo de matriz de caracteres, cujos elementos armazenam valores arbitrários. Esses objetos da matriz de caracteres são internos ao objeto streambuf, mas é fornecido acesso direto aos elementos da matriz para permitir que sejam utilizados com operações de E/S, como as operações de envio ou recebimento de um socket:

* A sequência de entrada do streambuf é acessível através da função membro `data()`. O tipo de retorno dessa função atende aos requisitos `ConstBufferSequence`.

* A sequência de saída do streambuf é acessível através da função membro `prepare()`. O tipo de retorno dessa função atende aos requisitos de `MutableBufferSequence`.

* Os dados são transferidos sequência frontal de saída para a parte de trás da sequência de entrada chamando a função membro `commit()`.

* Os dados são removidos da sequência frontal de entrada chamando a função de membro `consume()`.

O construtor `streambuf` aceita um argumento `size_t` especificando a soma máximo dos tamanhos da sequência de entrada e de saída. Qualquer operação que, se for bem-sucedida, aumentará os dados internos além desse limite, lançará uma exceção `std::length_error`.