# Transações

Uma das melhores introduções sobre transações vem diretamente do livro "Mastering Bitcoin" que, em tradução livre, diz:

>"Transações são a parte mais importante do sistema bitcoin. Todo o resto é desenhado para garantir que as transações possam ser criadas, propagadas na rede, validadas, e finalmente adicionadas ao livro-razão de transações (a blockchain). Transações são estruturas de dados que codificam a transferência de valor entre participantes do sistema bitcoin. Cada transação é uma entrada pública na blockchain do bitcoin, o livro-razão de dupla entrada global."

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

[INPUT/OUTPUT chain image PLACEHOLDER]

Tendo como única exceção as chamadas transações *coinbase* que são as transações criadas pelos mineradores ao incluirem um novo bloco na blockchain (mais detalhes em [Mineração](mineracao.md)) e recolherem o prêmio pelo trabalho.

[coinbase tx image PLACEHOLDER]

Como já dito, é importante lembrar que os *UTXO* quando usados numa nova transação sempre devem ser gastos **completamente**, o que faz que seja comum ter transações com um ou mais endereços de troco da transação já que nem sempre você terá *UTXOs* disponíveis formando o valor exato que deseja enviar para alguma transferência de valor.

#### *Outputs*

Todas transações criam um ou mais *outputs* para serem destrancados posteriormente quando usados em outra transação e a maioria deles podem ser gastos. Agora, nosso entendimento sobre transações fica mais interessante e preciso ao dissecarmos a estrutura do *output* para entendermos cada uma de suas peças:

| Campo                                | Descrição                                                          | Tamanho          |
|--------------------------------------|--------------------------------------------------------------------|------------------|
| *value*                              | Número de *satoshis* (BTC/10^8) a serem transferidos               | 8 *bytes*        |
| *locking-script length*              | O tamanho do *locking script* em *bytes*                           | 1-9 *bytes*      |
| *locking-script*                     | Um *script* com as condições necessárias para o *output* ser gasto | Tamanho variável |

*"Mas o que é um locking-script?"*

Os *locking-scripts* são uma peça fundamental para que você possa contemplar como as transações Bitcoin realmente funcionam. Eles são escritos na linguagem *Script* do Bitcoin (um pouco mais sobre a linguagem adiante) e ditam a condição necessária para que aquele *output* possa ser gasto; é como uma pergunta que precisa de uma resposta para que o *output* seja destrancado. Talvez você já tenha ouvido falar de *smart contracts* e este tipo de *script* é a forma que o Bitcoin aborda *smart contracts* desde 2009. As transações mais comuns são transações em que se transfere um certo número de bitcoins que poderão ser gastos por quem provar que tem o controle sob a chave privada de algum outro endereço, mas as condições podem ser bastante variadas permitindo, até mesmo, a criação de *outputs* comprovadamente impossíveis de serem gastos; o que atende a certos casos de uso como veremos adiante.

#### *Inputs*

Os *inputs* são referências aos *UTXOs* (*outputs* não gastos) contendo a resposta à condição necessária para gastar os *UTXOs*. A estrutura de um *input* de transação é:

| Campo                     | Descrição                                                                                | Tamanho          |
|---------------------------|------------------------------------------------------------------------------------------|------------------|
| *tx Hash*                 | Identificador da transação que contém os *UTXOs* a serem gastos                          | 32 *bytes*       |
| *output index*            | Índice do *UTXO* a da transação a ser gasto                                              | 4 *bytes*        |
| *unlocking-script length* | Tamanho do *unlocking-script* em *bytes*                                                 | 1-9 *bytes       |
| *unlocking-script*        | O *unlocking-script* que responde as condições do *locking-script* do *UTXO* a ser gasto | Tamanho variável |
| *sequence number          | Sequência para ser usada por *feature* de substituição de transação                      | 4 *bytes*        |
