# Newsbot360 Documentation Guide

Welcome to the comprehensive documentation for Newsbot360! This guide will help you navigate through all available documentation based on your needs.

## 📚 Documentation Structure

### For New Users

If you're new to Newsbot360, we recommend reading the documentation in this order:

1. **[README.md](README.md)** - Start here for a quick overview
   - What is Newsbot360?
   - Key features
   - Quick start guide
   - Technology stack overview

2. **[SETUP.md](SETUP.md)** - Installation and configuration
   - Prerequisites and requirements
   - API key setup
   - Step-by-step installation
   - Configuration options
   - Troubleshooting guide

3. **[WORKFLOW.md](WORKFLOW.md)** - Understand how it works
   - Complete step-by-step walkthrough
   - Application startup process
   - Daily scraping mechanism
   - Vector embedding process
   - Query processing flow
   - Real-world examples

4. **[ARCHITECTURE.md](ARCHITECTURE.md)** - Deep technical details
   - System components breakdown
   - Technology choices and rationale
   - Data flow diagrams
   - Design decisions
   - Scalability considerations

### For Developers

If you're planning to contribute or modify the code:

1. Start with **[ARCHITECTURE.md](ARCHITECTURE.md)** to understand the system design
2. Review **[WORKFLOW.md](WORKFLOW.md)** to see how components interact
3. Refer to **[SETUP.md](SETUP.md)** for development environment setup
4. Check the inline code comments in Python files for implementation details

### For Users

If you just want to use Newsbot360:

1. Read **[README.md](README.md)** for an overview
2. Follow **[SETUP.md](SETUP.md)** to install and configure
3. Optionally read **[WORKFLOW.md](WORKFLOW.md)** to understand what happens when you ask questions

### For System Administrators

If you're deploying Newsbot360:

1. Review **[SETUP.md](SETUP.md)** for deployment requirements
2. Study **[ARCHITECTURE.md](ARCHITECTURE.md)** for scalability and security considerations
3. Check **[WORKFLOW.md](WORKFLOW.md)** to understand system behavior

## 📖 Document Summaries

### README.md
**Purpose:** Project overview and quick start  
**Length:** ~100 lines  
**Key Topics:**
- Project description
- Feature highlights
- Quick installation steps
- Technology stack
- Links to detailed docs

**When to read:** First time learning about the project

---

### SETUP.md
**Purpose:** Complete installation and configuration guide  
**Length:** ~370 lines  
**Key Topics:**
- System requirements
- API account creation
- Step-by-step installation
- Environment variable configuration
- Pinecone and MongoDB setup
- Testing procedures
- Troubleshooting common issues
- Security best practices

**When to read:** When setting up the project for the first time or deploying to a new environment

---

### WORKFLOW.md
**Purpose:** Detailed step-by-step explanation of how the system works  
**Length:** ~870 lines  
**Key Topics:**
- Application startup sequence
- Web scraping process (with code examples)
- Text chunking and embedding generation
- Vector database storage
- Query processing pipeline
- Multi-query retrieval
- Result reranking
- Response generation
- Complete end-to-end examples

**When to read:** When you want to understand exactly what happens behind the scenes, or when debugging issues

---

### ARCHITECTURE.md
**Purpose:** Technical architecture and system design documentation  
**Length:** ~270 lines  
**Key Topics:**
- System component descriptions
- Technology stack justification
- Data flow diagrams
- Key design decisions
- Security considerations
- Performance optimizations
- Scalability limitations
- Future enhancements

**When to read:** When planning modifications, optimizing performance, or understanding architectural decisions

## 🎯 Use Cases and Recommended Reading

### "I want to install and run Newsbot360"
→ Read: [README.md](README.md) → [SETUP.md](SETUP.md)

### "I'm getting an error during installation"
→ Read: [SETUP.md](SETUP.md) (Troubleshooting section)

### "I want to understand what happens when I ask a question"
→ Read: [WORKFLOW.md](WORKFLOW.md) (User Query Processing section)

### "I want to modify the scraping logic"
→ Read: [ARCHITECTURE.md](ARCHITECTURE.md) (Web Scraper component) → [WORKFLOW.md](WORKFLOW.md) (Daily News Scraping section) → Review `webscrape.py`

### "I want to change the LLM or embedding model"
→ Read: [ARCHITECTURE.md](ARCHITECTURE.md) (Technology Stack) → [SETUP.md](SETUP.md) (Configuration Options) → Review `chtbot.py` and `vect_embed.py`

### "I'm concerned about security"
→ Read: [ARCHITECTURE.md](ARCHITECTURE.md) (Security Considerations) → [SETUP.md](SETUP.md) (Security Best Practices)

### "The system is too slow"
→ Read: [ARCHITECTURE.md](ARCHITECTURE.md) (Performance Optimizations) → [WORKFLOW.md](WORKFLOW.md) (understand bottlenecks)

### "I want to deploy to production"
→ Read: [SETUP.md](SETUP.md) (Production Deployment) → [ARCHITECTURE.md](ARCHITECTURE.md) (Scalability Considerations)

### "I want to add new news sources"
→ Read: [WORKFLOW.md](WORKFLOW.md) (Scraping process) → [ARCHITECTURE.md](ARCHITECTURE.md) (Web Scraper) → Modify `webscrape.py` and `files/moneycontrol_urls.txt`

## 📝 File Reference Guide

### Python Files

| File | Purpose | Documentation Reference |
|------|---------|------------------------|
| `app.py` | Main Streamlit application | ARCHITECTURE.md (Core Application), WORKFLOW.md (Application Startup) |
| `webscrape.py` | News scraping logic | ARCHITECTURE.md (Web Scraper), WORKFLOW.md (Daily News Scraping) |
| `vect_embed.py` | Embedding generation | ARCHITECTURE.md (Vector Embedding System), WORKFLOW.md (Vector Embedding Process) |
| `chtbot.py` | Chatbot RAG pipeline | ARCHITECTURE.md (Chatbot Logic), WORKFLOW.md (User Query Processing) |
| `dbase.py` | MongoDB operations | ARCHITECTURE.md (Database Management), WORKFLOW.md (Chat History) |

### Configuration Files

| File | Purpose | Documentation Reference |
|------|---------|------------------------|
| `.env` | Environment variables | SETUP.md (Configure Environment Variables) |
| `requirements.txt` | Python dependencies | SETUP.md (Install Dependencies) |
| `files/moneycontrol_urls.txt` | URLs to scrape | SETUP.md (Prepare News URL File), WORKFLOW.md (Scraping) |

### Data Files

| File/Directory | Purpose | Documentation Reference |
|----------------|---------|------------------------|
| `files/YYYY-MM-DD.txt` | Daily scraped news | WORKFLOW.md (Daily News Scraping) |
| `.langchain.db` | LLM response cache | ARCHITECTURE.md (Vector Embedding System - Caching) |

## 🔍 Quick Reference

### Key Numbers
- **Chunk Size:** 1500 characters
- **Chunk Overlap:** 200 characters
- **Chat History:** Last 5 conversations
- **Vector Dimensions:** 1024 (Voyage AI)
- **Max Pages per Section:** 15,655
- **Request Timeout:** 20 seconds
- **Error Sleep Time:** 80 seconds

### Key Technologies
- **Web Interface:** Streamlit
- **Scraping:** Beautiful Soup + Requests
- **Embeddings:** Voyage AI
- **Vector DB:** Pinecone (index: "trial")
- **LLM:** Mixtral-8x7B-Instruct (via HuggingFace)
- **Reranking:** Cohere
- **Chat History:** MongoDB Atlas
- **Cache:** SQLite

### Key API Keys Required
1. Cohere API Key
2. Pinecone API Key + Environment
3. Voyage AI API Key
4. HuggingFace Token
5. MongoDB Connection String

See [SETUP.md](SETUP.md) for detailed setup instructions.

## 💡 Tips for Reading

1. **Use the search function** in your editor/browser to find specific topics
2. **Follow the links** between documents to get context
3. **Code examples** in WORKFLOW.md show actual implementation
4. **Diagrams** in ARCHITECTURE.md visualize data flow
5. **Troubleshooting sections** in SETUP.md address common issues

## 🤝 Contributing to Documentation

If you find errors or want to improve the documentation:

1. Check which file needs updating (use this guide)
2. Make changes with clear, concise language
3. Add code examples where helpful
4. Update cross-references between documents
5. Test any commands or code snippets
6. Submit a pull request

## 📞 Getting Help

If the documentation doesn't answer your question:

1. Check the **Troubleshooting** section in [SETUP.md](SETUP.md)
2. Search **GitHub Issues** for similar problems
3. Review the **code comments** in the relevant Python files
4. Create a new **GitHub Issue** with:
   - What you're trying to do
   - What documentation you've read
   - What error/problem you're experiencing
   - Steps to reproduce

## 📅 Documentation Maintenance

These documents should be updated when:

- New features are added
- Dependencies are changed
- API services are updated
- System architecture changes
- Common issues are discovered
- Configuration options change

---

**Last Updated:** November 11, 2024

**Documentation Version:** 1.0

**Project Version:** Compatible with current main branch
