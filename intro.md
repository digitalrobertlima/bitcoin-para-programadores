# Introdução

## O que é Bitcoin?

### De Forma Resumida

Em tradução livre do repositório da principal implementação atualmente ([Bitcoin Core](https://github.com/bitcoin/bitcoin)):


>*"Bitcoin é uma nova moeda digital experimental que permite pagamento instantâneo para qualquer pessoa, em qualquer lugar do mundo. Bitcoin usa tecnologia peer-to-peer (P2P) para operar sem autoridade central: a gerência de transações e a emissão de dinheiro é executada coletivamente pela rede. Bitcoin Core é o nome do software open source que habilita o uso desta moeda."*

### Indo um Pouco Além...

Bitcoin é a união de tecnologias e abstracões que possbilitam que o consenso entre atores não necessariamente conhecidos possa ser alcançado de forma descentralizada sem que a confiaça tenha que ser depositada em um ponto de controle central ou que a segurança rede esteja sujeita à um único ponto de falha. Estas tecnologias em conjunto formam as bases para a existência de uma moeda digital descentralizada independente de autoridades centrais como bancos ou governos e de qualquer outro elemento que possamos abstrair para um modelo baseado em consenso - como contratos.

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

Com estas funções em mão conseguimos verificar integridade de informações enviadas à rede, gerar e verificar prova de trabalho computacional ou *proof-of-work* e, com isso, criar a "cola" criptográfica fundamental para a segurança da blockchain (mais sobre isto em [Mineração](mineracao.md)).

A compreensão sobre *hashes* neste nível de abstração já é suficiente para o entendimento do valor de suas propriedades no Bitcoin e o uso consciente destas propriedades que você verá adiante em mais exemplos. Caso você queira aprofundar o seu entendimento das abstrações matemáticas que possibilitam estas funções, recomendo: [ links list placeholder ]

 ### Criptografia de Chave Pública
