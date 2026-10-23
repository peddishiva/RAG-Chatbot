# Enterprise Knowledge Assistant — RAG Chatbot

An AI-powered company knowledge assistant built with **n8n, Retrieval-Augmented Generation (RAG), Google Drive, Pinecone, Gemini Embeddings, and OpenRouter**. The workflow ingests company documents, retrieves relevant passages through semantic search, and uses an AI agent to answer questions grounded in the available document content.

> **Project status:** Core workflow configured and manually tested. Source citations, automated answer evaluation, and additional resilience improvements are planned.

## Overview

Company information often lives in PDFs and internal documents. Finding a specific answer manually can be time-consuming. This project automates document ingestion, embedding generation, vector storage, semantic retrieval, and AI-powered question answering using a visual n8n workflow.

## Current Features

- **Google Drive ingestion:** Watches a dedicated folder for new and updated documents.
- **Document processing:** Downloads files and loads their content with n8n document-processing nodes.
- **Text chunking:** Splits content into manageable passages with a recursive character text splitter.
- **Embedding generation:** Uses Google Gemini Embeddings to represent document chunks as vectors.
- **Vector storage and retrieval:** Stores and searches vectors in Pinecone.
- **AI-powered answers:** Uses OpenRouter-backed chat models for the AI Agent and document-search tool.
- **Chat interface:** Lets users ask questions about ingested company documents in n8n.
- **Source-grounded intent:** Instructs the assistant to answer using retrieved company-document content and acknowledge when the information cannot be found.

## Architecture

### Document ingestion

```text
Google Drive folder
       |
       v
File-created / file-updated trigger
       |
       v
Download document
       |
       v
Default Data Loader
       |
       v
Recursive Character Text Splitter
       |
       v
Gemini Embeddings
       |
       v
Pinecone Vector Store
```

### Question answering

```text
User question
     |
     v
n8n Chat Trigger
     |
     v
AI Agent <---- OpenRouter Chat Model
     |
     v
Vector Store Tool <---- OpenRouter Chat Model
     |
     v
Pinecone retrieval <---- Gemini Embeddings
     |
     v
Retrieved document passages
     |
     v
AI-generated response
```

## Technology Stack

| Technology | Role |
|---|---|
| [n8n](https://n8n.io/) | Workflow orchestration and chat interface |
| [Google Drive](https://drive.google.com/) | Source document storage |
| [Google Gemini API](https://ai.google.dev/) | Document embeddings |
| [Pinecone](https://www.pinecone.io/) | Vector database and semantic retrieval |
| [OpenRouter](https://openrouter.ai/) | Chat-model access for the agent and retrieval tool |
| RAG | Grounds generated answers in retrieved document content |

## How It Works

1. **Watch the source folder:** Google Drive triggers detect document creation or updates.
2. **Download and load:** n8n downloads each file and extracts its content.
3. **Split into chunks:** The text splitter breaks content into smaller passages.
4. **Generate embeddings:** Gemini converts passages into vectors.
5. **Store vectors:** Pinecone stores the vectors for semantic search.
6. **Receive a question:** A user sends a message through the n8n chat interface.
7. **Retrieve relevant content:** The AI Agent uses the Vector Store Tool to search Pinecone.
8. **Generate a response:** The OpenRouter-backed chat model uses the retrieved context to answer.

## Prerequisites

- A running n8n instance (local or hosted).
- Google Drive access and Google OAuth credentials configured in n8n.
- A Google AI Studio API key for embeddings.
- A Pinecone account, API key, and vector index.
- An OpenRouter account and API key.
- An embedding index dimension that matches the embedding output configuration.


## Setup Instructions

Follow these steps to configure the RAG chatbot in your own n8n instance.

### 1. Prerequisites

Create accounts or set up access to the following services:

- **n8n:** A local or hosted n8n instance.
- **Google Drive:** Stores the company documents used by the chatbot.
- **Google AI Studio:** Provides the Gemini API key used for generating document embeddings.
- **Pinecone:** Stores document embeddings and supports semantic retrieval.
- **OpenRouter:** Provides the chat models used by the AI Agent and Vector Store Tool.

### 2. Configure Google Drive

1. Open Google Drive and create a dedicated folder named `Company RAG Documents`.
2. Open your local n8n dashboard.
3. Navigate to **Credentials** and create a Google Drive OAuth2 credential.
4. Follow the Google sign-in and authorization process.
5. If your Google OAuth application is in testing mode, add your Google account under **Google Cloud Console → Google Auth Platform → Audience → Test users**, if required.
6. Open both Google Drive Trigger nodes in the workflow.
7. Configure them to monitor the `Company RAG Documents` folder.
8. Set the trigger events to detect file creation and file updates.
9. Test the triggers to verify that n8n can access the selected folder.

### 3. Configure Google Gemini Embeddings

1. Open [Google AI Studio](https://aistudio.google.com/).
2. Generate a Gemini API key.
3. In n8n, navigate to **Credentials** and create a Google Gemini API credential.
4. Enter the API key and save the credential.
5. Open the `Embeddings Google Gemini` node in the document-ingestion branch.
6. Select the Gemini API credential and configure the embedding model as `models/gemini-embedding-001`.
7. Repeat this configuration for `Embeddings Google Gemini (retrieval)`.
8. Ensure both embedding nodes use compatible output dimensions.

**Important:** The Pinecone index dimension must match the actual embedding output dimension. If you change the embedding model or dimensions later, re-embed and re-ingest the documents.

### 4. Configure Pinecone

1. Create an account at [Pinecone](https://www.pinecone.io/).
2. Open the Pinecone dashboard and create an API key.
3. Create a dense vector index named `company-files`.
4. Configure the index dimension to match the embedding output dimension and select the cosine similarity metric.
5. In n8n, create a Pinecone API credential using your Pinecone API key.
6. Open the `Pinecone Vector Store` node in the ingestion branch.
7. Select the Pinecone credential and the `company-files` index.
8. Open `Pinecone Vector Store (Retrieval)` and select the same credential and index.
9. Save your configuration.

### 5. Configure OpenRouter Chat Models

OpenRouter is used for chat and reasoning instead of the Gemini chat models.

1. Open [OpenRouter](https://openrouter.ai/).
2. Create an API key and keep it private.
3. In n8n, navigate to **Credentials** and create an OpenRouter credential using the API key.
4. Open the `OpenRouter Chat Model` node connected to `AI Agent`.
5. Select your OpenRouter credential and choose an available model. The `openrouter/free` router can be used if available to your account.
6. Open `OpenRouter Chat Model1`, connected to `Vector Store Tool`.
7. Select the same OpenRouter credential and configure an available model.
8. Verify that the first OpenRouter node connects to the AI Agent's **Chat Model** input and the second connects to the Vector Store Tool's **Model** input.
9. Leave the Gemini embedding nodes configured as described in Step 3.

**Note:** OpenRouter free models may have request limits, rate limits, and changing availability.

### 6. Import the Workflow

1. Download the workflow JSON file from this repository.
2. Open your n8n dashboard.
3. Navigate to **Workflows**.
4. Select **Import from File**.
5. Choose the workflow JSON file.
6. Open the imported workflow and configure your own credentials and resource selections.
7. Save the workflow.

### 7. Ingest Company Documents

1. Upload a test PDF to the `Company RAG Documents` folder in Google Drive.
2. Verify that the Google Drive trigger detects the file.
3. Confirm that the download and document-loading nodes execute successfully.
4. Check that the Recursive Character Text Splitter divides the document into chunks.
5. Verify that Gemini embeddings are generated and the chunks are stored in the `company-files` Pinecone index.

### 8. Test the Chatbot

1. Open the chat interface in n8n.
2. Ask a question whose answer is present in the uploaded document.
3. Verify that the AI Agent uses the Vector Store Tool to retrieve relevant passages.
4. Compare the generated answer with the source document.
5. Test a question whose answer is absent from the documents to check how the chatbot handles missing information.

The workflow is ready for further improvements once document ingestion and question answering both complete successfully.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
