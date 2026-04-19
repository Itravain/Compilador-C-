# Compiladores · C- Compiler

[![Made with Flex & Bison](https://img.shields.io/badge/Made%20with-Flex%20%26%20Bison-1f6feb)](https://www.gnu.org/software/bison/)
[![Language](https://img.shields.io/badge/lang-C-blue.svg)](src/)
[![Build](https://img.shields.io/badge/build-Makefile-success)](Makefile)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](#license)

Compilador acadêmico para a linguagem C- com pipeline completo: scanner, parser, AST, análise semântica, TAC, assembly e binário.

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
Este projeto implementa um compilador para C- com geração de código intermediário e binário final.

Fluxo principal:
- Entrada do programa pela stdin.
- Geração de AST, tabela de símbolos e TAC.
- Emissão de assembly em [outputs/assembly.asm](outputs/assembly.asm).
- Montagem do assembly em binário com o montador Python.

---

## Recursos
- Variáveis globais, locais, parâmetros e arrays.
- Controle de fluxo com if/else, while e return.
- Expressões aritméticas, relacionais e chamadas de função.
- Geração de assembly e binário final.

---

## Pré-requisitos
- Linux, GNU Make
- gcc, flex, bison, python3
- valgrind, apenas se você for usar `make debug`
- Montador Python: [Assembler/assembler.py](Assembler/assembler.py)

Execute os comandos na raiz do projeto.

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

Saídas típicas:
- [outputs/arvore.txt](outputs/arvore.txt)
- [outputs/tabsimb.txt](outputs/tabsimb.txt)
- [outputs/codInterm.txt](outputs/codInterm.txt)
- [outputs/assembly.asm](outputs/assembly.asm)
- binário em [bin/](bin/)

Observação: o compilador lê o programa pela entrada padrão. `make compile` só redireciona o arquivo informado em `INPUT` para o executável `compiler` e monta o binário ao final.

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
Fonte de exemplo: [test_codes/fatorial.c](test_codes/fatorial.c).
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
- [outputs/codInterm.txt](outputs/codInterm.txt)
- [outputs/assembly.asm](outputs/assembly.asm)
- binário em [bin/](bin/)

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
- AST, tabela de símbolos e TAC são gerados no pipeline principal do compilador.
- O assembly final é produzido por [src/gerador_assembly.c](src/gerador_assembly.c).
- Vetores especiais (`VIDEO_MEMORY`, `RAM_MEMORY`, `INSTR_MEMORY`, `HD_MEMORY` e `TIMER_CONF`) são tratados como memória mapeada, com endereços base definidos em [globals.h](globals.h).

## Dicas e solução de problemas
- Se `make compile` falhar ao abrir arquivos de saída, verifique se está na raiz do projeto.
- `outputs/` guarda os artefatos intermediários e `bin/` guarda o binário final.
- O montador Python escreve o binário em stdout; o `Makefile` faz o redirecionamento.
- `make debug` exige `valgrind` instalado.
- Arquivos de teste em `test_codes/` funcionam, mas qualquer fonte C- compatível pode ser usado.

---



## Autores
- Eduardo Bouhid
- Ícaro Travain
