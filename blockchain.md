# Blockchain

A blockchain - ou block chain - é uma estrutura de dados ordenada composta por blocos de transações criptograficamente ligados por uma referência direta ao bloco anterior, servindo como livro-razão público no Bitcoin. A propriedade mais interessante da blockchain para o consenso de um sistema descentralizado é este elo criptográfico que liga os seus blocos entre si, que é criado pelo esforço computacional do algoritmo de *proof-of-work*. Sendo apenas mais uma estrutura de dados sem propriedades excepcionalmente úteis por si só fora do contexto apropriado; em um sistema descentralizado é onde esta estrutura de dados engenhosa se torna uma peça-chave na manutenção de consenso e segurança. A blockchain foi dada à luz por Satoshi Nakamoto como uma abordagem inteligente a um problema inerente a sistemas descentralizados: confiança.

O primeiro bloco da blockchain do Bitcoin é o bloco 0 chamado de bloco genesis. Este é o bloco minerado por Satoshi Nakamoto que serve como ponto de partida comum a todas as implementações do Bitcoin e é escrito diretamente no código referência para este fim. Ele contém a famosa frase escolhida por Satoshi Nakamoto:

```
The Times 03/Jan/2009 Chancellor on brink of second bailout for banks
```

Esta frase é serve o propósito de ser uma prova da data mínima em que a rede Bitcoin foi iniciada por se tratar da manchete de um jornal e, também, como mensagem clara sobre a motivação para a concepção de uma tecnologia como o Bitcoin para os dias de hoje. Ela pode ser encontrada diretamente no código-fonte do Bitcoin Core no [arquivo chainparams.cpp linha 53](https://github.com/bitcoin/bitcoin/blob/5851915a006ace71493f54665322e41001fb3ac3/src/chainparams.cpp#L53).

## Blocos

Cada bloco é uma estrutura de dados que contém as transações a serem incluidas na blockchain por meio do trabalho dos mineradores na rede.

 De uma forma geral, o bloco é composto por um cabeçalho contendo metadados sobre o bloco e uma lista de transações:

| Campo                  | Tamanho     | Descrição                                                       |
|------------------------|-------------|-----------------------------------------------------------------|
| Tamanho do Bloco       | 4 *bytes*   | tamanho do bloco (*block size*) em *bytes* a partir deste campo |
| Cabeçalho do Bloco     | 80 *bytes*  | o cabeçalho do bloco (*block header*) contendo metadados        |
| Contador de Transações | 1-9 *bytes* | número de transações neste bloco                                |
| Transaçòes             | Variável    | as transações deste bloco                                       |

Como na descrição, o cabeçalho do bloco é responsável por conter os metadados referentes ao bloco e é nele que está a "cola" criptográfica fundamental para a segurança da blockchain. A estrutura do cabeçalho do bloco contém os seguintes campos:

| Campo                                   | Tamanho     | Descrição                                                                    |
|-----------------------------------------|-------------|------------------------------------------------------------------------------|
| Versão                                  | 4 *bytes*   | número de versão do bloco indica que regras este bloco segue                 |
| *Hash* do Cabeçalho do Bloco Anterior   | 32 *bytes*  | *hash* do cabeçalho do bloco anterior (*parent block*) a este na blockchain  |
| Raiz de Merkle                          | 32 *bytes*  | *hash* da raiz de Merkle das transações deste bloco                          |
| *Timestamp*                             | 4 *bytes*   | hora aproximada da criação deste tempo em segundos no padrão UNIX            |
| Dificuldade Alvo                        | 4 *bytes*   | dificuldade alvo do algoritmo de *proof-of-work* para este bloco             |
| *Nonce*                                 | 4 *bytes*   | contador utilizado como *nonce* no algoritmo de *proof-of-work*              |

Como pode ver, os carregam todas as informações necessárias para servirem como páginas de um livro-razão das transações confirmadas na rede. Porém, sem o processo de mineração, os blocos e a blockchain estão "mortos". A partir da mineração a vida e utilidade desta estrutura ganha sentido ao se encaixar perfeitamente à esta corrida de processamento (e um pouco de sorte) acontecendo neste exato instante entre milhares de computadores ao redor do mundo. Alguns itens diretamente ligados ao processo de mineração - dificuldade, *nonce* e *timestamp* - serão revistos no [capítulo sobre mineração](mineracao.md); No momento, estamos interessados em ver os elementos que compõem blockchain para que o processo de atualizaçã dela possa fazer sentido.

## O Elo entre os Blocos

Como dito, no cabeçalho dos blocos está a chave para a segurança da blockchain proporcionada pelo algoritmo de *proof-of-work*. Este elo é *hash* do cabeçalho do bloco anterior.

O que ocorre é que cada bloco tem um identificador único que é criado a partir do *hash* de seu cabeçalho que serve tanto como identificador único deste objeto na rede quanto como prova de toda informação - incluindo as transações - contida nele. Para esta identificacão, apenas precisamos do cabeçalho de cada bloco já que a Raiz de Merkle (explicada um pouco adiante) serve como prova de todas as transações incluidas no bloco da qual ela faz parte. Esta característica faz com que a blockchain possa ser visualizada como uma corrente de blocos com os *hashes* do bloco anterior como elo criptográfico entre cada bloco:

![blockchain](/images/blockchain/blockchain.png)

Na prática, cada novo bloco tem ligação matematicamente comprovada com todos os blocos anteriores a ele, pois cada bloco tem um *hash* como identificador **único** que inclui, em seu resultado, o *hash* do bloco anterior a ele na blockchain. E, junto com o esforço computacional comprovado pelo *proof-of-work* responsável por gerar cada bloco, esta propriedade cria um elo que torna a forjabilidade da blockchain exponencialmente mais difícil a cada novo bloco.

Para visualizarmos a criação do *hash* do cabeçalho de um bloco - comumente chamado apenas de *hash* do bloco ou *block hash* - podemos usar um pequeno *script* Python3 para calcularmos o *hash* do [bloco #125552](https://blockr.io/block/info/125552). Para este fim, utilizamos os 6 campos do cabeçalho do bloco descritos acima concatenados em hexadecimal e ordem dos bytes *little-endian* para fazermos um *hash* SHA-256 seguido de outro *hash* SHA-256 com o resultado do anterior (*shasha*):

```
>>> import hashlib
>>> import codecs
>>> import binascii

#  blockheader com os 6 campos em hexadecimal concatenados...
>>> blockheader_hex = ("01000000" +  # <- versão
 "81cd02ab7e569e8bcd9317e2fe99f2de44d49ab2b8851ba4a308000000000000" + #  <- hash do bloco anterior
 "e320b6c2fffc8d750423db8b1eb942ae710e951ed797f7affc8892b0f1fc122b" +  # <- raiz de Merkle
 "c7f5d74d" +  # <- timestamp unix
 "f2b9441a" +  # <- dificuldade alvo em formato compacto
 "42a14695")  # <- nonce

# passando de hex para binário
>>> blockheader_bin = codecs.decode(blockheader_hex.encode('utf-8'), 'hex')
# crio o hash com uma dupla rodada de sha256 (shasha)
>>> blockheader_hash = hashlib.sha256(hashlib.sha256(blockheader_bin).digest()).digest()[::-1]  # <- trocando a ordem dos bytes para big-endian
# e de volta a hexadecimal com os bytes já na ordem certa
>>> blockheader_hash = binascii.hexlify(blockheader_hash)
>>> print(blockheader_hash)
b'00000000000000001e8d6829a8a21adc5d38d0a473b144b6765798e61f98bd1d'
```

E este é o *hash* do bloco 125552. Note a passagem da ordem dos *bytes* de *little-endian* para *big-endian* que é como as informações são transmitidas na rede e, geralmente, salvas desta forma.

### Árvores e Raízes de Merkle

Árvores de Merkle são estruturas de dados utilizadas para criar um resumo de dados com integridade criptograficamente verificável de forma eficiente quando em poder da raiz de Merkle - que vai no cabeçalho de cada bloco - e de um caminho de Merkle.

Para formar a raiz desta árvore binária com as transações, cada transação tem o seu *id* (o *hash* da transação) concatenado ao *id* da transação vizinha na árvore é submetida a uma dupla rodada da função *hash* SHA-256 sucessivamente até chegar à raiz. A visualização torna simples:

![Árvore de Merkle](/images/blockchain/simple-merkle-tree.png)

Cada um dos *hashes* passados é a mesma dupla rodada de SHA-256 já conhecida por nós. Logo, para formar o nó ```H AB``` e supondo que já temos o *id* das transações atribuídos às variáveis ```tx_a``` e ```tx_b``` basta que façamos isso:

```
>>> import hashlib
>>> H_ab = hashlib.sha256(hashlib.sha256(tx_a + tx_b).digest()):w
```

Para isso, todas as transações devem ser colocadas no mesmo nível como na imagem e formarem duplas para criarem os níveis acima sucessivamente. Em caso do bloco não ter um número par de transações, tudo que se faz é repetir a última transação:

![Árvore de Merkle com Transação Repetida](/images/blockchain/merkle-tree-repeated-tx.png)

Óbviamente, no Bitcoin, estas árvores são muito maiores e proporcionais ao número de transações de cada bloco e em cenários desta magnitude que esta estrutura se torna uma solução para comprovação eficiente da existência e integridade de uma transação num dado bloco. Cada *hash* deste tem o tamanho de 32 *bytes* e a complexidade de busca na árvore de Merkle cresce ```O(log2(N))``` na notação "Big-O" com N sendo o número de transações.

Vejamos um exemplo um pouco maior de uma árvore de Merkle criada a partir de 16 transações:

![Árvore de Merkle com 16 folhas](/images/blockchain/bigger-merkle-tree.png)

Caso precisemos comprovarmos que a transação M está incluida no bloco, precisamos de apenas 4 *hashes* de 32 *bytes* num total de 128 *bytes* para formar o noso caminho de Merkle. Veja como o caminho de Merkle junto com a raiz de Merkle é tudo que precisamos para comprovarmos que um bloco inclui a transação M:

[ BIGGER MERKLE TREE WITH MERKLE PATH HIGHLIGHTED IMAGE PLACEHOLDER ]

A eficiência da árvore de Merkle para este objetivo vai se tornando mais óbvia de acordo com que aumentamos o número de transações e comparamos com o número de *bytes* necessários para comprovar a existência de uma transação nestes números maiores:

| Número de Transações | Tamanho Aproximado do Bloco | Tamanho do Caminho de Merkle (*hashes*) | Tamanho do Caminho de Merkle (*bytes*) |
|----------------------|-----------------------------|-----------------------------------------|----------------------------------------|
| 16 transações        | 4 *kilobytes*               | 4 *hashes*                              | 128 *bytes*                            |
| 512 transações       | 128 *kilobytes*             | 9 *hashes*                              | 288 *bytes*                            |
| 2048 transações      | 512 *kilobytes*             | 11 *hashes*                             | 352 *bytes*                            |
| 65.525 transações    | 16 *megabytes*              | 16 *hashes*                             | 512 *bytes*                            |

 O que observamos é que com poucas transações não parece fazer muito sentido o uso da árvore de Merkle, mas logo que o número de transações começa a saltar podemos ver claramente a otimização que esta estrutura traz ao sistema do Bitcoin.

**Próximo capítulo:** [Mineração](mineracao.md)
