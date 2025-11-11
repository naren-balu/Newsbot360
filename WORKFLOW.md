# Newsbot360 Workflow - Step by Step

This document provides a detailed walkthrough of how Newsbot360 works from start to finish, explaining each step in the process.

## Table of Contents
1. [Application Startup](#application-startup)
2. [Daily News Scraping](#daily-news-scraping)
3. [Vector Embedding Process](#vector-embedding-process)
4. [User Query Processing](#user-query-processing)
5. [Response Generation](#response-generation)
6. [Complete Example Walkthrough](#complete-example-walkthrough)

---

## Application Startup

### Step 1: User Accesses the Application

When a user visits the Newsbot360 URL or runs `streamlit run app.py`:

**File: `app.py`**

```python
# Streamlit initializes the web interface
st.set_page_config(
    page_title="NewsBot360",
    page_icon=":newspaper:",
    layout="wide"
)
st.title("NewsBot360")
```

**What happens:**
- Streamlit creates a web server on `localhost:8501`
- The page loads with the title "NewsBot360" and a newspaper icon
- The interface is set to wide layout for better readability

---

### Step 2: Check for Today's News

**File: `app.py` (lines 13-20)**

```python
today = str(date.today())+'.txt'  # e.g., "2024-01-28.txt"
files = os.listdir('files')
if today not in files:
    with st.spinner("Scraping data..."):
        scrape('files/'+today)
        st.success()
```

**What happens:**
1. The system gets today's date (e.g., "2024-01-28")
2. Creates a filename: `2024-01-28.txt`
3. Checks if this file exists in the `files/` directory
4. **If the file doesn't exist:**
   - Shows "Scraping data..." spinner to the user
   - Calls the `scrape()` function to fetch today's news
   - Shows success message when complete
5. **If the file exists:**
   - Skips scraping (data already available)
   - Continues to chat interface

**Why this matters:** This ensures news data is always up-to-date without re-scraping unnecessarily.

---

### Step 3: Initialize Chat Interface

**File: `app.py` (lines 26-32)**

```python
if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
```

**What happens:**
1. Checks if chat history exists in session state
2. If not, creates an empty messages list
3. Displays all previous messages from the current session
4. Shows user messages on the right, bot messages on the left

**Note:** This is session-based memory (resets on page refresh), separate from MongoDB historical memory.

---

## Daily News Scraping

This process happens automatically when no file exists for today's date.

### Step 1: Initialize Scraper

**File: `webscrape.py` (lines 14-22)**

```python
def scrape(file_name):
    s = requests.Session()
    s.mount('https://', HTTPAdapter(max_retries=2))
    file = open(file_name, "w", encoding='utf-8')
    
    urls = [line.rstrip('\n') for line in open('files/moneycontrol_urls.txt')]
```

**What happens:**
1. Creates an HTTP session with retry logic (2 retries on failure)
2. Opens a file to write scraped news
3. Reads target URLs from `files/moneycontrol_urls.txt`:
   - Mutual Funds: `https://www.moneycontrol.com/news/business/mutual-funds/`
   - Economy: `https://www.moneycontrol.com/news/business/economy/`
   - Stocks: `https://www.moneycontrol.com/news/business/stocks/`
   - Markets: `https://www.moneycontrol.com/news/business/markets/`

---

### Step 2: Set Browser Headers

**File: `webscrape.py` (lines 27-30)**

```python
headers = {
    'Referer': 'https://www.moneycontrol.com',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...'
}
```

**What happens:**
- Sets headers to mimic a real browser
- Prevents the website from blocking automated requests
- Includes referer to appear as legitimate traffic

**Why this matters:** Without proper headers, MoneyControl would block the scraper as a bot.

---

### Step 3: Scrape Each News Section

**File: `webscrape.py` (lines 43-174)**

For each URL in the list, the scraper:

#### 3a. Fetch the Page

```python
response = s.get(url, headers=headers, timeout=20)
data = response.text
soup = BeautifulSoup(data, 'html.parser')
articles = soup.find_all('li', {'class':'clearfix'})
```

**What happens:**
1. Makes HTTP GET request to the URL
2. Receives HTML response
3. Parses HTML using Beautiful Soup
4. Finds all article elements with class "clearfix"

#### 3b. Extract Article Metadata

```python
for article in articles:
    title = article.find('h2').text
    link = article.find('a').get('href')
    date = article.find('span').text
```

**What happens:**
- Extracts article title from `<h2>` tag
- Gets article link from `<a>` tag's `href` attribute
- Gets publication date from `<span>` tag

**Example output:**
- Title: "RBI keeps repo rate unchanged at 6.5%"
- Link: "https://www.moneycontrol.com/news/business/economy/rbi-keeps..."
- Date: "January 28, 2024"

#### 3c. Fetch Full Article Content

```python
news_response = s.get(link, headers=headers, timeout=15)
news_soup = BeautifulSoup(news_data, 'html.parser')
news_text = news_soup.find('div', {'class':'arti-flow'})

# Remove scripts and styles
for x in news_text.find_all("script"):
    x.decompose()
for y in news_text.find_all('style'):
    y.decompose()

news = news_text.text
```

**What happens:**
1. Follows the article link to get full content
2. Finds the main article content div
3. Removes unnecessary script and style tags
4. Extracts clean text content

#### 3d. Clean the Text

```python
news = news.strip()
index_of_news = news.find('Related stories')
news = news[:index_of_news]
cleaned_text = ' '.join(news.split())
```

**What happens:**
1. Removes leading/trailing whitespace
2. Finds "Related stories" section (end of article)
3. Removes everything after "Related stories"
4. Collapses multiple spaces into single spaces

#### 3e. Format and Save

```python
if news and news != 'N/A':
    st = f"The news published on the date {date} with headline {title} is: \n {cleaned_text}"
else:
    st = f"The news published on the date {date} with headline {title}"

file.write(st)
file.write('\n \n')
```

**What happens:**
- Formats news as: "The news published on [date] with headline [title] is: [content]"
- Writes to the daily file
- Adds double newline as separator (used later for chunking)

**Example formatted output:**
```
The news published on the date January 28, 2024 with headline RBI keeps repo rate unchanged at 6.5% is:
The Reserve Bank of India (RBI) kept the repo rate unchanged at 6.5% for the sixth consecutive time...

The news published on the date January 28, 2024 with headline Sensex rises 500 points is:
Indian equity markets witnessed strong gains on Friday as the benchmark Sensex rose by 500 points...
```

---

### Step 4: Handle Pagination

**File: `webscrape.py` (lines 154-173)**

```python
url_tag = soup.find('a', {'class':'last'})
max_pages = 15655

if "void" in url_tag.get('href'):
    break
elif url_tag.get('href') and p < max_pages:
    url = 'https://www.moneycontrol.com' + url_tag.get('href')
    p += 1
else:
    break
```

**What happens:**
1. Finds the "Next Page" link at the bottom
2. Checks if it's a valid link (not "javascript:void(0)")
3. If valid and under page limit, continues to next page
4. If no more pages or limit reached, moves to next news section

**Why the limit:** Prevents infinite scraping; 15,655 pages is approximately 470,000+ articles.

---

### Step 5: Trigger Vector Embedding

**File: `webscrape.py` (line 175)**

```python
file.close()
db = vector_embedding(file_name)
```

**What happens:**
1. Closes the scraped news file
2. Immediately calls `vector_embedding()` to process the data
3. Returns the vector database for use

---

## Vector Embedding Process

This converts text news into searchable vector embeddings.

### Step 1: Load and Split Text

**File: `vect_embed.py` (lines 61-73)**

```python
with open(file_name, encoding='utf-8') as f:
    state_of_the_union = f.read()
    
text_splitter = RecursiveCharacterTextSplitter(
    separators=['\n \n'],
    chunk_size=1500,
    chunk_overlap=200,
    length_function=len,
)
docs = text_splitter.create_documents([state_of_the_union])
```

**What happens:**
1. Reads the entire scraped news file
2. Creates a text splitter with these rules:
   - Split on double newlines (`\n \n`) - our article separator
   - Each chunk: maximum 1500 characters
   - Overlap: 200 characters between chunks (for context continuity)
3. Splits the text into document chunks

**Example:**
- Chunk 1 (chars 0-1500): "The news published on the date January 28..."
- Chunk 2 (chars 1300-2800): "...rate unchanged at 6.5%. The news published..."
- Notice the 200-char overlap between chunks

**Why overlap matters:** Ensures questions about information near chunk boundaries can still find relevant context.

---

### Step 2: Create Embeddings

**File: `vect_embed.py` (lines 75-82)**

```python
llm_chain = create_hypothetical_chain()
embeddings = VoyageEmbeddings(voyage_api_key='...')

# Optional HyDE approach (commented out)
# embeddings = HypotheticalDocumentEmbedder(
#     llm_chain=llm_chain,
#     base_embeddings=base_embeddings
# )
```

**What happens:**

**Option A: Direct Embeddings (Current)**
1. Uses Voyage AI API to convert text to vectors
2. Each chunk becomes a 1024-dimensional vector
3. Vectors capture semantic meaning of the text

**Option B: Hypothetical Document Embeddings (Alternative)**
1. Generates a hypothetical answer to each chunk
2. Embeds both the chunk and its hypothetical answer
3. Can improve retrieval for abstract queries
4. Currently disabled but available in code

**Example vector (simplified):**
```
Chunk: "RBI keeps repo rate at 6.5%"
Vector: [0.23, -0.15, 0.87, ..., 0.45]  # 1024 numbers
```

---

### Step 3: Store in Pinecone

**File: `vect_embed.py` (lines 83-94)**

```python
pinecone.init(
    api_key=os.getenv("PINECONE_API_KEY"),
    environment=os.getenv("PINECONE_ENV")
)
index_name = "trial"

docsearch = Pinecone.from_documents(
    docs, 
    embeddings, 
    index_name=index_name
)
```

**What happens:**
1. Connects to Pinecone cloud service
2. Selects the "trial" index
3. Uploads all document chunks with their embeddings
4. Pinecone indexes them for fast similarity search

**Pinecone structure:**
```
Index: "trial"
├── Vector 1: [0.23, -0.15, ...] → "The news published on Jan 28..."
├── Vector 2: [-0.45, 0.67, ...] → "RBI keeps repo rate at 6.5%..."
├── Vector 3: [0.12, -0.34, ...] → "Sensex rises 500 points..."
└── ... (thousands more)
```

---

### Step 4: Initialize Cache

**File: `vect_embed.py` (lines 56-57, 95)**

```python
def create_cache():
    langchain.llm_cache = SQLiteCache(database_path=".langchain.db")

create_cache()
```

**What happens:**
1. Creates/opens `.langchain.db` SQLite database
2. Stores LLM responses for repeated queries
3. Reduces API calls and improves response time

**Example:**
- First time: "What is repo rate?" → API call → 2 seconds
- Second time: "What is repo rate?" → Cache hit → 0.1 seconds

---

## User Query Processing

When a user types a question and presses Enter.

### Step 1: Capture User Input

**File: `app.py` (lines 35-39)**

```python
if prompt := st.chat_input("Message NewsBot360..."):
    st.chat_message("user").markdown(prompt)
    st.session_state.messages.append({
        "role": "user", 
        "content": prompt
    })
```

**What happens:**
1. User types: "What is the latest news on RBI repo rate?"
2. Message displays in chat interface
3. Adds to session state for history display

---

### Step 2: Route to Chatbot

**File: `app.py` (line 41)**

```python
response = chtreply(prompt)
```

**What happens:**
- Sends the query to `chtreply()` function in `chtbot.py`
- This is where the RAG pipeline executes

---

### Step 3: Initialize RAG Components

**File: `chtbot.py` (lines 28-49)**

```python
vectDB = vector_embedding()  # Loads from Pinecone

# Multi-query retriever
repo_id = 'mistralai/Mixtral-8x7B-Instruct-v0.1'
llm = HuggingFaceHub(repo_id=repo_id, model_kwargs={"temperature": 0.5})
retriever = MultiQueryRetriever.from_llm(
    retriever=vectDB.as_retriever(), 
    llm=llm
)

# Compression retriever
compressor = CohereRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor, 
    base_retriever=retriever
)
```

**What happens:**
1. Loads vector database from Pinecone (no file_name = load existing)
2. Sets up Mixtral-8x7B-Instruct model via HuggingFace
3. Creates multi-query retriever (generates query variations)
4. Adds Cohere reranker for result quality

---

### Step 4: Generate Query Variations

**File: `chtbot.py` - MultiQueryRetriever (line 41)**

**Original query:** "What is the latest news on RBI repo rate?"

**Generated variations (example):**
1. "Recent updates on RBI repo rate"
2. "RBI monetary policy repo rate changes"
3. "Latest Reserve Bank of India interest rate decisions"
4. "Current RBI repo rate announcements"

**Why this matters:** Different phrasings can retrieve different relevant documents, increasing recall.

---

### Step 5: Retrieve Relevant Documents

**Process:**
1. Each query variation is embedded using Voyage AI
2. Pinecone finds top-k most similar document chunks for each variation
3. Results are combined and deduplicated
4. Typically retrieves 10-20 candidate documents

**Example results:**
```
Score: 0.89 → "The news published on Jan 28... RBI keeps repo rate unchanged at 6.5%..."
Score: 0.85 → "The news published on Jan 27... Experts predict RBI will maintain status quo..."
Score: 0.82 → "The news published on Jan 26... RBI monetary policy committee meeting..."
...
```

---

### Step 6: Rerank Results

**File: `chtbot.py` - CohereRerank (line 46)**

**What happens:**
1. Sends all 10-20 candidates to Cohere's reranking model
2. Cohere analyzes which documents best answer the specific query
3. Returns top 3-5 most relevant documents with new scores

**Before reranking:**
```
1. "RBI keeps repo rate unchanged" (0.89)
2. "Experts predict rate decisions" (0.85)
3. "RBI monetary policy meeting" (0.82)
```

**After reranking:**
```
1. "RBI keeps repo rate unchanged" (0.95) ← Most relevant
2. "RBI monetary policy meeting" (0.88) ← Moved up
3. "Experts predict rate decisions" (0.75) ← Moved down
```

---

### Step 7: Fetch Chat History

**File: `chtbot.py` (line 63) → `dbase.py` (lines 29-40)**

```python
latest_cht = get_chthistory()

def get_chthistory():
    cursor = cht_info.find().sort('time', pymongo.DESCENDING).limit(5)
    latest_cht_history = list()
    for doc in cursor:
        latest_cht_history.append((doc['query'], doc['response']))
    return latest_cht_history
```

**What happens:**
1. Connects to MongoDB
2. Gets last 5 conversations, newest first
3. Returns list of (query, response) tuples

**Example:**
```python
[
    ("What about stocks?", "Sensex rose 500 points today..."),
    ("Tell me about economy", "India's GDP growth..."),
    ("Any mutual fund news?", "Top performing mutual funds..."),
    ...
]
```

**Why this matters:** Provides conversation context for follow-up questions.

---

## Response Generation

### Step 1: Construct Prompt

**File: `chtbot.py` (lines 50-61)**

```python
system_template = """use only the following news to answer the question. 
Strictly answer only from the news below and don't try to answer from your knowledge.
{context}
"""

messages = [
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template("{question}")
]
qa_prompt = ChatPromptTemplate.from_messages(messages)
```

**Final prompt structure:**
```
SYSTEM: Use only the following news to answer the question. Strictly answer only from the news below...

CONTEXT:
The news published on Jan 28 with headline RBI keeps repo rate unchanged at 6.5% is:
The Reserve Bank of India kept the repo rate...

The news published on Jan 27 with headline RBI monetary policy meeting is:
The RBI monetary policy committee will meet...

CHAT HISTORY:
User: "What about economy?"
Assistant: "India's GDP growth..."

USER: "What is the latest news on RBI repo rate?"
```

---

### Step 2: Generate Response

**File: `chtbot.py` (lines 61-64)**

```python
qa = ConversationalRetrievalChain.from_llm(
    llm, 
    compression_retriever, 
    combine_docs_chain_kwargs={"prompt": qa_prompt}
)

resp = qa.run({
    'question': query,
    'chat_history': latest_cht
})
```

**What happens:**
1. Sends the complete prompt to Mixtral-8x7B model
2. Model generates answer based on:
   - Retrieved news articles (context)
   - Chat history (conversation flow)
   - User question
3. Model constrained to only use provided news (system prompt)

**Example response:**
```
"According to the latest news from January 28, 2024, the Reserve Bank of India 
(RBI) has kept the repo rate unchanged at 6.5% for the sixth consecutive time. 
This decision was made during the RBI monetary policy committee meeting..."
```

---

### Step 3: Save to Database

**File: `chtbot.py` (line 65) → `dbase.py` (lines 18-28)**

```python
insert_data(query, resp)

def insert_data(queryy, response):
    current_datetime = datetime.now()
    ist_timezone = pytz.timezone('Asia/Kolkata')
    current_time_ist = current_datetime.astimezone(ist_timezone)
    current_datetime = current_time_ist.strftime("%Y-%m-%d %H:%M:%S")
    
    cht_info.insert_one({
        'query': queryy,
        'response': response,
        'time': current_datetime
    })
```

**What happens:**
1. Gets current time in IST (Indian Standard Time)
2. Formats as "YYYY-MM-DD HH:MM:SS"
3. Inserts into MongoDB:
   - Query: User's question
   - Response: Bot's answer
   - Time: Timestamp in IST

**MongoDB document:**
```json
{
    "_id": "65b6f7a8...",
    "query": "What is the latest news on RBI repo rate?",
    "response": "According to the latest news from January 28, 2024...",
    "time": "2024-01-28 14:30:45"
}
```

---

### Step 4: Display Response

**File: `app.py` (lines 43-48)**

```python
with st.chat_message("assistant"):
    message_placeholder = st.empty()
    st.markdown(response)

st.session_state.messages.append({
    "role": "assistant", 
    "content": response
})
```

**What happens:**
1. Creates assistant message bubble in chat interface
2. Displays the response with markdown formatting
3. Adds to session state for display persistence
4. User sees the answer in the chat interface

---

## Complete Example Walkthrough

Let's follow a complete user interaction from start to finish.

### Scenario: User asks about stock market news

**1. User opens Newsbot360**
- Browser: `http://localhost:8501`
- System checks: No file for today (2024-11-11.txt)
- Action: Starts scraping MoneyControl.com

**2. Scraping process (10-30 minutes)**
```
Scraping: Mutual Funds section... 1000 articles
Scraping: Economy section... 1500 articles
Scraping: Stocks section... 2000 articles
Scraping: Markets section... 1200 articles
Total: ~5700 articles scraped
```

**3. Embedding process (5-10 minutes)**
```
Splitting into chunks... 4500 chunks created
Generating embeddings... Voyage AI processing
Uploading to Pinecone... 4500 vectors stored
Cache initialized... Ready for queries
```

**4. User interface loads**
```
Chat input box appears: "Message NewsBot360..."
User types: "What are the latest updates on the stock market?"
User presses Enter
```

**5. Query processing begins**
```
Query received: "What are the latest updates on the stock market?"

Multi-query generation:
→ "Latest stock market updates"
→ "Recent stock market news"
→ "Current stock market trends"
→ "Today's stock market movements"
```

**6. Document retrieval**
```
Searching Pinecone...
Query 1 found: 12 relevant documents
Query 2 found: 15 relevant documents
Query 3 found: 10 relevant documents
Query 4 found: 14 relevant documents
Combined & deduplicated: 28 documents
```

**7. Reranking**
```
Sending 28 documents to Cohere Rerank...
Top 5 after reranking:
1. "Sensex rises 500 points on strong global cues" (0.94)
2. "Nifty crosses 19,000 mark for first time" (0.91)
3. "Banking stocks lead market rally" (0.89)
4. "FII inflows boost equity markets" (0.86)
5. "IT stocks underperform amid recession fears" (0.82)
```

**8. Chat history retrieval**
```
Fetching from MongoDB...
Last 5 conversations: (empty - first query)
```

**9. Prompt construction**
```
SYSTEM: Use only the following news to answer...

CONTEXT:
The news published on Nov 11, 2024 with headline Sensex rises 500 points...
The news published on Nov 11, 2024 with headline Nifty crosses 19,000...
The news published on Nov 11, 2024 with headline Banking stocks lead...
[... 2 more articles ...]

USER: "What are the latest updates on the stock market?"
```

**10. Response generation**
```
Mixtral-8x7B processing...
Temperature: 0.5 (balanced creativity/accuracy)
Max length: 5000 tokens

Generated response:
"Based on the latest news from November 11, 2024, the Indian stock markets 
have shown strong performance today. The BSE Sensex rose by 500 points, 
while the Nifty crossed the 19,000 mark for the first time, driven by 
strong global cues and robust FII inflows. Banking stocks were the key 
drivers of this rally, with HDFC Bank and ICICI Bank leading the gains. 
However, IT stocks underperformed due to recession fears in key export markets."
```

**11. Save to MongoDB**
```
Inserting into database...
Document created:
{
    "query": "What are the latest updates on the stock market?",
    "response": "Based on the latest news from November 11...",
    "time": "2024-11-11 14:35:22"
}
```

**12. Display to user**
```
Chat interface updates:
┌─────────────────────────────────────────┐
│ User: What are the latest updates on    │
│ the stock market?                       │
├─────────────────────────────────────────┤
│ NewsBot360: Based on the latest news    │
│ from November 11, 2024, the Indian      │
│ stock markets have shown strong...      │
└─────────────────────────────────────────┘
```

**13. User asks follow-up question**
```
User: "Which banking stocks performed well?"

System:
1. Loads chat history (includes previous question + answer)
2. Uses conversation context to understand "which banking stocks" 
   refers to the stocks mentioned in previous answer
3. Retrieves relevant documents about specific banking stocks
4. Generates contextualized response
5. Saves to MongoDB
6. Displays answer
```

---

## Summary

The complete workflow involves:

1. **Daily Scraping**: Automated news collection from MoneyControl.com
2. **Embedding**: Converting text to searchable vectors via Voyage AI
3. **Storage**: Vectors in Pinecone, chat history in MongoDB, cache in SQLite
4. **Query**: Multi-query retrieval for better coverage
5. **Reranking**: Cohere improves result relevance
6. **Generation**: Mixtral-8x7B creates natural language answers
7. **Memory**: MongoDB stores all conversations for context

Each component is optimized for accuracy, speed, and cost-effectiveness, creating a seamless experience for users seeking economic news insights.
