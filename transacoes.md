# Transações

Uma das melhores introduções sobre transações vem diretamente do livro "Mastering Bitcoin" que, em tradução livre, diz:

>"Transações são a parte mais importante do sistema bitcoin. Todo o resto é desenhado para garantir que as transações possam ser criadas, propagadas na rede, validadas, e finalmente adicionadas ao livro-razão de transações (a blockchain). Transações são estruturas de dados que codificam a transferência de valor entre participantes do sistema bitcoin. Cada transação é uma entrada pública na blockchain do bitcoin, o livro-razão de dupla entrada global."

## Ciclo de Vida de uma Transação

O ciclo de vida de uma transação é todo o processo desde a criação de uma transação até a inclusão dela na blockchain por um minerador. De forma resumida, uma transação bem-sucedida é criada, assinada com a chave privada correspondente para "destrancar" os *outputs* da transação anterior utilizados como *inputs* da transação atual, enviada à rede, verificada e validada pelos nós na rede P2P Bitcoin, e propagada por eles até chegar ao minerador que incluirá a transação na blockchain (primeira confirmaçao).

Para mais detalhes sobre cada parte do ciclo de vida de uma transação do tipo mais comum, recomendo que leia  [O Ciclo de Vida de uma Transação Bitcoin](http://bitcoin.com.br/o-ciclo-de-vida-de-uma-transacao-bitcoin/) (arquivado [aqui](https://web.archive.org/web/20160428134050/https://bitcoin.com.br/o-ciclo-de-vida-de-uma-transacao-bitcoin/) caso não esteja disponível na URL original).

A melhor forma de entender como as transações funcionam na rede Bitcoin é conhecendo e "hackeando" cada peça desta estrutura. Comecemos por conhecer estas peças...

## Estrutura de uma Transação

Uma transação é a estrutura de dados responsável por formalizar em código a transferência de valor de um ou mais *inputs* (fonte dos fundos) para um ou mais *outputs* (destino dos fundos). Aqui está o primeiro nível e uma transação:

| Campo            | Descrição                                                                                      | Tamanho              | Tipo     |
|------------------|------------------------------------------------------------------------------------------------|----------------------|----------|
| *version*        | Identifica as regras que a transação segue                                                     | 4 *bytes*            | int32_t  |
| *tx_in count*    | Identifica quantos *inputs* a transação tem                                                    | 1-9 *bytes*          | var_int  |
| *tx_in*          | O(s) *input(s)* da transação                                                                   | Tamanho variável     | tx_in[]  |
| *tx_out count*   | Identifica quantos *outputs* a transação tem                                                   | 1-9 *bytes*          | var_int  |
| *tx_out*         | O(s) *output(s)* da transação                                                                  | Tamanho variável     | tx_out[] |
| *lock_time*      | Um *timestamp UNIX* ou um número de bloco a partir de quando/qual a transação será destrancada | 4 *bytes*            | uint32_t |

### Outputs e Inputs

Os *inputs* e *outputs* são os elementos fundamentais de uma transação. As transações são ligadas umas às outras por estes dois elementos; Os *inputs* de uma transação são, simplemente, os *outputs* de uma transação anterior. Estes *outputs* prontos para serem usado por uma nova transação são chamados de *UTXO* (*unspent transaction output*/*output* de transação não gasto).

A quantia de bitcoins que você tem são apenas uma abstração dos *UTXO* que você tem atribuídos a endereços sob o seu controle. Quando alguém diz que "tem 15 bitcoins" significa que esta pessoa tem uma quantidade de *UTXO* que atribuem, em seu total, 15 bitcoins para serem destrancados por endereços cujas chaves privadas estão sob controle dela. O sistema não tem um contador que diz algo como "15 bitcoins em sua conta"; ele apenas conhece *UTXOs* que são, por sua vez, abstraídos para valores como "15 bitcoins".

O modo como as transações funcionam, faz com que elas formem uma sucessiva corrente de *inputs* e *outputs* trancando e destrancando valores na rede.

[INPUT/OUTPUT chain image PLACEHOLDER]

Tendo como única exceção a este comportamento, as chamadas transações *coinbase* que são as transações criadas pelos mineradores ao incluirem um novo bloco na blockchain (mais detalhes em [Mineração](mineracao.md)) e recolherem o prêmio pelo trabalho.

[coinbase tx image PLACEHOLDER]


