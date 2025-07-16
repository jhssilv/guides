## O que é LangChain?

É um framework para desenvolver aplicações com LLMs. Ele age como uma "cola", conectando modelos de linguagem a fontes de dados externas e outras lógicas de programação.

## Componentes Principais

### 1. Models (Modelos)

A interface para interagir com os LLMs.

- **LLMs:** Entrada e saída de texto simples. `(string -> string)`
- **Chat Models:** Interface baseada em mensagens de chat (sistema, usuário, assistente). `(lista de mensagens -> mensagem)`
- **Text Embedding Models:** Transforma texto em vetores numéricos para comparação. `(texto -> vetor)`

### 2. Prompts (Instruções)

Templates para gerar as instruções enviadas aos modelos. Permitem reutilização e formatação dinâmica.

- **Objetivo:** Estruturar a entrada do modelo de forma consistente.
- **Exemplo:** `"Traduza '{texto}' para {idioma}."`

### 3. Chains (Cadeias)

Sequências de chamadas a componentes. É a forma de "acorrentar" ações.

- **Conceito:** A saída de um componente vira a entrada do próximo.
- **Exemplo de Fluxo:** `Input -> PromptTemplate -> Model -> OutputParser`
- **LCEL (LangChain Expression Language):** A sintaxe com `|` (pipe) para criar cadeias de forma declarativa.

### 4. Indexes (Índices) & RAG

Para conectar LLMs a dados privados. O processo é chamado de **RAG (Retrieval-Augmented Generation)**.

- **Objetivo:** Permitir que o LLM responda perguntas com base em documentos específicos.
    
- **Passos do RAG:**
    
    1. **Load:** Carregar documentos (PDF, TXT, etc.).
    2. **Split:** Quebrar em pedaços menores (`chunks`).
    3. **Embed:** Transformar os `chunks` em vetores.
    4. **Store:** Armazenar em um **Vector Store** (banco de dados vetorial).
    5. **Retrieve:** Buscar os `chunks` mais relevantes para uma pergunta e enviá-los como contexto para o LLM.

### 5. Agents (Agentes)

Usam um LLM como "motor de raciocínio" para decidir quais ações tomar.

- **Como funciona:** O Agente recebe acesso a um conjunto de **Tools** (ferramentas, ex: busca na web, calculadora) e decide qual usar para responder a uma pergunta.
- **Diferença para Chains:** Agentes têm um fluxo dinâmico e não pré-determinado.

### 6. Memory (Memória)

Permite que `Chains` e `Agents` "se lembrem" de interações passadas. Essencial para chatbots.

## Exemplo Prático: "Olá, Mundo!"

Código que cria uma cadeia simples para explicar um tópico.

**1. Instalação:**

```
pip install langchain-openai python-dotenv
```

2. Arquivo .env:

Crie o arquivo e adicione sua chave da OpenAI.

```
OPENAI_API_KEY="sua_chave_aqui"
```

**3. Código (`guia1_resumo.py`):**

```
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Carrega a chave da API do arquivo .env
load_dotenv()

# 1. Modelo: Interface para o GPT-3.5-Turbo
model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# 2. Prompt: Template com uma variável "topico"
prompt_template = ChatPromptTemplate.from_template(
    "Explique o que é '{topico}' em um parágrafo."
)

# 3. Parser: Extrai o texto da resposta do modelo
output_parser = StrOutputParser()

# 4. Chain: Conecta os componentes usando LCEL (|)
chain = prompt_template | model | output_parser

# 5. Execução: Invoca a cadeia com o tópico desejado
response = chain.invoke({"topico": "API REST"})

# 6. Resultado
print(response)
```

**4. Executar:**

```
python guia1_resumo.py
```