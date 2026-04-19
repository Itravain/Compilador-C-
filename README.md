# Compiladores · C- Compiler

[![Made with Flex & Bison](https://img.shields.io/badge/Made%20with-Flex%20%26%20Bison-1f6feb)](https://www.gnu.org/software/bison/)
[![Language](https://img.shields.io/badge/lang-C-blue.svg)](src/)
[![Build](https://img.shields.io/badge/build-Makefile-success)](Makefile)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](#license)

Compilador acadêmico para a linguagem C- com pipeline completo: Scanner (Flex), Parser (Bison), AST, Análise Semântica, TAC e Geração de Assembly + binário via montador Python.

<p align="center">
  <i>Tokens → Parser → AST → Semântica → TAC → Assembly → Binário</i>
</p>

---

## Sumário
- [Visão geral](#visão-geral)
- [Pipeline de compilação](#pipeline-de-compilação)
- [Recursos](#recursos)
- [Pré-requisitos](#pré-requisitos)
- [Como executar](#como-executar)
- [Comandos do Makefile](#comandos-do-makefile)
- [Exemplo de compilação](#exemplo-de-compilação)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Detalhes de implementação](#detalhes-de-implementação)
- [Dicas e solução de problemas](#dicas-e-solução-de-problemas)
- [Contribuição](#contribuição)
- [Licença](#license)
- [Autores](#autores)

---

## Visão geral
Este projeto implementa um compilador para C-:
- Análise léxica com Flex.
- Análise sintática com Bison.
- Geração de AST e Tabela de Símbolos.
- Análise semântica (tipos, escopos, declarações).
- Geração de Código Intermediário (TAC).
- Geração de Assembly e montagem para binário.

---

## Recursos
- Suporte a variáveis globais/locais e parâmetros (incluindo arrays).
- Controle de fluxo: if/else, while, return.
- Expressões aritméticas e relacionais.
- TAC (Three-Address Code) com suporte a labels e saltos.
- Geração de assembly com acesso a pilha/FP/dados globais.
- Makefile com alvos práticos para compilar, montar, depurar e limpar.

---

## Pré-requisitos
- Linux, GNU Make
- gcc, flex, bison, python3
- valgrind, apenas se você for usar `make debug`
- Montador Python: [Assembler/assembler.py](Assembler/assembler.py)

Antes de rodar os comandos, esteja na raiz do projeto:
```bash
cd /home/itravain/Faculdade/labs/Compiladores
```

---

## Como executar
O fluxo principal é:

1. Construir o compilador.
```bash
make
```

2. Compilar um programa C- de entrada e gerar o assembly intermediário em `outputs/assembly.asm`.
```bash
make compile INPUT=test_codes/fatorial.c
```

3. Montar um arquivo assembly já existente, se você quiser usar o montador separadamente.
```bash
make assemble INPUT_ASM=outputs/assembly.asm
```

4. Executar a versão de depuração com Valgrind.
```bash
make debug INPUT=test_codes/fatorial.c
```

5. Limpar artefatos gerados.
```bash
make clean
```

Saídas típicas da compilação:
- [outputs/arvore.txt](outputs/arvore.txt)
- [outputs/tabsimb.txt](outputs/tabsimb.txt)
- [outputs/codInterm.txt](outputs/codInterm.txt)
- [outputs/assembly.asm](outputs/assembly.asm)
- [bin/fatorial.bin](bin/fatorial.bin)

Observação: o compilador lê o programa de entrada pela entrada padrão. O alvo `compile` apenas redireciona o arquivo informado em `INPUT` para o executável `compiler` e depois usa o montador Python para produzir o binário.

---

## Comandos do Makefile
| Alvo | Descrição |
|------|-----------|
| `make` / `make all` | Constrói o compilador (`compiler`). |
| `make compile INPUT=arquivo.c [OUTPUT=bin/saida.bin]` | Compila um fonte C-, gera `outputs/assembly.asm` e monta o binário final. |
| `make assemble INPUT_ASM=arquivo.asm [OUTPUT_BIN=bin/saida.bin]` | Monta um arquivo assembly existente em binário. |
| `make debug INPUT=arquivo.c` | Executa o compilador com Valgrind e salva o log em `outputs/valgrind-out.txt`. |
| `make clean` | Remove binário do compilador, objetos e saídas geradas. |

---

## Exemplo de compilação
Fonte de exemplo: [test_codes/fatorial.c](test_codes/fatorial.c)
```c
int fatorial(int numero){
    if (numero <= 1) return 1;
    return numero * fatorial(numero-1);
}
int main(){
    int a; 
    a = 5;
    a = fatorial(a);
    output(a);
}
```

Comando:
```bash
make compile INPUT=test_codes/fatorial.c
```

Saídas:
- TAC: [outputs/codInterm.txt](outputs/codInterm.txt)
- Assembly: [outputs/assembly.asm](outputs/assembly.asm)
- Binário: bin/fatorial.bin

Trecho do assembly (exemplo):
```asm
; início de main
PUSH FP
MOV FP, SP
ADDI SP, SP, #N
...
```

Se quiser montar manualmente o assembly gerado:
```bash
make assemble INPUT_ASM=outputs/assembly.asm OUTPUT_BIN=bin/fatorial.bin
```

---

## Estrutura do repositório
```
.
├─ src/
│  ├─ main.c
│  ├─ arvore.c
│  ├─ tabSimbolos.c
│  ├─ analise_semantica.c
│  ├─ codigo_intermediario.c
│  ├─ gerador_assembly.c
│  └─ pilha.c
├─ Assembler/
│  └─ assembler.py
├─ test_codes/
├─ outputs/
├─ bin/
├─ globals.h
├─ analise_sintatica.y (Bison)
├─ lexical_analyser.l (Flex)
├─ Makefile
└─ README.md
```

---

## Detalhes de implementação
- AST: construída pelo parser ([src/main.c](src/main.c)) e impressa em [outputs/arvore.txt](outputs/arvore.txt).
- Tabela de símbolos: hash com escopos e offsets ([src/tabSimbolos.c](src/tabSimbolos.c)).
- TAC: gerado por [src/codigo_intermediario.c](src/codigo_intermediario.c) e salvo em [outputs/codInterm.txt](outputs/codInterm.txt).
- Assembly: gerado por [src/gerador_assembly.c](src/gerador_assembly.c).
- Considerações de registradores/stack para chamadas e recursão.
- Vetores especiais e memória mapeada: [src/tabSimbolos.c](src/tabSimbolos.c) trata `VIDEO_MEMORY`, `RAM_MEMORY`, `INSTR_MEMORY`, `HD_MEMORY` e `TIMER_CONF` como símbolos especiais, sem alocação normal de array. Na geração de assembly, o endereço final é calculado como base fixa + índice, usando os offsets definidos em [globals.h](globals.h): `RAM_BASE = 0`, `INSTR_BASE = 2048`, `VIDEO_BASE = 6144`, `HD_BASE = 11008` e `TIMER_BASE = 27392`.

## Dicas e solução de problemas
- Se `make compile` falhar ao abrir arquivos de saída, verifique se você está executando o comando na raiz do projeto.
- O diretório `outputs/` é usado para os artefatos intermediários e o diretório `bin/` para o binário final.
- O montador Python escreve o binário em stdout; por isso o `Makefile` redireciona a saída para o arquivo final.
- O alvo `make debug` exige `valgrind` instalado no sistema.
- O projeto aceita arquivos de teste em `test_codes/`, mas qualquer arquivo C- compatível pode ser usado com `make compile INPUT=...`.

<details>
<summary>Tokens do scanner (resumo)</summary>

| Categoria | Exemplos |
|---|---|
| Palavras-chave | `if`, `else`, `while`, `int`, `void`, `return` |
| Operadores | `+ - * / % && || ! != == = >= <= > <` |
| Símbolos | `{ } ( ) [ ] ; , .` |
</details>

---



## Autores
- Eduardo Bouhid
- Ícaro Travain
