# Endereços e Carteiras

Como visto em [A Criptografia no Bitcoin](intro.md#a-criptografia-no-bitcoin), a  criptografia de chave pública é essencial para resolver a questão de propriedade em sistemas computacionais como no caso do Bitcoin junto com o apoio criptográfico das funções *hash* criptográficas. O que antes eram apenas 0's e 1's indistinguíveis ganham algumas propriedades como escasses, autenticidade e integridade. E isto é a base para que se entenda o que os endereços representam e como eles são tratados pela rede.

## Criando Endereços

Cada endereço é um identificador único na rede; não de uma pessoa como *e-mails*, mas de uma chave pública. Para podermos criar um endereço sob nosso controle, primeiro temos que gerar uma chave privada de onde calcularemos a chave pública que será identificada como um endereço.

### Geração da Chave Privada

A chave privada é um número gerado de forma aleatória. E, além da aleatoriedade na geração deste número, outra qualidade importante para a segurança na implementação deste tipo de criptografia é que a chave privada é um número gigante. Com esta chave, as transações serão assinadas e valores serão transferidos na rede; gerar e manter esta chave de forma segura é **essencial** para a segurança de valores em bitcoin.

Para a geração segura de uma chave privada, primeiro precisamos de uma fonte segura de entropia para geração de uma chave de 256 *bits*. Normalmente, os *softwares* de carteira utilizam uma fonte de coleta de entropia do próprio sistema operacional - como ```/dev/urandom``` em sistemas UNIX ou ```cryptGenRandom()``` no Windows - e isto costuma ser o suficiente, mas para o nível máximo de paranoia é possível que utilizemos métodos físicos que excluem a possibilidade de falhas ou *backdoors* de *software*/*hardware* como o *diceware* descrito especialmente para o Bitcoin [aqui](#toDo-link). No mínimo, recomendo que gere a chave privada pelo método *diceware* descrito no link anterior pelo menos uma vez na vida devido à experiência que te ajudará a entender ou fortalecer na prática algumas coisas a mais sobre entropia.

<span style="color: #a94442;">**ATENÇÃO!**</span> Como programador(a) é a sua responsabilidade conhecer e entender a ferramenta que você estiver utilizando para a geração de números aleatórios criptograficamente seguros (*cryptographically secure pseudo-random number generator* - CSPRNG). Não use uma técnica própria desconhecida ao resto do mundo mesmo que você tenha a capacidade de cria uma técnica razoavelmente segura - parte fundamental na segurança de um algoritmo deste tipo é o fato de ele ter sido revisado por muitos especialistas e utilizado em "batalha" ao longo de muitos anos. Não confie nem mesmo nas ferramentas que eu apontar sem que você entenda porque elas são seguras e/ou que você possa verificar ampla confiança consolidada de outros especialistas nestas ferramentas. Dito isto, voltemos ao material...

Em Python, podemos utilizar o ```os.urandom``` que coleta bytes a partir do ```/dev/urandom``` em sistema UNIX e ```cryptGenRandom()``` no Windows para geração da chave privada em bytes:

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
>>> privkey_hex = hex(privkey_int)
>>> print(privkey_hex)
0xe87e8404367368d0494480916d2580be6efcdf67894aebfcdf7cc3056863fb9a
```

Agora temos 32 *bytes* (ou 256 *bits*) coletados. Certamente nenhum dos formatos mostrados parece com o que costumamos ler ao pedir que uma carteira exporte a chave privada para nós. Isso ocorre porque diferentemente de quando estamos passando informação pelos cabos, utilizamos um formato especial chamado WIF.

**WIF**: este é o formato que costumamos ler e é abreviação para *Wallet Import Format* (Formato de Importação de Carteira).
