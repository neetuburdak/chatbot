import os
import nltk
from whoosh import index
from whoosh.fields import Schema, TEXT
from whoosh.qparser import QueryParser

# Step 1: Document Ingestion
def ingest_documents(directory_path):
    documents = []
    for filename in os.listdir(directory_path):
        with open(os.path.join(directory_path, filename), 'r', encoding='utf-8') as file:
            text = file.read()
            documents.append({'filename': filename, 'text': text})
    return documents

# Step 2: Text Preprocessing
def preprocess_text(text):
    # Perform tokenization, stop word removal, stemming, etc.
    # You can use NLTK or other libraries for this.
    # Return the processed text.

# Step 3: Indexing
def create_index(documents, index_dir):
    schema = Schema(filename=TEXT(stored=True), content=TEXT)
    if not os.path.exists(index_dir):
        os.mkdir(index_dir)

    ix = index.create_in(index_dir, schema)
    writer = ix.writer()
    
    for doc in documents:
        writer.add_document(filename=doc['filename'], content=preprocess_text(doc['text']))

    writer.commit()

# Step 5: User Query Interface
def search_documents(query, index_dir):
    ix = index.open_dir(index_dir)
    with ix.searcher() as searcher:
        query_parser = QueryParser("content", ix.schema)
        user_query = query_parser.parse(query)
        results = searcher.search(user_query)
        return results

# Step 6: Handle user queries
while True:
    user_query = input("Enter your query (or 'exit' to quit): ")
    if user_query.lower() == 'exit':
        break
    else:
        results = search_documents(user_query, 'index_dir')
        for result in results:
            print(f"Document: {result['filename']}")
            print(f"Score: {result.score}")
            print(f"Content: {result['content']}\n")

# Example usage:
if __name__ == '__main__':
    documents = ingest_documents('your_document_directory')
    create_index(documents, 'index_dir')

