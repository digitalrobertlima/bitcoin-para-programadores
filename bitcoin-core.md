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
#[... outras tags ...]
v0.11.2
v0.11.2rc1
v0.12.0
v0.12.0rc1
v0.12.0rc2
v0.12.0rc3
v0.12.0rc4
v0.12.0rc5
v0.12.1
#[... outras tags ...]
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
#[...]
$ make
#[...]
$ make install  # opcional caso queira os binários no seu $PATH
#[...]
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

### Preparação

Para começar, chame o bitcoind e dê um CTRL+C logo em seguida apenas para que ele forme a estrutura da pasta .bitcoin automaticamente:

```
$ bitcoind
^C
```

Em seguida entre na pasta .bitcoin que, por padrão, fica no seu diretório $HOME e crie ou edite um arquivo chamado bitcoin.conf com um usuário para o servidor RPC e uma senha **forte** diferente da mostrada no exemplo. O arquivo ~/.bitcoin/bitcoin.conf deve ficar como este:

```
rpcuser=bitcoinrpc
rpcpassword=cF58sc+MuY5TM4Dhjs46U2MylXS/krSxB+YW1Ghidzg
```

\*Caso não esteja usando Linux, o seu diretório padrão será diferente:

Para Windows - ```%APPDATA%\Bitcoin```

Para OSX - ```~/Library/Applicatio Support/Bitcoin/```

Com isso feito, você já tem tudo pronto para rodar o *software* cliente como um *full-node* na rede escolhida. O bitcoind, por padrão, conecta-se à *mainnet* que é a rede Bitcoin "de produção" com real valor monetário e oferece as flags ```-testnet``` e ```-regtest``` para caso escolha se conectar na testnet3 ou usar regtest. Para a maior parte deste material, recomendo que siga rodando a mainnet ou testnet3; Aqui vão algumas informações sobre cada uma das três opções:

---

**mainnet:** A rede "de produção" do Bitcoin. Aqui é onde as transações movem valor monetário real e qualquer erro cometido, especialmente comum quando desenvolvendo, pode significar uma grande perda monetária para você. O *market cap* atual da rede é de cerca de 7 bilhões de dólares com um *hash rate* aproximado de 1 bilhão de *GigaHashes por segundo* (sim, isso é MUITO poder de processamento!). No momento, é recomendado que você tenha, pelo menos, 80GB de espaço de armazenamento livre para acomodar a blockchain de quase 70GB e deixar algum espaço para acompanhar o crescimento dela. Eu sincronizo com a mainnet para trabalho utilizando um disco externo; Caso você queira fazer algo parecido basta iniciar o bitcoind com a opção ```-datadir=<diretório que deseja popular>```.

**testnet3:** A rede de teste do Bitcoin. Esta rede tem uma blockchain separada da *mainnet* e é utilizada para testar novas implementações do protocolo (como, atualmente, *segwit* que foi ativada em dezembro de 2015 na *testnet3*), desenvolver sem arriscar valores monetários significativos ou a possibilidade de causar problemas não intencionais na *mainnet*. Esta é a terceira geração da *testnet*; A *testnet2* foi criada apenas pra reiniciar a rede com um novo bloco *genesis* já que algumas pessoas começaram a atribuir valor financeiro à *tesnet* e negociar seus *bitcoins* por dinheiro - o que não é a intenção da rede - e a *testnet3* além de introduzir um novo bloco *genesis* também trouxe algumas melhorias e facilidades para o desenvolvimento, como o ajuste diferente de dificuldade da rede. Os *bitcoins* da *testnet* sao comumente chamados de *test/tesnet coins* e os endereços nesta rede começam com "m" para *hashes* de chave pública e "2" para *script hashes* (mais sobre ambos tipos de endereço em [enderecos](enderecos.md)). Algo em torno de 10GB~15GB deve bastar para você fazer download da blockchain e trabalhar sem se preocupar com armazenamento. Para isso basta que inicie o bitcoind com a opção ```-testnet``` e, se quiser, pode utilizar a flag ```-datadir``` normalmente.

**regtest:** A rede de "teste de regressão" do Bitcoin. A dificuldade de mineração da *regtest* é praticamente 0 e, normalmente, você a inicia do 0 com nenhum bloco minerado e nenhum *peer* conectado. É como uma realidade paralela do Bitcoin em que você pode utilizar para fazer testes bastante interessantes e analisar o comportamento da rede em situações adversas completamente controladas por você. Você pode iniciar vários nós diferentes conectados à sua *regtest*, minerar, enviar transações entre os nós e testar praticamente qualquer situação necessária. Esta rede é especialmente útil para testar coisas que você não tem controle nas outras redes devido à participação de outros nós ou para fazer testes que não necessite da imprevisibilidade das redes públicas usando pouco espaço de armazenamento. A opção para iniciar o bitcoind na *regtest* é passada pela flag ```-regtest```.

---

### Interagindo com a API JSON-RPC

Tendo compreendido a diferença entre as redes, já podemos inciar o *bitcoind* com as opções escolhidas. Recomendo que dê uma lida nas opções disponíveis com o comando ```bitcoind --help```. Para iniciar o *bitcoind* conectado à *mainnet* em modo *daemon* para não segurar o terminal, basta:

```
$ bitcoind -daemon
Bitcoin server starting
```

E pronto... O seu nó começará a sincronizar com a rede e você já poderá se comunicar pela linha de comando via a API JSON-RPC (lembrando que o comando ```bitcoin-cli``` precisa da flag ```-testnet``` ou ```-regtest``` caso tenha iniciado o ```bitcoind``` com alguma destas opções). É uma ótima ideia ler cada comando disponível para se familiarizar:

```
$ bitcoin-cli help
== Blockchain ==
getbestblockhash
getblock "hash" ( verbose )
getblockchaininfo
getblockcount
getblockhash index
getblockheader "hash" ( verbose )
getchaintips
getdifficulty
getmempoolinfo
getrawmempool ( verbose )
gettxout "txid" n ( includemempool )
gettxoutproof ["txid",...] ( blockhash )
gettxoutsetinfo
verifychain ( checklevel numblocks )
verifytxoutproof "proof"

== Control ==
getinfo
help ( "command" )
stop

== Generating ==
generate numblocks
getgenerate
setgenerate generate ( genproclimit )

== Mining ==
getblocktemplate ( "jsonrequestobject" )
getmininginfo
getnetworkhashps ( blocks height )
prioritisetransaction <txid> <priority delta> <fee delta>
submitblock "hexdata" ( "jsonparametersobject" )

== Network ==
addnode "node" "add|remove|onetry"
clearbanned
disconnectnode "node" 
getaddednodeinfo dns ( "node" )
getconnectioncount
getnettotals
getnetworkinfo
getpeerinfo
listbanned
ping
setban "ip(/netmask)" "add|remove" (bantime) (absolute)

== Rawtransactions ==
createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,"data":"hex",...} ( locktime )
decoderawtransaction "hexstring"
decodescript "hex"
fundrawtransaction "hexstring" includeWatching
getrawtransaction "txid" ( verbose )
sendrawtransaction "hexstring" ( allowhighfees )
signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

== Util ==
createmultisig nrequired ["key",...]
estimatefee nblocks
estimatepriority nblocks
estimatesmartfee nblocks
estimatesmartpriority nblocks
validateaddress "bitcoinaddress"
verifymessage "bitcoinaddress" "signature" "message"

== Wallet ==
abandontransaction "txid"
addmultisigaddress nrequired ["key",...] ( "account" )
backupwallet "destination"
dumpprivkey "bitcoinaddress"
dumpwallet "filename"
encryptwallet "passphrase"
getaccount "bitcoinaddress"
getaccountaddress "account"
getaddressesbyaccount "account"
getbalance ( "account" minconf includeWatchonly )
getnewaddress ( "account" )
getrawchangeaddress
getreceivedbyaccount "account" ( minconf )
getreceivedbyaddress "bitcoinaddress" ( minconf )
gettransaction "txid" ( includeWatchonly )
getunconfirmedbalance
getwalletinfo
importaddress "address" ( "label" rescan p2sh )
importprivkey "bitcoinprivkey" ( "label" rescan )
importpubkey "pubkey" ( "label" rescan )
importwallet "filename"
keypoolrefill ( newsize )
listaccounts ( minconf includeWatchonly)
listaddressgroupings
listlockunspent
listreceivedbyaccount ( minconf includeempty includeWatchonly)
listreceivedbyaddress ( minconf includeempty includeWatchonly)
listsinceblock ( "blockhash" target-confirmations includeWatchonly)
listtransactions ( "account" count from includeWatchonly)
listunspent ( minconf maxconf  ["address",...] )
lockunspent unlock [{"txid":"txid","vout":n},...]
move "fromaccount" "toaccount" amount ( minconf "comment" )
sendfrom "fromaccount" "tobitcoinaddress" amount ( minconf "comment" "comment-to" )
sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendtoaddress "bitcoinaddress" amount ( "comment" "comment-to" subtractfeefromamount )
setaccount "bitcoinaddress" "account"
settxfee amount
signmessage "bitcoinaddress" "message"
```
#### Informações de Status

O comando RPC ```getinfo``` retorna informações básicas sobre o seu nó na rede, sua carteira e a blockchain:

```
$ bitcoin-cli getinfo
{
  "version": 120100,
  "protocolversion": 70012,
  "walletversion": 60000,
  "balance": 0.00000000,
  "blocks": 259896,
  "timeoffset": 1,
  "connections": 8,
  "proxy": "",
  "difficulty": 112628548.6663471,
  "testnet": false,
  "keypoololdest": 1461593328,
  "keypoolsize": 101,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}
```

E o ```getpeerinfo``` retorna informações sobre os nós conectados a você na rede:

```
$ bitcoin-cli getpeerinfo
[
  {
    "id": 13,
    "addr": "91.159.81.78:8333",
    "addrlocal": "209.222.18.59:44718",
    "services": "0000000000000005",
    "relaytxes": true,
    "lastsend": 1461603299,
    "lastrecv": 1461603300,
    "bytessent": 1688470,
    "bytesrecv": 1317410461,
    "conntime": 1461596517,
    "timeoffset": -4,
    "pingtime": 5.683412,
    "minping": 0.256795,
    "version": 70012,
    "subver": "/Satoshi:0.12.0/",
    "inbound": false,
    "startingheight": 408868,
    "banscore": 0,
    "synced_headers": 408876,
    "synced_blocks": 260036,
    "inflight": [
      260929, 
      260937, 
      260940, 
      260943, 
      260944, 
      260946, 
      260951, 
      260961, 
      260972, 
      260974, 
      260978, 
      260992, 
      261010, 
      261017, 
      261022, 
      261033
    ],
    "whitelisted": false
  },
#[... outros peers ...]
]
```

#### Configurando a Carteira

A primeira coisa que você deve fazer com a sua carteira é encriptar ela com uma senha forte. Para encriptar com a senha "naouseestasenhaidiota", entre:

```
$ bitcoin-cli encryptwallet naouseestasenhaidiota
wallet encrypted; Bitcoin server stopping, restart to run with encrypted wallet. The keypool has been flushed, you need to make a new backup.
```

Inicie o ```bitcoind``` novamente e verifique que sua carteira está encriptada com ```getinfo``` ao observar o novo atributo "unlocked_until" que deverá ter o valor 0:

```
$ bitcoin-cli getinfo
{
  "version": 120100,
  "protocolversion": 70012,
  "walletversion": 60000,
  "balance": 0.00000000,
  "blocks": 266096,
  "timeoffset": 1,
  "connections": 8,
  "proxy": "",
  "difficulty": 267731249.4824211,
  "testnet": false,
  "keypoololdest": 1461603754,
  "keypoolsize": 100,
  "unlocked_until": 0,  # <--- a carteira está encriptada
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}
```

Para "destrancar" a carteira, use o comando ```walletpassphrase``` com a senha e o tempo que deseja que a carteira permaneça "aberta" antes de se trancar novamente:

```
$ bitcoin-cli walletpassphrase naouseestasenhaidiota 120
```

E quando chamar ```getinfo``` novamente, poderá verificar que a carteira está "destrancada":

```
$ bitcoin-cli getinfo
{
  "version": 120100,
  "protocolversion": 70012,
  "walletversion": 60000,
  "balance": 0.00000000,
  "blocks": 269955,
  "timeoffset": 1,
  "connections": 8,
  "proxy": "",
  "difficulty": 510929738.0161518,
  "testnet": false,
  "keypoololdest": 1461603754,
  "keypoolsize": 101,
  "unlocked_until": 1461604440, #  <--- tempo em formato UNIX até quando a carteira permanecerá encriptada
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}

```

#### Criando e Restaurando Backups da Carteira

Agora, vamos criar ver como criar um arquivo de *backup* e restaurar a sua carteira a partir deste arquivo. Para criar o backup, entre o comando ```backupwallet``` com o destino do arquivo de *backup*:

```
$ bitcoin-cli backupwallet /home/user/carteira-2016-04-25.backup
```

E para restaurar a partir deste backup, simplemente use ```importwallet``` com a localização do arquivo como parâmetro - a carteira deve estar aberta para isso, caso já esteja com outra:

```
$ bitcoin-cli importwallet carteira-2016-04-25.backup
```

Caso precise da carteira em texto num formato legível, use ```dumpwallet``` com o arquivo destino do texto como parâmetro:

```
$ bitcoin-cli dumpwallet wallet.txt
```

E você terá um arquivo .txt com o *dump* da carteira inteira.


