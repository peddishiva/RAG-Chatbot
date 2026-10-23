# RAG Chatbot for Company Documents - Version 1

A Retrieval-Augmented Generation (RAG) chatbot built with **n8n**,
**Google Drive**, **Google Gemini embeddings**, **Pinecone**, and
**OpenRouter**. It ingests company documents from a dedicated Google
Drive folder, stores their embeddings in Pinecone, and retrieves
relevant document content to help answer questions in a chat interface.

> **Version 1 is the baseline workflow.** It demonstrates document
> ingestion and retrieval-based question answering. It is a
> learning/demo workflow and should be tested and hardened before
> production use.

## Features

-   Watches a Google Drive folder for newly created and updated files.
-   Downloads documents from Google Drive.
-   Loads document content and splits it into smaller chunks.
-   Uses Google Gemini embeddings to represent document chunks as
    vectors.
-   Stores document vectors in a Pinecone index named `company-files`.
-   Uses a Vector Store Tool to retrieve relevant company-document
    passages.
-   Uses an OpenRouter chat model with an n8n AI Agent to generate
    answers.
-   Includes a prompt instructing the assistant not to invent answers
    when the information is missing from the available documents.

## Architecture

### Document ingestion

1.  A Google Drive trigger detects a new or updated file in
    `Company RAG Documents`.
2.  n8n downloads the file.
3.  The Default Data Loader reads the binary file.
4.  The Recursive Character Text Splitter divides the content into
    chunks.
5.  Google Gemini generates embeddings for the chunks.
6.  The Pinecone Vector Store inserts the embedded documents into
    `company-files`.

### Question answering

1.  A user sends a question through the n8n chat trigger.
2.  The AI Agent receives the question.
3.  The agent uses the `company_documents_tool` Vector Store Tool to
    search the Pinecone index.
4.  Gemini embeddings are used to embed the retrieval query.
5.  The retrieved document content is provided to the OpenRouter chat
    model, which generates a response.

## Technology Stack

-   **Workflow automation:** n8n
-   **Document source:** Google Drive
-   **Embeddings:** Google Gemini (`models/gemini-embedding-001`, as
    intended in the setup)
-   **Vector database:** Pinecone (`company-files`)
-   **Chat model provider:** OpenRouter
-   **RAG orchestration:** n8n AI Agent and Vector Store Tool

## Requirements

Before importing the workflow, prepare:

-   An n8n instance (local or hosted).
-   A Google Drive folder for the source documents.
-   Google Drive OAuth2 credentials configured in n8n.
-   A Google AI Studio API key and Google Gemini credential in n8n.
-   A Pinecone account, API key, and dense vector index named
    `company-files`.
-   An OpenRouter API key and credential in n8n.

The Pinecone index dimension must match the actual output dimension of
the embedding model used. The ingestion and retrieval embedding
configurations must be compatible.

## Setup

### 1. Import the workflow

1.  Download `rag-chatbot-v1.json` from this folder.
2.  Open your n8n dashboard.
3.  Go to **Workflows** and choose **Import from File**.
4.  Select the JSON file and save the imported workflow.

### 2. Configure Google Drive

1.  Create a Google Drive folder named `Company RAG Documents`.
2.  Open both Google Drive Trigger nodes.
3.  Select your own folder in each trigger.
4.  Configure one trigger for file creation and the other for file
    updates.
5.  Select your own Google Drive OAuth2 credential.
6.  Test that n8n can access the folder.

### 3. Configure Gemini embeddings

1.  Create or select your Google Gemini API credential in n8n.
2.  Configure both embedding nodes: `Embeddings Google Gemini` and
    `Embeddings Google Gemini (retrieval)`.
3.  Select the intended embedding model, `models/gemini-embedding-001`,
    where available.
4.  Confirm that the output dimensions are compatible with the Pinecone
    index.

### 4. Configure Pinecone

1.  Create or select a Pinecone API credential in n8n.
2.  Create a dense index named `company-files`.
3.  Set the index dimension to match the embedding output and use cosine
    similarity if appropriate for your configuration.
4.  Select the same credential and index in both Pinecone Vector Store
    nodes.

### 5. Configure OpenRouter

1.  Create an OpenRouter API key and add it as an n8n credential.
2.  Configure `OpenRouter Chat Model1` for the AI Agent.
3.  Configure `OpenRouter Chat Model2` for the Vector Store Tool.
4.  Select a model available to your OpenRouter account. The
    `openrouter/free` router may be an option, but free-model
    availability and limits can change.

### 6. Test document ingestion

1.  Upload a test PDF or supported document to `Company RAG Documents`.
2.  Confirm the trigger detects the file.
3.  Check that the download, loader, text splitter, embedding, and
    Pinecone insertion nodes succeed.
4.  Verify that the document can be retrieved through the chat workflow.

### 7. Test the chatbot

1.  Open the n8n chat interface for the workflow.
2.  Ask a question whose answer is explicitly stated in the uploaded
    document.
3.  Compare the answer with the source document.
4.  Ask a question whose answer is not present and check that the
    assistant acknowledges the missing information.

## Current limitations

-   **Document updates may leave stale vectors:** The current ingestion
    path inserts content into Pinecone but does not implement a complete
    delete-and-replace process for previously stored chunks.
-   **Document deletion is not handled:** Removing a file from Google
    Drive does not automatically remove its vectors from Pinecone in
    this baseline workflow.
-   **Duplicate ingestion is possible:** Re-triggering ingestion for the
    same file may create duplicate or redundant chunks unless document
    identity and cleanup are implemented.
-   **Model availability may vary:** OpenRouter model availability, rate
    limits, and free-tier conditions can change.
-   **Production hardening is still needed:** Add robust error handling,
    logging, monitoring, access controls, and evaluation before using
    the workflow with sensitive company data.

## Security notes

-   Never commit API keys, OAuth tokens, passwords, or other secrets to
    GitHub.
-   Configure your own credentials after importing the workflow.
-   Use a sanitized workflow export for public sharing; do not publish
    personal folder IDs or unnecessary instance-specific identifiers.
-   Only upload documents you are authorized to process. Review the
    privacy and retention requirements of the services you use.

## Version 1 scope

Version 1 establishes the baseline RAG pipeline: **Google Drive →
document loading and chunking → Gemini embeddings → Pinecone → retrieval
tool → OpenRouter chat response**.

Planned improvements for later versions may include reliable document
update handling, deletion synchronization, duplicate prevention, better
source references, and stronger error handling.
