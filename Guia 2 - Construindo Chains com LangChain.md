## O que é uma Chain?

Uma `Chain` é uma sequência de componentes executados em uma ordem específica. A ideia central é que a **saída de um passo se torna a entrada do próximo**, criando um fluxo de processamento.

A maneira moderna de construir `Chains` é usando a **LangChain Expression Language (LCEL)**, que utiliza o operador `|` (pipe) para conectar os componentes.

**Fluxo Básico:** `Input | Componente 1 | Componente 2 | ... | Output`

## Tipos Comuns de Chains

### 1. LLMChain (A Cadeia Fundamental)

Esta é a cadeia mais básica e já foi vista no Guia 1. Ela conecta um `PromptTemplate` a um `Model` e, opcionalmente, a um `OutputParser`.

- **Objetivo:** Pegar a entrada do usuário, formatá-la com um template e obter a resposta do modelo.
    
- **Estrutura LCEL:** `prompt | model | parser`
    

**Relembrando o Código:**

```
# Estrutura de uma LLMChain básica
chain = prompt_template | model | output_parser
response = chain.invoke({"topico": "Inteligência Artificial"})
```

### 2. Cadeias Sequenciais (Sequential Chains)

Cadeias sequenciais são usadas quando você precisa executar uma série de passos em ordem, onde cada passo depende do resultado do anterior.

- **Objetivo:** Criar fluxos de trabalho com múltiplos estágios.
    
- **Exemplo de Fluxo:**
    
    1. **Cadeia 1:** Gera o nome de um produto com base em um conceito.
        
    2. **Cadeia 2:** Pega o nome do produto gerado e cria uma descrição de marketing para ele.
        

Para gerenciar as entradas e saídas entre as cadeias, usamos um objeto chamado `RunnablePassthrough`.

## Exemplo Prático: Cadeia Sequencial

Vamos criar uma cadeia que primeiro gera um nome para uma empresa de tecnologia e, em seguida, cria um slogan para essa empresa.

**1. Instalação (se ainda não tiver):**

```
pip install langchain-openai python-dotenv
```

**2. Arquivo `.env`:**

```
OPENAI_API_KEY="sua_chave_aqui"
```

**3. Código (`guia2_sequential.py`):**

```
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# Carrega a chave da API
load_dotenv()

# --- Modelo Compartilhado ---
# Usaremos o mesmo modelo para ambas as partes da cadeia
model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0.7)
parser = StrOutputParser()

# --- Cadeia 1: Gerar Nome da Empresa ---
# Esta cadeia recebe uma descrição do que a empresa faz e sugere um nome.
prompt_nome = ChatPromptTemplate.from_template(
    "Gere um nome criativo e curto para uma empresa focada em: {conceito}"
)
chain_nome = prompt_nome | model | parser

# --- Cadeia 2: Gerar Slogan ---
# Esta cadeia recebe o nome da empresa e gera um slogan.
prompt_slogan = ChatPromptTemplate.from_template(
    "Crie um slogan impactante para uma empresa chamada: {nome_empresa}"
)
chain_slogan = prompt_slogan | model | parser


# --- Cadeia Sequencial Completa ---
# Agora, vamos conectar as duas cadeias.
# Usamos o RunnablePassthrough para passar a entrada original ("conceito")
# junto com a saída da primeira cadeia ("nome_empresa").
# O dicionário define as entradas para a segunda cadeia.
cadeia_sequencial = (
    {"nome_empresa": chain_nome, "conceito": RunnablePassthrough()}
    | chain_slogan
)

# --- Execução ---
# O dicionário de entrada para .invoke() agora só precisa do "conceito",
# pois a cadeia foi montada para passar o resultado adiante.
conceito_empresa = "Inteligência Artificial para análise de dados"
print(f"Conceito da empresa: {conceito_empresa}\n")

# Invocando a primeira cadeia separadamente (opcional, para ver o passo intermediário)
nome_gerado = chain_nome.invoke({"conceito": conceito_empresa})
print(f"Nome gerado (passo 1): {nome_gerado}")

# Invocando a cadeia sequencial completa
slogan_gerado = cadeia_sequencial.invoke({"conceito": conceito_empresa})
print(f"Slogan gerado (passo 2): {slogan_gerado}\n")
```

**4. Executar:**

```
python guia2_sequential.py
```

Este exemplo demonstra o poder das cadeias para automatizar tarefas de múltiplos passos de forma estruturada e reutilizável.