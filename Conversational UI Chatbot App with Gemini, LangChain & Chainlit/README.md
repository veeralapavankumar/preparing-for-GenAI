# 🤖 Conversational UI Chatbot App with Gemini, LangChain & Chainlit

![Python](https://img.shields.io/badge/Python-3.12-blue?style=for-the-badge&logo=python&logoColor=white)
![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter&logoColor=white)
![Google Gemini](https://img.shields.io/badge/Google-Gemini%20Pro-4285F4?style=for-the-badge&logo=google&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-0.1.12-green?style=for-the-badge)
![Chainlit](https://img.shields.io/badge/Chainlit-1.0.401-blueviolet?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

---

## 📖 Overview

This project demonstrates how to build a **production-ready, conversational AI chatbot** powered by **Google Gemini Pro** using **LangChain** for orchestration and **Chainlit** for a polished, interactive web UI.

The chatbot goes far beyond a minimal viable product — it features conversational memory that allows multi-turn dialogues, real-time streaming of responses token-by-token, and a customizable landing page. Deployment is handled seamlessly on Google Colab with **ngrok** to expose the local server to the public internet.

---

## 🎯 Problem Statement

Building conversational AI assistants with LLMs typically involves three core challenges:

1. **Statelessness** — LLMs do not remember past interactions by default, resulting in context loss across conversation turns.
2. **Latency perception** — Waiting for an entire AI response before displaying it creates a poor user experience.
3. **Deployment complexity** — Serving a full-stack chatbot UI from a cloud notebook environment (e.g., Google Colab) requires an additional tunneling layer.

This project solves all three challenges by combining LangChain's `ConversationBufferWindowMemory` for context retention, Chainlit's streaming callback handler for real-time output, and ngrok for zero-friction public URL exposure.

---

## ✨ Features

* 🧠 **Conversational Memory** — Retains the last 20 exchanges (configurable window `k=20`) to maintain full context across multi-turn conversations
* ⚡ **Real-Time Response Streaming** — Responses are streamed token-by-token to the UI for an instant, ChatGPT-like experience
* 🎨 **Custom Landing Page** — A branded welcome screen rendered via `chainlit.md` with a personalized greeting
* 🔗 **LangChain LCEL Pipeline** — Modular, composable chain built using LangChain Expression Language (LCEL) with `RunnablePassthrough`, `RunnableLambda`, and `StrOutputParser`
* 🌐 **Public URL via ngrok** — Instantly accessible from any browser through a secure ngrok HTTPS tunnel
* 🛡️ **Safe & Honest AI Behaviour** — System prompt instructs the model not to fabricate answers
* 🔑 **Secure API Key Management** — API credentials loaded from YAML files, keeping secrets out of source code

---

## 🛠️ Tech Stack

| Layer | Technology | Version |
|---|---|---|
| **LLM** | Google Gemini Pro | via `google-generativeai 0.4.1` |
| **LLM Orchestration** | LangChain | `0.1.12` |
| **Gemini Integration** | langchain-google-genai | `0.0.11` |
| **Chat UI Framework** | Chainlit | `1.0.401` |
| **Tunneling** | pyngrok | `7.1.5` |
| **Runtime** | Python | `3.12` |
| **Environment** | Google Colab | — |

---

## 📁 Project Structure

```text
conversational-chatbot-gemini/
├── Conversational_UI_Chatbot_App_with_GEMINI,_LangChain_and_Chainlit.ipynb  # Main notebook
├── app.py                     # Auto-generated Chainlit application file
├── chainlit.md                # Custom landing page content for Chainlit UI
├── gemini_key.yml             # Gemini API credentials (NOT committed to Git)
├── ngrok_credentials.yml      # ngrok auth token (NOT committed to Git)
├── logs.txt                   # Chainlit server logs (auto-generated at runtime)
└── README.md                  # Project documentation
```

> ⚠️ **Security Note:** Never commit `gemini_key.yml` or `ngrok_credentials.yml` to a public repository. Add them to `.gitignore`.

---

## ⚙️ Installation

### Prerequisites

* Python 3.9+
* A valid [Google AI Studio API Key](https://aistudio.google.com/app/apikey)
* A valid [ngrok auth token](https://dashboard.ngrok.com/get-started/your-authtoken) (for Colab tunneling)

### Clone & Setup

```bash
git clone https://github.com/<your-username>/conversational-chatbot-gemini.git
cd conversational-chatbot-gemini
pip install -r requirements.txt
```

### Create Credentials Files

Create `gemini_key.yml`:

```yaml
gemini_key: "YOUR_GOOGLE_GEMINI_API_KEY"
```

Create `ngrok_credentials.yml`:

```yaml
ngrok_key: "YOUR_NGROK_AUTH_TOKEN"
```

---

## 📦 Requirements

Generate requirements based on the libraries used in the notebook:

```text
langchain==0.1.12
langchain-google-genai==0.0.11
langchain-core>=0.1.31
langchain-community>=0.0.28
chainlit==1.0.401
pyngrok==7.1.5
pydantic>=2.0
python-dotenv>=1.0.0
pyyaml>=5.1
```

> Save this as `requirements.txt` and install with `pip install -r requirements.txt`.

---

## 🔄 Workflow

```
1. Environment Setup
       │
       ▼
2. Load Gemini API Credentials (from gemini_key.yml)
       │
       ▼
3. Write Chainlit App (app.py via %%writefile)
       │
       ├── Initialize Gemini Pro LLM (streaming=True)
       ├── Define System Prompt
       ├── Build ChatPromptTemplate with MessagesPlaceholder
       ├── Initialize ConversationBufferWindowMemory (k=20)
       └── Compose LCEL Conversation Chain
       │
       ▼
4. Customize Landing Page (chainlit.md)
       │
       ▼
5. Start Chainlit Server (port 8989, background process)
       │
       ▼
6. Expose Server via ngrok Tunnel
       │
       ▼
7. Interact with Chatbot via Public HTTPS URL
       │
       ▼
8. Cleanup (kill ngrok tunnel & Chainlit process)
```

---

## 🧩 Core Components

### Application — `app.py`

The Chainlit application is structured around two core event handlers:

| Handler | Trigger | Responsibility |
|---|---|---|
| `@cl.on_chat_start` → `when_chat_starts()` | App session begins | Initialises Gemini LLM, prompt template, memory, and LCEL chain; stores in user session |
| `@cl.on_message` → `on_user_message()` | User sends a message | Retrieves chain & memory, streams Gemini response in real-time, saves context to memory |

### LangChain LCEL Chain

```
RunnablePassthrough.assign(history=memory.load_memory_variables | itemgetter("history"))
    │
    ▼
ChatPromptTemplate  [SystemMessage + History + HumanMessage]
    │
    ▼
ChatGoogleGenerativeAI  (gemini-pro, temperature=0.1, streaming=True)
    │
    ▼
StrOutputParser  →  Real-time streamed text to Chainlit UI
```

---

## 🤖 Models Used

| Model | Provider | Purpose | Key Settings |
|---|---|---|---|
| `gemini-pro` | Google DeepMind | Conversational response generation | `temperature=0.1`, `streaming=True`, `convert_system_message_to_human=True` |

> **Why `temperature=0.1`?** A low temperature makes the model more deterministic and factually consistent — ideal for an assistant that must not fabricate answers.

---

## 🧠 Memory Management

| Parameter | Value | Effect |
|---|---|---|
| Memory type | `ConversationBufferWindowMemory` | Stores raw message objects |
| Window size `k` | `20` | Keeps the last 20 conversation turns in context |
| `return_messages` | `True` | Returns `HumanMessage` / `AIMessage` objects for LangChain compatibility |

---

## 🌐 Deployment (Google Colab + ngrok)

```bash
# 1. Start the Chainlit server in the background
!chainlit run app.py --port=8989 --watch &> ./logs.txt &

# 2. Expose it publicly via ngrok
# (handled by the pyngrok code block in the notebook)
# Output example:
# Chainlit App: https://rewire-bulldog-frostlike.ngrok-free.dev
```

The notebook outputs a public ngrok URL that you can open in any browser to interact with the chatbot live.

---

## 📊 Results

This is a **generative AI application**, not a supervised ML model — hence traditional metrics (accuracy, F1, RMSE) do not apply. The quality of results is evaluated qualitatively:

| Dimension | Outcome |
|---|---|
| **Context retention** | ✅ Multi-turn conversations maintain coherent context across up to 20 turns |
| **Response quality** | ✅ Factual, concise answers with refusal to fabricate (enforced by system prompt) |
| **Streaming latency** | ✅ First token appears almost instantly; no waiting for full response |
| **UI experience** | ✅ Clean, professional Chainlit chat interface with custom welcome page |
| **Deployment** | ✅ Publicly accessible HTTPS URL generated in seconds via ngrok |

---

## 🖼️ Visualizations

Since this is a chatbot application, visual outputs are in the form of UI screenshots rather than data plots:

| UI Element | Description |
|---|---|
| **Landing Page** | Custom `chainlit.md` welcome screen: *"Welcome I am AI Assistant 🤖 — How can I help you today?"* |
| **Chat Interface** | Real-time token-streaming chat window with user and AI message bubbles |
| **ngrok Public URL** | HTTPS link printed after tunnel setup, e.g., `https://rewire-bulldog-frostlike.ngrok-free.dev` |

---

## 🚀 How to Run

### Option A — Google Colab (Recommended)

1. Open the `.ipynb` notebook in Google Colab
2. Create `gemini_key.yml` and `ngrok_credentials.yml` in the Colab file system
3. Run all cells sequentially
4. Copy the ngrok URL printed in the last cell and open it in your browser

### Option B — Local Machine

```bash
# Step 1: Install dependencies
pip install langchain==0.1.12 langchain-google-genai==0.0.11 chainlit==1.0.401 pyngrok==7.1.5

# Step 2: Set your Gemini API key as an environment variable
export GEMINI_API_KEY="your_api_key_here"

# Step 3: Write the app.py file (copy the app code from the notebook)
# or extract it manually from the %%writefile cell

# Step 4: Run the Chainlit app
chainlit run app.py --port=8989 --watch

# Step 5: Open http://localhost:8989 in your browser
```

---

## 🔮 Future Improvements

* 🗂️ **Document Q&A (RAG)** — Extend the app with a file upload feature and RAG pipeline so users can chat with their own PDFs, CSVs, or Word documents
* 🧑‍💼 **Multi-persona Support** — Add selectable system prompts to switch the AI's role (e.g., tutor, code reviewer, creative writer)
* 🌍 **Multilingual Support** — Leverage Gemini's multilingual capabilities with language detection and dynamic prompt adaptation
* 💾 **Persistent Chat History** — Replace in-memory storage with a database (e.g., SQLite, Firestore) to preserve sessions across restarts
* 📊 **Usage Analytics Dashboard** — Track token usage, session length, and response times with an admin panel
* 🔐 **User Authentication** — Add Chainlit's built-in authentication to restrict access and personalise sessions
* 🧪 **Upgrade to Gemini 1.5 Pro / 2.0 Flash** — Migrate to newer Gemini models with larger context windows and improved capabilities

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

```bash
# 1. Fork the repository
# 2. Create a feature branch
git checkout -b feature/your-feature-name

# 3. Commit your changes
git commit -m "feat: add your feature description"

# 4. Push to your fork and open a Pull Request
git push origin feature/your-feature-name
```

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

<table>
  <tr>
    <td align="center">
      <strong>AI/ML Developer & GenAI Enthusiast</strong><br/>
      <em>Analytics Vidhya — GenAI Pinnacle Program</em><br/><br/>
      📧 Connect on <a href="https://github.com">GitHub</a> &nbsp;|&nbsp;
      💼 Connect on <a href="https://linkedin.com">LinkedIn</a>
    </td>
  </tr>
</table>

> *Built as part of the Analytics Vidhya GenAI Pinnacle Program — Module 3: Building End-to-End Generative AI Applications.*

---

<div align="center">
  <strong>⭐ If you found this project helpful, please give it a star!</strong>
</div>
