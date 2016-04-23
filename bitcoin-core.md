# Bitcoin Core

>"O Bitcoin Core é um projeto *open-source* que mantém e publica o *software* cliente chamado 'Bitcoin Core'.

>É descendente direto do *software* cliente original Bitcoin publicado por Satoshi Nakamoto após ele ter publicado o famoso [*whitepaper* do Bitcoin](https://bitcoincore.org/bitcoin.pdf).

>O Bitcoin Core consiste de um software *'full-node'* para completa validação da blockchain, assim como uma carteira bitcoin. O projeto, atualmente, também mantém *softwares* relacionados como a biblioteca de criptografia [libsecp256k1](https://github.com/bitcoin/secp256k1) e outros que podem ser encontrados no [GitHub](https://github.com/bitcoin).

>Qualquer um pode [contribuir para o Bitcoin Core](https://bitcoincore.org/en/contribute/)."

... e, por razões óbvias, esta implementação em C++ é a escolhida pelo autor neste material.

## Instalação

Você pode instalar o Bitcoin Core utilizando um dos binários disponíveis [aqui](https://bitcoin.org/en/download) ou compilar a partir do [código-fonte](https://github.com/bitcoin/bitcoin). Recomendo que você compile diretamente do código-fonte para que tenha maior autonomia para escolher opções de instalação personalizadas e possa auditar o código que rodará em sua máquina. Porém, você cosneguirá acompanhar o material normalmente se decidir por apenas instalar um binário para o seu sistema se preferir.

Para baixar o código a ser compilado recomendo que utilize [Git](https://git-scm.com/) para lidar com o repositório no GitHub, mas você também pode baixar a última *release* do código-fonte [aqui](https://github.com/bitcoin/bitcoin/releases) e seguir os mesmos passos da compilação.

Eu seguirei com a instalação geral para sistemas Unix e alguns passos podem ser diferentes para diferentes sistemas. Logo, o repositório também disponibiliza instruções específicas para [OpenBSD](https://github.com/bitcoin/bitcoin/blob/master/doc/build-openbsd.md), [OS X](https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md) e [Windows](https://github.com/bitcoin/bitcoin/blob/master/doc/build-windows.md).

### Dependências

Antes de compilar o código do Bitcoin Core em si, você terá que cuidar da instalação de algumas dependências. As 3 primeiras listadas são explicitamente necessárias como listado abaixo e, das opcionais, recomendo que você instale todas as necessárias para utilizar a *interface* gráfica e as funcionalidades de carteira do Bitcoin Core.

Cada uma destas dependências podem ser encontradas no *package manager* (APT, yum, dnf, brew...) que você utiliza e os nomes podem ser um pouco diferentes de entre cada um deles. Já para a instalação que eu recomendo do Berkeley DB, talvez você precise baixar e compilar diretamente [http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz](http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz).

Para resolver o Berkeley DB basta seguir estes comandos:

```
$ wget http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz
$ sha256sum db-4.8.30.NC.tar.gz 
# o último comando deve ter gerado o *hash* 12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef
$ tar -xvf db-4.8.30.NC.tar.gz
$ cd db-4.8.30.NC/build_unix
$ mkdir -p build
$ BDB_PREFIX=$(pwd)/build
$ ../dist/configure —disable-shared —enable-cxx —with-pic —prefix=$BDB_PREFIX
$ make install

```

Com isso, você poderá usar as funcionalidades de *wallet*.

Continue na mesma janela para manter a variável $BDB_PREFIX ou guarde este valor para usar na configuração do Bitcoin Core abaixo.

Aqui estão as dependências **necessárias** para a compilação:

 Biblioteca  | Propósito        | Descrição
 ------------|------------------|----------------------
 libssl      | Criptografia     | Geração Aleatória de Números, Criptografia de Curva Elíptica
 libboost    | Utilidade        | Biblioteca para *threading*, estruturas de dados, etc
 libevent    | Rede             | Operações de rede assíncronas independente de Sistema Operacional

E estas são as dependências **opcionais** são:

 Biblioteca  | Propósito        | Dscrição
 ------------|------------------|----------------------
 miniupnpc   | Suporte UPnP     | Suporte para *jumping* de Firewall
 libdb4.8    | Berkeley DB      | Armazenamento de carteira (apenas necessário com *wallet* habilitada)
 qt          | GUI              | Conjunto de ferramentas para *GUI* (apenas necessário com *GUI* habilitada)
 protobuf    | Pagamento na GUI | Intercâmbio de dados usado para o protocolo de pagamento (apenas necessário com *GUI* habilitada)
 libqrencode | Códigos QR na GUI| Opcional para geração de códigos QR (apenas necessário com *GUI* habilitada)
 univalue    | Utilidade        | *Parsing* e codificação de JSON (versão empacotada será usada a não ser que --with-system-univalue seja passada na *configure*)
 libzmq3     | Notificação ZMQ  | Opcional, permite geração de notificações ZMQ (requer servidor ZMQ versão >= 4.x)

### Compilando

Agora, com as dependências instaladas, você pode baixar o código-fonte utilizando Git para clonar o repositório ou direto da [página de *releases*](https://github.com/bitcoin/bitcoin/releases). Como já dito, recomendo a primeira opção.

Vale lembrar que você, provavelmente, terá um ou outro problema específico ao seu sistema com a instalação das dependências e que você irá descobrir neste próximo passo de preparação e compilação do código. No entanto, os *outputs* com os erros causados por falta de dependências corretas são bastante prestativos e as respostas para estes problemas são facilmente encontradas pela internet.

Seguindo com o Git, clone o repositório:

```
$ git clone https://github.com/bitcoin/bitcoin.git
$ cd bitcoin
```

Liste as *tags* de *releases* disponíveis:

```
$ git tag
[... outras tags ...]
v0.11.2
v0.11.2rc1
v0.12.0
v0.12.0rc1
v0.12.0rc2
v0.12.0rc3
v0.12.0rc4
v0.12.0rc5
v0.12.1
[... outras tags ...]
```

E dê um *checkout* para a última versão estável ou, se preferir, a última *release candidate*:

```
$ git checkout v0.12.1
Note: checking out 'v0.12.1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 9779e1e... Merge #7852: [0.12] Add missing reference to release notes
```

Agora basta fazer o *build* (obs.: $BDB_PREFIX definida acima):

```
$ ./autogen.sh
$ ./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/" --with-gui
[...]
$ make
[...]
$ make install  # opcional caso queira os binários no seu $PATH
[...]
```

Se tudo ocorreu bem, você já pode utilizar ```bitcoind``` e ```bitcoin-cli```:

```
$ bitcoind --version
Bitcoin Core Daemon version v0.12.1
Copyright (C) 2009-2016 The Bitcoin Core Developers

This is experimental software.

Distributed under the MIT software license, see the accompanying file COPYING
or <http://www.opensource.org/licenses/mit-license.php>.

This product includes software developed by the OpenSSL Project for use in the
OpenSSL Toolkit <https://www.openssl.org/> and cryptographic software written
by Eric Young and UPnP software written by Thomas Bernard.
```

```
$ bitcoin-cli --version
Bitcoin Core RPC client version v0.12.1
```

Basicamente, o bitcoind é o cliente Bitcoin que expõe uma API JSON-RPC quando em modo *server* e o bitcoin-cli é a ferramenta que utilizamos para nos comunicarmos com esta API pela linha de comando.

## API JSON-RPC 

Para começar, chame o bitcoind e dê um CTRL+C logo em seguida apenas para que ele forme a estrutura da pasta .bitcoin automaticamente:

```
$ bitcoind
^C
```

Em seguida entre na pasta .bitcoin que, por padrão, fica no seu diretório $HOME e crie ou edite um arquivo chamado bitcoin.conf com um usuário para o servidor RPC e uma senha **forte** diferente da mostrada. O arquivo ~/.bitcoin/bitcoin.conf deve ficar como este:

```
rpcuser=bitcoinrpc
rpcpassword=cF58sc+MuY5TM4Dhjs46U2MylXS/krSxB+YW1Ghidzg
```

*Caso não esteja usando Linux, o seu diretório padrão será diferente:

Para Windows - ```%APPDATA%\Bitcoin```

Para OSX - ```~/Library/Applicatio Support/Bitcoin/```
