# General guidelines for data chunking to generate embedding vectors

When using Natural Language Processing (NLP), the client libraries and REST APIs used generate embedding vectors for text fragments have maximum input limits. For example, the maximum length of input text for the [Azure OpenAI](https://learn.microsoft.com/azure/cognitive-services/openai/how-to/embeddings) embedding models is 2048 tokens (equivalent to around 2-3 pages of text). If you're using these models to generate embeddings, it's critical that the input text stays under the limit. Partitioning your content into chunks ensures that your data can be processed by the Large Language Models (LLM) used for indexing and queries.

There isn't native chunking capability in Neither Cognitive Search or Azure OpenAI, so if you have large documents, you'll need to insert a chunking step into indexing and query workflows that breaks up large text. On the development side, we are working with these libraries:

+ [LangChain](https://python.langchain.com/en/latest/index.html)
+ [Semantic Kernal](https://github.com/microsoft/semantic-kernel)

NOTE: It's on the roadmap to document chunking patterns and provide a sample, but that content isn't available at this time.

## Factors to consider when chunking data

When it comes to chunking data, think about these factors:

1. Shape and density of your documents. If you need intact text or passages, larger chunks and variable chunking that preserves sentence structure can produce better results.

1. User queries: Larger chunks and overlapping strategies help preserve context and semantic richness for queries that target specific information.

1. Large Language Models (LLM) have performance guidelines for chunk size. you'll need to set a chunk size that works best for all of the models you're using. For instance, if you use models for summarization and embeddings, choose an optimal chunk size that works for both.

## Common chunking techniques

Here are some common chunking techniques, starting with the most widely used method:

1. Fixed-size chunks: Define a fixed size that's sufficient for semantically meaningful paragraphs (for example, 200 words) and allows for some overlap (for example, 10-15% of the content) can produce good chunks as input for embedding vector generators.

1. Variable-sized chunks based on content: Partition your data based on content characteristics, such as end-of-sentence punctuation marks, end-of-line markers, or using features in the Natural Language Processing (NLP) libraries. Markdown language structure can also be used to split the data.

1. Customize or iterate over one of the above techniques. For example, when dealing with large documents, you might use variable-sized chunks, but also append the document title to chunks from the middle of the document to prevent context loss.

## Content overlap considerations

When chunking data, overlapping a small amount of text between chunks can help preserve context. We recommend starting with an overlap of approximately 10%. For example, given a fixed chunk size of 256 tokens, you would begin testing with an overlap of 25 tokens. The actual amount of overlap varies depending on the type of data and the specific use case, but we have found that 10-15% works for many scenarios.

## Simple approach of how to create chunks with sentences

This section demonstrates the logic of creating chunks out of sentences. For this example, assume the following:

+ Tokens are equal to words.
+ Input = `text_to_chunk(string)`
+ Output = `sentences(list[string])`

### Sample input

`"Barcelona is a city in Spain. It is close to the sea and /n the mountains. /n You can both ski in winter and swim in summer."`

+ Sentence 1 contains 6 words: `"Barcelona is a city in Spain."`
+ Sentence 2 contains 9 words: `"It is close to the sea /n and the mountains. /n"`
+ Sentence 3 contains 10 words: `"You can both ski in winter and swim in summer."`

### Approach 1: Sentence chunking with "no overlap"

Given a maximum number of tokens, iterate through the sentences and concatenate sentences until the maximum token length is reached. If a sentence is bigger than the maximum number of chunks, truncate to a maximimum amount of tokens, and put the rest in the next chunk.

NOTE: The examples ignore the newline `/n` character because it's not a token, but if the package or library detects new lines, then you'd see those line breaks here.

**Example: maximum tokens = 10**

```
Barcelona is a city in Spain.
It is close to the sea /n and the mountains. /n
You can both ski in winter and swim in summer.
```

**Example: maximum tokens = 16**

```
Barcelona is a city in Spain. It is close to the sea /n and the mountain. /n
You can both ski in winter and swim in summer.
```
    
**Example: maximum tokens = 6**

```
Barcelona is a city in Spain.
It is close to the sea /n
and the mountains. /n
You can both ski in winter
and swim in summer.
```

### Approach 2: Sentence chunking with "10% overlap"

Follow the same logic with no overlap approach, except that you create an overlap between chunks according to certain ratio.
A 10% overlap on maximum tokens of 10 is one token.

**Example: maximum tokens = 10**

```
Barcelona is a city in Spain.
Spain. It is close to the sea /n and the mountains. /n 
mountains. /n You can both ski in winter and swim in summer.
```

## Learn more about embedding models in Azure OpenAI

+ [Understanding embeddings in Azure OpenAI Service](https://learn.microsoft.com/azure/cognitive-services/openai/concepts/understand-embeddings)
+ [Learn how to generate embeddings](https://learn.microsoft.com/azure/cognitive-services/openai/how-to/embeddings?tabs=console)
+ [Tutorial: Explore Azure OpenAI Service embeddings and document search](https://learn.microsoft.com/azure/cognitive-services/openai/tutorials/embeddings?tabs=command-line)
