# stateful-chatbot-using-LangGraph-LangChain-and-Groq

This notebook demonstrates how to build a simple **stateful chatbot using LangGraph**, **LangChain**, and **Groq’s Gemma2-9b-It model**. 
### **Installing Required Libraries**

```python
!pip install langgraph langsmith
!pip install langchain langchain_groq langchain_community
```

* Installs the packages:

  * langgraph: For building dynamic, stateful computation graphs.
  * langsmith: For tracing and debugging LangChain executions.
  * langchain, langchain_groq, langchain_community: Core and integration libraries for LangChain, including Groq support.

---

### **Loading API Keys from `.env`**

```python
from dotenv import load_dotenv
load_dotenv()
groq_api_key = os.environ.get("GROQ_API_KEY")
langsmith_token = os.environ.get("LANGSMITH_API_KEY")
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"]="CourseLanggraph"
```

* Loads environment variables using dotenv.
* Sets up LangChain to trace execution under the project name "CourseLanggraph".

---

### **Setting up the LLM**

```python
from langchain_groq import ChatGroq
llm = ChatGroq(groq_api_key=groq_api_key, model_name="Gemma2-9b-It")
```

* Connects to **Groq’s hosted model (Gemma2-9b-It)** via LangChain.
* `ChatGroq` is a wrapper for calling the model.

---

### **Building a LangGraph**

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
```

* `StateGraph`: Core class to define computation steps (nodes) and their transitions.
* `START`, `END`: Special markers in the graph.

---

### **Defining the State**

```python
class State(TypedDict):
    messages: Annotated[list, add_messages]
```

* The state holds a list of messages (conversation history).
* `add_messages` ensures messages are appended in sequence automatically.

---

### **Graph Node Definition**

```python
def chatbot(state: State):
    return {"messages": llm.invoke(state["messages"])}
```

* A simple function that passes the state’s messages to the LLM and gets a response.

---

### **Building the Graph**

```python
graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)
graph = graph_builder.compile()
```

* Creates a graph:

  * **Start → chatbot → End**
* Once compiled, this graph can process messages using the `stream()` method.

---

### **Graph Visualization**

```python
display(Image(graph.get_graph().draw_mermaid_png()))
```

* Tries to draw the graph using Mermaid syntax as a PNG (optional and may not work in some environments).

---

### **Running the Chatbot**

```python
while True:
    user_input = input("user: ")
    if user_input.lower() in ["exit", "quit"]:
        print("Exiting the chatbot.")
        break
    for event in graph.stream({"messages":("user", user_input)}):
        print(event.values())
        for value in event.values():
            print(value["messages"])
            print("bot:", value["messages"].content)
```

* Takes user input in a loop.
* Feeds it to the graph using `graph.stream()`, which streams events (state transitions).
* Extracts and prints the bot's response.

---



This notebook builds a **very basic stateful chatbot** using:

* **LangGraph**: To manage the conversation state and control flow.
* **LangChain**: For LLM integration and state logic.
* **Groq (Gemma2-9b-It)**: As the LLM backend.
* **LangSmith (optional)**: For execution tracing and debugging.


![image](https://github.com/user-attachments/assets/748738d0-cff0-4ad9-bf21-d2cef989b216)
## Langsmith Tracing

![image](https://github.com/user-attachments/assets/3d8ab218-3d89-44ad-ac26-4cd5f4858988)

