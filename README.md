# README – Analisador Sintático LL(1) com Tabela de Parsing

Este projeto consiste na implementação de um **analisador sintático LL(1)** para uma **linguagem  C++**, contendo:

---

# **1. Analisador Léxico (Lexer)**

### Função: `tokenize()`

O léxico recebe o código fonte e transforma em uma lista de **tokens**:

* palavras-chave (`int`, `while`, `return`)
* operadores (`+`, `==`, `&&`)
* identificadores (`abc`, `x1`)
* números (`10`, `3.14`)
* símbolos (`(`, `)`, `{`, `}`, `;`)

📌 Expressões Regulares são usadas para cada categoria.

Quando uma correspondência é encontrada:

* se for espaço, é ignorado
* se for caractere inválido → erro
* caso contrário → token adicionado

⚠️ No final é adicionado:

```
('EOF')
```

permitindo o parser detectar o fim do arquivo.

---

# **2. Gramática LL(1)**

### Declarações

```
DECLARATION → TYPE ID DECLARATION_TAIL
DECLARATION_TAIL → ASSIGN EXPRESSION SEMI | SEMI
```

### Expressões aritméticas

```
EXPRESSION → TERM EXP_PRIME
EXP_PRIME → PLUS TERM EXP_PRIME | EPSILON
TERM → FACTOR TERM_PRIME
TERM_PRIME → MULT FACTOR TERM_PRIME | EPSILON
```

### Condições lógicas

```
CONDITION → LOGICAL_OR_EXPR
LOGICAL_OR_EXPR → LOGICAL_AND_EXPR LOGICAL_OR_PRIME
LOGICAL_OR_PRIME → OR LOGICAL_AND_EXPR LOGICAL_OR_PRIME | EPSILON
```

⚠️ **Produção que resolve conflito ID = expressão**

```
STATEMENT_ID_START → ID STATEMENT_ID_TAIL
STATEMENT_ID_TAIL → ASSIGN EXPRESSION SEMI
                  | TERM_PRIME EXP_PRIME SEMI
```

Assim, evita ambiguidade entre:

```
x = 10;
x * 2;
```

---

# **3. FIRST e FOLLOW**

Antes de montar a tabela LL(1), o algoritmo calcula:

* `FIRST(X)` → quais símbolos podem iniciar X
* `FOLLOW(X)` → quais símbolos podem seguir X

Ambos utilizam laços até **não haver mais mudanças**.

---

# **4. Construção da Tabela LL(1)**

Para cada produção `A → α`:

* gera FIRST(α)
* se aceita ε (vazio), usa FOLLOW(A)

A tabela resultante tem formato:

```
(par não-terminal, terminal) → produção
```

Exemplo:

```
(PROGRAM, MAIN) → MAIN_FUNCTION EOF
```

---

# **5. Parsing com Pilha**

### Estratégia:

1. Inicializa pilha:

```
[ PROGRAM ]
```

2. Lê token atual
3. Regras:

### ✔️ Se topo é terminal e casa com token

→ consome token

### ✔️ Se topo é não-terminal

→ consulta tabela LL(1)
→ empilha produção invertida

### ✔️ Se topo é EPSILON

→ remove

### ❌ Caso contrário: erro sintático

O algoritmo imprime:

```
PILHA | ENTRADA | AÇÃO
```

Permite acompanhar passo a passo.

---

# ✔️ **Exemplo de Sucesso**

Entrada:

```
main() { int x; x = 10; }
```

Saída:

```
SUCESSO: CÓDIGO ACEITO
```

---

# ❌ **Exemplo de Erro**

Entrada:

```
main( { }
```

Saída:

```
ERRO SINTÁTICO: Esperado ')', encontrado '{'
```
