## O que é RAG?

**RAG (Geração Aumentada por Recuperação)** é uma técnica que "aumenta" o conhecimento de um LLM fornecendo-lhe informações relevantes de uma base de dados externa no momento da pergunta.

- **Problema:** LLMs não conhecem seus dados privados ou eventos recentes.
- **Solução RAG:** Antes de o LLM responder, um sistema busca em seus documentos os trechos mais relevantes para a pergunta e os entrega como **contexto**.

## O Fluxo do RAG

O processo de RAG é dividido em duas fases: **Indexação** (feita uma vez) e **Recuperação/Geração** (feita a cada pergunta).

### Fase 1: Indexação (Preparando os Dados)

1. **Load (Carregar):** Carrega seus documentos de várias fontes (PDF, TXT, CSV, etc.) usando `Document Loaders`.
    
2. **Split (Dividir):** Quebra os documentos em pedaços menores (`chunks`) usando `Text Splitters`. Isso é necessário para caber no limite de contexto do LLM.
    
3. **Embed (Vetorizar):** Converte cada `chunk` em um vetor numérico usando um `Embedding Model`. Vetores representam o significado semântico do texto.
    
4. **Store (Armazenar):** Armazena os `chunks` e seus vetores correspondentes em um `Vector Store` (banco de dados vetorial).
    

### Fase 2: Recuperação e Geração (Respondendo a uma Pergunta)

1. **Retrieve (Recuperar):** Quando o usuário faz uma pergunta, ela também é convertida em um vetor. O `Retriever` busca no `Vector Store` os `chunks` cujos vetores são mais similares ao vetor da pergunta.
    
2. **Generate (Gerar):** Os `chunks` recuperados (o contexto) e a pergunta original são inseridos em um `PromptTemplate`. Este prompt final é enviado ao LLM, que gera a resposta com base no contexto fornecido.
    

## Exemplo Prático: Chat com seu Documento

Vamos criar um sistema que responde a perguntas com base em um arquivo de texto.

1. Instalação:

Você precisará de mais algumas bibliotecas para este exemplo.

```
pip install langchain-openai python-dotenv faiss-cpu tiktoken
```

- `faiss-cpu`: Uma biblioteca do Facebook AI para busca de similaridade eficiente (nosso Vector Store).
    
- `tiktoken`: Usado internamente pelo LangChain para contar tokens ao dividir o texto.
    

**2. Arquivo `.env`:**

```
OPENAI_API_KEY="sua_chave_aqui"
```

3. Crie um Documento de Exemplo:

Crie um arquivo chamado meu_documento.txt e cole o seguinte texto:

```
A empresa InovaTech anunciou o lançamento do seu novo produto, o 'QuantumLeap', um processador quântico para uso comercial.
O lançamento oficial será no dia 15 de agosto de 2025, em Lisboa.
O QuantumLeap promete ser 100 vezes mais rápido que os supercomputadores atuais e será focado em resolver problemas de otimização complexos para as indústrias farmacêutica e financeira.
O CEO da InovaTech, Dr. Arnaldo Silva, afirmou que este é um marco para a computação.
```

**4. Código (`guia3_rag.py`):**

```
import os
from dotenv import load_dotenv

# Componentes do LangChain
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# Componentes específicos para RAG
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_community.document_loaders import TextLoader

# Carrega a chave da API
load_dotenv()

# --- 1. Modelos ---
model = ChatOpenAI(model="gpt-3.5-turbo")
embeddings = OpenAIEmbeddings() # Modelo para criar os vetores
parser = StrOutputParser()

# --- FASE DE INDEXAÇÃO ---

# --- 2. Load (Carregar) ---
# Carrega o documento de texto.
print("Carregando documento...")
loader = TextLoader("meu_documento.txt", encoding="utf-8")
documents = loader.load()

# --- 3. Split (Dividir) ---
# Divide o documento em chunks menores.
print("Dividindo documento em chunks...")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = text_splitter.split_documents(documents)

# --- 4. Store (Armazenar) ---
# Cria o Vector Store usando FAISS a partir dos chunks e seus embeddings.
# Esta etapa cria os vetores e os armazena em memória.
print("Criando o Vector Store...")
vector_store = FAISS.from_documents(chunks, embeddings)

# --- FASE DE RECUPERAÇÃO E GERAÇÃO ---

# --- 5. Retrieve (Recuperar) ---
# Cria um retriever a partir do Vector Store. O retriever é o componente
# responsável por buscar os documentos relevantes.
print("Criando o retriever...")
retriever = vector_store.as_retriever()

# --- 6. Prompt Template ---
# Criamos um prompt que espera um contexto (os chunks recuperados) e a pergunta.
template = """
Você é um assistente de IA. Responda à pergunta do usuário com base
apenas no contexto fornecido. Se a resposta não estiver no contexto,
diga que você não sabe.

Contexto:
{context}

Pergunta:
{question}
"""
prompt = ChatPromptTemplate.from_template(template)


# --- 7. Construção da Cadeia RAG ---
# Usando LCEL, montamos a cadeia completa.
# O fluxo é:
# 1. A pergunta do usuário é passada para o retriever.
# 2. O retriever retorna os documentos relevantes (o contexto).
# 3. O contexto e a pergunta são passados para o prompt.
# 4. O prompt formatado é enviado para o modelo.
# 5. A resposta do modelo é processada pelo parser.
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | parser
)

# --- 8. Execução ---
print("\n--- Sistema de Perguntas e Respostas ---")
pergunta = "Qual o nome do novo produto da InovaTech e quando será lançado?"
print(f"Pergunta: {pergunta}")

# Invocamos a cadeia com a pergunta
resposta = rag_chain.invoke(pergunta)
print(f"Resposta: {resposta}\n")

pergunta_2 = "Quem é o presidente da empresa?"
print(f"Pergunta: {pergunta_2}")
resposta_2 = rag_chain.invoke(pergunta_2)
print(f"Resposta: {resposta_2}\n")

pergunta_3 = "Qual o preço do produto?"
print(f"Pergunta: {pergunta_3}")
resposta_3 = rag_chain.invoke(pergunta_3)
print(f"Resposta: {resposta_3}\n")
```

**5. Executar:**

```
python guia3_rag.py
```

Observe como o sistema responde corretamente às perguntas baseadas no texto e admite não saber a resposta quando a informação não está no contexto.