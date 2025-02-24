# Integrating a Custom Embedding Service

This is the third tutorial of this BentoML RAG example project.

In [the last tutorial](../01-simple-rag/), we built a RAG web service with BentoML. In this tutorial, we will enhance our RAG service by integrating a custom text embedding model, which will replace the default embedding service provided by OpenAI.

To incorporate our custom model into LlamaIndex, we need to do the following:

1. Create our own embedding web service.
2. Ensure LlamaIndex can use this new service by wrapping it in a class as specified in [the LlamaIndex documentation](https://docs.llamaindex.ai/en/stable/examples/embeddings/custom_embeddings/).

## Use SentenceTransformers with BentoML

BentoML provides [an example project](https://github.com/bentoml/BentoSentenceTransformers/) on creating a text embedding service using [SentenceTransformers](https://sbert.net). Since BentoML Services are composable, we will leverage this to integrate directly with our RAG service.

The Service source code of the BentoML embedding example project is already saved in `embedding.py`, with an additional wrapper class similar to LlamaIndex's [example code](https://docs.llamaindex.ai/en/stable/examples/embeddings/custom_embeddings/). 

## Run the updated Service

We need to import our embedding service directly in `service.py`. The modifications in `service.py` only need the following lines:

```diff
+ # import embedding service and wrapper class in service.py
+ from embedding import SentenceTransformers, BentoMLEmbeddings

...

 class RAGService:

+    embedding_service = bentoml.depends(SentenceTransformers)
+
     def __init__(self):
         openai.api_key = os.environ.get("OPENAI_API_KEY")
+        self.embed_model = BentoMLEmbeddings(self.embedding_service)

         from llama_index.core import Settings
         self.text_splitter = SentenceSplitter(chunk_size=1024, chunk_overlap=20)
         Settings.node_parser = self.text_splitter
+        Settings.embed_model = self.embed_model

```

Then, run with this Service by using `bentoml serve .` and interact with it in the same way as shown in the previous tutorial.

## Next step

Despite integrating our own embedding model, we continue to use OpenAI's service for the question-answering part. In the [next tutorial](../03-custom-llm/), we will replace this component with our own language model.