This guide covers practical, advanced techniques for building robust and maintainable applications with LangChain and LangGraph. These are patterns and tips that often emerge from experience rather than being front-and-center in the documentation.

## 1. Code Organization: The "Toolset" Pattern

As your agent grows, managing tools in a flat list becomes cumbersome. The "Toolset" pattern helps you group related tools, their configurations, and helper methods into a single, organized class.

**Concept:** A Toolset is a class that encapsulates a specific domain of functionality (e.g., `WebSearchToolset`, `DatabaseToolset`, `CodeExecutionToolset`).

**Why it's useful:**
*   **Modularity:** Keeps related logic and dependencies together.
*   **Configuration:** Easily manage API keys, database connections, or other settings for a group of tools.
*   **Readability:** Makes your agent's capabilities clear at a glance.
*   **Reusability:** Easily share and reuse toolsets across different agents and projects.

### Example: `WebSearchToolset`

```python
from langchain_core.tools import tool
from langchain_community.utilities.duckduckgo_search import DuckDuckGoSearchAPIWrapper

class WebSearchToolset:
    """A toolset for performing web searches and processing results."""

    def __init__(self, results_per_search: int = 4):
        """
        Initializes the web search toolset.
        
        Args:
            results_per_search: The number of results to return per search.
        """
        self.ddg_search = DuckDuckGoSearchAPIWrapper()
        self.results_per_search = results_per_search

    def web_search(self, query: str) -> str:
        """
        Performs a web search using DuckDuckGo and returns the results.
        Use this for general web searches.
        """
        return self.ddg_search.run(query)

    def search_and_summarize(self, query: str) -> str:
        """
        Performs a web search and summarizes the top results.
        Use this when you need a concise summary, not just raw links.
        """
        # In a real implementation, you would add summarization logic here.
        results = self.ddg_search.get_snippets(query)
        return f"Summarized results for '{query}':\n" + "\n".join(results)

    def get_tools(self):
        """Returns a list of all methods decorated with @tool."""
        return [self.web_search, self.search_and_summarize]

# --- Usage in your agent setup ---
# search_tools = WebSearchToolset(results_per_search=5)
# all_tools = search_tools.get_tools() 
# agent = create_agent(llm, all_tools, prompt)
```

## 2. Advanced State Management in LangGraph

The `State` object in LangGraph is powerful. Go beyond simple key-value pairs by using typed data classes and annotations for more complex and predictable state.

**Concept:** Define your graph's state using a `TypedDict` and use `Annotated` to specify how values should be accumulated.

**Why it's useful:**
*   **Type Safety:** Catch errors early and make your state predictable.
*   **Clear Accumulation Logic:** Explicitly define if a value should be overwritten (default), added to a list, or updated in a dictionary.
*   **IDE Support:** Get autocompletion and type checking for your state object.

### Example: A More Complex State

```python
from typing import TypedDict, Annotated, List
from langchain_core.messages import BaseMessage
import operator

class AgentState(TypedDict):
    # This will accumulate messages in a list
    messages: Annotated[List[BaseMessage], operator.add]
    
    # This will be overwritten each time it's set
    current_task: str
    
    # This will store the output of a specific tool
    retrieved_files: List[str]
```

## 3. Building Resilient Graphs: Handling Tool Errors

In the real world, tools fail. APIs go down, code has bugs, or inputs are malformed. A robust graph shouldn't crash when a tool fails.

**Concept:** Wrap tool executions within `try...except` blocks inside your graph nodes. Update the state with an error message instead of raising an exception. Use conditional edges to route to a fallback or human-in-the-loop node upon failure.

### Example: Error-Handling Node and Conditional Edge

```python
from langchain_core.messages import ToolMessage

def tool_executor_node(state: AgentState):
    """A node that executes tools and handles potential errors."""
    last_message = state['messages'][-1]
    tool_call = last_message.tool_calls[0]
    
    try:
        # ... logic to find and execute the tool ...
        tool_output = execute_tool(tool_call) 
        
        # On success, return a ToolMessage
        message = ToolMessage(
            content=str(tool_output), 
            tool_call_id=tool_call['id']
        )
    except Exception as e:
        # On failure, return a ToolMessage with an error
        message = ToolMessage(
            content=f"Error executing tool {tool_call['name']}: {e}",
            tool_call_id=tool_call['id']
        )
        
    return {"messages": [message]}

def route_after_tool(state: AgentState):
    """
    If the last message is a ToolMessage with an error,
    route to a human for help. Otherwise, continue.
    """
    last_message = state['messages'][-1]
    if isinstance(last_message, ToolMessage) and "Error executing tool" in last_message.content:
        return "human_in_the_loop"
    return "continue_to_agent"

# --- In your graph definition ---
# graph.add_node("tool_executor", tool_executor_node)
# graph.add_conditional_edges(
#     "tool_executor",
#     route_after_tool,
#     {"human_in_the_loop": "human_node", "continue_to_agent": "agent_node"}
# )
```

## 4. Streaming Responses from LangGraph

Users love seeing output as it's generated. While streaming is common in LangChain Expression Language (LCEL), it's also possible with LangGraph.

**Concept:** Use the `.stream()` or `.astream()` methods on your compiled graph. Instead of getting the final state, you'll get a stream of the outputs from each node as it completes.

**Why it's useful:**
*   **Improved UX:** Provides immediate feedback to the user.
*   **Debugging:** Lets you see the flow of the graph and the state changes in real-time.

### Example: Streaming Node Outputs

```python
# Assuming 'app' is your compiled LangGraph
# app = graph.compile()

for chunk in app.stream({"messages": [("user", "What's the weather in SF?")]}):
    # The chunk will be a dictionary with the node name as the key
    # and the node's output as the value.
    node_name, node_output = list(chunk.items())[0]
    print(f"--- Output from node: {node_name} ---")
    print(node_output)
    print("\n")
```
