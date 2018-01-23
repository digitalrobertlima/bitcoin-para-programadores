# Errata

Esta seção é dedicada a listar erros reportados por terceiros. Muito obrigado
aos que colaboram.

##0
Bug no código exemplo na "Introdução" no item "Funções *Hash* Criptográficas" reportado por **JC GreenMind**, em que um erro no nome da variável causa falha na execução.

### Diff da correção:
```bash
diff --git a/intro.md b/intro.md
index 9241f5f..4b08016 100644
--- a/intro.md
+++ b/intro.md
@@ -53,9 +53,9 @@ Exemplo de ambas funções sendo usadas em Python com a *string* "bitcoin" como
 1ed7259a5243a1e9e33e45d8d2510bc0470032df964956e18b9f56fa65c96e89
 
 >>> word2_ripemd160 = hashlib.new('ripemd160')
->>> word_ripemd160.update(word2.encode('utf-8'))
->>> print(word_ripemd160.hexdigest())
-5f67cd0e647825711eac0b0bf78e0487b149bc3a
+>>> word2_ripemd160.update(word2.encode('utf-8'))
+>>> print(word2_ripemd160.hexdigest())
+9d2028ac5216d10b85d1a3ab389ebcc57a3ee6eb
```
