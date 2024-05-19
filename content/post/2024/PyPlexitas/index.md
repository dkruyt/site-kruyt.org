---
title: "PyPlexitas: Open Source CLI Alternative to Perplexity AI"
date: 2024-05-19T19:45:00+00:00
draft: false
author: Dennis Kruyt
categories:
- ai
cover:
  image: /images/2024/PyPlexitas.webp
slug: pyplexitas
tags:
- llm
- ai
- search
---

PyPlexitas is a Python script that is designed to create an open-source alternative to Perplexity AI, a tool that provides users with detailed answers to their queries by searching the web, extracting relevant content, and using advanced language models to generate responses.

The script operates by first taking a user's query and using search engines like Bing or Google to find relevant web pages. It then scrapes the content from these web pages, processes the text into manageable chunks, and generates vector embeddings for these chunks. Vector embeddings are mathematical representations of text that allow for efficient searching and comparison of content. These embeddings are stored in a database, enabling quick retrieval of relevant information based on the user's query.

Once the content is processed and stored, the script uses a language model to generate a detailed answer to the user's query, using the information extracted from the web pages. This response is designed to be accurate and informative, drawing directly from the content found during the search process.

### Overview

PyPlexitas is designed to be a  efficient solution for retrieving and processing information from the web. It leverages the power of search engines, scraping content from URLs, generating embeddings, and storing them in a vector database. It then uses a language model to provide detailed answers to your queries.

### Key Features

1. **Search Engine Integration**: PyPlexitas can use either the Google or Bing API to fetch search results based on your query.
2. **Local LLM Option**: With support for Ollama, you can use a local language model, offering flexibility and potentially more privacy.
3. **Content Scraping**: The tool scrapes content from the URLs retrieved from search results, ensuring that relevant information is processed.
4. **Embeddings and Chunks**: Content is split into manageable chunks and transformed into embeddings, which are stored in a vector database.
5. **Vector Database**: PyPlexitas uses Qdrant, a highly efficient vector database, to store and retrieve embeddings.
6. **Question Answering**: The language model uses these embeddings to generate accurate and contextually relevant answers to your queries.

### How It Works

1. **Query Input**: The user provides a search query via the command line interface. For example:

    ```bash
    python PyPlexitas.py -q "When will the next model GPT-5 be released" -s 10 --engine google
    ```

2. **Choose Search Provider**: Depending on the user’s choice, PyPlexitas uses either the Google or Bing API to perform the search. The relevant API key (Google Custom Search API key or Bing Search API key) is required and configured in the `.env` file.

3. **Fetch and Scrape URLs**: PyPlexitas fetches search results and scrapes the content from the URLs. It parses the content to extract meaningful text, which is then processed further.

4. **Create Embeddings and Chunks**: The scraped content is split into smaller chunks. Each chunk is transformed into a vector (embedding) using either OpenAI or Ollama embeddings. These embeddings are numerical representations of the text, capturing the semantic meaning.

5. **Use a Vector Database**: The embeddings are stored in a vector database, Qdrant. A vector database allows for efficient storage, indexing, and querying of high-dimensional vectors. By using Qdrant, PyPlexitas can quickly search and retrieve relevant chunks based on the user's query.

6. **Answer Using LLM**: The relevant chunks are then fed into a language model (LLM), such as OpenAI’s GPT-4 or Ollama, to generate a detailed answer. The model processes the chunks to provide a coherent and contextually accurate response to the user’s query.

Output:
```
Searching for 🔎: When will the next model GPT-5 be released using google
Starting Google search ⏳
Google search returned 🔗: 10 results
From domains 🌐: mashable.com  www.reddit.com  www.tomsguide.com  www.datacamp.com  medium.com  www.standard.co.uk  www.theverge.com  arstechnica.com  
Scraping content from search results...
Embedding content ✨
Total embeddings 📊: 10
Total chunks processed 🧩: 7

Answering your query: When will the next model GPT-5 be released 🙋

The release date for GPT-5 is currently expected to be sometime in mid-2024, likely during the summer, according to a report from Business Insider [1][2]. OpenAI representatives have not provided a specific release date, and the timeline may be subject to change depending on the duration of safety testing and other factors [1][2]. OpenAI CEO Sam Altman has indicated that a major AI model will be released this year, but it is unclear whether it will be called GPT-5 or something else [1].

### Sources
1. Benj Edwards - https://arstechnica.com/information-technology/2024/03/gpt-5-might-arrive-this-summer-as-a-materially-better-update-to-chatgpt/
2. Saqib Shah - https://www.standard.co.uk/tech/openai-chatgpt-5-release-date-b1076129.html
```

### What is a Vector Database?

A vector database is a specialized type of database designed to store and query high-dimensional vector data. In the context of PyPlexitas, the vector database stores embeddings, which are numerical representations of text data. These embeddings capture semantic information, enabling efficient similarity searches. Qdrant, the vector database used by PyPlexitas, allows for fast and accurate retrieval of relevant information by comparing the query embedding with stored embeddings.

## Installation
1. Clone the repository:
    ```bash
    git clone https://github.com/dkruyt/PyPlexitas.git
    cd PyPlexitas
    ```

2. Install the required Python packages:
    ```bash
    pip install -r requirements.txt
    ```

3. Set up the Qdrant service using Docker:
    ```bash
    docker-compose up -d
    ```

## Configuration
Configure your environment variables by creating a `.env` file in the project root. Use the provided `example.env` as a template:
```bash
cp example.env .env
```
Fill in your API keys and other necessary details in the `.env` file:
- `OPENAI_API_KEY`
- `GOOGLE_API_KEY`
- `GOOGLE_CX`
- `BING_SUBSCRIPTION_KEY`

## Getting API Keys

### OpenAI API Key
1. Sign up or log in to your [OpenAI account](https://www.openai.com/).
2. Go to the API section and generate a new API key.
3. Copy the API key and add it to the `OPENAI_API_KEY` field in your `.env` file.

### Google Custom Search API Key and CX
1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or select an existing project.
3. Enable the Custom Search API in the API & Services library.
4. Go to the [Credentials](https://console.cloud.google.com/apis/credentials) page and create an API key.
5. Copy the API key and add it to the `GOOGLE_API_KEY` field in your `.env` file.
6. To get the Custom Search Engine (CX) ID, go to the [Custom Search Engine](https://cse.google.com/cse/) page.
7. Create a new search engine or select an existing one.
8. Copy the Search Engine ID (CX) and add it to the `GOOGLE_CX` field in your `.env` file.

### Bing Search API Key
1. Sign up or log in to your [Microsoft Azure account](https://azure.microsoft.com/).
2. Create a new Azure resource for Bing Search v7.
3. Go to the Keys and Endpoint section to find your API key.
4. Copy the API key and add it to the `BING_SUBSCRIPTION_KEY` field in your `.env` file.

## Usage
Run the PyPlexitas script with your query:
```bash
python PyPlexitas.py -q "Your search query" -s 10 --engine bing
```
Options:
- `-q, --query`: Search query (required)
- `-s, --search`: Number of search results to parse (default: 10)
- `--engine`: Search engine to use (`bing` or `google`, default: `bing`)
- `-l, --log-level`: Set the logging level (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`, default: `ERROR`)
- `-t, --max-tokens`: Maximum token limit for model input (default: 1024)
- `--quiet`: Suppress print messages

## Project Structure
- `PyPlexitas.py`: Main script for running the application.
- `example.env`: Example configuration file for environment variables.
- `docker-compose.yml`: Docker Compose configuration for Qdrant.
- `requirements.txt`: List of required Python packages.
- `README.md`: Project documentation.
