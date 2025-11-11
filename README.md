# Newsbot360

![Newsbot360 Logo](https://github.com/narenSb1837/Newsbot360/assets/89464601/c06f78d6-454b-4e0a-a4ce-00a94726b2fb)

## Overview

Newsbot360 is a sophisticated chatbot designed to provide users with real-time economic news data. It scrapes information from top economic news sites such as MoneyControl and Economic Times, allowing users to ask questions about specific events.

The chatbot is powered by various technologies, including Beautiful Soup for web scraping, the Langchain framework for RAG (Retrieval-Augmented Generation) pipeline implementation, Cohere embeddings for Pinecone vector database, and LLM Cohere for data processing. The incorporation of hypothetical vector embeddings ensures optimized performance.

## Website Output

![Screenshot (795)](https://github.com/narenSb1837/Newsbot360/assets/89464601/00d03b50-c7f4-49de-8471-92490d6401a8)
![Screenshot (789)](https://github.com/narenSb1837/Newsbot360/assets/89464601/1e5cf683-b035-4c4b-9fde-3616e25b4a22)
![Screenshot (790)](https://github.com/narenSb1837/Newsbot360/assets/89464601/c94cb982-58eb-4b99-8816-8dbcbfa41f8e)

## Features

- **Web Scraping:** Utilizes Beautiful Soup for efficient web scraping of economic news from sites like MoneyControl and Economic Times.
- **RAG Pipeline:** Implements the Langchain framework for an advanced Retrieval-Augmented Generation pipeline.
- **Embeddings:** Leverages Voyage AI embeddings for Pinecone vector database, enhancing data representation.
- **Multi-Query Retrieval:** Uses Mixtral-8x7B-Instruct to generate query variations for better document retrieval.
- **Contextual Reranking:** Employs Cohere's reranking model to improve result relevance.
- **Conversation Memory:** Stores chat history in MongoDB for context-aware responses.
- **Streamlit Deployment:** The chatbot is deployed using Streamlit, providing an interactive and user-friendly interface.

## 📚 Documentation

This project includes comprehensive documentation to help you understand and work with Newsbot360:

- **[DOCUMENTATION.md](DOCUMENTATION.md)** - 📖 Complete documentation guide
  - Navigation help for all documentation
  - Recommended reading order
  - Quick reference and use cases
  
- **[SETUP.md](SETUP.md)** - ⚙️ Installation and configuration guide
  - Prerequisites and API key requirements
  - Step-by-step installation instructions
  - Configuration options and troubleshooting
  
- **[WORKFLOW.md](WORKFLOW.md)** - 🔄 Step-by-step process walkthrough
  - Daily news scraping process
  - Vector embedding generation
  - User query processing flow
  - Complete example scenarios
  
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - 🏗️ Technical architecture and system design
  - Detailed component descriptions
  - Technology stack explanation
  - Data flow diagrams
  - Design decisions and trade-offs

**New to Newsbot360?** Start with [DOCUMENTATION.md](DOCUMENTATION.md) for guidance on which documents to read first.

## Quick Start

1. Clone the repository:
```bash
git clone https://github.com/naren-balu/Newsbot360.git
cd Newsbot360
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Configure API keys in `.env` file (see [SETUP.md](SETUP.md) for details)

4. Run the application:
```bash
streamlit run app.py
```

For detailed setup instructions, please refer to [SETUP.md](SETUP.md).

## How It Works

Newsbot360 follows this workflow:

1. **Daily Scraping**: Automatically scrapes latest news from MoneyControl.com
2. **Text Processing**: Cleans and chunks news articles into manageable segments
3. **Embedding Generation**: Converts text to semantic vectors using Voyage AI
4. **Vector Storage**: Stores embeddings in Pinecone for fast similarity search
5. **Query Processing**: Uses multi-query retrieval and reranking for accurate results
6. **Response Generation**: Mixtral-8x7B generates natural language answers from retrieved context
7. **Memory Management**: Stores conversation history in MongoDB for context-aware interactions

For a detailed step-by-step walkthrough, see [WORKFLOW.md](WORKFLOW.md).

## Technology Stack

### AI/ML Services
- **Voyage AI** - Semantic embeddings
- **Mixtral-8x7B-Instruct** (via HuggingFace) - Query expansion and response generation
- **Cohere** - Result reranking and optional hypothetical embeddings

### Storage & Databases
- **Pinecone** - Vector database for semantic search
- **MongoDB Atlas** - Conversation history storage
- **SQLite** - LLM response caching

### Frameworks & Libraries
- **Streamlit** - Web interface
- **LangChain** - RAG pipeline orchestration
- **Beautiful Soup** - Web scraping
- **Requests** - HTTP client

For detailed architecture information, see [ARCHITECTURE.md](ARCHITECTURE.md).
