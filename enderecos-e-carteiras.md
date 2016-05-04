# Endereços e Carteiras

Como visto em [A Criptografia no Bitcoin](intro.md#a-criptografia-no-bitcoin), a  criptografia de chave pública é essencial para resolver a questão de propriedade em sistemas computacionais como no caso do Bitcoin junto com o apoio criptográfico das funções *hash* criptográficas. O que antes eram apenas 0's e 1's indistinguíveis ganham algumas propriedades como escasses, autenticidade e integridade. E isto é a base para que se entenda o que os endereços representam e como eles são tratados pela rede.

## Criando Endereços

Cada endereço é um identificador único na rede; não de uma pessoa como *e-mails*, mas de uma chave pública. Para podermos criar um endereço sob nosso controle, primeiro temos que gerar uma chave privada de onde calcularemos a chave pública que será identificada como um endereço.

### Geração da Chave Privada

A chave privada é um número gerado de forma aleatória. E, além da aleatoriedade na geração deste número, outra qualidade importante para a segurança na implementação deste tipo de criptografia é que a chave privada é um número gigante. Com esta chave, as transações serão assinadas e valores serão transferidos na rede; gerar e manter esta chave de forma segura é **essencial** para a segurança de valores em bitcoin.

Para a geração segura de uma chave privada, primeiro precisamos de uma fonte segura de entropia para geração de uma chave de 256 *bits*. Normalmente, os *softwares* de carteira utilizam uma fonte de coleta de entropia do próprio sistema operacional - como ```/dev/urandom``` em sistemas UNIX ou ```cryptGenRandom()``` no Windows - e isto costuma ser o suficiente, mas para o nível máximo de paranoia é possível que utilizemos métodos físicos que excluem a possibilidade de falhas ou *backdoors* de *software*/*hardware* como o *diceware* descrito especialmente para o Bitcoin [aqui](#toDo-link). No mínimo, recomendo que gere a chave privada pelo método *diceware* descrito no link anterior pelo menos uma vez na vida devido à experiência que te ajudará a entender ou fortalecer na prática algumas coisas a mais sobre entropia.

<span style="color: #a94442;">**ATENÇÃO!**</span> Como programador(a) é a sua responsabilidade conhecer e entender a ferramenta que você estiver utilizando para a geração de números aleatórios criptograficamente seguros (*cryptographically secure pseudo-random number generator* - CSPRNG). Não use uma técnica própria desconhecida ao resto do mundo mesmo que você tenha a capacidade de cria uma técnica razoavelmente segura - parte fundamental na segurança de um algoritmo deste tipo é o fato de ele ter sido revisado por muitos especialistas e utilizado em "batalha" por muitos anos.
