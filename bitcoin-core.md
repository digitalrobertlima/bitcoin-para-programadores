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

**mainnet:** A rede "de produção" do Bitcoin. Aqui é onde as transações movem valor monetário real e qualquer erro cometido, especialmente comum quando desenvolvendo, pode significar uma grande perda monetária para você. O *market cap* atual da rede é de cerca de 7 bilhões de dólares com um *hash rate* aproximado de 1 bilhão de *GigaHashes por segundo* (sim, isso é MUITO poder de processamento!). No momento, é recomendado que você tenha, pelo menos, 80GB de espaço de armazenamento livre para acomodar a blockchain de quase 70GB e deixar algum espaço para acompanhar o crescimento dela caso não use a opção ```prune``` (mais sobre esta opção logo abaixo). Eu sincronizo com a mainnet para trabalho utilizando um *drive* USB externo com a opção ```prune``` em 10GB economizando muito espaço; Caso você queira fazer algo parecido basta iniciar o bitcoind com a opção ```-datadir=<diretório que deseja popular> -prune=10000```.

**testnet3:** A rede de teste do Bitcoin. Esta rede tem uma blockchain separada da *mainnet* e é utilizada para testar novas implementações do protocolo (como, atualmente, *segwit* que foi ativada em dezembro de 2015 na *testnet3*), desenvolver sem arriscar valores monetários significativos ou a possibilidade de causar problemas não intencionais na *mainnet*. Esta é a terceira geração da *testnet*; A *testnet2* foi criada apenas pra reiniciar a rede com um novo bloco *genesis* já que algumas pessoas começaram a atribuir valor financeiro à *tesnet* e negociar seus *bitcoins* por dinheiro - o que não é a intenção da rede - e a *testnet3* além de introduzir um novo bloco *genesis* também trouxe algumas melhorias e facilidades para o desenvolvimento, como o ajuste diferente de dificuldade da rede. Os *bitcoins* da *testnet* sao comumente chamados de *test/tesnet coins* e os endereços nesta rede começam com "m" para *hashes* de chave pública e "2" para *script hashes* (mais detalhes sobre ambos tipos de endereço em [enderecos](enderecos-e-carteiras.md)). Algo em torno de 10GB~15GB deve bastar para você fazer download da blockchain e trabalhar sem se preocupar com armazenamento, além de poder usar a opção ```prune``` com um valor menor para armazenamento. Para isso basta que inicie o bitcoind com a opção ```-testnet``` e, se quiser, pode utilizar a flag ```-datadir``` normalmente.

**regtest:** A rede de "teste de regressão" do Bitcoin. A dificuldade de mineração da *regtest* é praticamente 0 e, normalmente, você a inicia do 0 com nenhum bloco minerado e nenhum *peer* conectado. É como uma realidade paralela do Bitcoin em que você pode utilizar para fazer testes bastante interessantes e analisar o comportamento da rede em situações adversas completamente controladas por você. Você pode iniciar vários nós diferentes conectados à sua *regtest*, minerar, enviar transações entre os nós e testar praticamente qualquer situação necessária. Esta rede é especialmente útil para testar coisas que você não tem controle nas outras redes devido à participação de outros nós ou para fazer testes que não necessite da imprevisibilidade das redes públicas usando pouco espaço de armazenamento. A opção para iniciar o bitcoind na *regtest* é passada pela flag ```-regtest```.

\*Nota sobre a opção ```prune```: A partir do Bitcoin Core v0.11.0, você pode utilizar a opção ```prune``` - "poda"/"podar" em português - sem possibilidade de utilizar a funcionalidade de carteira do Core, e a partir do Bitcoin Core v0.12.0 já é possível utilizar a funcionalidade de carteira do *software* junto com a opção ```prune```. Esta opção permite que você trabalhe guardando uma porção menor da blockchain, ficando apenas com os últimos N blocos com N sendo o número de MB que você decidir para esta opção. É importante notar que esta opção lida apenas com o tamanho da blockchain (na pasta blocks) e não conta o espaço necessário para o armazenamento dos outros arquivos - o que não é muito. Para usar esta opção para limitar os blocos para algo em torno de 10GB, você pode iniciar o ```bitcoind``` com a flag ```-prune=10000` ou colocar uma linha em seu ```bitcoin.conf``` assim:

```
rpcuser=bitcoinrpc
rpcpassword=cF58sc+MuY5TM4Dhjs46U2MylXS/krSxB+YW1Ghidzg
prune=10000  # <--- limita o armazenamento de blocos
```

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

#### Transações

O seu *software* cliente gera endereços automaticamente e os mantém em uma *pool* de endereços. O atributo *keypoolsize* que indica quantos endereços públicos já gerados a sua carteira tem pode ser obtido com *getinfo*. Para pegar um desses endereços, você simplemente usa o comando ```getnewaddress```:

```
$ bitcoin-cli getnewaddress
1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC
```

Envie alguns *millibits* para o endereço que você acabou de pegar na linha de comando para você poder ver como checar a quantia recebida por endereço. No meu caso, vou enviar 10 *millibits* (0.010 bitcoins) para o endereço.

Após enviar, você pode checar o valor recebido com o comando ```getreceivedbyaddress```, porém, por padrão, ele precisa de alguns números de confirmação antes de mostrar a quantia - você pode configurar isso no arquivo bitcoin.conf; para que mostre a quantia recebida mesmo sem confirmação, chamaremos este comando com o parâmetro de confirmações mínimas como 0;

```
$ bitcoin-cli getreceivedbyaddress 1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC 0
0.01000000
```

Se você tentar este mesmo comando sem o parâmetro 0, ele usará o número configurado como ```minconf``` no seu bitcoin.conf.

Você também pode listar todas as transações recebidas pela carteira em todos os endereços com o comando ```listtransactions``` :

```
$ bitcoin-cli listtransactions
[
  {
    "account": "",
    "address": "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC",
    "category": "receive",
    "amount": 0.01000000,
    "label": "",
    "vout": 0,
    "confirmations": 0,
    "trusted": false,
    "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
    "walletconflicts": [
    ],
    "time": 1462453065,
    "timereceived": 1462453065,
    "bip125-replaceable": "no"
  }
]

```

Também podemos listar todos os endereços de uma conta com o comando ```getaddressesbyaccount``` com o identificador da conta como parâmetro - a padrão inicial é uma *string* vazia.:

```
$ bitcoin-cli getaddressesbyaccount ""
[
  "115PHNcNbBtaWeb1iYfX96hYiehgnxamL2", 
  "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC", 
  "1HFJpe3knukVwfqj22m9mVvLz9R4Ac76f9", 
  "1JEqZijxDfTyXHYjiaJTy6SgSCJNdmTayr", 
  "1KCoEFDXmtde2LEshZfbDZFwRwrX8pg8H9", 
  "1PU7nWdqJsev4sWmgcDJwZG9EZRuS7nLGU"
]
```

E para ver a quantia total de todos endereços da carteira com o número de confirmações mínimos da configuração ```minconf```, basta usar o ```getbalance```:

```
$ bitcoin-cli getbalance
0.01000000
```

Vamos, agora, ver como a transação é vista pelo *client* usando o comando ```gettransaction``` com o txid (id da transação; o *hash* da transação) que pegamos com o comando ```listtransactions``` como parâmetro:

```
$ bitcoin-cli gettransaction 01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2
{
  "amount": 0.01000000,
  "confirmations": 1,
  "blockhash": "0000000000000000042e065b770fb46aba51bd389d086a1a2a47c5d9e5ffe575",
  "blockindex": 2408,
  "blocktime": 1462453539,
  "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
  "walletconflicts": [
  ],
  "time": 1462453065,
  "timereceived": 1462453065,
  "bip125-replaceable": "no",
  "details": [
    {
      "account": "",
      "address": "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC",
      "category": "receive",
      "amount": 0.01000000,
      "label": "",
      "vout": 0
    }
  ],
  "hex": "0100000001e6980adab5435ca0214cb5e047c334a4d8f01d5c7b6f55f022ac903db7e75cb7010000006b483045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f0121037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388ffffffff0240420f00000000001976a914ae2c30a645a14fa36425119e1d174b4d201b8f5988acc8684200000000001976a91474e79a48b73182083df37632bba597961ef51da788ac00000000"
}
```

Esta é a transação em sua forma simplificada, mostrando a quantia transacionada (```amount```), número de confirmações da transação (```confirmations```), o ID da transação (```txid```), a hora da transação (```time```), a hora que seu *client* ficou sabendo da transação (```timereceived```) e os detalhes da transação (```details```).

Para ver a transação no formato completo, você precisa usar o comando ```getrawtransaction``` com o ```txid``` como parâmetro que retornará a transação no formato de uma *string* hexadecimal para que você decodifique com o ```decoderawtransaction```:

```
$ bitcoin-cli getrawtransaction 01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2
0100000001e6980adab5435ca0214cb5e047c334a4d8f01d5c7b6f55f022ac903db7e75cb7010000006b483045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f0121037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388ffffffff0240420f00000000001976a914ae2c30a645a14fa36425119e1d174b4d201b8f5988acc8684200000000001976a91474e79a48b73182083df37632bba597961ef51da788ac00000000
$ bitcoin-cli decoderawtransaction 0100000001e6980adab5435ca0214cb5e047c334a4d8f01d5c7b6f55f022ac903db7e75cb7010000006b483045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f0121037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388ffffffff0240420f00000000001976a914ae2c30a645a14fa36425119e1d174b4d201b8f5988acc8684200000000001976a91474e79a48b73182083df37632bba597961ef51da788ac00000000
{
  "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
  "size": 226,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "b75ce7b73d90ac22f0556f7b5c1df0d8a434c347e0b54c21a05c43b5da0a98e6",
      "vout": 1,
      "scriptSig": {
        "asm": "3045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f[ALL] 037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388",
        "hex": "483045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f0121037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.01000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 ae2c30a645a14fa36425119e1d174b4d201b8f59 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914ae2c30a645a14fa36425119e1d174b4d201b8f5988ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC"
        ]
      }
    }, 
    {
      "value": 0.04352200,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 74e79a48b73182083df37632bba597961ef51da7 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91474e79a48b73182083df37632bba597961ef51da788ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1Bf8qHi9q3ASGMHwf123E5RbTfMiky7TSy"
        ]
      }
    }
  ]
}
```

Ou juntando tudo em uma linha para melhorar a visualização...

```
$ bitcoin-cli decoderawtransaction $(bitcoin-cli getrawtransaction [TXID PLACEHOLDER])
{
  "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
  "size": 226,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "b75ce7b73d90ac22f0556f7b5c1df0d8a434c347e0b54c21a05c43b5da0a98e6",
      "vout": 1,
      "scriptSig": {
        "asm": "3045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f[ALL] 037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388",
        "hex": "483045022100ddb1a8477d5ac64c688122f9ecb6306991ed3148d93639601f0ac145ca7bb49602203b306c2d60555dc2ccf89636b7994b04a03960a6fc541c673a04036da641940f0121037d1588b4d483db39a0d076f78d03181cf10dcf362cc06f2a01ff20d447c94388"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.01000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 ae2c30a645a14fa36425119e1d174b4d201b8f59 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914ae2c30a645a14fa36425119e1d174b4d201b8f5988ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC"
        ]
      }
    }, 
    {
      "value": 0.04352200,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 74e79a48b73182083df37632bba597961ef51da7 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91474e79a48b73182083df37632bba597961ef51da788ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1Bf8qHi9q3ASGMHwf123E5RbTfMiky7TSy"
        ]
      }
    }
  ]
}
```

Como você pode ver, desta forma nós conseguimos ver todos os elementos de uma transação com tudo que a rede precisa para verificar e validar ela. Nesta transação, eu usei N inputs e gerei N outputs sendo 10 *millibits* direcionado ao endereço da carteira no Bitcoin Core. Note que não existe a *fee* (ou taxa) de transação explícita; Ela nada mais é do que a diferença entre o todos os *inputs* e o total de todos *outputs*. Isso significa que qualquer fundo não gasto é considerado uma taxa de transação para recompensar o minerador responsável por incluir ela na blockchain.

Feito isso, vamos usar o Bitcoin Core para enviar estes bitcoins para outro endereço pela linha de comando. O primeiro comando que precisamos é o ```listunspent``` para listar todos os *outputs* de nossa carteira que já foram confirmados:

```
$ bitcoin-cli listunspent
  {
    "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
    "vout": 0,
    "address": "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC",
    "account": "",
    "scriptPubKey": "76a914ae2c30a645a14fa36425119e1d174b4d201b8f5988ac",
    "amount": 0.01000000,
    "confirmations": 1,
    "spendable": true
  }
]

```

Aqui, podemos ver que em ```txid``` está o *hash* da transação responsável por criar o *output* ```vout``` 0 contendo 0.01000000 designados ao endereço ```1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC```, assim como o número de confirmações e o ```scriptPubKey``` (mais detalhes em [Transações](transacoes.md))

Para enviarmos estes bitcoins, teremos que criar uma transação que usa o *output* ```vout``` 0 da transação [TXID PLACEHOLDER] e o referenciar no *input* da próxima transação que enviará para o próximo endereço.

Nós podemos ter uma visão mais detalhada do *output* que queremos usar com o comando ```gettxout``` dando o ```txid``` e o número do ```vout``` como parâmetros:

```
$ bitcoin-cli gettxout 01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2 0
{
  "bestblock": "0000000000000000042e065b770fb46aba51bd389d086a1a2a47c5d9e5ffe575",
  "confirmations": 1,
  "value": 0.01000000,
  "scriptPubKey": {
    "asm": "OP_DUP OP_HASH160 ae2c30a645a14fa36425119e1d174b4d201b8f59 OP_EQUALVERIFY OP_CHECKSIG",
    "hex": "76a914ae2c30a645a14fa36425119e1d174b4d201b8f5988ac",
    "reqSigs": 1,
    "type": "pubkeyhash",
    "addresses": [
      "1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC"
    ]
  },
  "version": 1,
  "coinbase": false
}

```

Então, como você pode ver, este *output* designou 10 *millibits* para o endereço ```1GswWkrWvxGhCARW4w2ibnwk3pDURM6GpC``` ter o direito de gastar como pode ser verificado junto com o *script* de transação mais comum da rede (mais detalhes sobre isso em [Transações](transacoes.md)). Para continuar a operação de envio dos bitcoins, vamos criar dois novos endereços para enviar os bitcoins:

```
$ bitcoin-cli getnewaddress
1N8A7WLw4TA8Vbu6GQ9JSotbvtTPY5uKhf  # <--- endereço recipiente principal
$ bitcoin-cli getnewaddress
1EoPUX89wRFGHRNGJFTzqfNcRgdzgs5JhK  # <--- geramos um outro para o "troco" da transação
```

O próximo comando que usaremos para criar a transação é o ```createrawtransaction```, porém, antes devemos entender uma coisa importante sobre transações: **todos** os *inputs* devem ser gastos **completamente**. Isso é importante saber desde já, pois qualquer erro de programação na hora de montar uma transação em que não se gaste todos os *inputs* significará perda financeira. O que acontece é que qualquer valor que não seja explicitamente gasto numa transação será considerado como *fee* de mineração para o minerador que incluir a sua transação na blockchain. Logo, a diferença da soma de todos os *inputs* menos a soma de todos os *outputs* é igual à taxa que será paga ao minerador (soma dos *inputs* - soma dos *outputs* = taxa de mineração). Entendendo isso, podemos criar a transação com o ```createrawtransaction```:

```
$ bitcoin-cli createrawtransaction '[{"txid" : "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2", "vout" : 0}]' '{"1N8A7WLw4TA8Vbu6GQ9JSotbvtTPY5uKhf": 0.006, "1EoPUX89wRFGHRNGJFTzqfNcRgdzgs5JhK": 0.0039}'
0100000001e2b5d0fc7d087c784a1477ed7049c1dc70fcfff8f30ee19b5848b196ccaeec010000000000ffffffff02c0270900000000001976a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac70f30500000000001976a914975f87b998f171ee6c6426e2b018414d2abec50f88ac00000000
```

O que criamos acima é uma transação enviando ```0.006``` bitcoins para ```1N8A7WLw4TA8Vbu6GQ9JSotbvtTPY5uKhf``` como simulação de um endereço para um pagamento ou qualquer outra coisa, usamos o endereço ```1EoPUX89wRFGHRNGJFTzqfNcRgdzgs5JhK``` como endereço de troco atribuindo ```0.0039``` bitcoins a ele; o que deixa ```0.0001``` (```0.006 - 0.0039 = 0.0001```) como taxa de transação.

Para verificar que a transação foi formada corretamente como o esperado, podemos usar o comando ```decoderawtransaction``` com a transação codificada como parâmetro:

```
$ bitcoin-cli decoderawtransaction 0100000001e2b5d0fc7d087c784a1477ed7049c1dc70fcfff8f30ee19b5848b196ccaeec010000000000ffffffff02c0270900000000001976a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac70f30500000000001976a914975f87b998f171ee6c6426e2b018414d2abec50f88ac00000000
{
  "txid": "88f124d1cc2198c8103cfb707849f091589734a42413baf2323936463e905f53",
  "size": 119,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00600000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7b528dec5f14d053d6f32d8508ccb287932c431 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1N8A7WLw4TA8Vbu6GQ9JSotbvtTPY5uKhf"
        ]
      }
    }, 
    {
      "value": 0.00390000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 975f87b998f171ee6c6426e2b018414d2abec50f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914975f87b998f171ee6c6426e2b018414d2abec50f88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1EoPUX89wRFGHRNGJFTzqfNcRgdzgs5JhK"
        ]
      }
    }
  ]
}
```

Parece tudo certo. E, como pode ver, o ```scriptSig``` desta transação está vazio, pois não assinamos ainda. Sem a assinatura digital, esta transação é apenas um *template* que qualquer um poderia ter formado sem valor financeiro algum. Nós precisamos da assinatura para destrancar os *inputs* que estamos usando na transação e antes devemos destrancar a carteira com a senha para podermos expor a chave privada para a assinatura:

```
$ bitcoin-cli walletpassphrase naouseestasenhaidiota 200
```

E assinamos...

```
$ bitcoin-cli signrawtransaction 
{
  "hex": "0100000001e2b5d0fc7d087c784a1477ed7049c1dc70fcfff8f30ee19b5848b196ccaeec01000000006a47304402205984d90f15c019c0804ae140613f6ab5819bd44b6e1faa08cf9aba39655d32f802202b3e5f40dd0f7c8b17381cfff466fae17f7c80e20190ce0caf81962389b80aaf0121039e129c7aec71c34073909fa428982674d305f984fbbc4acda68046a9cd07598bffffffff02c0270900000000001976a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac70f30500000000001976a914975f87b998f171ee6c6426e2b018414d2abec50f88ac00000000",
  "complete": true
}
```

Agora temos a transação assinada, retornada pelo ```signrawtransaction``` como podemos verificar com o ```decoderawtransaction```:

```
$ bitcoin-cli decoderawtransaction 0100000001e2b5d0fc7d087c784a1477ed7049c1dc70fcfff8f30ee19b5848b196ccaeec01000000006a47304402205984d90f15c019c0804ae140613f6ab5819bd44b6e1faa08cf9aba39655d32f802202b3e5f40dd0f7c8b17381cfff466fae17f7c80e20190ce0caf81962389b80aaf0121039e129c7aec71c34073909fa428982674d305f984fbbc4acda68046a9cd07598bffffffff02c0270900000000001976a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac70f30500000000001976a914975f87b998f171ee6c6426e2b018414d2abec50f88ac00000000
{
  "txid": "51cf064af1e25c9a7a299bd3fec3a7a41f318febd17c661656c5134f2ca63044",
  "size": 225,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2",
      "vout": 0,
      "scriptSig": {
        "asm": "304402205984d90f15c019c0804ae140613f6ab5819bd44b6e1faa08cf9aba39655d32f802202b3e5f40dd0f7c8b17381cfff466fae17f7c80e20190ce0caf81962389b80aaf[ALL] 039e129c7aec71c34073909fa428982674d305f984fbbc4acda68046a9cd07598b",
        "hex": "47304402205984d90f15c019c0804ae140613f6ab5819bd44b6e1faa08cf9aba39655d32f802202b3e5f40dd0f7c8b17381cfff466fae17f7c80e20190ce0caf81962389b80aaf0121039e129c7aec71c34073909fa428982674d305f984fbbc4acda68046a9cd07598b"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 0.00600000,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 e7b528dec5f14d053d6f32d8508ccb287932c431 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1N8A7WLw4TA8Vbu6GQ9JSotbvtTPY5uKhf"
        ]
      }
    }, 
    {
      "value": 0.00390000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 975f87b998f171ee6c6426e2b018414d2abec50f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914975f87b998f171ee6c6426e2b018414d2abec50f88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1EoPUX89wRFGHRNGJFTzqfNcRgdzgs5JhK"
        ]
      }
    }
  ]
}
```

Okay, nosso *input* agora tem o ```scriptSig``` preenchido e passa a ter valor real. Só assim ela pode ser verificada e validada pelos outros nós na rede.

Finalmente, vamos enviar a transação assinada para a rede com ```sendrawtransaction``` para que a rede conheça e propague nossa transação:

```
$ bitcoin-cli sendrawtransaction 0100000001e2b5d0fc7d087c784a1477ed7049c1dc70fcfff8f30ee19b5848b196ccaeec01000000006a47304402205984d90f15c019c0804ae140613f6ab5819bd44b6e1faa08cf9aba39655d32f802202b3e5f40dd0f7c8b17381cfff466fae17f7c80e20190ce0caf81962389b80aaf0121039e129c7aec71c34073909fa428982674d305f984fbbc4acda68046a9cd07598bffffffff02c0270900000000001976a914e7b528dec5f14d053d6f32d8508ccb287932c43188ac70f30500000000001976a914975f87b998f171ee6c6426e2b018414d2abec50f88ac00000000

51cf064af1e25c9a7a299bd3fec3a7a41f318febd17c661656c5134f2ca63044
```

Agora temos o *hash* desta transação (*txid*) e podemos usar sobe ele os mesmos comandos que usamos com outras transações para olhar cada detalhe desta transação. Vale lembrar que, atualmente, este *txid* ainda pode mudar até que esta transação seja incluída na blockchain com o *txid* final.

Uma última coisa a se reparar é o fato de o Bitcoin, diferente de cartões de crédito e outros meios do sistema financeiro atual ser muito mais seguro pelo simples fato de que nenhuma informação secreta teve que ser enviada à rede - como nome, documentos, número de cartão de crédito, etc. Tudo que enviamos é uma transação com dados públicos e que não pode ser forjada por ninguém com a informação contida nela sem a chave privada para assinar digitalmente a transação. Num mundo conectado onde o roubo de identidade é um dos crimes mais lucrativos do mundo, esta característica é claramente uma gigantesca vantagem deste sistema.

#### Blocos

Você também pode explorar blocos com a API JSON-RPC do Core. Vejamos o bloco da transação de 0.010 btc recebida usando o comando ```getblock``` com o *hash* do bloco como parâmetro (presente na transação após confirmada):

```
$ bitcoin-cli getblock 0000000000000000042e065b770fb46aba51bd389d086a1a2a47c5d9e5ffe575
{
  "hash": "0000000000000000042e065b770fb46aba51bd389d086a1a2a47c5d9e5ffe575",
  "confirmations": 6,
  "size": 998202,
  "height": 410350,
  "version": 805306368,
  "merkleroot": "df4984d291da6a74e2fad8f88f2df313c153815b3a11420403f72b2707b061ef",
  "tx": [
    "3f3de9824ef4fd1f65f8335e2022db559547dc8a6377124f1ef585bddfeef63e", 
    "9733a90a77833f51886884f76cb50a3ca358615d3b28193ad51e4d3668b6de83", 
    #  [... Muito mais transações ...]
    "76eb383c8a198b3ab56b4d469bb759e8ac75f04dc5e8e3a59b39c55bef93afcb", 
    "01ecaecc96b148589be10ef3f8fffc70dcc14970ed77144a787c087dfcd0b5e2", <--- Nossa transação recebida
    "3c48a57facd63881c4df869a680f73ed751e2a9d00ff16dd5dfcbe54b61bfa23", 
    # [... Muito mais transações ...]
  ],
  "time": 1462453539,
  "mediantime": 1462448476,
  "nonce": 1818589289,
  "bits": "18062776",
  "difficulty": 178659257772.5273,
  "chainwork": "0000000000000000000000000000000000000000001858c55a5df5a426cda2d0",
  "previousblockhash": "0000000000000000021ff8d9b19b4b760670d593e2958a4410880dc6ce14f05a",
  "nextblockhash": "00000000000000000520dd8c8ca2ce480d6bceee1d51916f9befbba0e00d0359"
}
```

Para pegarmos o *hash* de um bloco pela *height* (ou altura) dele na blockchain, usamos o comando ```getblockhash``` e o índice do bloco como parâmetro. A altura ou indíce do primeiro bloco é 0 e, desde então, a cada novo bloco ela é incrementada. Para vermos o famoso bloco *genesis* minerado por Satoshi Nakamoto, basta que usemos o parâmetro 0:

```
$ bitcoin-cli getblockhash 0
000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
```

E agora podemos usar o comando anterior ```getblock``` com o *hash* retornado como parâmetro:

```
$ bitcoin-cli getblock 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
{
    "hash" : "000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f",
    "confirmations" : 286388,
    "size" : 285,
    "height" : 0,
    "version" : 1,
    "merkleroot" : "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b",
    "tx" : [
        "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b"
    ],
    "time" : 1231006505,
    "nonce" : 2083236893,
    "bits" : "1d00ffff",
    "difficulty" : 1.00000000,
    "chainwork" : "0000000000000000000000000000000000000000000000000000000100010001",
    "nextblockhash" : "00000000839a8e6886ab5951d76f411475428afc90947ee320161bbf18eb6048"
}
```

Os comandos ```gettransaction```, ```getblock``` e ```getblockhash``` tem tudo o que você precisa para explorar a blockchain com o Bitcoin Core passando de uma transação para outra e de um bloco para outro como necessário.

#### Outras Implementações e Ferramentas

O cliente bitcoind do Bitcoin Core é a implementação referência atual, porém, pela natureza * open-source* do Bitcoin, há uma variedade enorme de implementações e ferramentas alternativas em diferentes linguagens e com diferentes intuitos. Algumas delas são:

[libbitcoin](https://github.com/libbitcoin/libbitcoin): Ferramenta para desenvolvimento Bitcoin multi-plataforma implementada em C++

[libbitcoin explorer](https://github.com/libbitcoin/libbitcoin-explorer): Ferramenta de Linha de Comando para o Bitcoin

[libbitcoin server](https://github.com/libbitcoin/libbitcoin-server): *Fullnode* e *Query Server* Bitcoin

[btcd](https://github.com/btcsuite/btcd): Implementação do Bitcoin escrita em Go (golang)

[bitcoinj](https://bitcoinj.github.io): *Library* Bitcoin escrita em Java

[python-bitcoinlib](https://github.com/petertodd/python-bitcoinlib): *Library* Bitcoin escrita em Python

[picocoin](https://github.com/jgarzik/picocoin): *Library* Bitcoin escrita em C

... e muitas outras.

*Próximo capítulo:* [Endereços e Carteiras](enderecos-e-carteiras.md)
