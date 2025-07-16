## Por que LangGraph?

As `Chains` do LangChain (LCEL) são **Grafos Acíclicos Dirigidos (DAGs)**. Isso significa que o fluxo de dados segue uma direção única, sem a possibilidade de voltar a um passo anterior (ciclos).

Para agentes complexos, que precisam, por exemplo, usar uma ferramenta, analisar o resultado e talvez decidir usar outra ferramenta, um fluxo linear não é suficiente. É necessário ter **ciclos** e **lógica condicional**.

**LangGraph resolve isso**, permitindo a criação de grafos com estado onde o fluxo pode ser direcionado dinamicamente.

## Conceitos Chave do LangGraph

### 1. State (Estado)

É um objeto central (geralmente um `TypedDict`) que carrega os dados através do grafo. Cada nó pode ler e modificar o estado.

```
from typing import TypedDict, List

class AgentState(TypedDict):
    question: str
    intermediate_steps: List[tuple]
    answer: str
```

### 2. Nodes (Nós)

São as unidades de trabalho do grafo. Um nó é uma função ou um `Runnable` (como uma `Chain`) que recebe o estado atual e retorna um dicionário com as atualizações para esse estado.

### 3. Edges (Arestas)

São as conexões que definem o fluxo entre os nós.

- **Aresta Padrão (`add_edge`):** Conecta um nó a outro incondicionalmente.
    
- **Aresta Condicional (`add_conditional_edges`):** Conecta um nó a múltiplos outros. Uma função de lógica decide qual caminho seguir com base no estado atual. É aqui que a "mágica" dos ciclos e da ramificação acontece.
    

## Exemplo Prático: Agente com Ferramenta e Ciclo

Vamos construir um agente simples que pode usar uma "ferramenta de busca" falsa. O agente decidirá se precisa da ferramenta. Se precisar, ele a usará e reavaliará a situação. Se não, ele responderá diretamente.

**1. Instalação:**

```
pip install langchain-openai python-dotenv langgraph
```

**2. Arquivo `.env`:**

```
OPENAI_API_KEY="sua_chave_aqui"
```

**3. Código (`guia4_langgraph.py`):**

```
import os
from dotenv import load_dotenv
from typing import TypedDict, Annotated, List
from langchain_core.messages import BaseMessage, SystemMessage
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver

# Carrega a chave da API
load_dotenv()

# --- 1. Definição do Estado do Grafo ---
# O estado define a estrutura de dados que flui através do grafo.
class AgentState(TypedDict):
    messages: Annotated[List[BaseMessage], lambda x, y: x + y]

# --- 2. Definição das Ferramentas ---
# Uma ferramenta simples e "falsa" para simular uma busca.
def search_tool(query: str):
    print(f"--- Usando a ferramenta de busca para: '{query}' ---")
    if "lisboa" in query.lower() and "lançamento" in query.lower():
        return "O lançamento do QuantumLeap será no dia 15 de agosto de 2025, em Lisboa."
    return "Não encontrei informações sobre isso."

# --- 3. Definição dos Nós do Grafo ---

# Modelo que será o "cérebro" do agente
model = ChatOpenAI(model="gpt-4o", temperature=0)

# Nó do Agente: decide se usa a ferramenta ou responde.
def agent_node(state: AgentState):
    # Adiciona uma instrução para o modelo decidir.
    system_prompt = SystemMessage(
        content="""Você é um agente. Decida se precisa usar a ferramenta de busca.
        Se a pergunta for sobre o lançamento do 'QuantumLeap', use a ferramenta.
        Caso contrário, responda diretamente.
        Para usar a ferramenta, responda com 'search: [termo de busca]'.
        Para responder diretamente, apenas forneça a resposta."""
    )
    # Adiciona a mensagem do sistema à lista de mensagens do estado
    messages_with_prompt = [system_prompt] + state['messages']
    
    response = model.invoke(messages_with_prompt)
    return {"messages": [response]}

# Nó da Ferramenta: executa a busca e retorna o resultado.
def tool_node(state: AgentState):
    last_message = state["messages"][-1].content
    # Extrai o termo de busca
    search_query = last_message.replace("search:", "").strip()
    result = search_tool(search_query)
    return {"messages": [SystemMessage(content=result)]}

# --- 4. Lógica Condicional ---
# Esta função decide para qual nó ir após o passo do agente.
def should_continue(state: AgentState):
    last_message = state["messages"][-1].content
    if "search:" in last_message:
        return "tool"  # Vai para o nó da ferramenta
    else:
        return END  # Termina o fluxo

# --- 5. Construção do Grafo ---
graph_builder = StateGraph(AgentState)

# Adiciona os nós
graph_builder.add_node("agent", agent_node)
graph_builder.add_node("tool", tool_node)

# Define o ponto de entrada
graph_builder.set_entry_point("agent")

# Adiciona a aresta condicional
graph_builder.add_conditional_edges(
    "agent",
    should_continue,
    {"tool": "tool", END: END}
)

# Adiciona a aresta que cria o ciclo: depois da ferramenta, volta para o agente
graph_builder.add_edge("tool", "agent")

# Compila o grafo
memory = MemorySaver()
graph = graph_builder.compile(checkpointer=memory)

# --- 6. Execução ---
def run_agent(question: str):
    thread_config = {"configurable": {"thread_id": "1"}}
    response = graph.invoke({"messages": [("user", question)]}, thread_config)
    final_answer = response["messages"][-1].content
    print(f"\nPergunta: {question}")
    print(f"Resposta Final: {final_answer}")

print("--- Execução 1: Usando a ferramenta ---")
run_agent("Qual a data de lançamento do QuantumLeap em Lisboa?")

print("\n\n--- Execução 2: Sem usar a ferramenta ---")
run_agent("Olá, tudo bem?")
```

**6. Executar:**

```
python guia4_langgraph.py
```

Observe como na primeira execução o grafo passa pelo `agent_node`, decide ir para o `tool_node`, e depois volta para o `agent_node` para formular a resposta final. Na segunda, ele para no primeiro passo. Isso demonstra o poder dos ciclos e da lógica condicional.