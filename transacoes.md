# Transações

Uma das melhores introduções sobre transações vem diretamente do livro "Mastering Bitcoin" que, em tradução livre, diz:

> **Quote** "Transações são a parte mais importante do sistema bitcoin. Todo o resto é desenhado para garantir que as transações possam ser criadas, propagadas na rede, validadas, e finalmente adicionadas ao livro-razão de transações (a blockchain). Transações são estruturas de dados que codificam a transferência de valor entre participantes do sistema bitcoin. Cada transação é uma entrada pública na blockchain do bitcoin, o livro-razão de dupla entrada global."

## Ciclo de Vida de uma Transação

O ciclo de vida de uma transação é todo o processo desde a criação de uma transação até a inclusão dela na blockchain por um minerador. De forma resumida, uma transação bem-sucedida é criada, assinada com a chave privada correspondente para "destrancar" os *outputs* da transação anterior referenciados nos *inputs* da transação atual, enviada à rede, verificada e validada pelos nós na rede P2P Bitcoin, e propagada por eles até chegar ao minerador que incluirá a transação na blockchain (primeira confirmaçao).

Para mais detalhes sobre cada parte do ciclo de vida de uma transação do tipo mais comum, recomendo que leia  [O Ciclo de Vida de uma Transação Bitcoin](http://bitcoin.com.br/o-ciclo-de-vida-de-uma-transacao-bitcoin/) (arquivado [aqui](https://web.archive.org/web/20160428134050/https://bitcoin.com.br/o-ciclo-de-vida-de-uma-transacao-bitcoin/) caso não esteja disponível na URL original).

A melhor forma de entender como as transações funcionam na rede Bitcoin é conhecendo e "hackeando" cada peça desta estrutura. Comecemos por conhecer estas peças...

## Estrutura de uma Transação

Uma transação é a estrutura de dados responsável por formalizar em código a transferência de valor de um ou mais *inputs* (fonte dos fundos) para um ou mais *outputs* (destino dos fundos). Aqui está o primeiro nível e uma transação:

| Campo            | Descrição                                                                                            | Tamanho              |
|------------------|------------------------------------------------------------------------------------------------------|----------------------|
| *version*        | Identifica as regras que a transação segue                                                           | 4 *bytes*            |
| *tx_in count*    | Identifica quantos *inputs* a transação tem                                                          | 1-9 *bytes*          |
| *tx_in*          | O(s) *input(s)* da transação                                                                         | Tamanho variável     |
| *tx_out count*   | Identifica quantos *outputs* a transação tem                                                         | 1-9 *bytes*          |
| *tx_out*         | O(s) *output(s)* da transação                                                                        | Tamanho variável     |
| *lock_time*      | Um *timestamp UNIX* ou um número de bloco a partir de quando/qual a transação poderá ser destrancada | 4 *bytes*            |

Para que o nosso entendimento do papel de cada um destes campos fique bem claro: O campo ```version``` diz a versão desta transação para que todos nós na rede saberem que regras ela segue para que possam decidir como verificá-la ou, até, descartá-la em caso de incompatibilidade; os ```tx_in count``` e ```tx_out count``` servem como contadores dos *inputs* e *outputs*, respectivamente, para que a iteração por ambos seja simples ao se saber o número de elementos a se esperar; os ```tx_in``` e ```tx_out``` tem um ou mais *inputs* e *outputs* de transação, respectivamente em sua estrutura; E, finalmente, o  ```lock_time``` é o tempo em formato *UNIX* ou número de bloco a partir do qual a transação será considerada válida pela rede - normalmente, este valor é ```0``` para indicar que a transação pode ser gasta assim que recebida e validada.

### *Outputs* e *Inputs*

Os *inputs* e *outputs* são os elementos fundamentais de uma transação. As transações são ligadas umas às outras por estes dois elementos; Os *inputs* de uma transação são, simplemente, referências aos *outputs* de uma transação anterior. Estes *outputs* prontos para serem usado por uma nova transação são chamados de *UTXO* (*unspent transaction output*/*output* de transação não gasto).

A quantia de bitcoins que você tem são apenas uma abstração dos *UTXO* que você tem atribuídos a endereços sob o seu controle. Quando alguém diz que "tem 15 bitcoins" significa que esta pessoa tem uma quantidade de *UTXO* que atribuem, em seu total, 15 bitcoins para serem destrancados por endereços cujas chaves privadas estão sob controle dela. O sistema não tem um contador que diz algo como "15 bitcoins em sua conta"; ele apenas conhece *UTXOs* que são, por sua vez, abstraídos para valores como "15 bitcoins".

O modo como as transações funcionam, faz com que elas formem uma sucessiva corrente de *inputs* e *outputs* trancando e destrancando valores na rede.

![Corrente de Transações](/images/transacoes/tx-chain.png)

Tendo como única exceção as chamadas transações *coinbase* que são as transações criadas pelos mineradores ao incluirem um novo bloco na blockchain (mais detalhes em [Mineração](mineracao.md)) e recolherem o prêmio pelo trabalho.

Como já dito, é importante lembrar que os *UTXO* quando usados numa nova transação sempre devem ser gastos **completamente**, o que faz que seja comum ter transações com um ou mais endereços de troco da transação já que nem sempre você terá *UTXOs* disponíveis formando o valor exato que deseja enviar para alguma transferência de valor.

#### *Outputs*

Todas transações criam um ou mais *outputs* para serem destrancados posteriormente quando usados em outra transação e a maioria deles podem ser gastos. Agora, nosso entendimento sobre transações fica mais interessante e preciso ao dissecarmos a estrutura do *output* para entendermos cada uma de suas peças:

| Campo                                | Descrição                                                          | Tamanho          |
|--------------------------------------|--------------------------------------------------------------------|------------------|
| *value*                              | Número de *satoshis* (BTC/10^8) a serem transferidos               | 8 *bytes*        |
| *locking-script length*              | O tamanho do *locking script* em *bytes*                           | 1-9 *bytes*      |
| *locking-script*                     | Um *script* com as condições necessárias para o *output* ser gasto | Tamanho variável |

*"Mas o que é um locking-script?"*

Os *locking-scripts* são uma peça fundamental para que você possa contemplar como as transações Bitcoin realmente funcionam. Eles são escritos na linguagem *Script* do Bitcoin (um pouco mais sobre a linguagem adiante) e ditam a condição necessária para que aquele *output* possa ser gasto; é como um desafio que precisa de uma solução para que o *output* seja destrancado. As transações mais comuns são transações em que se transfere um certo número de bitcoins que poderão ser gastos por quem provar que tem o controle sob a chave privada de algum outro endereço, mas as condições podem ser bastante variadas permitindo, até mesmo, a criação de *outputs* provadamente impossíveis de serem gastos; o que atende a certos casos de uso como veremos adiante.

#### *Inputs*

Os *inputs* são referências aos *UTXOs* (*outputs* não gastos) contendo a resposta à condição necessária para gastar os *UTXOs*. A estrutura de um *input* de transação é:

| Campo                     | Descrição                                                                                       | Tamanho          |
|---------------------------|-------------------------------------------------------------------------------------------------|------------------|
| *tx Hash*                 | Identificador da transação que contém os *UTXOs* a serem gastos                                 | 32 *bytes*       |
| *output index*            | Índice do *UTXO* a da transação a ser gasto                                                     | 4 *bytes*        |
| *unlocking-script length* | Tamanho do *unlocking-script* em *bytes*                                                        | 1-9 *bytes       |
| *unlocking-script*        | O *unlocking-script* que responde as condições do *locking-script* do *UTXO* a ser gasto        | Tamanho variável |
| *sequence number          | Sequência para ser usada por *feature* de substituição de transação (atualmente, não utilizado) | 4 *bytes*        |

O *unlocking-script*, como você já deve estar concluindo, é a solução para o desafio colocado no *locking-script* do *output* ao qual o *input* atual fizer referência. Se o *UTXO* sendo gasto dizia *"Apresente assinatura pertencente ao endereço 1exemplo"*, o *unlocking-script* deve responder com a assinatura do endereço "1exemplo" para que a transação seja considerada válida.

O que nos leva a entender como esta linguagem funciona...

### Linguagem *Script* Bitcoin

Como já percebido, a alma da validação de uma transação Bitcoin está nos *locking script* - responsável por trancar os *outputs* - e *unlocking script* - responsável por destrancar os *UTXOs* utilizados num *input*. Mas como isso é lido e interpretado por cada nó?

A resposta está na linguagem *Script* do Bitcoin. Chamada apenas de *Script* é uma linguagem similar à [Forth](https://pt.wikipedia.org/wiki/Forth) com notação em [Polonesa Reversa/Inversa](https://pt.wikipedia.org/wiki/Nota%C3%A7%C3%A3o_polonesa_inversa) com método de execução em pilha interpretada da esquerda para a direita e foi intencionalmente limitada - por exemplo, ela é *Turing* incompleta - para evitar problemas inesperados na rede e garantir a robustez das transações. A explicação ficará simples quando visualizada.

Primeiro, vejamos como ambos *scripts* ficam organizados para serem interpretados na validação de uma transação. Cada nó responsável por validar uma transação utiliza o ```scriptPubKey``` do *UTXO* referenciado no *input* atual e utiliza o ```scriptSig``` deste *input* para formar a expressão a ser verificada. Aqui está um exemplo do tipo mais comum de transacão - a *Pay to Pubkey Hash* (*P2PKH*) - em que "enviamos" um certo número de bitcoins para ser gasto pelo detentor da chave privada de outro endereço:

![Script de Transação](/images/transacoes/tx-script.png)

Mas antes de resolvermos esta expressão, vamos visualizar a resolução de uma expressão com operações matemáticas básicas para que entendamos como os *scripts* são interpretados e validados na rede Bitcoin. Tomemos como exemplo este *locking script* contendo apenas operações aritméticas:

```
7 OP_ADD 3 OP_SUB 6 OP_EQUAL
```

Contemple o fato de que este *locking script* é sintaticamente correto e poderia ser o *script* trancando um *output* caso alguém quisesse uma "doação" ao primeiro que quisesse destrancar este *output*. As palavras com prefixo "OP_" são chamados de **opcodes** e são funções da linguagem de *script* do Bitcoin utilizadas nas operações; Uma lista de **opcodes** existentes pode ser encontrada [aqui](https://en.bitcoin.it/wiki/Script#Words).

A solução para este *script* pode ser dada apenas com:

```
2
```

A explicação para isso é que o programa de validação pega o *locking script* e o *unlocking script* e os coloca juntos numa expressão assim:

```
2 7 OP_ADD 3 OP_SUB 6 OP_EQUAL
```

E a execução desta expressão acontece como na imagem a seguir:

![Execução de Script de Transação](/images/transacoes/tx-script-exec.png)

Este *unlocking script* satisfaz o "desafio" proposto pelo *locking script* ao retornar TRUE que é representado pelo número 1 em hexadecimal ```0x01```. Caso a solução fosse inválida, ```OP_EQUAL``` retornaria FALSE ou ```0x00``` para a pilha.

Agora, entendendo a forma de execução podemos ver o que cada *opcode* na expressão inicial muito comum em transações Bitcoin faz com cada valor anterior.

```OP_DUP```: Duplica o valor no topo da pilha, ou seja, se ```x``` estiver no topo ```xx``` será retornado.
```OP_HASH160```: Aplica duas funções *hash* no *input*. Primeiro um SHA-256 e depois um RIPEMD-160, o que a partir de uma chave pública formará o endereço Bitcoin correspondente como visto em [Endereços e Carteiras](enderecos-e-carteiras.md).
```OP_EQUALVERIFY```: Aplica ```OP_EQUAL``` e ```OP_VERIFY``` logo em seguida, o que significa que checa se os *inputs* são iguais (e retorna TRUE se forem) e, então, marca o *script* como inválido se o valor no topo da pilha for FALSE e nada se for TRUE.
```OP_CHECKSIG```: Aplica uma função *hash* sobre os *outputs*, *inputs* e *script* da transação e verifica se a assinatura realmente pertence à chave esperada.

A execução do *script* inicialmente proposto retornará TRUE se a assinatura corresponder com a esperada e fará com que a transação seja considerada válida se todo o resto estiver certo - como possíveis tentativas de *double spend* ou valores errados.

#### Outros Scripts Comuns

Para finalizar, agora que entendemos que estes *scripts* de trancamento e destrancamento de transações são tão variados quanto a possibilidade de combinação dos *opcodes* que couberem numa transação, não podemos deixar de ver a notação de alguns dos outros *scripts* mais comuns na rede.

**Multi-assinatura (*multisig*)**: Este *script* é utilizado para criar uma conidição de destrancamento da transação na qual um número M de assinaturas devem ser apresentadas de N assinaturas especificadas no *locking-script*. Normalmente, chamado de M de N, um exemplo simples e comum é um esquema de *multisig* 2-de-3 no qual ao menos 2 assinaturas devem ser apresentadas para a transação de 3 chaves previamente especificadas para que os fundos possam ser movidos. Um exemplo de um *script* de trancamento e um de destrancamento para um esquema *multisig* seria como a seguir:


```
# Primeiro o script de trancamento 2-de-3...
2 <Chave Pública A> <Chave Pública B> <Chave Pública C> 3 OP_CHECKMULTISIG
```

```
# E para solucionar o script...
OP_0  <Assinatura A> <Assinatura C>
```

```
# Juntos ficam...
OP_0 <Assinatura A> <Assinatura C> 2 <Chave Pública A> <Chave Pública B> <Chave Pública C> 3 OP_CHECKMULTISIG
```

E esta expressão retornará TRUE se as 2 assinaturas corresponderem a 2 assinaturas **diferentes** das 3 especificadas no *script* de trancamento.

**OP_RETURN**: Um *script* normalmente chamado apenas de ```OP_RETURN``` devido ao uso deste *opcode* de mesmo nome é utilizado para criar um *output* chamado de *provably unspendable* (ou provadamente não gastável) e normalmente associado com algum dado. Um dos exemplos comuns de uso deste operador é a prova de existência, em que se grava o *hash* de aguma informação para ser adicionado na blockchain e comprovar que tal informação existia, pelo menos, a partir daquele ponto em que foi escrita na blockchain; criando, assim, uma prova matematicamente verificável por qualquer um com acesso à blockchain de que uma informação realmente existia naquele ponto da história em que esta transação foi adicionada à blockchain. Um exemplo deste *script* ficaria assim:

```
OP_RETURN <informação a ser gravada; normalmente um *hash* feito a partir de uma informação (como um arquivo) devido ao limite de 40 *bytes*>
```

Dois exemplos de utilização desta capacidade da linguagem de *script* são os sites [Proof of Existence](https://www.proofofexistence.com/) e o brasileiro [OriginalMy](https://originalmy.com/) que geram provas de existência da informação suprida.

**Pay to Script Hash (P2SH)**: Este tipo de *script* facilitou muito a criação de condições mais complexas para a liberação de *outputs* e, inclusive, substitui muitos esquemas de *multisig* devido à sua facilidade para envio de pagamentos para este tipo de esquema de assinatura. Para entendermos, vamos pegar um exemplo de *script* apresentado em *multisig*:

```
2 <Chave Pública A> <Chave Pública B> <Chave Pública C> 3 OP_CHECKMULTISIG
```

O problema que se encontra com este tipo de esquema é que, além do espaço necessário para adicionar um certo número de chaves, como pode notar, qualquer pessoas que esteja criando o pagamento teria que criar uma transação que gerasse um *output* desta forma. Isso, em larga escala, especialmente quando lidando com usuários comuns na rede é inviável visto que a maioria dos *softwares* de carteira nem mesmo geram este tipo de pagamento. A solução para isso introduzida em 2012 com *P2SH* é criar um *hash* a partir do *script* que desejamos usar como condição de destrancamento dos fundos e quando quisermos gastar estes fundos deveremos apresentar o *script* que formou o *hash* junto com a condição de destrancamento para ele. Logo, um *script* de trancamento como o de cima seria reduzido para o resultado do *hash* SHA-256 e, logo após, a aplicação do *hash* RIPEMD-160 por cima deste, fazendo com que possamos criar uma condição assim:

```
OP_HASH160 ab8648c91206a3770b8aaf6f5153b6b35423caf0 OP_EQUAL
```

O que garantiria que o *script* de destrancamento seria verificado e bateria com o *hash* esperado.

Isso resolve alguns problemas, mas a capacidade mais interessante vem com a *feature* do P2SH codificar o *hash* do *script* como um endereço Bitcoin como pode ser visto na [BIP0013](https://github.com/bitcoin/bips/blob/master/bip-0013.mediawiki) que é chamada de **Pay to Script Hash Address*. Com isso, após o trabalho de criação do *hash* do script, a única coisa que precisa ser feita é criar um endereço a partir deste *script* aplicando a codificação *Base58Check* utilizada para endereços Bitcoin resultando em um endereço Bitcoin normal com o prefixo legível 3 como visto em [Endereços e Carteiras](enderecos-e-carteiras.md) sobre o significado de prefixos de endereços.

Finalmente, a partir de agora, qualquer transação enviada para este endereço com prefixo 3 respeitará as regras de destrancamento ditadas pelo *script* utilizado para formar este endereço. Resolvendo, entre outros problemas descritos acima, a questão de facilidade de pagamento já que ningueḿ que envia a transação para este endereço precisa conhecer o *script* que o formou.


**Próximo capítulo:** [Rede P2P](rede-p2p.md)
