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

## Setup

### 1. Import the workflow

1. Download the workflow JSON from this repository once it is published.
2. In n8n, open **Workflows** and select **Import from File**.
3. Select the workflow JSON.
4. Reconfigure credentials in your own n8n instance; credentials are not included in the shared workflow.

### 2. Configure Google Drive

1. Create a folder for the documents you want the chatbot to search.
2. Configure Google Drive OAuth credentials in n8n.
3. Select the correct folder in the Google Drive triggers.
4. Verify that file download and document loading succeed.

### 3. Configure embeddings and Pinecone

1. Add your Gemini API credential to the embedding nodes.
2. Use the same embedding model and output dimension for document ingestion and retrieval.
3. Configure the Pinecone credential and select your vector index.
4. Ensure that the Pinecone index dimension matches the embedding output.
5. Re-ingest documents after changing the embedding model or vector dimension.

### 4. Configure OpenRouter

1. Create an OpenRouter API key.
2. Add the credential in n8n.
3. Configure both OpenRouter Chat Model nodes: one for the AI Agent and one for the Vector Store Tool.
4. Select a model available to your OpenRouter account. Free models can have usage limits, rate limits, and availability changes.

### 5. Test

1. Add a sample document to your configured Google Drive folder.
2. Confirm that ingestion completes and the vectors are stored.
3. Open the n8n chat.
4. Ask a question that can be answered from the source document.
5. Compare the response against the original document for completeness and accuracy.

## Security Notes

- **Never commit API keys, OAuth tokens, passwords, or other secrets.**
- Review exported workflow JSON before publishing. Remove credential values, tokens, private URLs, and sensitive node data where present.
- n8n credential references may remain in the workflow export, but actual secrets should be managed through n8n credentials and configured separately after import.
- Do not publish confidential company documents or private source material.
- Prefer synthetic or non-sensitive example documents for public demonstrations.

## Current Limitations and Planned Improvements

The current version demonstrates core document ingestion, vector search, and question answering. Further improvements are planned:

- **Source citations:** Return document names and relevant passage references with answers.
- **Error handling:** Improve handling of model outages, rate limits, and failed ingestion.
- **Answer evaluation:** Build a repeatable test set to assess relevance and faithfulness.
- **Retrieval tuning:** Evaluate chunk sizes, overlap, and retrieval count.
- **Fallback behavior:** Clearly indicate when the available documents do not contain an answer.
- **Demonstration assets:** Add screenshots and example questions after final validation.

## Repository Contents

The repository will contain the sanitized n8n workflow export and this documentation. Importing the workflow does not provision external services or credentials automatically; you must configure those in your own n8n instance.

## License

No license has been added yet. Unless a license is provided, standard copyright applies and reuse permissions are not granted automatically.
