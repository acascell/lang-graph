# lang-graph notes
LLM stateful orchestration framework

Lang graph is being defined as orchestration framework to direct AI chaos and reduce as well hallucination.
It defines multiple layers within a directed graph which designates the execution workflow. It uses:

1. nodes (the actual steps to perform)
2. edges (transitions between the steps)
3. state (the actual memory)

The main component of the frameowork is the graph (DAG or cylic) - This the workflow engine, to use a metaphore is very similar to how a step function works in AWS, coordinating the workflow, sharing the state and deciding what action to perform on the next step.

1. Nodes -> is the function (step) that runs some logic - for instance call a large language model - or query a database - or call an external API - or deciding the next step of the workflow

```python
def my_node(state)
  return {"result": "soomething}
```
Key idea: nodes are deterministic or LLM driven 

2. Edge -> the actual link between the nodes defining the execution order
Normal edge - always go to the next node
Conditional edge -> routing based logic
-> its a decition point with a branching statement
   ```python
   if state["needs_tool"]:
     tool_node
   else:
     answer_node
   ```

3. state -> reppresent the shared object passed between nodes (functions) -> usually its a dicionary or a pydantic model
```python
state = {
    "question": "...",
    "context": "...",
    "answer": "..."
  }
```
The shared state role is to store the memory, track the progress and enable multi step reasoning

The interaction between node <-> state works in a way that each node reads state and return partial updates. LangGraph eventually merges updates automatically

LLM Pattern
LLM → decide → tool → result → LLM → ...

Tool -> An external capability the agent can actually call to perform API calls, db queries, aws services, calculation etc...

Loop/cycle -> graph can loop until a certain condition is met -> this enables 
1. multi step reasoning
2. iterative refinement

Start node
```python
graph.set_entry_point("start_node")
```

End node -> special function which terminates the execution

Checkpointing -> saving state between steps, used mostly for recovery, persistence, long running workflows - Think -> resume agent later

Streaming --> its a partial output that gets emitted during the exeuction and its useful for UI updates and for real-time feedback

Memory can be of different types 1. short tem -> stored in the current state 2. long term -> stored in external storage like db or vector db

Subgraph -> graph another graph - enables modular design and reusable workflows

deterministic flow -> has a fixed path and its predictible 
agentic flow -> LLM decies and has dynamic routing  - real systems use both approaches

MENTAL MODEL
- state = data
- nodes = functions
- edges = control flow
- LLM = decision engne
- graph = whole system

FLOW EXAMPLE

User question -> retriever node (RAG)
-> LLM decision mode -> needs tool?
-> yes - tool node - back to llm
-> no - answer node - END
