# Newsbot360 Setup Guide

## Prerequisites

### System Requirements
- Python 3.8 or higher
- 4GB RAM minimum (8GB recommended)
- Internet connection for API access
- 2GB free disk space for dependencies and data

### Required Accounts and API Keys
You'll need to create accounts and obtain API keys from the following services:

1. **Cohere** (for LLM operations)
   - Sign up at: https://cohere.ai/
   - Get your API key from the dashboard

2. **Pinecone** (for vector database)
   - Sign up at: https://www.pinecone.io/
   - Create a free tier account
   - Create an index named "trial"
   - Get API key and environment name

3. **HuggingFace Hub** (for Mixtral model access)
   - Sign up at: https://huggingface.co/
   - Get your API token from settings

4. **Voyage AI** (for embeddings)
   - Sign up at: https://www.voyageai.com/
   - Get your API key

5. **MongoDB Atlas** (for chat history)
   - Sign up at: https://www.mongodb.com/cloud/atlas
   - Create a free cluster
   - Get connection string

## Installation Steps

### Step 1: Clone the Repository

```bash
git clone https://github.com/naren-balu/Newsbot360.git
cd Newsbot360
```

### Step 2: Create Virtual Environment

It's recommended to use a virtual environment to avoid dependency conflicts:

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

This will install all required packages including:
- streamlit (web interface)
- langchain (RAG framework)
- cohere (LLM provider)
- pinecone-client (vector database)
- beautifulsoup4 (web scraping)
- pymongo (MongoDB client)
- And many more...

### Step 4: Configure Environment Variables

#### Option A: Using .env File (Recommended)

1. Create a `.env` file in the project root:

```bash
touch .env
```

2. Add your API keys to `.env`:

```env
# Cohere API Key
COHERE_API_KEY=your_cohere_api_key_here

# Pinecone Configuration
PINECONE_API_KEY=your_pinecone_api_key_here
PINECONE_ENV=your_pinecone_environment_here

# Voyage AI
VOYAGE_API_KEY=your_voyage_api_key_here

# HuggingFace Hub
HUGGINGFACEHUB_API_TOKEN=your_huggingface_token_here

# MongoDB Connection String
MONGODB_CONNECTION_STRING=your_mongodb_connection_string_here

# Optional: Other API Keys (if needed)
QDRANT_API_KEY=your_qdrant_key_here
GOOGLE_API_KEY=your_google_key_here
APIFY_API_TOKEN=your_apify_token_here
```

3. **Important**: Update the Python files to load from environment variables

Currently, the API keys are hardcoded in the files. You should update them to read from environment variables. Install `python-dotenv`:

```bash
pip install python-dotenv
```

Then modify each Python file that uses API keys to include at the top:

```python
from dotenv import load_dotenv
load_dotenv()

# Then use os.getenv() instead of hardcoded values
os.environ["COHERE_API_KEY"] = os.getenv("COHERE_API_KEY")
```

#### Option B: Direct Environment Variables

Set environment variables directly in your shell:

```bash
# On macOS/Linux:
export COHERE_API_KEY="your_key_here"
export PINECONE_API_KEY="your_key_here"
export PINECONE_ENV="your_env_here"

# On Windows (Command Prompt):
set COHERE_API_KEY=your_key_here
set PINECONE_API_KEY=your_key_here
```

### Step 5: Set Up Pinecone Index

Before running the app, create a Pinecone index:

1. Log in to Pinecone dashboard
2. Create a new index with these settings:
   - **Name**: `trial`
   - **Dimensions**: 1024 (for Voyage AI embeddings)
   - **Metric**: cosine
   - **Pod Type**: Starter (free tier)

### Step 6: Set Up MongoDB

1. Create a MongoDB Atlas cluster (free tier is sufficient)
2. Create a database named `chatHistory`
3. Create a collection named `cht_info`
4. Get your connection string (format: `mongodb+srv://username:password@cluster.mongodb.net/`)
5. Replace the hardcoded connection string in `dbase.py` with your own

### Step 7: Prepare News URL File

The `files/moneycontrol_urls.txt` file should contain the URLs to scrape. Default content:

```
https://www.moneycontrol.com/news/business/mutual-funds/
https://www.moneycontrol.com/news/business/economy/
https://www.moneycontrol.com/news/business/stocks/
https://www.moneycontrol.com/news/business/markets/
```

You can add more MoneyControl news section URLs if needed.

## Running the Application

### Development Mode

```bash
streamlit run app.py
```

The application will:
1. Start on `http://localhost:8501` (default)
2. Open automatically in your browser
3. Check if today's news has been scraped
4. Start scraping if needed (this may take 10-30 minutes on first run)

### Production Deployment

For deploying to Streamlit Cloud:

1. Push your code to GitHub
2. Go to https://share.streamlit.io/
3. Connect your GitHub repository
4. Add secrets in the Streamlit Cloud dashboard:
   - Go to App Settings > Secrets
   - Add all API keys in TOML format:
   
```toml
COHERE_API_KEY = "your_key"
PINECONE_API_KEY = "your_key"
PINECONE_ENV = "your_env"
VOYAGE_API_KEY = "your_key"
HUGGINGFACEHUB_API_TOKEN = "your_token"
```

5. Deploy the app

## Configuration Options

### Scraping Configuration

Edit `webscrape.py` to customize:

- **Max pages per section**: Change `max_pages = 15655` (line 158)
- **Request timeout**: Modify `timeout=20` (line 56)
- **Sleep time on errors**: Adjust `time.sleep(80)` (line 60, 119)
- **URLs to scrape**: Edit `files/moneycontrol_urls.txt`

### Embedding Configuration

Edit `vect_embed.py` to customize:

- **Chunk size**: Change `chunk_size=1500` (line 68)
- **Chunk overlap**: Change `chunk_overlap=200` (line 69)
- **Embedding model**: Modify `VoyageEmbeddings` initialization (line 78)
- **Pinecone index name**: Change `index_name = "trial"` (line 84)

### Chatbot Configuration

Edit `chtbot.py` to customize:

- **LLM model**: Change `repo_id='mistralai/Mixtral-8x7B-Instruct-v0.1'` (line 37)
- **Temperature**: Adjust `temperature: 0.5` (line 39)
- **Max length**: Modify `max_length: 5000` (line 39)
- **Chat history length**: Change `.limit(5)` in `dbase.py` (line 31)

## Testing the Installation

### 1. Test Web Scraping

```bash
python -c "from webscrape import scrape; scrape('test_output.txt')"
```

This will scrape news and save to `test_output.txt`. Should take 10-30 minutes.

### 2. Test Vector Embedding

```bash
python -c "from vect_embed import vector_embedding; db = vector_embedding('test_output.txt')"
```

This will create embeddings and store them in Pinecone.

### 3. Test Chatbot

```bash
python -c "from chtbot import chtreply; print(chtreply('What is the latest news on stocks?'))"
```

This should return a relevant news summary.

### 4. Run Full Application

```bash
streamlit run app.py
```

Open browser and try asking questions like:
- "What's happening in the stock market?"
- "Tell me about mutual funds news"
- "Any updates on the economy?"

## Troubleshooting

### Common Issues

#### 1. ModuleNotFoundError
**Problem**: Missing Python packages
**Solution**: 
```bash
pip install -r requirements.txt --upgrade
```

#### 2. Pinecone Connection Error
**Problem**: Invalid API key or environment
**Solution**: 
- Verify API key in Pinecone dashboard
- Check environment name (e.g., "gcp-starter", "us-east-1")
- Ensure index "trial" exists with correct dimensions

#### 3. MongoDB Connection Error
**Problem**: Cannot connect to MongoDB
**Solution**:
- Verify connection string format
- Check network access (whitelist your IP in MongoDB Atlas)
- Ensure database and collection exist

#### 4. HuggingFace Rate Limit
**Problem**: Too many requests to HuggingFace
**Solution**:
- Upgrade to HuggingFace Pro account
- Or use local model with `transformers` library
- Add delays between requests

#### 5. Web Scraping Blocked
**Problem**: MoneyControl blocking requests
**Solution**:
- Check if website structure has changed
- Increase sleep time between requests
- Verify headers in `webscrape.py` are up to date

#### 6. Streamlit Port Already in Use
**Problem**: Port 8501 is occupied
**Solution**:
```bash
streamlit run app.py --server.port 8502
```

#### 7. Memory Error During Embedding
**Problem**: Out of memory when processing large files
**Solution**:
- Reduce chunk size in `vect_embed.py`
- Process in smaller batches
- Increase system RAM

### Getting Help

If you encounter issues:
1. Check the GitHub Issues page
2. Review error logs carefully
3. Ensure all API keys are valid and have sufficient credits
4. Verify internet connection for API calls

## Next Steps

After successful setup:
1. Read [WORKFLOW.md](WORKFLOW.md) to understand how the system works
2. Read [ARCHITECTURE.md](ARCHITECTURE.md) for technical details
3. Try different queries to test the chatbot
4. Monitor API usage to stay within free tier limits
5. Consider implementing suggested security improvements

## Security Best Practices

Before deploying to production:

1. **Never commit API keys to Git**
   - Add `.env` to `.gitignore`
   - Use environment variables or secrets management

2. **Rotate exposed keys**
   - If keys were committed, rotate them immediately
   - Update both the source and the service

3. **Use different keys for dev/prod**
   - Separate API keys for development and production
   - Easier to track usage and revoke if needed

4. **Monitor API usage**
   - Set up alerts for unusual usage patterns
   - Track costs to avoid unexpected charges

5. **Implement rate limiting**
   - Add rate limits to prevent abuse
   - Use caching to reduce API calls
