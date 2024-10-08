![CompleteFlow Logo](https://github.com/atchai/completeflow-agents/blob/main/frontend/public/assets/completeflow-logo.png?raw=true)
# CompleteFlow Agents

CompleteFlow Agents is a fully private conversational AI framework powered by state-of-the-art LLMs that can even be run locally on CPU or GPU.

## Features ✨

- 🖥️ **UI for interacting with agents**
- 🧑‍💼 **Delegator / supervisor agent for directing requests to the appropriate agent**
- 🧩 **Modular framework that allows for adding new agents**
- 🤖 **3 agents packaged - RAG agent, data agent, swap agent**
- 🔐 **Support for browser crypto wallets**
- 📁 **Support for file uploading**
- 🐳 **Containerized with Docker**


## Pre-requisites

* [Download Ollama](https://ollama.com/ )for your operating system
* Then after finishing installation pull these two models:

```ollama pull llama3```

```ollama pull nomic-embed-text```


## Run with Docker Compose

Docker compose will build and run two containers.  One will be for the agents, the other will be for the UI. 

```docker-compose up```

or for Apple silicon

```docker-compose -f docker-compose-apple.yml up```

Open in the browser: ```http://localhost:3333/```

Docker build will download the model.  The first time that one of the agents are called, the model will be loaded into memory and this instance will be shared between all agents.

## Agents
Three agents are included:

### RAG Agent
This agent will answer questions about an uploaded PDF file.

### Data Agent

This agent provides real-time pricing and other cryptoasset metrics by pulling data from CoinGecko and DefiLlama APIs.

It currently supports the following metrics:

- current price of coins
- current price of NFT collections
- market cap
- fully diluted valuation
- total value locked

It is possible to ask questions about assets by referring to them either by their name or their ticker symbol.

### Swap Agent
This agent will enable you to perform swaps between cryptoassets. It should be used with the accompanying UI which provides a browser-based front-end to chat with the agent, display quotes and sign transactions.

A typical flow looks like this:

- User requests a swap, e.g "I want to swap ETH for USDC"
- The agent requests any missing information, e.g. in this case the amount is missing
- Once all the information hase been collected, the agent looks up the assets on the current chain, retrieves contract addresses and generates a quote if available.
- The quote is shown to the user, who may either proceed or cancel
- If the user accepts the quote, the swap may proceed. The back-end will generate transactions which will be sent to the front-end to be signed by the user's wallet.
- If the allowance for the token being sold is too low, an approval transaction will be generated first

Chains supported:
- Ethereum
- Arbitrum
- Optimism
- Polygon
- BSC



# Delegator
The Delegator handles user queries by analyzing the prompt and delegating it to the appropriate agent.

## API Endpoints

1. **Chat Functionality**
   - Endpoint: `POST /`
   - Handles chat interactions, delegating to appropriate agents when necessary.

2. **Message History**
   - Endpoint: `GET /messages`
   - Retrieves chat message history.

3. **Clear Messages**
   - Endpoint: `GET /clear_messages`
   - Clears the chat message history.

4. **Swap Operations**
   - Endpoints:
     - `POST /tx_status`: Check transaction status
     - `POST /allowance`: Get allowance
     - `POST /approve`: Approve transaction
     - `POST /swap`: Perform swap

5. **File Upload**
   - Endpoint: `POST /upload`
   - Handles file uploads for RAG (Retrieval-Augmented Generation) purposes.



# Adding a New Agent

## Overview

Each agent is configured in the `config.py` file, which specifies the agent's path, class, and other details.

## Steps to Add a New Agent

### 1. Create a New Agent Folder

1. **Create a new folder** in the `agents/src` directory for your new agent.
2. **Implement the agent logic** within this folder. Ensure that the agent class is defined and ready to handle the specific type of queries it is designed for.

### 2. Update `config.py`

1. **Open the `config.py` file** located in the `agents/src` directory.
2. **Add a new entry** in the `DELEGATOR_CONFIG` dictionary with the following details:
   - `path`: The path to the agent's module.
   - `class`: The class name of the agent.
   - `detail`: A description of when to use this agent.
   - `name`: A unique name for the agent.
   - `upload`: A boolean indicating if the agent requires a file to be uploaded from the front-end before it should be called.

#### Example:
```python:agents/src/config.py
DELEGATOR_CONFIG = {
    "agents": [
        # ... existing agents ...
        {
            "path": "new_agent.src.agent",
            "class": "NewAgent",
            "description": "if the prompt is related to new functionality, choose new agent",
            "name": "new agent",
            "upload": false
        }
    ]
}
```


### 3. Implement Agent Logic

1. **Define the agent class** in the specified path.
2. **Ensure the agent can handle the queries** it is designed for.

#### Example:
```python:agents/src/new_agent/src/agent.py
class NewAgent:
    def __init__(self, agent_info, llm, llm_ollama, embeddings, flask_app):
        """
        Initialize the NewAgent.

        Parameters:
        - agent_info (dict): Configuration details for the agent.
        - llm (object): The main language model instance.
        - llm_ollama (object): An additional language model instance from Ollama.
        - embeddings (object): Embedding model for handling vector representations.
        - flask_app (Flask): The Flask application instance.
        """
        self.agent_info = agent_info
        self.llm = llm
        self.llm_ollama = llm_ollama
        self.embeddings = embeddings
        self.flask_app = flask_app

    def chat(self, request):
        # Implement chat logic
        pass

    # Add other methods as needed
```


### 4. Handle Multi-Turn Conversations

Agents can handle multi-turn conversations by returning a next_turn_agent which indicates the name of the agent that should handle the next turn.

#### Example:
```python
class NewAgent:
    def __init__(self, agent_info, llm, llm_ollama, embeddings, flask_app):
        """
        Initialize the NewAgent.

        Parameters:
        - agent_info (dict): Configuration details for the agent.
        - llm (object): The main language model instance.
        - llm_ollama (object): An additional language model instance.
        - embeddings (object): Embedding model for handling vector representations.
        - flask_app (Flask): The Flask application instance.
        """
        self.agent_info = agent_info
        self.llm = llm
        self.llm_ollama = llm_ollama
        self.embeddings = embeddings
        self.flask_app = flask_app

    def chat(self, request, user_id):
        # Process the query and determine the next agent
        next_turn_agent = self.agent_info["name"]
        
        # Generate response where we want to initiate a multi-turn conversation with the same agent.

        return response, next_turn_agent

```

### 5. Integration

The `Delegator` will automatically:
- Import the agent module.
- Instantiate the agent class.
- Add the agent to its internal dictionary.

### 6. Test the New Agent

1. **Ensure the `Delegator` can properly route requests** to the new agent.
2. **Test the agent's functionality** through the chat interface.
