# Endereços e Carteiras

Como visto em [A Criptografia no Bitcoin](intro.md#a-criptografia-no-bitcoin), a  criptografia de chave pública é essencial para resolver a questão de propriedade em sistemas computacionais como no caso do Bitcoin junto com o apoio criptográfico das funções *hash* criptográficas. O que antes eram apenas 0's e 1's indistinguíveis ganham algumas propriedades como escasses, autenticidade e integridade. E isto é a base para que se entenda o que os endereços representam e como eles são tratados pela rede.

## Criando Endereços

Cada endereço é um identificador único na rede; não de uma pessoa como *e-mails*, mas de uma chave pública. Para podermos criar um endereço sob nosso controle, primeiro temos que gerar uma chave privada de onde calcularemos a chave pública que será identificada como um endereço.

### Geração da Chave Privada

A chave privada é um número gerado de forma aleatória. E, além da aleatoriedade na geração deste número, outra qualidade importante para a segurança na implementação deste tipo de criptografia é que a chave privada é um número gigante. Com esta chave, as transações serão assinadas e valores serão transferidos na rede; gerar e manter esta chave de forma segura é **essencial** para a segurança de valores em bitcoin.

Para a geração segura de uma chave privada, primeiro precisamos de uma fonte segura de entropia para geração de uma chave de 256 *bits*. Normalmente, os *softwares* de carteira utilizam uma fonte de coleta de entropia do próprio sistema operacional - como ```/dev/urandom``` em sistemas UNIX ou ```cryptGenRandom()``` no Windows - e isto costuma ser o suficiente, mas para o nível máximo de paranoia é possível que utilizemos métodos físicos que excluem a possibilidade de falhas ou *backdoors* de *software*/*hardware* como o *diceware* descrito especialmente para o Bitcoin [aqui](#toDo-link). No mínimo, recomendo que gere a chave privada pelo método *diceware* descrito no link anterior pelo menos uma vez na vida devido à experiência que te ajudará a entender ou fortalecer na prática algumas coisas a mais sobre entropia.

<span style="color: #a94442;">**ATENÇÃO!**</span> Como programador(a) é a sua responsabilidade conhecer e entender a ferramenta que você estiver utilizando para a geração de números aleatórios criptograficamente seguros (*cryptographically secure pseudo-random number generator* - CSPRNG). Não use uma técnica própria desconhecida ao resto do mundo mesmo que você tenha a capacidade de cria uma técnica razoavelmente segura - parte fundamental na segurança de um algoritmo deste tipo é o fato de ele ter sido revisado por muitos especialistas e utilizado em "batalha" ao longo de muitos anos. Não confie nem mesmo nas ferramentas que eu apontar sem que você entenda porque elas são seguras e/ou que você possa verificar ampla confiança consolidada de outros especialistas nestas ferramentas. Dito isto, voltemos ao material...

Em Python, podemos utilizar o ```os.urandom``` que coleta *bytes* a partir do ```/dev/urandom``` em sistema UNIX e ```cryptGenRandom()``` no Windows para geração da chave privada em *bytes*:

```
import os

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

**WIF**: este é o formato que costumamos ler e é abreviação para *Wallet Import Format* (Formato de Importação de Carteira). Este formato é usado para facilitar a leitura e cópia das chaves por humanos. O processo é simples: pegamos a chave privada, concatenamos o *byte* ```0x80``` para ```mainet``` ou ```0xef``` para a ```testnet``` como prefixo e o *byte* ```0x01``` como sufixo se a chave privada for corresponder a uma chave pública comprimida - uma forma de representar chaves públicas usando menos espaço -, realizamos uma função *hash* SHA-256 duas vezes seguidas - comumente referido como *shasha* -, pegamos os 4 primeiros *bytes* do resultado - este é o *checksum* -, adicionamos estes 4 *bytes* ao final do resultado da chave pública antes do *shasha* e, então, usamos a funcão *Base58Check* para codificar o resultado final.

**Base58Check**: é uma versão modificada da função de Base58 utilizada no Bitcoin para codificar chaves e endereços na rede para produzir um formato que facilita a digitação por humanos, assim como diminui drasticamente a chance de erros na digitação.

Então, vamos pegar a chave privada que criamos e transformar ela para o formato WIF. Ao finalizarmos, como teste do resultado, pegue a chave no formato WIF e importe ela em algum *software* de carteira e verá que a sua carteira criará endereços a partir desta chave privada normalmente. Aqui está uma forma como podemos transformar a chave privada que obtivemos acima para o formato WIF:

```
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

### Geração da Chave Pública

Aqui é onde faremos a primeira operação exclusiva de Criptografia de Chave Pública ao usarmos o ECDSA (*Elliptic Curve Digital Signature Algorithm*) para gerarmos a nossa chave pública a partir da chave privada que criamos anteriormente. Para este tipo de operação, recomendo que utilize *libraries* como [python-ecsda](https://github.com/warner/python-ecdsa), [secp256k1-py](https://github.com/ludbb/secp256k1-py) (*binding* direto com a [secp256k1](https://github.com/bitcoin-core/secp256k1) escrita em C) ou alguma *lib* já bem utilizada em produção na sua linguagem de preferência - melhor ajudar a melhorar estas *libs* do que fazer uma nova implementação à toa e apresentar um novo risco sem benefício. Mas, para nosso fim didático deste material, vamos ver como calculamos a chave pública a partir da chave privada em Python para simplificar a visualização e entendimento.

<span style="color: #a94442;">**ATENÇÃO!**</span> O código abaixo é uma versão útil apenas como referência didática. Não recomendo que use este código em produção de forma alguma! Este código não foi testado e revisado como necessário para os padrões de segurança compatíveis com criptografia, e, provavelmente, há mais razões para não fazer cálculos criptográficos em puro Python do que átomos no Universo observável. **VOCÊ FOI AVISADO(A)**.

Agora que você sabe que é uma péssima ideia usar o código abaixo em qualquer ambiente de produção com valores reais, podemos continuar com a geração da chave pública. Já tendo a chave privada que criamos acima na variável ```privkey```...

```
>>> print(privkey)
b'\x9a\xfbch\x05\xc3|\xdf\xfc\xebJ\x89g\xdf\xfcn\xbe\x80%m\x91\x80DI\xd0hs6\x04\x84~\xe8'
```

Comecemos a dissecar os cálculos necessários o algoritmo e implementar passo-a-passo.
