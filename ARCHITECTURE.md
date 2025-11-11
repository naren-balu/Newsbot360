# Newsbot360 Architecture

## Overview
Newsbot360 is a conversational AI system that provides real-time economic news information through a RAG (Retrieval-Augmented Generation) pipeline. The system scrapes, processes, and stores news data, then uses advanced NLP techniques to answer user queries.

## System Components

### 1. Core Application (`app.py`)
The main entry point built with Streamlit that provides the web interface.

**Key Functions:**
- Creates the Streamlit web interface
- Checks if today's news has been scraped (stored in `files/YYYY-MM-DD.txt`)
- Manages chat session state and message history
- Routes user queries to the chatbot system
- Displays responses in a conversational interface

**Flow:**
1. On startup, checks if today's news file exists
2. If not, triggers web scraping automatically
3. Initializes/loads chat message history from session state
4. Processes user input through the chatbot
5. Displays responses and maintains conversation context

### 2. Web Scraper (`webscrape.py`)
Automated news extraction system targeting MoneyControl.com.

**Key Features:**
- Scrapes multiple news sections (mutual funds, economy, stocks, markets)
- Handles pagination automatically
- Implements retry logic for failed requests
- Uses browser-like headers to avoid blocking
- Cleans and formats extracted news text

**Process:**
1. Reads URLs from `files/moneycontrol_urls.txt`
2. For each URL section:
   - Iterates through paginated results
   - Extracts article metadata (title, date, link)
   - Follows article links to get full content
   - Removes unnecessary HTML elements (scripts, styles)
   - Cleans text by removing "Related stories" and extra whitespace
3. Formats each article: "The news published on [date] with headline [title] is: [content]"
4. Saves to daily text file
5. Triggers vector embedding process

**Error Handling:**
- Request timeout handling with 20-second timeout
- 80-second sleep on connection errors
- Maximum of 15,655 pages per section
- Graceful handling of missing elements

### 3. Vector Embedding System (`vect_embed.py`)
Creates and manages semantic embeddings for efficient news retrieval.

**Key Components:**

#### a) Text Processing
- Uses `RecursiveCharacterTextSplitter` to split news into chunks
- Chunk size: 1500 characters
- Chunk overlap: 200 characters
- Separates on double newlines (`\n \n`)

#### b) Embedding Generation
- Uses **Voyage AI embeddings** for semantic representation
- Alternative: Hypothetical Document Embeddings (HyDE) with Cohere LLM
- HyDE generates hypothetical answers to improve retrieval accuracy

#### c) Vector Storage
- **Pinecone** vector database (index: "trial")
- Stores embeddings for fast similarity search
- Supports incremental updates with new daily data

#### d) Caching
- SQLite cache (`.langchain.db`) for LLM responses
- Reduces API calls and improves response time

**Functions:**
- `vector_embedding(file_name)`: Processes new scraped data and creates embeddings
- `create_hypothetical_chain()`: Creates LLM chain for HyDE approach
- `create_cache()`: Initializes SQLite cache for LangChain

### 4. Chatbot Logic (`chtbot.py`)
Implements the conversational AI interface using a RAG pipeline.

**Architecture:**

#### a) Multi-Query Retrieval
- Uses **MultiQueryRetriever** to generate multiple query variations
- Increases recall by searching from different perspectives
- Powered by Mixtral-8x7B-Instruct model via HuggingFace Hub

#### b) Contextual Compression
- **CohereRerank** compressor refines retrieved results
- Re-ranks documents by relevance to the query
- Reduces noise and improves answer quality

#### c) Conversational Chain
- **ConversationalRetrievalChain** maintains conversation context
- System prompt: "Use only the following news to answer the question"
- Strictly answers from retrieved news, not from model's general knowledge
- Includes last 5 chat exchanges for context

#### d) Response Generation
- Combines retrieved news context with user query
- Generates natural language answers
- Stores conversation in MongoDB for history tracking

**Flow:**
1. Receives user query
2. Loads vector database from Pinecone
3. Generates multiple query variations (MultiQueryRetriever)
4. Retrieves relevant news chunks
5. Re-ranks results (CohereRerank)
6. Fetches last 5 chat exchanges from database
7. Generates response using conversation history + retrieved context
8. Saves query and response to MongoDB
9. Returns answer to user

### 5. Database Management (`dbase.py`)
Manages conversation history using MongoDB.

**MongoDB Structure:**
- Database: `chatHistory`
- Collection: `cht_info`
- Document fields:
  - `query`: User's question
  - `response`: Bot's answer
  - `time`: Timestamp in IST (Asia/Kolkata)

**Functions:**
- `insert_data(query, response)`: Saves each conversation turn
- `get_chthistory()`: Retrieves last 5 conversations in descending order by time
- Provides conversation context for better responses

## Technology Stack

### Core Frameworks
- **Streamlit**: Web interface and chat UI
- **LangChain**: RAG pipeline orchestration
- **Beautiful Soup**: Web scraping and HTML parsing

### AI/ML Services
- **Cohere**: LLM for hypothetical document generation
- **HuggingFace Hub**: Mixtral-8x7B-Instruct for query expansion
- **Voyage AI**: Semantic embeddings
- **Pinecone**: Vector database for similarity search

### Data Storage
- **MongoDB**: Conversation history (cloud-hosted)
- **SQLite**: LLM response caching
- **Local Files**: Daily scraped news in text format

### Supporting Libraries
- **Requests**: HTTP client for web scraping
- **NLTK**: Natural language processing utilities
- **PyTZ**: Timezone handling for IST timestamps

## Data Flow

```
User Query
    ↓
Streamlit UI (app.py)
    ↓
Chatbot Logic (chtbot.py)
    ↓
Multi-Query Generation → Mixtral-8x7B
    ↓
Vector Retrieval → Pinecone
    ↓
Re-ranking → Cohere Rerank
    ↓
Context Assembly (news + chat history)
    ↓
Response Generation → Mixtral-8x7B
    ↓
Save to MongoDB (dbase.py)
    ↓
Display Response in UI
```

## Scraping and Indexing Flow

```
Daily Trigger
    ↓
Web Scraper (webscrape.py)
    ↓
MoneyControl.com
    ↓
Extract Articles (title, date, content, link)
    ↓
Clean and Format Text
    ↓
Save to files/YYYY-MM-DD.txt
    ↓
Vector Embedding (vect_embed.py)
    ↓
Chunk Text (1500 chars, 200 overlap)
    ↓
Generate Embeddings → Voyage AI
    ↓
Store in Pinecone
    ↓
Ready for Queries
```

## Key Design Decisions

### 1. Daily Scraping
- News is scraped once per day to minimize server load
- Stored in date-stamped files for auditability
- Automatic triggering on first user visit each day

### 2. RAG over Fine-tuning
- RAG allows real-time updates without model retraining
- Maintains factual accuracy with source attribution
- More cost-effective for frequently changing data

### 3. Multi-Stage Retrieval
- Multi-query generation increases recall
- Reranking improves precision
- Balance between retrieval quality and latency

### 4. Conversation History
- MongoDB storage enables cross-session history
- Last 5 exchanges provide sufficient context
- IST timezone for Indian users

### 5. Hypothetical Document Embeddings (Optional)
- Can be enabled for better retrieval on abstract queries
- Currently using direct Voyage AI embeddings for simplicity
- HyDE chain available in code for future use

## Security Considerations

⚠️ **Critical Security Issues:**
1. **Hardcoded API Keys**: All API keys are exposed in source code
2. **MongoDB Credentials**: Database connection string is in plain text
3. **Public Repository**: Credentials are visible to anyone

**Recommendations:**
- Move all credentials to environment variables
- Use `.env` file and load with `python-dotenv`
- Add `.env` to `.gitignore`
- Rotate all exposed API keys immediately
- Use secret management services for production

## Performance Optimizations

1. **Caching**: SQLite cache reduces redundant LLM API calls
2. **Chunk Overlap**: 200-character overlap ensures context continuity
3. **Request Retry**: Automatic retry on network failures
4. **Batch Processing**: Processes multiple articles before saving
5. **Vector Indexing**: Pinecone provides sub-second retrieval

## Scalability Considerations

### Current Limitations
- Single Pinecone index for all news
- Daily full scraping (no incremental updates)
- No sharding for large data volumes
- Single LLM endpoint (HuggingFace Hub)

### Future Enhancements
- Date-based index partitioning in Pinecone
- Incremental scraping (only new articles)
- Multiple LLM endpoints with load balancing
- Caching popular queries at application level
- Asynchronous scraping with task queue
