# 10MS_RAG_Application
This repository implements a Retrieval-Augmented Generation (RAG) system for processing English and Bengali queries using the "HSC26 Bangla 1st Paper" PDF as the knowledge base. It supports short-term (conversation history) and long-term (vector database) memory, with a REST API for interaction.

## Setup Guide

### Prerequisites

- **Environment**: Google Colab (GPU/CPU).
- **API Keys**:
  - Google Gemini (`GOOGLE_AOI_KEY_2`)
  - Pinecone (`Pinecone_API_KEY`)
  - OpenAI (`my_GPT_key_2`)
  - Ngrok (`NGROK_AUTH_TOKEN`)
- **Dependencies**: `poppler-utils`
- **Document**: `HSC26-Bangla1st-Paper.pdf`

### Installation

1. **Clone Repository**:
   ```bash
   git clone <repository_url>
   cd <repository_directory>
   ```

2. **Install Libraries**:
   ```bash
   !pip install pdf2image python-docx docx2txt nltk indic-nlp-library sentence-transformers pinecone openai flask pyngrok -q
   !pip install git+https://github.com/csebuetnlp/normalizer -q
   !apt-get update && apt-get install -y poppler-utils
   ```

3. **Set API Keys**: Add keys to Colab Secrets.

4. **Upload PDF**:
   ```python
   from google.colab import files
   files.upload()
   ```

5. **Run Scripts**:
   - **Preprocess**: `python pdf_preprocessing.py` (outputs `preprocessed_text.docx`).
   - **RAG System**: `python rag_system.py` (starts Flask API, outputs Ngrok URL).

## Tools, Libraries, and Packages

- **Python Libraries**: `pdf2image`, `python-docx`, `docx2txt`, `nltk`, `indic-nlp-library`, `sentence-transformers` (`intfloat/multilingual-e5-large`), `pinecone`, `openai` (`gpt-4o`), `flask`, `pyngrok`, `normalizer` (`csebuetnlp/normalizer`), `requests`, `csv`, `re`, `numpy`, `collections`, `threading`, `json`, `socket`, `subprocess`, `time`.
- **System Tools**: `poppler-utils`, Google Colab.

## Sample Queries and Outputs

Tested on Google Colab, based on *Aparichita* from "HSC26 Bangla 1st Paper".

### Bengali Queries

- **Query**: অনুপমের ভাষায় সুপরুষ কাকে বলা হয়েছে?  
  - **Expected**: শম্ভুনাথকে  
  - **Actual**: অনুপমের ভাষায় সুপুরুষ শম্ভুনাথকে বলা হয়েছে।  
- **Query**: তিনি কল্যাণীর সম্পর্কে কে হন?  
  - **Expected**: পিতা  
  - **Actual**: তিনি কল্যাণীর পিতা হন।  
- **Query**: কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?  
  - **Expected**: মামাকে  
  - **Actual**: অনুপমের ভাগ্য দেবতা বলে তার মামাকে উল্লেখ করা হয়েছে।  
- **Query**: 'রসনচৌকি' শব্দের অর্থ কি?  
  - **Expected**: শানাই ঢোল ও কাঁসি এই তিনটি বাদ্যযন্ত্রের সৃষ্ট ঐকতানবাদন  
  - **Actual**: 'রসনচৌকি' শব্দের অর্থ হলো শানাই, ঢোল ও কাঁসি এই তিনটি বাদ্যযন্ত্রের সৃষ্ট ঐকতানবাদন।

### English Queries

- **Query**: Who is the main character in the story?  
  - **Expected**: অনুপম  
  - **Actual**: The main character in the story "অপরিচিতা" is অনুপম.  
- **Query**: What is the name of Anupam's friend?  
  - **Expected**: হরিশ  
  - **Actual**: Anupam's friend is named Harish.

## API Documentation

### Endpoint: `/api/query`

- **Method**: POST
- **Content-Type**: `application/json`
- **Payload**:
  ```json
  {"query": "<query_in_english_or_bengali>"}
  ```

- **Response**:
  - **Success (200)**:
    ```json
    {"answer": "<generated_answer>"}
    ```
  - **Error (400, 500)**:
    ```json
    {"error": "<error_message>"}
    ```

## Evaluation

### Evaluation Process

The RAG system was evaluated using two metrics:

- **Groundedness**: Checked if the expected answer appears in the top 10 retrieved chunks (from Pinecone, using `intfloat/multilingual-e5-large` embeddings and cosine similarity). A chunk containing the expected answer (case-insensitive substring match) is marked "SUPPORTED."
- **Relevance**: Calculated as the average cosine similarity score of the top 10 retrieved chunks, indicating how closely the retrieved content matches the query.

The `evaluate_rag` function computed these metrics for each sample query by retrieving chunks, checking for the expected answer, and averaging similarity scores. The `test_query` function sent queries to the API, retrieved chunks, and printed results.

### Evaluation Results

| Query | Expected Answer | Actual Answer | Groundedness | Relevance (Avg. Cosine Similarity) |
|-------|-----------------|---------------|--------------|------------------------------------|
| অনুপমের ভাষায় সুপরুষ কাকে বলা হয়েছে? | শম্ভুনাথকে | অনুপমের ভাষায় সুপুরুষ শম্ভুনাথকে বলা হয়েছে। | SUPPORTED | 0.819 |
| তিনি কল্যাণীর সম্পর্কে কে হন? | পিতা | তিনি কল্যাণীর পিতা হন। | SUPPORTED | 0.824 |
| কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে? | মামাকে | অনুপমের ভাগ্য দেবতা বলে তার মামাকে উল্লেখ করা হয়েছে। | SUPPORTED | 0.809 |
| 'রসনচৌকি' শব্দের অর্থ কি? | শানাই ঢোল ও কাঁসি এই তিনটি বাদ্যযন্ত্রের সৃষ্ট ঐকতানবাদন | 'রসনচৌকি' শব্দের অর্থ হলো শানাই, ঢোল ও কাঁসি এই তিনটি বাদ্যযন্ত্রের সৃষ্ট ঐকতানবাদন। | SUPPORTED | 0.798 |
| Who is the main character in the story? | অনুপম | The main character in the story "অপরিচিতা" is অনুপম. | SUPPORTED | 0.795 |
| What is the name of Anupam's friend? | হরিশ | Anupam's friend is named Harish. | SUPPORTED | 0.764 |
