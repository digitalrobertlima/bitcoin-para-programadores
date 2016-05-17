# Endereços e Carteiras

Como visto em [A Criptografia no Bitcoin](intro.md#a-criptografia-no-bitcoin), a  criptografia de chave pública é essencial para resolver a questão de propriedade em sistemas computacionais como no caso do Bitcoin junto com o apoio criptográfico das funções *hash* criptográficas. O que antes eram apenas 0's e 1's indistinguíveis ganham algumas propriedades como escasses, autenticidade e integridade. E isto é a base para que se entenda o que os endereços representam e como eles são tratados pela rede.

## Criando Endereços

Cada endereço é um identificador único na rede; não de uma pessoa como *e-mails*, mas de uma chave pública. Para podermos criar um endereço sob nosso controle, primeiro temos que gerar uma chave privada de onde calcularemos a chave pública que será identificada como um endereço.

### Gerando a Chave Privada

A chave privada é um número gerado de forma aleatória. E além da aleatoriedade na geração deste número, outra qualidade importante para a segurança na implementação deste tipo de criptografia é que a chave privada é um número gigante. Com esta chave, as transações serão assinadas e valores serão transferidos na rede; gerar e manter esta chave de forma segura é **essencial** para a segurança de valores em bitcoin.

Para a geração segura de uma chave privada, primeiro precisamos de uma fonte segura de entropia para geração de uma chave de 256 *bits*. Normalmente, os *softwares* de carteira utilizam uma fonte de coleta de entropia do próprio sistema operacional - como ```/dev/urandom``` em sistemas UNIX ou ```cryptGenRandom()``` no Windows - e isto costuma ser o suficiente, mas para o nível máximo de paranoia é possível que utilizemos métodos físicos que excluem a possibilidade de falhas ou *backdoors* de *software*/*hardware* como o *diceware* descrito especialmente para o Bitcoin [aqui](#toDo-link). No mínimo, recomendo que gere a chave privada pelo método *diceware* descrito no link anterior pelo menos uma vez na vida devido à experiência que te ajudará a entender ou fortalecer na prática algumas coisas a mais sobre entropia.

> **Danger** Como programador(a) é a sua responsabilidade conhecer e entender a ferramenta que você estiver utilizando para a geração de números aleatórios criptograficamente seguros (*cryptographically secure pseudo-random number generator* - CSPRNG). Não use uma técnica própria desconhecida ao resto do mundo mesmo que você tenha a capacidade de cria uma técnica razoavelmente segura - parte fundamental na segurança de um algoritmo deste tipo é o fato de ele ter sido revisado por muitos especialistas e utilizado em "batalha" ao longo de muitos anos. Não confie nem mesmo nas ferramentas que eu apontar sem que você entenda porque elas são seguras e/ou que você possa verificar ampla confiança consolidada de outros especialistas nestas ferramentas. Dito isto, voltemos ao material...

Em Python, podemos utilizar o ```os.urandom``` que coleta *bytes* a partir do ```/dev/urandom``` em sistema UNIX e ```cryptGenRandom()``` no Windows para geração da chave privada em *bytes*:

```
>>> import os

>>> privkey = os.urandom(32)
>>> print(privkey)
b'\x9a\xfbch\x05\xc3|\xdf\xfc\xebJ\x89g\xdf\xfcn\xbe\x80%m\x91\x80DI\xd0hs6\x04\x84~\xe8'
#  para termos uma ideia do tamanho do número, veja em int...
privkey_int = int.from_bytes(privkey, byteorder='little')
>>> print(privkey_int)
105160114745571781986276553867283233264261203826988260355664227503456152451994
# e em hexadecimal como é comum ser enviado e recebido pelos cabos...
# (obviamente, em sistema seguros e de seu controle caso seja REALMENTE necessário)
>>> privkey_hex = hex(privkey_int)
>>> print(privkey_hex)
0xe87e8404367368d0494480916d2580be6efcdf67894aebfcdf7cc3056863fb9a
```

Agora temos 32 *bytes* (ou 256 *bits*) coletados. Certamente nenhum dos formatos mostrados parece com o que costumamos ler ao pedir que uma carteira exporte a chave privada para nós. Isto ocorre porque, diferente de quando estamos passando informação pelos cabos, costumamos utilizar um formato especial chamado WIF.

> **Info** **WIF**: este é o formato que costumamos ler e é abreviação para *Wallet Import Format* (Formato de Importação de Carteira). Este formato é usado para facilitar a leitura e cópia das chaves por humanos. O processo é simples: pegamos a chave privada, concatenamos o *byte* ```0x80``` para ```mainet``` ou ```0xef``` para a ```testnet``` como prefixo e o *byte* ```0x01``` como sufixo se a chave privada for corresponder a uma chave pública comprimida - uma forma de representar chaves públicas usando menos espaço -, realizamos uma função *hash* SHA-256 duas vezes seguidas - comumente referido como *shasha* -, pegamos os 4 primeiros *bytes* do resultado - este é o *checksum* -, adicionamos estes 4 *bytes* ao final do resultado da chave pública antes do *shasha*, e então, usamos a funcão *Base58Check* para codificar o resultado final.


>**Base58Check**: é uma versão modificada da função de Base58 utilizada no Bitcoin para codificar chaves e endereços na rede para produzir um formato que facilita a digitação por humanos, assim como diminui drasticamente a chance de erros na digitação limitando os endereços a caracteres específicos que não sejam visualmente idênticos em algumas fontes - como 1 e I ou 0 e O - e outros problemas parecidos no envio e recebimento de endereços por humanos.

Então, vamos pegar a chave privada que criamos e transformar ela para o formato WIF. Ao finalizarmos, como teste do resultado, pegue a chave no formato WIF e importe ela em algum *software* de carteira e verá que a sua carteira criará endereços a partir desta chave privada normalmente. Aqui está uma forma como podemos transformar a chave privada que obtivemos acima para o formato WIF:

```
#!/usr/bin/env python3
import hashlib
from base58 import b58encode

# definimos a dupla rodada de sha-256 para melhorar a legibilidade...
def shasha(data):
    """SHA256(SHA256(data)) -> HASH object"""
    result = hashlib.sha256(hashlib.sha256(data).digest())
    return result

# agora criamos a função que passará a chave para formato WIF...
def privkey_to_wif(rawkey, compressed=True):
    """Converte os bytes da chave privada para WIF"""
    k = b'\x80' + rawkey  # adicionamos o prefixo da mainet

    # por padrão criamos formado comprimido
    if compressed:
        k += b'\x01'  # sufixo para indicar chave comprimida

    checksum = shasha(k).digest()[:4]  # os primeiros 4 bytes da chave como checksum
    key = k + checksum

    b58key = b58encode(key)
    return b58key

# agora podemos usar a função privkey_to_wif com a nossa chave privada
# privkey obtida no exemplo anterior
privkey_wif = privkey_to_wif(privkey)
print(privkey_wif)
# L2QyYCh5nFDe4yRX8hBRMhAGNnHYQmyrWbH1HT7YJohYULHbthxg
```

Este é o formato final que nos permite importar para as carteiras que permitem este tipo de operação. Note este mesmo formato ssendo exportado pelo Bitcoin Core, por exemplo, com o comando ```dumpprivkey```:

```
$ bitcoin-cli getnewaddress
193AUxttHmHQLajJ1pnHMvk5d9WvbuvvFR
$ bitcoin-cli dumpprivkey "193AUxttHmHQLajJ1pnHMvk5d9WvbuvvFR"
KxBBVbkgku7f5XudmAizo51h8pfBTHDhL1u167EMgKS7PK3bnrwc
```

### Gerando a Chave Pública

Aqui é onde faremos a primeira operação exclusiva de Criptografia de Chave Pública ao usarmos o ECDSA (*Elliptic Curve Digital Signature Algorithm*) para gerarmos a nossa chave pública a partir da chave privada que criamos anteriormente. Para este tipo de operação, recomendo que utilize *libraries* como [python-ecsda](https://github.com/warner/python-ecdsa), [secp256k1-py](https://github.com/ludbb/secp256k1-py) (*binding* direto com a [secp256k1](https://github.com/bitcoin-core/secp256k1) escrita em C) ou alguma *lib* já bem utilizada em produção na sua linguagem de preferência - melhor ajudar a melhorar estas *libs* do que fazer uma nova implementação à toa e apresentar um novo risco sem benefício. Mas, para nosso fim didático deste material, vamos ver como calculamos a chave pública a partir da chave privada em Python para simplificar a visualização e entendimento.

> **Danger** O código abaixo é uma versão útil apenas como referência didática. Não recomendo que use este código em produção de forma alguma! Este código não foi testado e revisado como necessário para os padrões de segurança compatíveis com criptografia, e provavelmente, há mais razões para não fazer operações criptográficas em puro Python do que átomos no Universo observável. **VOCÊ FOI AVISADO(A)**.

Agora que você sabe que é uma péssima ideia usar o código abaixo em qualquer ambiente de produção com valores reais, podemos continuar com a geração da chave pública. Já tendo a chave privada que criamos acima na variável ```privkey``` que será utilizada ao longo deste exemplo:

```
>>> print(privkey)
b'\x9a\xfbch\x05\xc3|\xdf\xfc\xebJ\x89g\xdf\xfcn\xbe\x80%m\x91\x80DI\xd0hs6\x04\x84~\xe8'
```

Como comentado anteriormente, o Bitcoin utiliza a implementação de Curva Elíptica secp256k1 definida pela equação ```y^2 == x^3 + 7 (mod p)``` sendo ```p = 2^256 - 2^32 - 977```. A curva elíptica que estamos trabalhando pode ser graficamente representada por:

![Curva Elíptica](/images/enderecos-e-carteiras/ecc.png)

Para que calculemos a curva usada em nosso esquema de criptografia de chave pública, temos que ter o poder de duas operações: Adição e multiplicação de pontos. A utilidade deste método para o fim que pretendemos começa a ser visualizada aqui ao perceber que as únicas operações que podemos fazer com pontos são adição e multiplicação.

Para adicionarmos o Ponto A ao Ponto B de uma curva como esta, primeiro desenhamos uma linha entre os dois pontos A e B, esta linha fará uma interseção com um terceiro ponto na curva, então só precisamos refletir esta interseção passando pelo eixo x para que tenhamos o resultado da soma dos pontos A e B:

![Adição de Pontos](/images/enderecos-e-carteiras/ecc-add.png)

E para adicionarmos um mesmo ponto A a ele mesmo, nós desenhamos uma linha tangencial à curva tocando no Ponto A, esta linha terá uma interseção com a curva formando um segundo ponto, daí basta que façamos a reflexão como no primeiro caso e achamos o resultado de ```2 * Ponto A```:

![Dobro de um Ponto](/images/enderecos-e-carteiras/ecc-double.png)

Como multiplicação nada mais é do que adicionar um mesmo valor por ele *n* vezes, nós já temos o que precisamos para realizar as operações necessárias.

Para obtermos a chave pública, a operação que faremos será a multiplicação da nossa chave privada ```privkey``` por um ponto inicial definido pela implementação da curva conhecido por todos. Este ponto se chama Ponto Gerador (abreviado como G em futuras referências) e na secp256k1, ele é:

```
Gx = 55066263022277343669578718895168534326250603453777594175500187360389116729240
Gy = 32670510020758816978083085130507043184471273380659243275938904335757337482424
```

Sim, os números precisam ser gigantes para que este tipo de esquema criptográfico seja seguro. Como nota de curiosidade, o número total de chaves privadas possíveis no Bitcoin é algo em torno de 2**256; isto é um número de grandeza comparável com o número estimado de átomos no Universo observável (sim, de verdade) que está entre 10**77 e 10**82.

A nossa chave pública, então, será o ponto gerado a partir da multiplicação da chave privada por G (```privkey * (Gx, Gy)```). Assim, temos que implementar uma multiplicação escalar para multiplicarmos  a ```privkey``` como ```int``` pelo vetor G implementado como um ```set```. Vamos abstrair esta operação em quatro funções: uma para calcular a inversa modular de ```x (mod p)```, uma para caluclar o dobro de um ponto - ou seja, a soma de um ponto por ele mesmo -, mais uma para adicionar um ponto a outro ponto qualquer e uma outra função para multiplicar um ponto por valores arbitrários (no caso, a chave privada):

```
#!/usr/bin/env python3

# primeiro vamos definir o "p" da fórmula usada no bitcoin "y^2 == x^3 + 7 (mod p)"
p = 2**256 - 2**32 - 977

def inverse(x, p):
    """
    Calcula inversa modular de x (mod p)
    A inversa modular de um número é definida:
    (inverse(x, p) * x) == 1
    """
    inv1 = 1
    inv2 = 0
    while p != 1 and p != 0:
        inv1, inv2 = inv2, inv1 - inv2 * (x // p)
        x, p = p, x % p

    return inv2


def double_point(point, p):
    """
    Calcula point + point (== 2 * point)
    """
    (x, y) = point
    if y == 0:
        return None

    # Calculate 3*x^2/(2*y)  modulus p
    slope = 3*pow(x, 2, p) * inverse(2 * y, p)

    xsum = pow(slope, 2, p) - 2 * x
    ysum = slope * (x - xsum) - y
    return (xsum % p, ysum % p)


def add_point(p1, p2, p):
    """
    Calcula p1 + p2
    """
    (x1, y1) = p1
    (x2, y2) = p2
    if x1 == x2:
        return double_point(p1, p)

    # calcula (y1-y2)/(x1-x2)  modulo p
    # slope é o coeficiente angular da curva
    slope = (y1 - y2) * inverse(x1 - x2, p)
    xsum = pow(slope, 2, p) - (x1 + x2)
    ysum = slope * (x1 - xsum) - y1
    return (xsum % p, ysum % p)


def point_mul(point, a, p):
    """
    Multiplicação escalar: calcula point * a
    """
    scale = point
    acc = None
    while a:
        if a & 1:
            if acc is None:
                acc = scale
            else:
                acc = add_point(acc, scale, p)
        scale = double_point(scale, p)
        a >>= 1
    return acc
```

Agora utilizamos estas funções com os valores que temos da ```privkey``` e as coordenadas de G escritas em hexadecimal em ```int```:

```
>>> privkey = 0xe87e8404367368d0494480916d2580be6efcdf67894aebfcdf7cc3056863fb9a
>>> g_x = 0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798
>>> g_y = 0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8
>>> g = (g_x, g_y)

>>> pub_x, pub_y = point_mul(g, privkey, p)
>>> print("x: %x, y: %x" % (pub_x, pub_y))
# Com os valores acima, print deve imprimir as coordenadas da nossa chave pública na curva que é:
# 273f9c55a1c8976f87032aade62b794df31e64327386b403d7438c735b2f7c89, 9848db72f0b79646364e508d0f591d3a80541f8138f44722ada5220608f79805
```

O resultado da final da chave pública no ECDSA é simplesmente as coordenadas da chave pública (32 *bytes* para cada uma) com um *byte* ```0x04``` como sufixo. Podemos simplesmente fazer algo assim:

```
>>> pubkey = '04' + format(pub_x, 'x') + format(pub_y, 'x')
>>> print(pubkey)
04273f9c55a1c8976f87032aade62b794df31e64327386b403d7438c735b2f7c899848db72f0b79646364e508d0f591d3a80541f8138f44722ada5220608f79805
```

Agora que temos a nossa chave pública, podemos seguir com a geração do endereço Bitcoin correspondente a ela.

### Gerando o Endereço Bitcoin

Continuando com os valores que conseguimos acima para a nossa chave pública, vamos gerar um endereço Bitcoin de acordo com as regras no protocolo.

Primeiro, pegamos o valor de ```pubkey``` e aplicamos a função *hash* SHA-256 em seu *bytes*:

```
>>> import codecs
>>> import hashlib

>>> pubkey_bytes = codecs.decode(pub.encode('utf-8'), 'hex')
>>> print(pubkey_bytes)
b'\x04\'?\x9cU\xa1\xc8\x97o\x87\x03*\xad\xe6+yM\xf3\x1ed2s\x86\xb4\x03\xd7C\x8cs[/|\x89\x98H\xdbr\xf0\xb7\x96F6NP\x8d\x0fY\x1d:\x80T\x1f\x818\xf4G"\xad\xa5"\x06\x08\xf7\x98\x05'
>>> pubkey_sha256 = hashlib.sha256(pubkey_bytes)
>>> print(pubkey_sha256.hexdigest())
'd3feedd34006df75cba68a5de79228e4a6956436f46aba74332eeada8ac2ec47'
```

Segundo, aplicamos a função *hash* RIPEMD-160 no resultado ```pubkey_sha256```:

```
>>> pubkey_ripe = hashlib.new('ripemd160')
>>> pubkey_ripe.update(pubkey_sha256.digest())
>>> print(pubkey_ripe.hexdigest())
'0fc8aa8b93103a388fd562514ec250be2d403a27'
```
Terceiro, adicionamos o byte ```0x00``` para identificar a chave para a *mainet*. De forma simples, podemos:

```
>>> raw_address = '00' + pubkey_ripe.hexdigest()
>>> print(raw_address)
000fc8aa8b93103a388fd562514ec250be2d403a27
```

E este resultado é o endereço Bitcoin. Guarde-o, pois já usaremos ele novamente. Mais uma vez não se parece com os endereços que costumamos ver e escrever. Isto ocorre porque este é o endereço puro em hexadecimal sem Base58Check. Para termos um endereço com proteção contra erros de digitação como a maioria dos endereços que lidamos no dia-a-dia, seguimos os seguintes passos.

Primeiro vamos pegar 4 *bytes* para usar como *checksum* fazendo o *shasha* nos bytes do endereço. Como dito acima costumamos chamar de *shasha* quando tiramos um *hash* SHA-256 e passamos este resultado novamente na função *hash* SHA256. Para isso, fazemos:

```
>>> raw_addr_bytes = codecs.decode(raw_address.encode('utf-8'), 'hex')
>>> print(raw_addr_bytes)
b"\x00\x0f\xc8\xaa\x8b\x93\x10:8\x8f\xd5bQN\xc2P\xbe-@:'"
>>> addr_shasha = hashlib.sha256(hashlib.sha256(raw_addr_bytes).digest())
>>> print(addr_shasha.hexdigest())
455a7335c8c121fbb90e22e0dbb77ff3d7137908a052b895d6933f53032b2d27
# E pegamos os 4 primeiros bytes como checksum
>>> checksum = addr_shasha.digest()[:4]
>>> print(checksum)
b'EZs5'
# Em hexadecimal...
>>> checksum = addr_shasha.hexdigest()[:8]
>>> print(checksum)
'455a7335'
```

Agora adicionamos o *checksum* ao final do endereço ```000fc8aa8b93103a388fd562514ec250be2d403a27``` que pegamos um pouco mais acima, e finalmente, passamos os *bytes* pela função para codificar em Base58:

```
>>> from base58 import b58encode
>>> address = raw_address + checksum
>>> addr_bytes = codecs.decode(address.encode('utf-8'), 'hex')
>>> address = b58encode(addr_bytes)
>>> print(address)
12STXQicaWRh4RFfjW6T6y6ZAqQVytTKXa
```

E este é o endereço que podemos usar para receber transações nos *softwares* de carteira controlado pela chave privada criada no início.

> **Note** Repare que com o uso de 4 *bytes* como *checksum* junto om o Base58Check, nós temos um endereço com proteção contra erros de digitação contanto que o *software* de carteira o implemente corretamente. O Base58Check serve para que os endereços gerados sejam de mais facilidade visual e o *checksum* serve para garantir que o endereço realmente foi digitado corretamente antes de enviar uma transação. Sem esta proteção, a chance de erros com perdas financeiras seria muito maior para os usuários.

## Carteiras

Carteiras ou *wallets* são *softwares* ou arquivos - geralmente, refere-se a *softwares* - que contém chaves privadas e/ou disponibilizam funcionalidades para administração destas chaves, como: *backup*, envio de transações, monitoramento de recebimento de transações, *UTXOs*, geração de endereços, etc. As carteiras podem guardar outras informações sobre as transações para melhor usabilidade como anotações sobre as transações, e outras utilidades como assinatura de textos ou arquivos com suas chaves privadas.

O *software* de carteira mais simples costuma fazer, pelo menos, as operações: gerar chaves privadas, derivar chaves públicas correspondentes, monitorar *UTXOs* para para estas chave, e criar e assinar transações. Existem outras carteiras com menos ou mais funcionalidades de acordo com o caso de uso (exemplo: uma empresa pode utilizar uma carteira com o código enxuto para ter uma menor superfície de contato em um computador desconectado da Internet com a única funcionalidade de proteger as chaves privadas e assinar transações para transmitir estas transações por outra máquina seguindo um protocolo de segurança mais rigoroso). No entanto, a diferença fundamental nas carteiras mais utilizadas está no esquema de geração de chaves que elas utilizam. Abaixo podemos ver os tipos de carteiras mais comuns e um pouco do contexto técnico que justifica a utilidade de cada tipo. Vale notar que os tipos listados abaixo não são necessariamente exclusivos entre si.

### Tipos de Carteiras: Método de Segurança das Chaves

**Hot Wallets:** são todas as carteiras que, em algum momento, tem as suas chaves privadas expostas em um ambiente com conexão disponível à Internet - especialmente, a Internet. Estas são, provavelmente, são as carteiras mais usadas e que apresentam o menor nível de segurança para o usuário. Pelo menos, toda vez em que uma nova transação precisa ser assinada, o usuário digita uma senha ou PIN para desencriptar a chave privada e assinar a transação em um ambiente com conexão à Internet, fazendo com que a chave fique exposta mesmo que por alguns momentos.

**Cold Wallets:** são todas as carteiras rodando em um ambiente sem conexão à Internet. Outro nome comum para definir este esquema é *Cold Storage* e a chave privada NUNCA é exposta em um computador com acesso à internet. Normalmente são criadas e usadas para acumular bitcoins e, quando usadas, descartadas e trocadas por novas chaves com menos chance de terem sido comprometidas. Também são usadas como carteira em ambiente totalmente *offline* apenas para assinar transações e ter as transações levadas de alguma outra forma ao conhecimento da rede bitcoin. O ideal para a segurança é que as chaves sejam sempre renovadas quando uma transação for feita.

**Paper Wallets:** podem ser consideradas um tipo de *cold storage*, já que são carteiras com as chaves privadas impressas em papel para proteção física da chave privada. A efetividade da segurança deste método depende de como as chaves foram geradas, em que ambiente elas foram geradas (ex.: um computador sem conexão) e como elas são armazenadas com segurança contra roubo ou destruição. Um método interessante para este tipo de carteira é o M-de-N que permite que você tenha um número N de papéis e precise apenas de M destes papéis para gastar seus bitcoins. Assim, se você tiver uma 3-de-5 que é um esquema bem comum, você pode ter 5 papéis separados fisicamente em locais seguros e precisará de 3 destas partes para gastar os bitcoins; Desta forma além da proteção das chaves não estarem conectadas, poderá existir a proteção contra um único ponto de falha já que, caso um papel seja destruído ou roubado, os bitcoins ainda estarão seguros para serem enviados para uma nova carteira segura.

**Hardware Wallets:** também podem ser consideradas um tipo de *cold storage*. O interessante desta carteira é a possibilidade de ter um nível de segurança razoável em comparação aos outros métodos ao mesmo tempo que se pode ter uma carteira boa para ser utilizada com mais frequência no dia-a-dia. Técnicamente, este tipo de carteira se vale de uma separação de *hardware*, sendo um dispositivo seguro, desconectado de qualquer rede e que não confia no computador a que ela é conectada. Tudo que ela faz é assinar transações com as chaves privadas armazenadas nela e verificar a validade de endereços de recebimento para proteção contra certos tipos de ataque *man-in-the-middle*. Normalmente, os passos para assinatura de uma transação são como seguem: O usuário, com a carteira conectada ao computador, cria uma transação no *software* de carteira, o *software* compatível com a *hardware wallet* envia esta transação para ser assinada pela *hardware wallet*, o usuário confirma a transação na própria *hardware wallet* em um visor separado (ou por algum outro esquema como cartões PIN) e confirma, logo em seguida tudo que sai da *hardware wallet* é a transação assinada e nada mais para que o *software* de carteira possa transmitir esta transação à rede.

### Tipos de Carteira: Método de Geração de Chaves

**Single-Address Wallets:** são carteiras que utilizam um único endereço para recebimento de transações e trocos, envio, etc. Não são um bom tipo de carteira para privacidade e segurança, e são cada vez menos utilizadas substituídas por métodos superiores para a maioria dos casos de uso. 

**Nondeterministic (Random) Address Wallets:** carteiras não-determinísticas tem um armazenamento de tamanho fixo de endereços gerados aleatoriamente. As chaves privadas são geradas de forma aleatória, tem seus endereços correspondentes gerados e ficam armazenadas. Este esquema tem alguns problemas para a segurança dos bitcoins caso *backups* não sejam feitos regularmente. Para entender o problema, digamos que você tenha uma *pool* de no máximo 100 endereços na sua carteira, você faz um *backup* da carteira antes de começar a utilizar ela, então, depois de um tempo utilizando com centenas ou milhares de transações, você tem o seu disco rígido destruído perdendo o acesso a carteira. Ótimo, você pega o seu *backup* e restaura em um novo computador... mas, de repente, você nota que está sem parte ou todos os seus bitcoins e, sim, você os perdeu. O que aconteceu é que quando você fez o backup da primeira vez, o seu backup tinha guardado as 100 primeiras chaves privadas da *pool* da sua carteira, mas, com o tempo, você foi utilizando e criando novos endereços, recebendo em outros e ec, e estas chaves estavam sendo armazenadas em seu computador danificado, mas não no *backup*. Neste tipo de carteira, você gera as chaves de forma aleatória e as armazena, você não tem como saber quais chaves foram geradas no computador caso não tenha armazenado as novas chaves no *backup*.

**Deterministic Address Wallets:** carteiras determinísticas geram chaves privadas derivadas a partir de uma *seed* (semente) em comum. Para a recuperação de seus bitcoin neste tipo de carteira, você só precisa estar em posse de sua *seed* inicialmente gerada. Neste tipo de carteira, diferente das não-determinísticas, há a garantia de que todas as chaves privadas serão geradas na mesma ordem não importando o dispositivo em que ela estiver rodando. Isto facilita a segurança e usabilidade já que não é mais necessário repetir o processo de *backup* da carteira original de tempos em tempos.

**Hierachical Deterministic (HD) Wallets**: carteiras hierárquicas determinísticas como definidas pelas [BIP0032](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) e [BIP0044](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) são um tipo avançado de carteira determinística em que as chaves também são geradas a partir de uma *seed* e todas são geradas em uma ordem não aleatória. A diferença aparece na *feature* que este tipo de carteira apresenta ao gerar suas chaves formando uma estrutura de árvore com cada folha na árvore tendo a possibilidade de gerar as chaves filhas e não as acima delas. Isto possibilita uma organização estrutural superior da carteira com facilidades de organização, divisão de carteiras e, inclusive, a possibilidade de ter uma carteira dividida por suas folhas por pessoas de uma empresa de acordo com a estrutura organizacional sem comprometer as chaves privadas mestras. Para um melhor entendimento, veja esta imagem mostrando da *seed* até as folhas filhas:

![Chaves Hierárquica Determinísticas](/images/enderecos-e-carteiras/hd-wallet.png)

**Próximo capítulo:** [Transações](transacoes.md)
