# Mineração

A mineração é o processo responsável por atualizar a blockchain e, até atingir o limite de cerca de 21 milhões *satoshis*, trazer novas moedas à rede por meio de uma competição de processamento intenso com o intuito de alcançar um *hash* de um bloco com transações válidas menor ou igual ao resultado esperado pelo resto da rede. Esta competição propositalmente pesada para os recursos computacionais produz o *proof-of-work* - ou prova de trabalho - essencial para a segurança do consenso na rede. O algoritmo usado no Bitcoin é o *hashcash* criado em 1997 por Adam Back.

## O Propósito da Mineração

### Segurança

Em [Blockchain](blockchain.md) podemos ver que cada bloco é diretamente ligado ao anterior pelo campo que contém o *hash* do cabeçalho do bloco anterior, servindo de prova de integridade da blockchain. A mineração, por sua vez, adiciona o toque final à segurança da blockchain trazendo a necessidade de um esforço computacional mínimo para a adição de cada novo bloco e, assim, fazendo com que o esforço de qualquer tentativa de mudança aos blocos da blockchain cresça exponencialmente a cada bloco anterior que se tentar realizar a mudança; ao mesmo tempo em que a rede continua trabalhando adicionando novos blocos com mais *proof-of-work* em cada um deles, fazendo a blockchain cada vez mais segura e praticavelmente imutável. Graças ao trabalho da mineração, se eu quiser alterar um bloco de altura 410500, eu terei que provar para a rede o trabalho computacional de todos os blocos subsequentes a este desde o bloco mais alto aceito pela rede; No momento, isso é simplesmente impossível em meu tempo de vida, pois dependeria de um trabalho computacional que, mesmo com muito processamento disponível, levaria um tempo fora de minha compreensão humana de espaço-tempo.

### Novas Moedas

Outro propósito da mineração está em um dos incentivos oferecido aos mineradores como recompensa pelo esforço computacional gasto na segurança da blockchain contra alterações maliciosas. Junto com as taxas de mineração recebidas de todas as transações incluidas no bloco, o minerador também recebe uma recompensa que tem a dupla função de servir de subsídio ao trabalho despendido e de trazer novas moedas à existência na rede. Este subsídio, atualmente, é de 25 bitcoins e, em breve diminuirá para 12.5 bitcoins; Esta diminuição no subsídio da rede é conhecida como *halving* e acontece a cada 210.000 blocos - aproximadamente 4 anos -, quando esta recompensa é cortada pela metade.

Todo minerador que escreve um novo bloco na blockchain ganha o direito de criar uma transação chamada *coinbase* que é uma exceção por não ter *inputs* e ter apenas o *output* com a recompensa atual da rede para o endereço escolhido pelo minerador. Esta geração de moedas a partir das transações *coinbase* apresenta um crescimento logarítimico no número total de moedas circulando na rede ao longo do tempo:

![oferta de bitcoins](/images/mineracao/bitcoin-supply.png)

O número total de bitcoins que serão criados na rede é aproximadamente 21 milhões de bitcoins; precisamente 2099999997690000 *satoshis*. Podemos visualizar este crescimento com um pequeno *script* em Python3:

```
#!/usr/bin/env python3
# Ano em que a rede foi iniciada por Satoshi Nakamoto
START_YEAR = 2009
# Intervalo de anos com blocos de 10 minutos
YEAR_INTERVAL = 4
# A recompensa inical de 50 bitcoins em satoshis
START_BLOCK_REWARD = 50 * 10**8
# Intervalo de blocos entre o halving da recompensa
REWARD_INTERVAL = 210 * 10**3


def show_mine_progress():
    curr_year = START_YEAR
    curr_reward = START_BLOCK_REWARD
    total_coins = 0
    while curr_reward > 0:
        print("Ano: %d, Recompensa atual: %d, Moedas em circulação: %d satoshis" %
              (curr_year, curr_reward, total_coins))
        # O número de moedas é somado com todas as recompensas de 210000 blocos
        # gerados em 4 anos
        total_coins += curr_reward * REWARD_INTERVAL
        curr_year += YEAR_INTERVAL
        # empurra os bits para a direita uma vez, efetivamente dividindo
        # a recompensa pela metade.
        # Ex.: 4 em binário (100) com os bits empurrados para a
        # direita será 2 (010)
        curr_reward >>= 1
    print("\nTotal de moedas em circulação: %d satoshis" % total_coins)
```

O *output* desta função será:

```
>>> show_mine_progress()
Ano: 2009, Recompensa atual: 5000000000, Moedas em circulação: 0 satoshis
Ano: 2013, Recompensa atual: 2500000000, Moedas em circulação: 1050000000000000 satoshis
Ano: 2017, Recompensa atual: 1250000000, Moedas em circulação: 1575000000000000 satoshis
Ano: 2021, Recompensa atual: 625000000, Moedas em circulação: 1837500000000000 satoshis
Ano: 2025, Recompensa atual: 312500000, Moedas em circulação: 1968750000000000 satoshis
Ano: 2029, Recompensa atual: 156250000, Moedas em circulação: 2034375000000000 satoshis
Ano: 2033, Recompensa atual: 78125000, Moedas em circulação: 2067187500000000 satoshis
Ano: 2037, Recompensa atual: 39062500, Moedas em circulação: 2083593750000000 satoshis
Ano: 2041, Recompensa atual: 19531250, Moedas em circulação: 2091796875000000 satoshis
Ano: 2045, Recompensa atual: 9765625, Moedas em circulação: 2095898437500000 satoshis
Ano: 2049, Recompensa atual: 4882812, Moedas em circulação: 2097949218750000 satoshis
Ano: 2053, Recompensa atual: 2441406, Moedas em circulação: 2098974609270000 satoshis
Ano: 2057, Recompensa atual: 1220703, Moedas em circulação: 2099487304530000 satoshis
Ano: 2061, Recompensa atual: 610351, Moedas em circulação: 2099743652160000 satoshis
Ano: 2065, Recompensa atual: 305175, Moedas em circulação: 2099871825870000 satoshis
Ano: 2069, Recompensa atual: 152587, Moedas em circulação: 2099935912620000 satoshis
Ano: 2073, Recompensa atual: 76293, Moedas em circulação: 2099967955890000 satoshis
Ano: 2077, Recompensa atual: 38146, Moedas em circulação: 2099983977420000 satoshis
Ano: 2081, Recompensa atual: 19073, Moedas em circulação: 2099991988080000 satoshis
Ano: 2085, Recompensa atual: 9536, Moedas em circulação: 2099995993410000 satoshis
Ano: 2089, Recompensa atual: 4768, Moedas em circulação: 2099997995970000 satoshis
Ano: 2093, Recompensa atual: 2384, Moedas em circulação: 2099998997250000 satoshis
Ano: 2097, Recompensa atual: 1192, Moedas em circulação: 2099999497890000 satoshis
Ano: 2101, Recompensa atual: 596, Moedas em circulação: 2099999748210000 satoshis
Ano: 2105, Recompensa atual: 298, Moedas em circulação: 2099999873370000 satoshis
Ano: 2109, Recompensa atual: 149, Moedas em circulação: 2099999935950000 satoshis
Ano: 2113, Recompensa atual: 74, Moedas em circulação: 2099999967240000 satoshis
Ano: 2117, Recompensa atual: 37, Moedas em circulação: 2099999982780000 satoshis
Ano: 2121, Recompensa atual: 18, Moedas em circulação: 2099999990550000 satoshis
Ano: 2125, Recompensa atual: 9, Moedas em circulação: 2099999994330000 satoshis
Ano: 2129, Recompensa atual: 4, Moedas em circulação: 2099999996220000 satoshis
Ano: 2133, Recompensa atual: 2, Moedas em circulação: 2099999997060000 satoshis
Ano: 2137, Recompensa atual: 1, Moedas em circulação: 2099999997480000 satoshis

Total de moedas em circulação: 2099999997690000 satoshis
```

E assim fica fácil observar que o número de bitcoins que existirão na rede será aproximadamente 20 milhões. A transação *coinbase* tem a sua recompensa calculada no [arquivo miner.cpp linha 279](https://github.com/bitcoin/bitcoin/blob/86b800c6a299455580fe76e5fb43218f0222e179/src/miner.cpp#L279) no Bitcoin Core e é verificada pelos nós como visto no [arquivo main.cpp linha 2393](https://github.com/bitcoin/bitcoin/blob/f7a21dae5dbf71d5bc00485215e84e6f2b309d0a/src/main.cpp#L2393); ambos utilizando as funções ```GetBlockSubsidy``` com a ```chainparams.GetConsensus``` como parametro.

## Como funciona

A rede tem um parametro chamado ```difficulty``` que indica o *hash* do bloco esperado pela rede para que o minerador seja autorizado a atualizar a blockchain. Esta dificuldade é ajustada, se necessário, a cada 2016 blocos com o objetivo de manter um alvo que necessite de 10 minutos **em média** para ser alcançado por algum minerador via *proof-of-work*. Você pode verificar a dificuldade atual pela API JSON-RPC do Bitcoin Core com o comando ```getdifficulty```:

```
$ bitcoin-cli getdifficulty
194254820283.444
```

A dificuldade é uma medida do quão difícil é para achar um *hash* abaixo de um certo alvo. O alvo inicial da rede é ```0x00000000FFFF0000000000000000000000000000000000000000000000000000``` (em *float*, truncado) e é o máximo alvo possível no Bitcoin que representa a mínima dificuldade possível ```1``` ajustada, desde então, a cada 2016 blocos. O alvo esperado pela rede representa o número mínimo de 0's que o *hash* do próximo bloco deve ter para que seja aceito na atualização da blockchain, ou seja, o minerador deve conseguir criar um *hash* que represente um número menor ou igual ao alvo atual. Este mecanismo no *hashcash* serve como prova de computação devido ao fato de que quantos mais 0's um *hash* tem em seu início, mais trabalho computacional deve ser despendido para que se consiga achar este *hash* e este trabalho computacional é previsível. Para calcular a notação do número retornado pelo comando ```getdifficulty```, usamos esta fórmula como base:

```
dificuldade_atual = dificuldade_inicial / alvo_atual
```

No Bitcoin Core, a você pode ver o cálculo da dificuldade no [arquivo main.cpp linha 3304](https://github.com/bitcoin/bitcoin/blob/master/src/main.cpp#L3304) com o uso da função ```GetNextWorkRequired``` definida no [arquivo pow.cpp linha 13](https://github.com/bitcoin/bitcoin/blob/20f9ecd343bbd305f0aeb829f42e61edea8de62f/src/pow.cpp#L13).

Para conseguir criar *hashes* diferentes com o mesmo bloco, os mineradores podem alterar o campo *nonce* arbitrariamente para criarem *hashes* completamente diferentes do mesmo bloco. Então, para um minerador provar que alcançou o resultado esperado pela rede basta que apresente o bloco com todos seus elementos incluindo este *nonce* para que qualquer um possa verificar a validade deste *hash* e da validade das transações contidas naquele bloco. No entanto, um valor de 32 *bytes* com suas 4 bilhões de possibilidades diferentes já não é suficiente para um *proof-of-work* que desde 2011 já necessita de mais de 1 quadrilhão de *hashes* para ser resolvido por algum minerador, logo, os mineradores passaram a usar otimizações ao algoritmo de *proof-of-work* original para alterarem o *timestamp* em alguns segundos, a própria transação *coinbase* e a ordem ou composição da lista de transações. 

Agora, para facilitar a nossa visualização do trabalho computacional despendido no *proof-of-work*, podemos implementar um script em Python3 para minerar um bloco razoavelmente fácil para uma CPU comum, o bloco 1 após logo após o bloco gênesis. Não utilizarei o próprio bloco gênesis para o exemplo para podermos ver um bloco comum como outros, já que o gênesis é um caso especial e não tem referência ao bloco anterior. Este exemplo é bastante ineficiente para a tarefa e é muito mais útil para entendermos melhor o processo por meio de visualização, no entanto ele pode ser alterado para outros blocos como quiser:

```
#!/usr/bin/env python3
import hashlib
import struct
import codecs
import time

# versao para o bloco
version = 1
# o bloco anteiror (aqui, o bloco genesis)
prev_block = "000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f"
# a raiz de merkle formada das transacoes... nao estamos formando a nossa;
# apenas pegamos a formada no bloco original
merkle_root = "0e3e2357e806b6cdb1f70b54c3a3a17b6714ee1f0e68bebb44a74b1efd512098"
start_time = int(time.time())
bits = 486604799
p = ''

# calculando a string do alvo para checarmos
exp = bits >> 24
mant = bits & 0xffffff
target = mant * (1 << (8 * (exp - 3)))
target_hexstr = '%064x' % target
target_str = codecs.decode(target_hexstr, 'hex')
nonce = 100000000

# apenas printando algumas informacoes sobre o bloco, bloco anterior e alvo
print("Hash do bloco anterior:", prev_block)
print("Raiz de Merkle:", merkle_root)
print("Alvo atual (hex):", target_hexstr)
print()

print("Iniciando mineração...")
while True:
    nonce += 1
    # este e o cabecalho do bloco
    header = (struct.pack('<L', version) +
              codecs.decode(prev_block, 'hex')[::-1] +
              codecs.decode(merkle_root, 'hex')[::-1] +
              struct.pack('<LLL', start_time, bits, nonce))

    # passando o cabecalho na shasha; shasha = sha256(sha256().digest())
    blockhash = hashlib.sha256(hashlib.sha256(header).digest()).digest()
    blockhash_hex = codecs.encode(blockhash[::-1], 'hex')

    # printando a cada hash com um 0 a mais conseguido
    if blockhash_hex.startswith(p.encode('utf-8')):
        print('\nnonce:', nonce,
              '\nblockhash (hex):', blockhash_hex.decode('utf-8'),
              '\ntempo corrido: %.2f segundos' % (time.time() - start_time),
              '\nnumero de zeros no inicio do hash:', len(p))
        p += '0'

    # se o hash do bloco for menor ou igual ao target, finalizamos
    if blockhash[::-1] <= target_str:
        print("Sucesso!")
        print("Bloco minerado em %d segundos." % (time.time() - start_time))
        break
```

No *output* deste último programa, podemos ver a progressão do esforço computacional até conseguirmos minerar o bloco... o que levou algum tempo:

```
Hash do bloco anterior: 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
Raiz de Merkle: 0e3e2357e806b6cdb1f70b54c3a3a17b6714ee1f0e68bebb44a74b1efd512098
Alvo atual (hex): 00000000ffff0000000000000000000000000000000000000000000000000000

Iniciando mineração...

nonce: 100000001 
blockhash (hex): 771be0368aaaa3fc1521f4d6db2fd1dfc6e9ef0952fd4bd822ad3b4610f7aebe 
tempo corrido: 0.34 segundos 
numero de zeros no inicio do hash: 0

nonce: 100000003 
blockhash (hex): 0fa9339e51cefb79cc6c61d602cbf549fbe29e8bf17427d840277f2b5037769a 
tempo corrido: 0.34 segundos 
numero de zeros no inicio do hash: 1

nonce: 100000230 
blockhash (hex): 00b911eccc58ef739f7e87cc45fcd445b746ca95f4eec0f3b4ea7854bcdd850e 
tempo corrido: 0.34 segundos 
numero de zeros no inicio do hash: 2

nonce: 100001054 
blockhash (hex): 000046253349900d528f82b17365378925e80e08e4404ce86d0a81be592332ba 
tempo corrido: 0.36 segundos 
numero de zeros no inicio do hash: 3

nonce: 100075230 
blockhash (hex): 0000e9a6a93a2d8c19bc35a797bc558afecb0a4d53247946fc2460dd749a7948 
tempo corrido: 1.80 segundos 
numero de zeros no inicio do hash: 4

nonce: 100236426 
blockhash (hex): 00000a8d845ea7b92ea3f589b0c8e3210e4351aa988daac57bc9b5b46cab26fc 
tempo corrido: 4.82 segundos 
numero de zeros no inicio do hash: 5

nonce: 121523146 
blockhash (hex): 00000048b1a635b6b7fd726bbf7be991dbe2ced3e894df08619e1c081e93ded8 
tempo corrido: 425.43 segundos 
numero de zeros no inicio do hash: 6

nonce: 148058503 
blockhash (hex): 0000000a8256eb559ad433b492a5fbee291a457d4f522631afd5a086a18cbcbd 
tempo corrido: 959.89 segundos 
numero de zeros no inicio do hash: 7

# [... um tempo de espera até conseguir o alvo... ]
```

Como pode ver pelo *output* com um computador pessoal, levamos um tempo considerável para minerar este bloco; no total * segundos.

E, para finalizarmos esta parte, podemos ver uma implementação simplificada do algoritmo de *proof-of-work* para visualizarmos a progressão dos números com a dificuldade em *bits* aumentando progressivamente:

```
#!/usr/bin/env python3
import hashlib
import time

max_nonce = 2**32


def proof_of_work(header, difficulty_bits):
    # calcula o alvo da dificuldade
    target = 2**(256-difficulty_bits)

    for nonce in range(max_nonce):
        utf8_header = header.encode('utf-8')
        utf8_nonce = str(nonce).encode('utf-8')
        hash_result = hashlib.sha256(utf8_header + utf8_nonce).hexdigest()

        # checa se e um resultado valido
        if int(hash_result, 16) < target:
            print("Sucesso com o nonce %d" % nonce)
            print("Hash é %s" % hash_result)
            return(hash_result, nonce)
    print("Falhou após %d (max_nonce) tentativas" % nonce)
    return nonce

if __name__ == '__main__':
    nonce = 0
    hash_result = ''

    # dificuldade de 0 a 31 bits
    for difficulty_bits in range(32):
        difficulty = 2**difficulty_bits
        print("Dificuldade: %d (%d bits)" % (difficulty, difficulty_bits))

        print("Começando busca...")

        # marca a hora inicial
        start_time = time.time()

        # cria um novo bloco que inclui o hash do anterior
        # usamos apenas uma string vazia como um mock das transacoes
        new_block = 'bloco teste com transações' + hash_result

        hash_result, nonce = proof_of_work(new_block, difficulty_bits)

        # marca a hora final
        end_time = time.time()

        elapsed_time = end_time - start_time
        print("Tempo Corrido: %.4f segundos" % elapsed_time)

        if elapsed_time > 0:
            # uma estimativa de hashes por segundo
            hash_power = float(nonce/elapsed_time)
            print("Poder de Hashing: %d hashes por segundo" % hash_power)
        print("\n")
```

Rodando este código, podemos observar o aumento da dificuldade em *bits* que representa o número de 0's ao início do valor alvo e observar quanto tempo nosso computador demora para achar uma solução em cada dificuldade. Logo vemos que o tempo aumenta de bastante - com espaço para alguma sorte - de acordo com o aumento do número que o alvo vai ficando menor (com mais 0's ao início) como neste output em um computador pessoal:

```
Dificuldade: 2 (1 bits)
Começando busca...
Sucesso com o nonce 0
Hash é 065243447c61432cce2e6a047e9ee07d104a1ce9a28f9f7a70a12cf84de09b02
Tempo Corrido: 0.0001 segundos
Poder de Hashing: 0 hashes por segundo

# [... outras dificuldades... ]
Dificuldade: 4194304 (22 bits)
Começando busca...
Sucesso com o nonce 7782755
Hash é 00000318dddbe8ba0b1272f58c3d4273640fd26413a41766de3a15f416cd5ddd
Tempo Corrido: 28.6132 segundos
Poder de Hashing: 271998 hashes por segundo

# [...]
Dificuldade: 67108864 (26 bits)
Começando busca...
Sucesso com o nonce 63590018
Hash é 00000018645fee7587700bff0af594aeeb2f0973831df1b8c26ea5a589fcd0a2
Tempo Corrido: 244.7094 segundos
Poder de Hashing: 259859 hashes por segundo

# [...]

```

Com números de uma magnitude muito maior do que a que estamos fazendo em nossos processadores para entendermos o processo, os mineradores estão, neste exato momento, competindo restringidos pelas leis da física para alcançar estes mesmos tipos de resultados. Entendendo isto, observe os *hashes* dos [últimos blocos](https://blockr.io) e contemple o processamento colossal necessário para calcular cada um deles para atualizar a blockchain de forma a aumentar exponencialmente a segurança da blockchain contra alterações forjadas, ocupando um papel essencial no estabelecimento de consenso numa rede descentralizada de transferência de valores como o Bitcoin.

<!-- ## Formas de Minerar

### Solo

### Pool

#### Protocolo Stratum

## Forks

### Riscos

### Resolução -->
