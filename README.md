
# RAG (Retriever-Augmented Generation) Architecture Example

This repository demonstrates the implementation of the Retriever-Augmented Generation (RAG) architecture using OpenAI's GPT model and the `datasets` library for document retrieval.

RAG combines a retrieval component with a generative model to improve the accuracy and relevance of answers to questions. The retrieval component fetches relevant context (documents) based on the user's query, and the generative model uses this context to produce a more informed and relevant response.

## Overview

This example illustrates the following steps:

1. **Loading the dataset**: A dataset is loaded from the Hugging Face `datasets` library (in this case, "epfl-llm/guidelines").
2. **Embedding Generation**: Embeddings for the query (e.g., a question) and the documents (context) are generated using OpenAI's API.
3. **Similarity Calculation**: The similarity between the query embedding and document embeddings is computed using cosine similarity.
4. **Context Construction**: Based on the similarity score, the top document(s) are selected and concatenated to create a context.
5. **Response Generation**: The context and query are fed into the GPT model for generating a response.

## Requirements

- Python 3.x
- Install the required dependencies using the following command:
  ```bash
  pip install datasets openai pandas numpy
  ```

## Setup

1. **Install the dependencies**:
   Run the following command to install the required libraries:
   ```bash
   pip install datasets openai pandas numpy
   ```

2. **Set up OpenAI API Key**:
   To access the OpenAI API, you'll need an API key. You can set it in the script or environment:
   ```python
   client = OpenAI(api_key="YOUR_API_KEY")
   ```

3. **Download and Preprocess Dataset**:
   The dataset "epfl-llm/guidelines" is used to retrieve context for the query.

## Code Walkthrough

1. **Load the Dataset**:
   ```python
   from datasets import load_dataset
   df = load_dataset("epfl-llm/guidelines")
   df = df['train'].select(range(500))
   ```

2. **Preprocess Data**:
   Extract and prepare the text for the retriever:
   ```python
   df_new = pd.DataFrame()
   df_new['text'] = df['clean_text']
   ```

3. **Embedding Function**:
   Embedding generation for the text and the query:
   ```python
   def get_embeddings(text, model='text-embedding-3-small'):
       text = truncate_text(text)
       return client.embeddings.create(input={text}, model=model).data[0].embedding
   ```

4. **Calculate Similarity**:
   Calculate the similarity between the query and document embeddings using cosine similarity:
   ```python
   def fn(page_embedding):
       return np.dot(page_embedding, question_embedding)
   ```

5. **Retrieve Top Context**:
   Sort the documents by similarity and concatenate the top ones to form the context:
   ```python
   context = df_new['text'].iloc[0] + "
" + df_new['text'].iloc[1] + "
" + df_new['text'].iloc[2]
   ```

6. **Generate Response**:
   Generate a response using the GPT model, feeding in the context and question:
   ```python
   response = client.chat.completions.create(
       model='gpt-3.5-turbo',
       messages=[
           {"role": "system", "content": "You are a helpful assistant."},
           {"role": "user", "content": question}
       ]
   )
   print(response.choices[0].message.content)
   ```

## Example Usage

To test this implementation, you can run the script. It will fetch relevant documents from the dataset and generate a response to the input question.

### Input:
```text
What is a food allergy?
```

### Output:
A response generated by GPT-3 based on the relevant documents fetched by the retriever.

## How RAG Works

1. **Retriever**: The retriever searches for relevant documents in a large corpus based on the user's query.
2. **Generator**: The generator (GPT model) uses the retrieved documents as context to generate a more accurate and context-aware response.

By combining both retrieval and generation, RAG models can provide more relevant and precise answers, especially in scenarios where the model has limited prior knowledge or when the question is highly domain-specific.
