# Instalando Asio

Inicialmente existe duas versões disponíveis para download:

- [Asio Standalone](https://think-async.com/Asio) (sem boost) - **Recomendação:** C++11[`std::system_error`] ou posterior;

- [Boost Asio](https://www.boost.org) (normalmente mais utilizado).

- [Networking-TS](https://cplusplus.github.io/networking-ts/draft.pdf) - Baseado no asio/boost::asio, proposto pela ISO para standard library `std::net`.

Instalar Asio não é difícil. Pois ele possui apenas arquivos headers.

Os exemplos abaixo citarão a instalação do `boost::asio`.

## Linux

`Ubuntu`:

	$ sudo apt-get install boost

`Arch Linux`:

	$ sudo pacman -S boost

No linux para compilar um programa, o parâmetro `-pthread` se faz necessário:

	$ g++ client.cpp -o client -pthread -lboost_system

## BSD

`OpenBSD`:

	$ pkg_add boost

Quando estiver compilando um programa, favor vincular as bibliotecas `boost`.

Ex.:

	$ c++ -L/usr/local/lib client.cpp -o client -lboost_system

## Windows

Se você utiliza MSVC, então poderá optar por diversas opções:

* Baixar pré-compiladas: [Sourceforge](https://sourceforge.net/projects/boost/files/boost-binaries)

* Usar [`vcpkg`](https://github.com/microsoft/vcpkg) com o seguinte parâmetro, ex.:
	
	x86:

		vcpkg install boost:x86-windows

	ou
	
	x64:
	
		vcpkg install boost:x64-windows

* Usar [`conan`](https://conan.io) (**Requer:** python) com o seguinte parâmetro, ex.:

	Versão [1.72]:

		conan install Boost/1.72.0@bincrafters/stable

	**Nota:** Para que a instalação com conan funcione neste repositório precisará utilizar este comando antes:

		conan remote add bincrafters https://api.bintray.com/conan/bincrafters/public-conan

	Para compilar no Windows usando MSVC no prompt de comando use:

		cl /EHsc /I C:\Program Files\boost\boost_1_72_0 example.cpp /link /LIBPATH:C:\Program Files\boost\boost_1_72_0\lib

Caso queira utilizar MinGW(Minimal GNU for Windows) terá duas opções:

* [MSYS2](http://www.msys2.org):

	x86:

		$ pacman -S mingw-w64-i386-boost

	x86_64:

		$ pacman -S mingw-w64-x86_64-boost

	**Nota:** Por mais que parece ser um ambiente linux (minimalista), não requer uso do `sudo` na instalação

* Compilar manualmente seguindo a documentação boost: [Windows - Getting Start](https://www.boost.org/doc/libs/1_72_0/more/getting_started/windows.html)
