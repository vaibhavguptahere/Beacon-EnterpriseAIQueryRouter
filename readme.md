# Beacon — Enterprise AI Query Router

**One conversational interface for HR, IT, and Finance knowledge.**

Beacon is an AI-powered enterprise assistant designed to help employees find answers across different organizational knowledge bases through a single chat interface. Instead of searching through multiple documents, policies, and departmental resources manually, users can ask questions in natural language and receive context-aware answers grounded in relevant enterprise documents.

The project combines **intent classification, intelligent query routing, Retrieval-Augmented Generation (RAG), local LLM-based orchestration, and cloud-based AI services** to make enterprise information easier to access.

---

## ✨ Key Features

- **Unified AI Chat Interface:** Ask questions about HR, IT, and Finance from one conversational interface.
- **Intelligent Query Classification:** Identify the likely intent and domain of an incoming query before retrieving information.
- **Two-Tier Classification:** Use a lightweight classification path for straightforward queries and an Ollama-based fallback for ambiguous or more complex queries.
- **Domain-Based Routing:** Route queries to the relevant departmental knowledge base.
- **Retrieval-Augmented Generation (RAG):** Retrieve relevant document sections and use them as context for generating answers.
- **Hybrid Search:** Combine keyword-based and vector-based retrieval through Azure AI Search.
- **Grounded Responses:** Generate answers using retrieved enterprise context instead of relying only on the language model's general knowledge.
- **Source References:** Return retrieved source information alongside the generated answer.
- **Authentication:** Use Supabase Authentication to support user login and signup.
- **Chat Persistence:** Store chat sessions and individual messages in PostgreSQL through Supabase.
- **Modern Interface:** Responsive Next.js interface with animations and interactive visual elements.

---

## 🧠 How Beacon Works

Beacon follows a classification → routing → retrieval → generation workflow.

```text
                  USER
                    |
                    v
          Next.js Chat Interface
                    |
                    v
              FastAPI Backend
                    |
                    v
           Intent Classification
                    |
          +---------+----------+
          |                    |
          v                    v
    Fast Classifier       Ollama Fallback
          |             (Ambiguous Queries)
          +---------+----------+
                    |
                    v
              Query Router
                    |
          +---------+---------+
          |         |         |
          v         v         v
         HR         IT      Finance
          |         |         |
          +---------+---------+
                    |
                    v
            Azure AI Search
          (Hybrid Retrieval)
                    |
                    v
          Retrieved Document
                Context
                    |
                    v
          Azure OpenAI Generation
                    |
                    v
          Answer + Source Details
                    |
                    v
             Chat Interface
```

### 1. User Query

The user enters a natural-language question through the Next.js chat interface.

Example questions:

- "How many days of annual leave can I take?"
- "How do I reset my company VPN?"
- "What is the reimbursement policy?"

### 2. Intent Classification

The FastAPI backend analyzes the incoming query to determine its intent and relevant domain.

Beacon uses a two-tier classification approach:

- **Fast classification path:** A lightweight classifier uses sentence embeddings and a trained classification model to identify familiar query patterns.
- **Fallback classification path:** When a query is ambiguous or requires more contextual reasoning, the system can use a local Ollama model named `beacon-router`.

The classification and routing layer also uses structured Pydantic schemas to represent routing outcomes, including single-intent, multi-intent, clarification-required, and out-of-scope cases.

### 3. Query Routing

Based on the classification result, the orchestration layer selects the relevant domain:

- HR — employee policies and human resources information.
- IT — technical support and IT-related documentation.
- Finance — financial policies and related information.

This separates query understanding from document retrieval and makes the system easier to extend with additional domains.

### 4. Knowledge Retrieval

The RAG pipeline converts the query into an embedding and searches the configured Azure AI Search index.

Beacon uses hybrid retrieval, combining:

- Keyword-based search.
- Vector-based semantic search.

The retrieved document sections provide relevant context for answering the user's question.

### 5. Answer Generation

The retrieved context is passed to Azure OpenAI for answer generation.

The generation prompt is designed to keep responses grounded in the retrieved information and avoid unsupported answers.

The backend returns the generated answer along with available source details to the frontend.

---

## 🤖 Models and AI Technologies

Beacon combines local models with cloud AI services.

| Component | Model / Technology | Purpose |
|---|---|---|
| Fast intent classification | Sentence Transformers — `all-MiniLM-L6-v2` | Convert queries into sentence embeddings |
| Classification | Trained classification model (`intent_classifier.pkl`) | Identify likely query intent |
| Fallback classification | BART zero-shot classification | Support classification when the fast path is insufficient |
| Local LLM orchestration | Ollama — `beacon-router` | Handle ambiguous and complex routing decisions |
| Embeddings | Azure OpenAI embeddings deployment | Generate embeddings for document and query retrieval |
| Vector and keyword retrieval | Azure AI Search | Retrieve relevant enterprise document context |
| Answer generation | Azure OpenAI deployment | Generate context-grounded responses |

**Why use both local and cloud models?**

The local Ollama model provides a self-hosted option for query classification and fallback reasoning. Azure AI services handle the embedding, search, and answer-generation components of the RAG pipeline.

This combination allows Beacon to use local orchestration alongside managed cloud AI infrastructure.

---

## 🏗️ Technology Stack

### Frontend

- Next.js 15 — App Router
- React 19
- TypeScript
- Tailwind CSS v4
- Vanilla CSS
- Framer Motion
- GSAP
- Three.js through `@react-three/fiber` and `@react-three/drei`
- Radix UI Primitives
- Supabase SSR and Supabase JavaScript Client

### Backend

- Python 3.10+
- FastAPI
- Pydantic
- Ollama
- Transformers
- PyTorch
- Sentence Transformers
- SentencePiece

### AI and Retrieval

- Azure OpenAI
- Azure AI Search
- Hybrid keyword and vector search
- Retrieval-Augmented Generation (RAG)
- Ollama local model orchestration

### Database and Authentication

- Supabase
- PostgreSQL
- Supabase Authentication
- JWT-based authentication
- Chat and message persistence

---

## 📂 Project Structure

The repository is organized into frontend, backend, orchestration, and RAG components.

```text
Beacon/
│
├── app/
│   ├── auth/
│   │   └── page.tsx
│   │
│   └── assistant/
│       └── page.tsx
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   └── routes/
│   │   │       └── chat.py
│   │   │
│   │   ├── orchestration/
│   │   │   ├── classifier.py
│   │   │   └── router.py
│   │   │
│   │   ├── schemas/
│   │   │   └── routing.py
│   │   │
│   │   ├── rag/
│   │   │   ├── search.py
│   │   │   ├── generator.py
│   │   │   └── pipeline.py
│   │   │
│   │   └── models/
│   │       └── intent_classifier.pkl
│   │
│   ├── ingestion/
│   │   └── ingest.py
│   │
│   └── data/
│
├── public/
├── package.json
└── README.md
```

*This is a simplified view of the main components; the repository may contain additional configuration, UI, and supporting files.*

---

## 🚀 Run Beacon on Your Local Machine

You can fork this repository and run your own local instance of Beacon. The application requires frontend and backend configuration, a Supabase project, and access to the configured Azure AI services.

### Prerequisites

Install the following:

- Node.js and npm
- Python 3.10 or later
- Git
- Ollama
- An Ollama model named `beacon-router`, or the equivalent model configuration
- A Supabase project
- Azure AI Search service and search index
- Azure OpenAI resource with the required embedding and chat deployments

### 1. Fork and Clone the Repository

Fork the repository using GitHub's **Fork** button.

Then clone your fork:

```bash
git clone https://github.com/<your-username>/Beacon-EnterpriseAIQueryRouter.git

cd Beacon-EnterpriseAIQueryRouter
```

### 2. Configure the Backend

Navigate to the backend directory:

```bash
cd backend
```

Create and activate a virtual environment.

**Windows:**

```bash
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

Install the backend dependencies:

```bash
pip install -r requirements.txt
```

If your checkout does not contain a `requirements.txt`, install the dependencies listed in the backend's dependency configuration.

### 3. Configure Environment Variables

Create the backend environment file using the variable names expected by the application.

You will need to configure the relevant credentials and deployment details for:

- Azure OpenAI endpoint and API key.
- Azure OpenAI embedding and generation deployment names.
- Azure AI Search endpoint, API key, and index name.
- Ollama connection settings.
- Any other backend configuration required by the application.

Never commit API keys, database credentials, or other secrets to GitHub.

### 4. Set Up Ollama

Install Ollama from:

https://ollama.com/

Start the Ollama service and make sure the configured `beacon-router` model is available locally.

If you have the model definition used by this project, build it with:

```bash
ollama create beacon-router -f Modelfile
```

Start or verify the model:

```bash
ollama run beacon-router
```

The model name and Modelfile must match the configuration in the backend.

### 5. Configure Supabase

Create a Supabase project:

https://supabase.com/

Configure the frontend and backend with the appropriate Supabase project URL and keys.

The application uses Supabase for authentication and PostgreSQL-backed application data, including:

- `public.profiles`
- `public.chats`
- `public.messages`

Make sure the database schema and any required authentication triggers and access policies are configured before using the application.

### 6. Configure Azure AI Services

Create and configure:

1. An Azure AI Search service and the required search index.
2. An Azure OpenAI resource.
3. An embedding deployment for document and query embeddings.
4. A chat/completion deployment for answer generation.

Add the corresponding endpoint, credentials, deployment names, and index configuration to the backend environment.

The RAG pipeline depends on these services. Running the frontend and FastAPI server alone is not sufficient to reproduce cloud-backed retrieval and generation.

### 7. Run the Backend

From the backend directory, start the FastAPI application using its configured entry point.

For example, if the application entry point is `app.main:app`:

```bash
uvicorn app.main:app --reload
```

The API documentation is generally available at:

```text
http://127.0.0.1:8000/docs
```

Check the actual application entry point and configured port in your checkout before running the command.

### 8. Run the Frontend

Open a second terminal and navigate to the project root:

```bash
npm install
```

Create the frontend environment file with the Supabase URL, public key, and backend API URL expected by the application.

Start the development server:

```bash
npm run dev
```

Open:

```text
http://localhost:3000
```

Create an account or sign in, then start asking questions through the assistant interface.

---

## 💬 Example Queries

Once the required services and departmental documents are configured, you can try questions such as:

| Domain | Example Query |
|---|---|
| HR | What is the annual leave policy? |
| HR | What is the process for requesting leave? |
| IT | How can I troubleshoot VPN connectivity? |
| IT | What should I do if I forget my system password? |
| Finance | What is the employee reimbursement process? |
| Finance | Which expenses are eligible for reimbursement? |

The answers depend on the documents indexed in your Azure AI Search knowledge base. These examples are illustrative; the system does not guarantee an answer unless the relevant information exists in the indexed documents.

---

## 📄 Document Ingestion

Beacon includes a document ingestion component for preparing departmental knowledge sources for retrieval.

The ingestion workflow is designed to:

1. Read supported PDF documents.
2. Extract their text.
3. Divide the extracted content into manageable sections.
4. Generate embeddings using the configured Azure OpenAI embedding deployment.
5. Index the processed content in Azure AI Search.

The repository includes sample HR, IT, and Finance documents under `backend/data`.

To use Beacon with your own organization's information, configure the ingestion process and search index for your documents. Ensure that you have permission to process and use those documents.

---

## 🔐 Security and Configuration Notes

- Keep API keys and service credentials in environment variables.
- Do not commit `.env` files or secrets to a public repository.
- Configure Supabase authentication and database access policies for your deployment.
- Restrict access to sensitive HR, IT, and Finance documents.
- Use appropriate Azure AI Search and Azure OpenAI access controls.
- Do not index confidential company documents without authorization.

---

## 🔧 Customization and Extension

Beacon is designed to support additional enterprise knowledge domains.

Potential extensions include:

- Adding new departmental knowledge bases.
- Expanding intent categories and routing rules.
- Improving multi-intent handling.
- Adding document formats and ingestion pipelines.
- Supporting additional embedding or language models.
- Introducing role-based access to departmental information.
- Adding monitoring, evaluation, and feedback mechanisms.

The orchestration layer separates classification, routing, and retrieval, making these components easier to modify independently.

---

## 🤝 Contributing

Contributions, improvements, and suggestions are welcome.

If you want to contribute:

1. Fork the repository.
2. Clone your fork.
3. Create a feature branch.
4. Make and test your changes.
5. Commit and push your changes.
6. Open a Pull Request describing your contribution.

Example:

```bash
git checkout -b feature/your-feature
git add .
git commit -m "Add your feature"
git push origin feature/your-feature
```

---

## 👥 Team

**Beacon — Enterprise AI Query Router**

Developed as a team project at Bennett University.

---

## 📌 Project Status

Beacon is an evolving project focused on enterprise query understanding, domain-based routing, and document-grounded AI responses.

The current implementation combines a Next.js frontend, FastAPI backend, local Ollama-based orchestration, Supabase authentication and persistence, and Azure-powered RAG services.

Some advanced orchestration capabilities, such as fully parallel multi-domain retrieval and synthesized multi-agent responses, can be developed further as the project evolves.

---

**Built to make enterprise knowledge easier to find, understand, and use.**