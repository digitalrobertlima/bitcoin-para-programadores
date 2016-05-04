# Introdução

## O que é Bitcoin?

### De Forma Resumida

Em tradução livre do repositório da principal implementação atualmente ([Bitcoin Core](https://github.com/bitcoin/bitcoin)):


>*"Bitcoin é uma nova moeda digital experimental que permite pagamento instantâneo para qualquer pessoa, em qualquer lugar do mundo. Bitcoin usa tecnologia peer-to-peer (P2P) para operar sem autoridade central: a gerência de transações e a emissão de dinheiro é executada coletivamente pela rede. Bitcoin Core é o nome do software open source que habilita o uso desta moeda."*

### Indo um Pouco Além...

Bitcoin é a união de tecnologias e abstracões que possibilitam que o consenso entre atores não necessariamente conhecidos possa ser alcançado de forma descentralizada sem que a confiaça tenha que ser depositada em um ponto de controle central ou que a segurança rede esteja sujeita à um único ponto de falha. Estas tecnologias em conjunto formam as bases para a existência de uma moeda digital descentralizada e para qualquer outro caso de uso que possamos abstrair para um modelo baseado em consenso - como contratos - de forma independente de autoridades centrais como bancos ou governos.

E, é importante reparar que o mesmo termo "**B**itcoin" com "B" maiúsculo é comumente utilizado para designar a tecnologia como um todo, a rede P2P Bitcoin ou o protocolo Bitcoin enquanto **b**itcoin(s) com "b" minúsculo é utilizado para designar a unidade de conta usada na rede.

## A Criptografia no Bitcoin

Bitcoin é uma cripto-moeda; e isto se deve ao fato de a Criptografia ser uma parte essencial em seu funcionamento.

A Criptografia é um ramo da matemática que, em sua definição moderna, acolhe toda a tecnologia criada e utilizada para restringir verdades fundamentais da natureza da informação com o intuito de alcançar objetivos como: esconder mensagens, provar a existência de um segredo sem a necessidade de revelar o segredo, provar autenticidade e integridade de dados, provar trabalho computacional etc...

A princípio, no Bitcoin, estamos interessados em atingir os seguintes objetivos com uso de algoritmos criptográficos: Garantia de integridade e consistência de dados na rede e prova de trabalho computacional utilizando *hashes* e autenticidade das transações utilizando assinaturas digitais de Criptografia de Chave Pública.

### Funções *Hash* Criptográficas

Este tipo de função é usado como bloco fundamental em muitas aplicações criptográficas e tem como comportamento básico receber um conjunto de dados de tamanho arbitrário como *input* e produzir um valor *hash* de um tamanho fixo como *output* que chamamos de *digest* como forma de representação do dado de entrada. Chamamos de funções *hash* **criptográficas**, todas as funções *hash* que atendem aos seguintes requisitos: ela deve resistir a todo tipo de ataque cripto-analítico conhecido, deve ter resistência à colisão - ou seja, a geração de um mesmo *digest* para a *inputs* diferentes deve ser impraticável -, ser computacionalmente eficiente, agir como uma função matemática trap-door - o que significa dizer que deve ser impraticável/improvável reverter fazer o caminho contrário e reverter o *digest* de volta ao *input* original, além de ser impraticável a retirada de qualquer dado útil do *digest* que possa dizer algo sobre o *input* usado na função.

As duas funções *hash* utilizadas no Bitcoin são a SHA-256 (*Secure Hash Algorithm*) que retorna *digests* de 256 *bits* e a RIPEMD-160 (*RACE Integrity Primitives Evaluation Message Digest*) que retorna *digests* de 160 *bits*.

Exemplo de ambas funções sendo usadas em Python com a *string* "bitcoin" como *input* e *print* do *digest* no formato mais comum em hexadecimal:

```python
>>> import hashlib

>>> word = "bitcoin"

>>> word_sha256 = hashlib.sha256(word.encode('utf-8'))
>>> print(word_sha256.hexdigest())
6b88c087247aa2f07ee1c5956b8e1a9f4c7f892a70e324f1bb3d161e05ca107b

>>> word_ripemd160 = hashlib.new('ripemd160')
>>> word_ripemd160.update(word.encode('utf-8'))
>>> print(word_ripemd160.hexdigest())
5891bf40b0b0e8e19f524bdc2e842d012264624b

# hashes completamente diferentes são formados com pequenas alterações no input
>>> word2 = "bitcoin2"

>>> word2_sha256 = hashlib.sha256(word2.encode('utf-8'))
>>> print(word2_sha256.hexdigest())
1ed7259a5243a1e9e33e45d8d2510bc0470032df964956e18b9f56fa65c96e89

>>> word2_ripemd160 = hashlib.new('ripemd160')
>>> word_ripemd160.update(word2.encode('utf-8'))
>>> print(word_ripemd160.hexdigest())
5f67cd0e647825711eac0b0bf78e0487b149bc3a
```

Com estas funções em mão conseguimos verificar integridade de informações enviadas à rede, gerar e verificar prova de trabalho computacional ou *proof-of-work* e, com isso, criar a "cola" criptográfica fundamental para a segurança da blockchain (mais detalhes em [Mineração](mineracao.md)).

A compreensão sobre *hashes* neste nível de abstração já é suficiente para o entendimento do valor de suas propriedades no Bitcoin e o uso consciente destas propriedades que você verá adiante em mais exemplos. Caso você queira aprofundar o seu entendimento das abstrações matemáticas que possibilitam estas funções, recomendo: [ links list placeholder ]

 ### Criptografia de Chave Pública

Outra tecnologia fundamental para o funcionamento do Bitcoin é a Criptografia de Chave Pública. Esta tecnologia possibilita que pessoas possam usar ferramentas criptográficas para encriptar e provar/verificar autenticidade de informações trocadas sem a necessidade do compartilhamento de um segredo.

No Bitcoin, estamos mais interessados nas assinaturas digitais produzidas por este tipo de criptografia, na qual eu consigo provar que tenho um segredo (a Chave Privada ou *Private Key*) ao mesmo tempo em que autentico uma transação com este segredo sem a necessidade de compartilhar este segredo; tendo apenas que compartilhar a minha Chave Pública (*Public Key*). Assim, garantindo que apenas o detentor da Chave Privada correta poderá movimentar fundos na rede.

Eu acho que a melhor explicação simplificada ao estilo *Explain Like I'm 5* que já achei e vai te ajudar a ver o problema com mais simplicidade é um trecho do ótimo artigo sobre Assinaturas Digitais de Curva Elíptica (o método utilizado para as assinaturas no Bitcoin) do falecido (e, por enquanto, "ressuscitado") blog [The Royal Fork](http://royalforkblog.github.io/2014/09/04/ecc/) que, em tradução livre e um pouco alterada para adequação, descreve:

>*"Imagine uma turma de crianças nos primeiros anos de escola que sabem multiplicação, mas ainda não aprenderam divisão. No início do ano, o professor proclama 'Meu número especial é 3'. Numa manhã, a mensagem 'Sempre foi assim e sempre irá ser' - assinado 'Professor - 11' aparece no quadro negro. Como os alunos sabem que esta mensagem veio do professor e não de um fraudador que gosta de recitar frases de filmes? Eles multiplicam o "número especial" do professor - 3 - pelo "número da assinatura" - 11 - e se eles obtiverem o número de caracteres contidos na mensagem (33 caracteres), eles julgam a assinatura válida, e estão confiantes de que a mensagem realmente foi escrita pelo professor. Sem a mágica da divisão, os alunos não conseguem produzir uma assinatura válida para qualquer mensagem arbitrária, e porque a assinatura é baseada no tamanho da mensagem, os estudantes não podem mudar a mensagem sem invalidar a assinatura.*

>*Isto é fundamentalmente como o Algoritmo de Assinaturas Digitais de Curva Elíptica funciona; Ao conhecedor da chave privada é concedido o poder da divisão, enquanto os conhecedores da chave pública estão restritos à multiplicação, o que os permite checarem se uma assinatura é válida ou não."*

Há mais de um método matemático para alcançar este tipo de funcionalidade e no Bitcoin é utilizado o Algoritmo de Assinaturas Digitais de Curva Elípica (ou *Elliptic Curve Digital Signature Algorithm*) ao qual vou me referenciar a partir de agora pela sigla em inglês ECDSA. A implementação utilizada no Bitcoin Core - a implementação referência atual - é a [libsecp256k1](https://github.com/bitcoin/secp256k1) desenvolvida pelo Bitcoin Core *developer* Pieter Wuille para substituir a implementação anterior que utilizava a biblioteca OpenSSL com o objetivo de remover uma dependência do código do Core, além de ser uma implementação muito mais eficiente do algoritmo com um código melhor testado do que na OpenSSL.

Este tipo de assinatura é o que permite que todos na rede possam comprovar que uma transação foi enviada pelo detentor de uma certa chave privada - que nada mais é que um número gigante de 256 *bits* obtido, se feito corretamente, de forma criptograficamente aleatória - sendo, assim, essencial para o funcionamento correto do Bitcoin.

Por exemplo, digamos que Maria envia 1 bitcoin para o endereço bitcoin de João e João, por sua vez, envia 1 bitcoin para o endereço de Raphael. Todas estas transações são apenas mensagens que dizem "passar *n* bitcoins de *x* para *y*" e todas estas mensagens precisam ser assinadas por quem a rede considera o atual detentor dos bitcoins para que sejam consideradas válidas e incluidas na blockchain por algum minerador.

Em [Endereços e Carteiras](enderecos-e-carteiras.md), veremos como a chave privada pode ser gerada e como fazemos para derivar a chave pública a partir da chave privada para, finalmente, gerar um endereço Bitcoin. Em [Transações](transacoes.md) veremos como as assinaturas são realmente enxergadas na rede e como assinar transações utilizando os comandos RPC do Bitcoin Core e implementaremos um código Python com o mesmo intuito. E, para caso você queira se aprofundar mais nas abstrações matemáticas que dão à luz esta tecnologia amplamente utilizada atualmente, recomendo: [links list placeholder]

*Próximo capítulo*: [Bitcoin Core](bitcoin-core.md)
