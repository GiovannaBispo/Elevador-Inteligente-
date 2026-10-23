# 🛗 Elevador Inteligente — Validadores, Máquina de Estados e Scanner

**Integrantes**

| RA | Nome |
|---|---|
| 2392072 | Diego Azevedo Dias Inacio |
| 2334425 | Gabriella Pereira Rodrigues |
| 2367880 | Giovanna Bispo Da Silva |
| 2499948 | José Henrique Guimarães Galvão Silva |

**Disciplina:** Compiladores — Ciência da Computação, 8º semestre
**Atividade:** Aula CP 08 (Prática 3) — Autômatos Finitos na Prática
**Tema:** Elevador inteligente

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU_USUARIO/SEU_REPOSITORIO/blob/main/Elevador_Inteligente.ipynb)

---

## 📌 O que o projeto faz

Modelamos o software de um elevador com três tipos de autômato:

| Parte | O que é | Seção do notebook |
|---|---|---|
| **Validadores (AFD)** | andar (`T`, `S1`, `S2`, `1`–`20`) e código de manutenção (`MNT-0042`) | 1 e 2 |
| **Máquina de estados** | ciclo de porta e movimento: `PARADO → PORTA_ABRINDO → PORTA_ABERTA → PORTA_FECHANDO → SUBINDO/DESCENDO` | 3 |
| **Mini-scanner** | linguagem de comandos: `CHAMAR 7 SUBIR` · `IR S2` · `EMERGENCIA` | 4 |
| **Interface** | painel `ipywidgets` com 4 abas | 5 |
| **Bônus** | AFD mínimo, desenho com Graphviz, scanner → máquina de estados | 6 |

---

## ▶️ Como rodar (Google Colab)

1. Clique no botão **Abrir no Colab** acima (ou abra `Elevador_Inteligente.ipynb` no Colab: *Arquivo → Abrir notebook → GitHub*).
2. Menu **Ambiente de execução → Executar tudo**.
3. Nada precisa ser instalado: usamos só a biblioteca padrão do Python, mais `ipywidgets` e `graphviz`, que já vêm no Colab.
4. A interface aparece na saída da **Célula 14**. Os desenhos do Graphviz aparecem na **Célula 16**.

> Para rodar localmente: `pip install ipywidgets graphviz jupyter` (e o programa `dot` do Graphviz para os desenhos), depois abra o `.ipynb` no Jupyter ou VS Code.

---

## 1️⃣ Validador de andar

Aceita: `T` (térreo), `S1`, `S2` (subsolos) e os números de `1` a `20`.

### Classes de caracteres

| Classe | Caracteres |
|---|---|
| `0` | 0 |
| `1` | 1 |
| `2` | 2 |
| `3-9` | 3 a 9 |
| `T` | T |
| `S` | S |
| `outro` | qualquer outro (vai para o estado morto) |

As classes `0`, `1`, `2` e `3-9` são as classes "finas": `1` e `2` precisam ser separados porque podem ser um andar sozinho, o começo de `10`–`19`/`20` ou o número de `S1`/`S2`; o `0` só é válido depois de `1` ou `2`.

### Desenho

```
 ──► ( A0 ) ──T───► (( AT ))
        │
        ├──S───► ( AS ) ──1, 2────────────► (( AF ))
        │
        ├──1───► (( A1 )) ──0,1,2,3-9─────► (( AF ))     # 10 a 19
        │
        ├──2───► (( A2 )) ──0─────────────► (( AF ))     # 20
        │
        └──3-9─► (( AD ))
```

`(( ))` = estado de aceitação. Qualquer seta que não aparece leva ao estado morto. O estado `AF` aparece repetido só para o desenho ficar legível: é um único estado.

### Saída de `tabela()`

```
📋 Tabela de transição — Andar do elevador
   estado     0    1    2  3-9    S    T
→  A0         —   A1   A2   AD   AS   AT
 * A1        AF   AF   AF   AF    —    —
 * A2        AF    —    —    —    —    —
 * AD         —    —    —    —    —    —
 * AF         —    —    —    —    —    —
   AS         —   AF   AF    —    —    —
 * AT         —    —    —    —    —    —
```

### Rastreamento (`rodar()`)

```
🔎 Andar do elevador lendo 'S2'
   passo lido  classe  de     para
   1     'S'   S       A0     AS
   2     '2'   2       AS     AF
   ✅ ACEITA: terminou em AF, que é de aceitação

🔎 Andar do elevador lendo '21'
   passo lido  classe  de     para
   1     '2'   2       A0     A2
   2     '1'   1       A2     💀 morto
   ❌ REJEITADA: não há transição de A2 com '1'
```

**Por que `21` foi rejeitada?** Ela morreu **no meio**: depois do `2` o autômato está em `A2`, e a única seta que sai de `A2` é com `0` (o andar 20). Chegou `1`, não há seta, estado morto.

`S` também é rejeitada, mas de outro jeito: não morre, termina em `AS`, que não é de aceitação (faltou o número do subsolo).

### Bateria AFD × RegEx

RegEx equivalente: `T|S[12]|20|1[0-9]|[1-9]`

| Aceitas (9) | Rejeitadas (13) |
|---|---|
| `T` `S1` `S2` `1` `7` `9` `10` `19` `20` | `""` (vazia) `0` `21` `S` `S0` `S3` `T1` `01` `100` `t` `s1` `"1 "` `30` |

**Divergências: 0.**

---

## 2️⃣ Validador de código de manutenção

Formato: `MNT-` mais **exatamente 4 dígitos** (`MNT-0042`).

| Classe | Caracteres |
|---|---|
| `M`, `N`, `T`, `-` | cada símbolo do prefixo é uma classe |
| `D` | dígitos 0–9 |
| `outro` | qualquer outro |

### Desenho

```
 ──► (M0) ─M─► (M1) ─N─► (M2) ─T─► (M3) ─-─► (M4) ─D─► (M5) ─D─► (M6) ─D─► (M7) ─D─► ((M8))
```

### Saída de `tabela()`

```
📋 Tabela de transição — Código de manutenção
   estado     -    D    M    N    T
→  M0         —    —   M1    —    —
   M1         —    —    —   M2    —
   M2         —    —    —    —   M3
   M3        M4    —    —    —    —
   M4         —   M5    —    —    —
   M5         —   M6    —    —    —
   M6         —   M7    —    —    —
   M7         —   M8    —    —    —
 * M8         —    —    —    —    —
```

### Rastreamento

```
🔎 Código de manutenção lendo 'MNT-0042'
   1 'M' M  M0 -> M1
   2 'N' N  M1 -> M2
   3 'T' T  M2 -> M3
   4 '-' -  M3 -> M4
   5 '0' D  M4 -> M5
   6 '0' D  M5 -> M6
   7 '4' D  M6 -> M7
   8 '2' D  M7 -> M8
   ✅ ACEITA: terminou em M8, que é de aceitação

🔎 Código de manutenção lendo 'MNT-42'
   ...
   6 '2' D  M5 -> M6
   ❌ REJEITADA: terminou em M6, que NÃO é de aceitação

🔎 Código de manutenção lendo 'MNT-00425'
   ...
   9 '5' D  M8 -> 💀 morto
   ❌ REJEITADA: não há transição de M8 com 'D'
```

- `MNT-42` terminou em `M6`: faltaram 2 dígitos.
- `MNT-00425` morreu no 5º dígito: `M8` é de aceitação, mas não tem nenhuma seta de saída.

(A saída completa passo a passo está na Célula 5 do notebook.)

### Bateria AFD × RegEx

RegEx equivalente: `MNT-[0-9]{4}`

| Aceitas (5) | Rejeitadas (8) |
|---|---|
| `MNT-0042` `MNT-0000` `MNT-0001` `MNT-1234` `MNT-9999` | `""` (vazia) `MNT-` `MNT-42` `MNT0042` `mnt-0042` `MNT-00425` `MNT-004A` `ABC-0042` |

**Divergências: 0.**

---

## 3️⃣ Máquina de estados do elevador

Os "símbolos" aqui são **eventos**. **7 estados** e **16 transições**.

**Regra de segurança que escolhemos:** o elevador só se move depois de um ciclo completo de porta. Estado de aceitação (ciclo finalizado): `PARADO`.

```
 ──► ( PARADO ) ─abrir─► ( PORTA_ABRINDO ) ─porta_aberta─► ( PORTA_ABERTA ) ─fechar─► ( PORTA_FECHANDO )
        ▲                                                                                │          │
        │                                                                          subir │          │ descer
        │                                                                                ▼          ▼
        │                                                                         ( SUBINDO )  ( DESCENDO )
        │                                                                                │          │
        └──────────────────────────────────── chegar ────────────────────────────────────┴──────────┘
```

| De | Evento | Para | Regra de negócio |
|---|---|---|---|
| `PORTA_FECHANDO` | `obstaculo` | `PORTA_ABRINDO` | sensor da porta: alguém no caminho, reabre |
| `PORTA_FECHANDO` | `porta_fechada` | `PARADO` | ninguém chamou, fica parado |
| qualquer estado (menos `EMERGENCIA`) | `emergencia` | `EMERGENCIA` | botão de emergência |
| `EMERGENCIA` | `resolver` | `PARADO` | só a manutenção libera |

### Fluxos válidos

```
🛗 eventos: ['abrir', 'porta_aberta', 'fechar', 'subir', 'chegar']
   PARADO          --abrir--> PORTA_ABRINDO
   PORTA_ABRINDO   --porta_aberta--> PORTA_ABERTA
   PORTA_ABERTA    --fechar--> PORTA_FECHANDO
   PORTA_FECHANDO  --subir--> SUBINDO
   SUBINDO         --chegar--> PARADO
   ✅ ciclo finalizado em PARADO

🛗 eventos: ['abrir', 'porta_aberta', 'fechar', 'descer', 'emergencia', 'resolver']
   ...
   DESCENDO        --emergencia--> EMERGENCIA
   EMERGENCIA      --resolver--> PARADO
   ✅ ciclo finalizado em PARADO
```

### Fluxos inválidos (o código mostra os eventos permitidos)

```
🛗 eventos: ['abrir', 'subir']
   PARADO          --abrir--> PORTA_ABRINDO
   ❌ evento 2 'subir' não é permitido em PORTA_ABRINDO. Permitidos: ['porta_aberta', 'emergencia']

🛗 eventos: ['abrir', 'porta_aberta', 'fechar', 'subir', 'abrir']
   ...
   ❌ evento 5 'abrir' não é permitido em SUBINDO. Permitidos: ['chegar', 'emergencia']
```

---

## 4️⃣ Mini-scanner da linguagem Elevador

### Tabela de tokens (10 tipos)

| Token | Padrão | Exemplos |
|---|---|---|
| `RESERVADA` | `CHAMAR IR SUBIR DESCER EMERGENCIA ESPERAR DEFINIR SE MANUTENCAO` (9) | `CHAMAR` |
| `ID` | letra ou `_`, seguida de letras, dígitos ou `_` | `andar`, `velocidade` |
| `NUM_INT` | um ou mais dígitos | `7` |
| `NUM_REAL` | dígitos, ponto, dígitos | `2.5` |
| `ANDAR_TERREO` | a letra `T` sozinha | `T` |
| `ANDAR_SUBSOLO` | `S1` ou `S2` | `S2` |
| `COD_MANUT` | `MNT-` + exatamente 4 dígitos | `MNT-0042` |
| `ATRIB` | `=` | `=` |
| `IGUAL` | `==` | `==` |
| `PONTO_VIRGULA` | `;` | `;` |
| *(ignorados)* | espaços, tabs, quebras de linha, comentários `# ...` | |

### Maximal Munch (3 casos)

| Entrada | Resultado | Por quê |
|---|---|---|
| `= ==` | `ATRIB`, `IGUAL` | lookahead de um caractere |
| `2 2.5` | `NUM_INT`, `NUM_REAL` | o laço só para quando não dá mais para crescer |
| `MNT MNT-0042` | `ID`, `COD_MANUT` | depois de `MNT`, se vem `-` e dígitos, leva o código inteiro |

### Erros léxicos

Todos informam **linha, coluna e dica**, e o scanner **continua** depois do erro. Dicas específicas do domínio:

| Situação | Dica |
|---|---|
| vírgula (`3,5`) | use ponto como separador decimal |
| hífen fora de código (`-`) | o hífen só é válido dentro de um código de manutenção, como `MNT-0042` |
| ponto sem dígitos (`5.`, `.5`) | número real precisa de dígitos antes e depois do ponto |
| código malformado (`MNT-42`) | use `MNT-` seguido de exatamente 4 dígitos |

### Exemplo de saída

Entrada (resumida):

```
# roteiro de teste do elevador
DEFINIR velocidade = 1.5
CHAMAR 7 SUBIR
IR S2
...
MANUTENCAO MNT-0042
CHAMAR 3,5 DESCER @
IR MNT-42
IR 5.
```

Saída (trecho):

```
TIPO           LEXEMA     LIN COL
RESERVADA      DEFINIR      2   1
ID             velocidade   2   9
ATRIB          =            2  20
NUM_REAL       1.5          2  22
RESERVADA      CHAMAR       3   1
NUM_INT        7            3   8
RESERVADA      SUBIR        3  10
RESERVADA      IR           4   1
ANDAR_SUBSOLO  S2           4   4
...
RESERVADA      MANUTENCAO   9   1
COD_MANUT      MNT-0042     9  12

⚠️ Linha 10, coluna 9: caractere inesperado ',' (use ponto como separador decimal)
⚠️ Linha 10, coluna 19: caractere inesperado '@' (caractere fora da linguagem)
⚠️ Linha 11, coluna 4: código de manutenção malformado 'MNT-42' (use MNT- seguido de exatamente 4 dígitos, ex.: MNT-0042)
⚠️ Linha 12, coluna 5: caractere inesperado '.' (número real precisa de dígitos antes e depois do ponto)
```

---

## 5️⃣ Interface

A **Célula 14** monta um painel `ipywidgets` com 4 abas:

| Aba | Para quê |
|---|---|
| **Validadores** | escolher o AFD (andar ou manutenção), digitar a palavra e ver o passo a passo + o que a RegEx diz |
| **Scanner** | digitar comandos e ver a tabela de tokens e os erros léxicos |
| **Máquina** | digitar eventos separados por vírgula e ver o fluxo |
| **Roteiro** | digitar comandos e ver o scanner comandando a máquina de estados (bônus) |

---

## ⭐ Bônus

**AFD mínimo.** A função `minimizar()` (refinamento de partições) junta estados equivalentes. O AFD do andar passa de **7 para 5 estados**, porque `AT`, `AD` e `AF` aceitam e não têm saída. O AFD mínimo também passa na bateria com **0 divergências**. O de manutenção já é mínimo (9 → 9).

**Desenho automático com Graphviz.** `desenhar(afd)` gera o grafo a partir do próprio `delta` (setas com mesma origem e destino são juntadas). Foram desenhados o AFD do andar, o AFD mínimo, o AFD de manutenção e a máquina de estados.

**Tudo conectado.** `executar_roteiro()` escaneia um roteiro e transforma os tokens em eventos da máquina de estados, validando andares e códigos com os AFDs. Exemplo:

```
▶ linha 4: EMERGENCIA
   PARADO          --emergencia--> EMERGENCIA
▶ linha 5: IR T  (andar -2 -> 0)
   ❌ evento 1 'abrir' não é permitido em EMERGENCIA. Permitidos: ['resolver']
▶ linha 6: MANUTENCAO MNT-0042  (código validado pelo AFD)
   EMERGENCIA      --resolver--> PARADO
```

Depois da emergência, só um código de manutenção válido libera o elevador.

---

## 🗂️ Arquivos

| Arquivo | Conteúdo |
|---|---|
| `Elevador_Inteligente.ipynb` | notebook completo (rodar no Colab) |
| `README.md` | este arquivo |

## 📚 Referências

- CARMO, Robson. *Compiladores*. São Paulo: Centro Universitário FMU, 2026. (Apostila da disciplina — Unidades 5 e 6.)
- AHO, A. V.; LAM, M. S.; SETHI, R.; ULLMAN, J. D. *Compiladores: Princípios, Técnicas e Ferramentas*. 2. ed. Pearson, 2008. (Cap. 3.)
- SIPSER, M. *Introdução à Teoria da Computação*. Cengage, 2007. (Cap. 1.)
- Documentação do módulo `re` do Python: <https://docs.python.org/3/library/re.html>
- Documentação do ipywidgets: <https://ipywidgets.readthedocs.io>
