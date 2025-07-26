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

# Answers to the Submission Question

## What method or library did you use to extract the text, and why? Did you face any formatting challenges with the PDF content?

I used the Gemini API (`gemini-2.0-flash`) with `pdf2image` to extract text from "HSC26-Bangla1st-Paper.pdf" page-by-page, chosen for its superior Bengali script recognition and ability to handle scanned PDFs. Text was cleaned with regex to preserve Bengali characters and saved to a `.docx` file using `python-docx` for RAG compatibility.

**Challenges and Failures**: Initial attempts with `pdfplumber` and `pytesseract` produced garbled text due to the PDF's scanned or non-standard font content. A second attempt with enhanced OCR settings failed similarly. Using Gemini for whole-PDF extraction improved results but was incomplete. Page-by-page extraction missed some pages due to API limits.

**Final Success**: The final code implemented retries (three attempts per page) and high-resolution (300 DPI) image conversion, with a 5-second delay to avoid rate limits. This approach extracted most content accurately, overcoming previous failures.


## 1. Chunking Strategy
**Strategy**: Sentence-based chunking using `indic-nlp-library` for Bengali and `nltk` for English.

**Why Effective**: Sentences preserve semantic units, balancing granularity to avoid irrelevant details (paragraphs) or fragmented meaning (character limits). Bengali-specific tokenization ensures accurate splitting.

## 2. Embedding Model
**Model**: `intfloat/multilingual-e5-large`.

**Why Chosen**: Supports Bengali and English with 1024-dimensional embeddings, capturing nuanced semantics. Pretrained efficiency eliminates fine-tuning needs.

**How It Captures Meaning**: This transformer-based model uses attention to encode contextual word relationships, aligning similar meanings across languages in a shared vector space, ideal for multilingual retrieval.

## 3. Query Comparison and Storage
**Method**: Cosine similarity in Pinecone vector database, retrieving the top 10 chunks.

**Why Chosen**: Cosine similarity measures semantic alignment, robust for high-dimensional embeddings. Pinecone provides scalable, managed storage, supporting multilingual embeddings.

## 4. Meaningful Comparison
**Ensuring Comparison**:
- Normalize queries and documents for consistency.
- Use `multilingual-e5-large` for a shared vector space.
- Retrieve the top 10 chunks for diverse context.
- Prompt `gpt-4o` to resolve pronouns using conversation history.

**Vague Queries**: The system generally performs well, as shown below:

- **Query**: অনুপমের ভাষায় সুপরুষ কাকে বলা হয়েছে?  
  **Actual Answer**: অনুপমের ভাষায় সুপুরুষ শম্ভুনাথকে বলা হয়েছে।  
  **Expected Answer**: শম্ভুনাথকে 
- **Query**: তিনি কল্যাণীর সম্পর্কে কে হন?  
  **Actual Answer**: তিনি কল্যাণীর পিতা হন।  
  **Expected Answer**: পিতা  

However, it failed in the following case due to pronoun ambiguity:

- **Query**: তিনি কি করেন?  
  **Actual Answer**: তিনি ওকালতি করে প্রচুর টাকা রোজগার করেছেন, কিন্তু ভোগ করার সময় পাননি।  
  **Expected Answer**: তিনি ডাক্তারি করেন।  

**Conclusion**: Ambiguous queries (e.g., "তিনি কি করেন?") may retrieve irrelevant chunks, leading to incorrect answers (e.g., lawyer instead of doctor). **Mitigation**: Add entity metadata, enhance prompts to clarify pronouns, or rephrase queries using conversation history.

## 5. Result Relevance
**Relevance**: Most results are relevant (5/6 queries match, cosine similarity 0.764–0.824). Ambiguous queries fail due to unresolved pronouns.

**Improvements**:
- **Chunking**: Use overlapping chunks for better context.
- **Embedding**: Fine-tune the model on Bengali literature.
- **Corpus**: Expand the document with related texts.
- **Pronoun Resolution**: Store entity metadata, refine prompts to prioritize recent history, or request clarification for ambiguous pronouns.
