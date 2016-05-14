# Rede P2P

A rede Bitcoin tem uma arquitetura *peer-to-peer* onde todos os participantes da rede são nós (ou *nodes*) que funcionam tanto como servidores quanto como clientes sem hierarquia especial entre um nó ou outro. Todos são considerados hierarquicamente iguais e devem servir a rede com certos recursos. Este modelo aberto e descentralizado faz com que a rede não tenha uma barreira de entrada arbitrariamente escolhida; basta que você esteja rodando um nó e você será parte da rede.

Os nós na rede trocam mensagens contendo transações, blocos e endereços (IP) de outros *peers* entre si. Ao conectar à rede seu *client* realiza o processo chamado *bootstrap* para achar e se conectar a outros *peers* na rede, e começa a baixar blocos para a cópia local da blockchain de *peers* mais adiantados na rede. Ao mesmo tempo o seu *client* já pode começar a prover dados para os *peers* conectados e ajudar a rede enquanto se atualiza com o consenso da rede. Quando você cria uma transação e a envia para a rede - como fizemos em [Bitcoin Core: API JSON-RPC](bitcoin-core.md#api-jsonrpc) - o seu *client* envia esta transação para alguns *peers*, estes *peers* enviam para outros e assim, sucessivamente, até que um minerador valide a sua transação e a inclua num bloco, e, então, transmita este bloco aos *peers* de forma sucessiva até que seu *client* receba o novo bloco e confirme que a transação foi confirmada.

Quando nos referimos à "rede Bitcoin", estamos falando do protocolo P2P Bitcoin. No entanto, também existem outros protocolos como o *Stratum*, por exemplo, que é usado por mineradores e carteiras *lightweight*, e forma parte do que chamamos de rede estendida Bitcoin.

## Tipos de Nós na Rede

Existem 4 características/funções básicas que um nó pode ter na rede: carteira, minerador, blockchain completa e roteamento na rede P2P. Nem todos os nós apresentam todas as quatro características/funções, podendo apresentar desde apenas uma - rede - ou mais delas até todas elas. A falta de hierarquia especial na rede continua a mesma; a única coisa que muda é a funcionalidade que o operador do nó **escolhe** ter. 

Os *softwares* dos nós podem ser modificados nas funcionalidades que apresentam e de que forma as implementam de acordo com a necessidade do operador. Por exemplo, uma *exchange* - e a maioria das pessoas - não precisa ter a funcionalidade de mineração em seu nó. O mínimo que se precisa para ser considerado um nó na rede Bitcoin é a participação com a funcionalidade de rede passando e recebendo mensagens, porém, a rede estendida Bitcoin apresenta outros tipos e nó fora desta rede original com outros protocolos e eles estarão listados abaixo. Aqui está uma lista dos tipos mais comuns:

| Tipo                                  | Funções                                                                                                      |
|---------------------------------------|--------------------------------------------------------------------------------------------------------------|
| **Cliente Referência (Bitcoin Core)** | **carteira**, **minerador**, cópia da **blockchain completa** e roteamento na **rede** P2P.                  |
| **Nó de Blockchain Completa**         | cópia da **blockchain completa**, e roteamento na **rede** P2P.                                              |
| **Minerador Solo**                    | **minerador**, cópia da **blockchain completa** e roteamento na **rede** P2P.                                |
| **Carteira Leve**                     | **carteira** e roteamento na rede.                                                                           |
| **Servidores Stratum**                | roteamento na **rede** P2P e funcionam como *gateways* para nós rodando *Stratum*.                           |
| **Minerador Leve**                    | **minerador** e **rede** conectada a algum nó ou *pool Stratum*.                                             |
| **Carteira Leve Stratum**             | **carteira** e **rede** conectada a algum nó via protocolo *Stratum*.                                        |

> **Info** *Full-node* se refere aos nós na rede P2P Bitcoin que mantém uma cópia completa da *blockchain* e, assim, tem independência completa para verificar blocos e transações.

## Conectando-se à Rede

O Bitcoin Core - e a maioria dos *clients* seguindo a referência - costumam seguir o mesmo processo de conexão à rede P2P Bitcoin. No caso do Core, assim que você começa a rodar com o comando ```bitcoind``` este processo é iniciado e passa por verificação dos últimos blocos - 288 por *default* configurável pela *flag* ```-checkblocks``` -, descoberta de *peers* para que a troca de inventório possa começar a ser feita junto com o roteamento de mensagens (transações, blocos, **peers**...).

Acho uma boa ideia que você veja os *logs* referentes a este processo acontecendo para captar mais informações por você mesmo. Para isso, basta seguir os arquivo ```debug.log``` no diretório que seu *client* estiver utilizando. Você pode simplesmente usar o comando tail:

```
$ tail -f ~/.bitcoin/debug.log
```

Agora vamos aos passos-a-passo do processo de conexão...

**Descoberta de Peers:** O primeiro método utilizado na primeira inicialização de seu Bitcoin Core para achar outros *peers* na rede é puxar DNS's usando as chamadas *DNS seeds* que são servidores DNS que disponibilizam uma rede de IPs de outros nós na rede. Esta lista de *DNS seeds* é escrita diretamente no código e, no momento, pode ser encontrada no [arquivo chainparams.cpp (linha 112)](https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp#L112) do código fonte do Bitcoin Core. Como teste, podemos usar o comando ```nslookup``` no terminal para ver a lista que um destes servidores retornará:

```
$ nslookup seed.bitcoin.sipa.be
Server:		10.137.4.1
Address:	10.137.4.1#53

Non-authoritative answer:
Name:	seed.bitcoin.sipa.be
Address: 88.198.60.110
Name:	seed.bitcoin.sipa.be
Address: 73.71.114.219
Name:	seed.bitcoin.sipa.be
Address: 213.91.211.17
Name:	seed.bitcoin.sipa.be
Address: 85.218.136.63
Name:	seed.bitcoin.sipa.be
Address: 188.226.188.160
Name:	seed.bitcoin.sipa.be
Address: 185.25.49.184
Name:	seed.bitcoin.sipa.be
Address: 204.68.122.11
#[... Mais outros IPs... ]
```

> **Info** O Bitcoin Core, atualmente, vem com 6 *DNS seeds* escritas diretamente no código e você pode escolher se utilizará estes servidores com a opção ```-dnsseed``` que, por padrão, vem configurada com o valor ```1```.

Você também pode escolher passar a informação ```-dnsseed``` definindo o IP do *peer* para que o *nslookup* seja feito ou usar diretamente a opção ```-addnode``` para se conectar diretamente a um outro nó que esteja disponível na rede.

No entanto, caso o seu *client* já tenha se conectado à rede antes, ele usará como primeiro método a tentativa de se conectar à lista de nós que ele estava conectado antes de sair da rede.

**Handshake:** Ao se conectar a um nó na rede, a primeira coisa que ambos devem fazer é um *handshake* para se certificarem de que eles conseguem se comunicar de acordo com as mesmas regras. O primeiro nó transmitirá uma mensagem de versão contendo:

| Campo            | Descrição                                                                                |
|------------------|------------------------------------------------------------------------------------------|
| PROTOCOL_VERSION | constante que define a versão do protocolo P2P que o *client* utiliza para se comunicar. |
| nLocalServices   | lista de serviços suportados pelo nó.                                                    |
| nTime            | hora atual                                                                               |
| addrYou          | endereço de IP do outro nó como visto por ele.                                           |
| addrMe           | endereço IP do nó local.                                                                 |
| subver           | Uma sub-versão que identifica o tipo de *software* que o nó está rodando.                |
| BestHeight       | A altura do bloco mais recente da blockchain deste nó.                                   |

O outro nó, então, continuará com o *handshake* retornando uma mensagem *verack* em resposta ao recebimento de versão junto com a própria mensagem de versão dele em caso de ter aceitado continua a conexão, reconhecendo que ambos estão "falando" de formas compatíveis.

O primeiro nó envia, agora, as mensagens *getaddr* e *addr* para pegar endereços de outros *peers* conectados ao segundo nó e começar o mesmo processo de estabelecimento de conexão com estes outros e ter o seu próprio endereço repassado a outros nós conectados ao seu *peer* atual. E, este mesmo processo de conexão ocorrerá com mais outros nós a fim de se estabelecer na rede já que nenhum nó tem a garantia de que permanecerá conectado e o processo de descoberta de novos nós deve continuar periodicamente para garantir que os nós que parem de responder sejam descartados e novos sejam adicionados como *peers*.

Você pode usar a API JSON-RPC para ver quais nós estão conectados ao seu *client* no momento junto com algumas informações sobre eles:

```
$ bitcoin-cli getpeerinfo
[
{
    "id": 2,
    "addr": "54.152.216.47:8333",
    "addrlocal": "179.43.176.98:47524",
    "services": "0000000000000001",
    "relaytxes": true,
    "lastsend": 1463261684,
    "lastrecv": 1463261685,
    "bytessent": 2153665,
    "bytesrecv": 455684694,
    "conntime": 1463258351,
    "timeoffset": 3,
    "pingtime": 9.469450999999999,
    "minping": 0.419712,
    "version": 70002,
    "subver": "/Satoshi:0.10.0/",
    "inbound": false,
    "startingheight": 411774,
    "banscore": 0,
    "synced_headers": 411777,
    "synced_blocks": 199986,
    "inflight": [
      200190, 
      200191, 
      200208, 
      200214, 
      200219, 
      200243, 
      200253, 
      200259, 
      200266, 
      200271, 
      200276, 
      200297, 
      200298, 
      200299, 
      200303, 
      200305
    
    ],
    "whitelisted": false
  
}, 
{
    "id": 3,
    "addr": "45.32.185.154:8333",
    "addrlocal": "179.43.176.98:50096",
    "services": "0000000000000001",
    "relaytxes": true,
    "lastsend": 1463261684,
    "lastrecv": 1463261685,
    "bytessent": 2485767,
    "bytesrecv": 533974518,
    "conntime": 1463258352,
    "timeoffset": 1,
    "pingtime": 8.236776000000001,
    "minping": 0.260138,
    "version": 70002,
    "subver": "/Satoshi:0.11.2/",
    "inbound": false,
    "startingheight": 411774,
    "banscore": 0,
    "synced_headers": 411777,
    "synced_blocks": 199986,
    "inflight": [
      200222, 
      200223, 
      200241, 
      200244, 
      200260, 
      200262, 
      200270, 
      200272, 
      200273, 
      200274, 
      200275, 
      200282, 
      200284, 
      200288, 
      200301, 
      200307
    
    ],
    "whitelisted": false
  
},
# [... Mais outros nós... ]
]
```

**Troca de Inventário:** Agora, para que o nó que acabou de entrar na rede possa ter uma cópia atualizada da blockchain, uma das primeiras coisas que ele faz ao se conectar com outros nós é a troca de inventários. A primeira mensagem enviada ao se conectar ao outro nó inclui o *bestHeight* que informa até que bloco ele tem conhecimento. Assim que ele receber a mensagem de versão de seu *peer* contendo o *bestHeight* dele, ele saberá se está precisando sincronizar a sua cópia local da blockchain com o resto do consenso da rede ou se é seu *peer* que está mais atrás. O *peer* que tiver a blockchain com mais longa (normalmente, a com mais trabalho), já sabendo quantos blocos o *peer* com a blockchain mais curta precisa, identificará os 500 primeiros blocos que o *peer* está precisando para alcançar o resto da rede e enviará uma mensagem *inv* com os *hashes* dos blocos para que, então, o *peer* que precisa se atualizar envie uma mensagem *getdata* pedingo a informação completa de cada bloco para que ele possa baixar os blocos e verificar um por um de forma independente.

Este nó, agora, faz parte da rede P2P Bitcoin e contribui recebendo, verificando e transmitindo transações, blocos e outras informações ao resto da rede.

> **Info** Uma dica para caso ainda esteja muitos blocos atrás na rede e queira acelerar a atualização é buscar por nós bem conectados e que estejam contribuindo bastante para a rede em locais como [esta lista](https://bitnodes.21.co/nodes/leaderboard/) e iniciar o seu client com a *flag* ```-addnode``` com o IP de alguns destes nós.

## Transmitindo e Propagando Transações

Toda vez que um nó conectado à rede cria uma transação, ele envia uma mensagem *inv* informando a nova transação para todos os seus *peers*. A mensagem *inv* é uma mensagem de inventório que apenas notifica outros nós sobre um novo objeto descoberto ou envia dados sobre algo que está sendo requisitado. O nó que recebe uma mensagem *inv* sabe se tem ou não um objeto porque esta mensagem inclui o *hash* do objeto. Este é o formato de uma mensagem *inv*:

| Campo | Tamanho    | Descrição                                      |
|-------|------------|------------------------------------------------|
| type  | 4 *bytes*  | tipo do objeto identificado neste inventório   |
| hash  | 32 *bytes* | *hash* do objeto                               |

Os outros nós que receberem a mensagem *inv* e não conhecerem este novo objeto - neste caso, uma nova transação - enviarão uma mensagem *getdata* incluindo o *hash* do objeto pedindo a informação completa. Ao receberem a transação, poderão verificar a validade dela por si próprios e, em caso da transação ser válida, repetirão o mesmo processo com os outros *peers* propagando a transação sucessivamente pela rede.

### Pools de Transação

Enquanto estas transações recebidas e validadas pelos nós não são incluidas por algum minerador à blockchain, elas, geralmente, permanecem em um espaço de memória volátil de cada *full-node* da rede. A maioria dos nós conectados à rede implementam a *pool* chamada *mempool* e, alguns outros, implementam *pools* separadas da *mempool* como as *orphan pools* para as chamadas transações orfãs e as *UTXO pools* que pode ser implementada em uma memória local persistente dependendo do tipo do *software* utilizado pelo nó.

**mempool:** guarda todas as transações recebidas e validadas pelo nó que não tenham sido escritas na blockchain por algum minerador. Quanto mais transações sendo propagadas pela rede, maior a *mempool* ficará até que ela vá sendo gradualmente esvaziada com as transações sendo aceitas pelos mineradores. Muitas implementações de carteiras usam a *mempool* para calcular as taxas de transação ideais para que a transação seja aceita o mais rápido quanto possível, criando uma sugestão de *fee* flutuante.

**orphan pool:** é uma *pool* separada da *mempool* existente em algumas versões de *full-node* na rede que guardam as transações que referenciem e dependam de uma transação anterior a elas, chamadas de *parent transactions* - transações pai/mãe - que ainda não tenham sido vistas por este nó.

**UTXO pool:** diferente da *mempool*, é uma *pool* de transações presente em algumas implementações que guarda em memória volátil ou persistente local milhões de entradas de *outputs* de transação criados e nunca gastos com *UTXOs* que podem datar, em alguns casos, de transações criadas em 2009. E, enquanto a *mempool* e a *orphan pool* apenas contém transações não confirmadas, esta *pool* apenas contém transações confirmadas na rede.

**Próximo capítulo:** [Blockchain](blockchain.md)
