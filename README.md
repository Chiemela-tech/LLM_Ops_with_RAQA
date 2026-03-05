# 🎀 Barbie RAQA — Retrieval Augmented Question Answering

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-1.x-green.svg)](https://www.langchain.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--3.5--turbo-orange.svg)](https://openai.com/)
[![FAISS](https://img.shields.io/badge/VectorStore-FAISS-purple.svg)](https://github.com/facebookresearch/faiss)
[![Chainlit](https://img.shields.io/badge/UI-Chainlit-pink.svg)](https://chainlit.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Retrieval Augmented Question Answering (RAQA) system built on top of **955 real IMDB reviews** of the 2023 Barbie movie. Ask questions about the movie and get answers grounded in actual audience reviews — powered by LangChain, FAISS, and OpenAI.

> 💬 Live Demo: [Barbie RAQA Chainlit App on HuggingFace Spaces](https://huggingface.co/spaces/EllaChinedum/Barbie-RAQA-Chainlit)

---

## Table of Contents

- [What does this project do?](#what-does-this-project-do)
- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Getting started](#getting-started)
  - [Installation](#installation)
  - [Environment setup](#environment-setup)
  - [Running the notebook](#running-the-notebook)
  - [Running the Chainlit app](#running-the-chainlit-app)
- [Project structure](#project-structure)
- [Example queries and results](#example-queries-and-results)
- [Deploying to HuggingFace Spaces](#deploying-to-huggingface-spaces)
- [Key challenges and solutions](#key-challenges-and-solutions)
- [License](#license)

---

## What does this project do?

This project scrapes IMDB reviews for the Barbie (2023) movie, builds a semantic search index using FAISS vector embeddings, and powers a conversational QA interface where users can ask natural language questions about the movie — all answered using real reviewer opinions as the knowledge base.

Example questions you can ask:
- *"How was Will Ferrell in this movie?"*
- *"Was the movie good for kids?"*
- *"What did people think of the movie's message?"*
- *"Do reviewers consider this movie Kenough?"*

---

## How it works

```
IMDB Reviews (955)
      ↓
  Selenium Scraper
      ↓
  barbie_reviews.csv
      ↓
  CSVLoader (LangChain)
      ↓
  RecursiveCharacterTextSplitter → 2,321 chunks
      ↓
  OpenAI Embeddings → FAISS VectorStore
      ↓
  RetrievalQA Chain (GPT-3.5-turbo)
      ↓
  Chainlit Chat Interface
```

1. **Scraping** — Selenium opens a real Chrome browser to bypass IMDB's bot detection, clicks "25 more" up to 50 times to load all reviews, and BeautifulSoup parses the HTML into a structured DataFrame.
2. **Indexing** — LangChain's `CSVLoader` loads the reviews, `RecursiveCharacterTextSplitter` chunks them into 1,000-character pieces with 100-character overlap, and OpenAI's embedding model converts them to vectors stored in a FAISS index.
3. **Retrieval** — At query time, the question is embedded and the 4 most semantically similar review chunks are retrieved from FAISS.
4. **Generation** — GPT-3.5-turbo synthesises an answer from the retrieved chunks and returns it with source citations.
5. **Interface** — A Chainlit web app wraps the chain in a real-time chat UI, with the AI responding in character as Ken from the movie.

---

## Requirements

- Python 3.11 or higher
- Google Chrome browser installed
- An OpenAI API key with available credits
- Dependencies listed in `requirements.txt`

---

## Getting started

### Installation

Clone the repository:

```bash
git clone https://github.com/Chiemela-tech/LLM_Ops_with_RAQA.git
cd LLM_Ops_with_RAQA
```

Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate       # Mac/Linux
venv\Scripts\activate          # Windows
```

Install dependencies:

```bash
pip install openai langchain langchain-openai langchain-community langchain-text-splitters
pip install faiss-cpu tiktoken selenium beautifulsoup4 pandas numpy python-dotenv notebook tqdm chainlit
```

### Environment setup

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=your_openai_api_key_here
```

> ⚠️ Never commit your `.env` file. It is already listed in `.gitignore`.

### Running the notebook

Launch Jupyter and open `barbieRAQA.ipynb`:

```bash
jupyter notebook
```

Run all cells from top to bottom. The notebook will:
1. Open a Chrome browser and scrape 955 IMDB reviews
2. Save them to `barbie_reviews.csv`
3. Build a FAISS vector store
4. Run example queries against the QA chain

> **Note:** The scraping cell opens a visible Chrome window — this is intentional. IMDB blocks headless browsers, so a real browser window is required.

### Running the Chainlit app

Make sure your `barbie_reviews.csv` is in the `data/` folder, then run:

```bash
chainlit run app.py
```

Open your browser at `http://localhost:8000` to chat with Ken about the Barbie movie.

---

## Project structure

```
LLM_Ops_with_RAQA/
├── barbieRAQA.ipynb        # Main notebook: scraping, indexing, and QA chain
├── app.py                  # Chainlit chat application
├── chainlit.md             # Chainlit welcome message
├── requirements.txt        # Python dependencies
├── Dockerfile              # Docker config for HuggingFace Spaces deployment
├── data/
│   └── barbie_reviews.csv  # Scraped IMDB reviews (955 rows)
├── .env                    # Your API key (never committed)
├── .env.example            # Example env file
└── .gitignore
```

---

## Example queries and results

**Query:** *"How was Will Ferrell in this movie?"*
> Will Ferrell's performance was criticised by many reviewers as ruining every scene he was in. Several compared his role to his character in The Lego Movie, calling the casting lazy and his character cartoonish — though some felt the satire worked in context.

**Query:** *"Was the movie good for kids?"*
> Based on reviewer feedback, the movie may not be ideal for young children. Reviewers noted the humour and social commentary are aimed at adults, the pacing can feel chaotic, and several jokes go over younger audiences' heads.

**Query:** *"What did people think of the movie's message?"*
> Opinions were divided. Some found the film's feminist message balanced and thought-provoking, while others felt it was preachy, overly focused on male bashing, or too superficial to land effectively.

---

## Deploying to HuggingFace Spaces

1. Create a new Space at [huggingface.co/new-space](https://huggingface.co/new-space) — choose **Docker** as the SDK.
2. Clone your Space repo and copy in all project files.
3. Push to HuggingFace:
   ```bash
   git add .
   git commit -m "Deploy Barbie RAQA app"
   git push
   ```
4. In your Space's **Settings → Repository secrets**, add:
   - `OPENAI_API_KEY` = your OpenAI key
5. The Space will build automatically — takes ~3-5 minutes.

---

## Key challenges and solutions

| Challenge | Solution |
|---|---|
| IMDB returns 403 Forbidden to headless Chrome | Used visible (non-headless) Chrome browser via Selenium |
| IMDB redesigned their page structure | Updated CSS selectors from `load-more-trigger` to `button.ipc-see-more__button` and `article.sc-fa7e37dc-0` |
| Apple Silicon (M1/M2) ChromeDriver incompatibility | Installed ARM-compatible ChromeDriver via Homebrew |
| `np.NaN` removed in NumPy 2.0 | Replaced all `np.NaN` with `np.nan` |
| LangChain 1.x removed `RetrievalQA` and moved modules | Rewrote chain using LCEL (`RunnableParallel`, `RunnablePassthrough`) and updated all imports to `langchain-openai`, `langchain-community`, `langchain-text-splitters` |
| OpenAI rate limit with 2,321 chunks | Used a 200-chunk subset to stay within API quota |

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

*Built as part of the [AI Maker Space LLM Ops Cohort](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1) curriculum.*
