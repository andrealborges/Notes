# Introduction

Generative AI, and technologies that implement it are increasingly in the public consciousness – even among people who don't work in technology roles or have a background in computer science or machine learning. The futurist and novelist Arthur C. Clarke is quoted as observing that "any sufficiently advanced technology is indistinguishable from magic". In the case of generative AI, it does seem to have an almost miraculous ability to produce human-like original content, including poetry, prose, and even computer code.

However, there's no wizardry involved in generative AI – just the application of mathematical techniques incrementally discovered and refined over many years of research into statistics, data science, and machine learning. You can gain a high-level understanding of how the magic trick is done by learning the core concepts and principles explored in this module. As you learn more about the generative AI technologies we have today, and how to use them responsibly, you can help society imagine new possibilities for AI tomorrow.

# What is generative AI?

Artificial Intelligence (AI) imitates human behavior by using machine learning to interact with the environment and execute tasks without explicit directions on what to output.

**Generative AI** describes a category of capabilities within AI that create original content. These capabilities include taking in natural language input, and returning appropriate responses in a variety of formats such as natural language, images, code, and more.  

## Natural language generation

To generate a natural language response, you might submit a request such as:  

*"Write a cover letter for a person with a bachelor's degree in history."*

A generative AI application might respond with a letter starting like this:

Dear Hiring Manager, I am writing to express my interest in the position of...

## Image generation

Some generative AI applications can interpret a natural language request and generate an appropriate image.  

**Example request:**  
*"Create a logo for a florist business."*  

A generative AI application could then return a new image based on the description you provided.

## Code generation

Some generative AI applications are designed to help software developers write code.  

**Example request:**  
*"Write Python code to add two numbers."*  

A generative AI application could generate the following response:

```python
def add_numbers(a, b):
    return a + b
```

# How do language models work?

Over the last decades, multiple developments in the field of natural language processing (NLP) have resulted in achieving **large language models (LLMs)**. The development and availability of language models led to new ways to interact with applications and systems, such as through generative AI assistants and agents.

Let's take a look back at historical developments for language models which include:

- **Tokenization:** enabling machines to read.  
- **Word embeddings:** enabling machines to capture the relationship between words.  
- **Architectural developments:** enabling them to capture word context.  

## Tokenization

Machines have a hard time deciphering text as they mostly rely on numbers. To read text, we therefore need to convert the presented text to numbers.  

One important development to allow machines to more easily work with text has been **tokenization**. Tokens are strings with a known meaning, usually representing a word. Tokenization is turning words into tokens, which are then converted to numbers.  

A statistical approach to tokenization is by using a pipeline:

1. Start with the text you want to tokenize.  
2. Split the words in the text based on a rule (for example, split where there's a white space).  
3. **Stop word removal:** remove noisy words that have little meaning, like *the* and *a*.  
4. Assign a number to each unique token.  

Tokenization allowed for text to be labeled. As a result, statistical techniques could be used to let computers find patterns in the data instead of applying rule-based models.

## Word embeddings

One of the key concepts introduced by applying **deep learning** techniques to NLP is **word embeddings**. Word embeddings address the problem of not being able to define the semantic relationship between words.

Word embeddings are created during the deep learning model training process. During training, the model analyzes the co-occurrence patterns of words in sentences and learns to represent them as **vectors**.  

A vector represents a path through a point in *n*-dimensional space. Semantic relationships are defined by how similar the angles of the lines are (the direction of the path). Because word embeddings represent words in a vector space, the relationship between words can be easily described and calculated.

To create a vocabulary that encapsulates semantic relationships between tokens, we define **contextual vectors (embeddings)** for them. Vectors are multi-valued numeric representations, for example `[10, 3, 1]`, where each element represents a particular attribute of the token. The categories of these attributes are determined during training, based on how commonly words are used together or in similar contexts.

- Semantically similar tokens result in vectors that point in the same direction.  
- **Cosine similarity** is used to measure the similarity of direction (regardless of distance).  

**Example:**  
- "dog" and "puppy" embeddings point in almost the same direction.  
- Their direction is similar to "cat".  
- "skateboard," however, points in a very different direction.  

## Architectural developments

The **architecture** (design) of a machine learning model describes the structure and organization of its various components and processes. It defines how data is processed, how models are trained and evaluated, and how predictions are generated.  

One of the first breakthroughs in language model architecture was the **Recurrent Neural Network (RNN)**.

### Why context matters
To understand text isn't just to understand individual words. Words can differ in meaning depending on the **context** in which they appear.  

RNNs account for context by processing inputs sequentially:

- Each step takes an input token and a hidden state (memory).  
- The hidden state stores the output of the previous step and passes it to the next.  
- This allows the network to "remember" previous words when predicting the next one.  

**Example sentence:**  
*"Vincent Van Gogh was a painter most known for creating stunning and emotionally expressive artworks, including ..."*  

To predict the missing last word, the model must remember "Vincent Van Gogh." Using the special `[MASK]` token, you can tell the model to predict the missing word:

*"Vincent was a painter known for [MASK]"*  

The RNN processes each token, updating its hidden state, and when it reaches `[MASK]`, it predicts **Starry Night**.

## Challenges with RNNs

- The hidden state contains **all tokens**, treating them equally.  
- Relevant information may be **diluted or overwritten** by newer, less relevant tokens.  
- Long-distance dependencies (like remembering "Vincent Van Gogh" at the beginning of a long sentence) are hard to preserve.  

As a result, important signals can become weak and be overlooked because irrelevant information dominates the hidden state.

✅ So far, we've seen how:  
- Language models read text through **tokenization**.  
- They understand relationships between words using **word embeddings**.  
- They capture context with early architectures like **RNNs**.  


# Understand how transformers advance language models

The generative AI applications we use today are made possible by utilizing **Transformer architecture**. Transformers were introduced in the *Attention is All You Need* paper by Vaswani et al. (2017).

Transformer architecture introduced concepts that drastically improved a model's ability to understand and generate text. Different models have been trained using adaptations of the Transformer architecture to optimize for specific NLP tasks.

## Understand Transformer architecture

There are two main components in the original Transformer architecture:

- **The encoder:** Processes the input sequence and creates a representation that captures the context of each token.  
- **The decoder:** Generates the output sequence by attending to the encoder's representation and predicting the next token in the sequence.  

The most important innovations presented in the Transformer architecture were **positional encoding** and **multi-head attention**.  

- In the **encoder layer**, an input sequence is encoded with positional encoding, after which multi-head attention is used to create a representation of the text.  
- In the **decoder layer**, an (incomplete) output sequence is encoded in a similar way. Multi-head attention is used twice:  
  - Once to process the encoded output sequence.  
  - Again to combine it with the encoder’s output.  

As a result, the model can generate text output.

## Understand positional encoding

The position of a word and the order of words in a sentence are important to understand meaning. To include this information without processing text sequentially, Transformers use **positional encoding**.

- Before Transformers, models used **word embeddings** to encode text into vectors.  
- In Transformers, **positional encoding** is added to embeddings.  
- The sum of word embeddings + positional vectors ensures each token carries both **semantic meaning** and **positional information**.  

A simple approach would be to assign an index to each word. However, large indices grow with sentence length, hold little meaning, and may cause instability during training. Positional encoding solves this problem by embedding richer positional information into the model.

## Understand attention

The most important technique used by Transformers is **attention**, which replaces recurrence.  

- RNNs process text **sequentially**, which is compute-intensive.  
- Transformers process words **in parallel**, using attention.  

**Attention** (also called *self-attention* or *intra-attention*) maps new information to learned information, helping the model determine what the new information entails.

- Each word is encoded (with positional encoding) and represented as a **query**.  
- Encoded words also produce **keys** and **values**.  
- The model compares the query with keys to find the closest match, and returns the associated value.  

**Example:**  
Sentence: *"Vincent van Gogh is a painter, known for his stunning and emotionally expressive artworks."*  
- Query: "Vincent van Gogh"  
- Key: "Vincent van Gogh" → Value: "painter"  

Later, given a new query like *"Shakespeare's work has influenced many movies, mostly thanks to his work as a..."*, the model finds:  
- Query: "Shakespeare"  
- Closest key: "William Shakespeare" → Value: "playwright"  

### How attention is calculated

1. Queries, keys, and values are encoded as **vectors**.  
2. The **scaled dot-product** is calculated between the query and keys to measure alignment.  
3. A **softmax function** converts these scores into a probability distribution.  
4. The key with the highest probability is selected, and its value is returned as output.  

## Multi-head attention

Transformers use **multi-head attention**, meaning tokens are processed by several attention mechanisms in parallel.  

- This enables the model to analyze the same word or sentence from multiple perspectives.  
- Different "heads" can capture different types of relationships or context.  


✅ With Transformers, models can process tokens in parallel and capture complex contextual relationships more efficiently, enabling the breakthroughs behind today’s **generative AI systems**.


# Understand Differences in Language Models

Today, importantly, developers don't need to train models from scratch. To build a generative AI application, you can use pretrained models. Some language models are open-source and publicly available. Others are offered in proprietary catalogs. Different models exist today which mostly differ by the specific data used to train them, or by how they implement attention within their architectures.

## Large and Small Language Models

In general, language models can be considered in two categories: **Large Language Models (LLMs)** and **Small Language Models (SLMs).**

| Large Language Models (LLMs) | Small Language Models (SLMs) |
|-------------------------------|-------------------------------|
| Trained with vast quantities of text that represent a wide range of general subject matter – typically by sourcing data from the Internet and other publications. | Trained with smaller, more subject-focused datasets. |
| Have many billions (even trillions) of parameters (weights applied to embeddings to calculate predicted token sequences). | Typically have fewer parameters than LLMs. |
| Exhibit comprehensive language generation capabilities across a wide range of conversational contexts. | More effective in specific conversational topics, but less effective for general language generation. |
| Large size impacts performance and makes them difficult to deploy locally. | Smaller size allows deployment options including local devices and on-premises computers


# Improve Prompt Results

The quality of responses from generative AI assistants not only depends on the language model used, but also on the types of prompts users provide. Prompts are ways we tell an application what we want it to do. You can get the most useful completions by being explicit about the kind of response you want.  

**Example:**  
*"Summarize the key considerations for adopting Copilot described in this document for a corporate executive. Format the summary as no more than six bullet points with a professional tone."*  

Users of generative AI can achieve better results when they submit clear, specific prompts.

## Ways to Improve Responses

- Start with a specific goal for what you want the assistant to do.  
- Provide a source to ground the response in a specific scope of information.  
- Add context to maximize response appropriateness and relevance.  
- Set clear expectations for the response.  
- Iterate based on previous prompts and responses to refine the result.  

## What Happens to Your Prompt

In most cases, an agent doesn't just send your prompt as-is to the language model. Usually, your prompt is augmented with:

- **System message:** Sets conditions and constraints for model behavior.  
  *Example: "You're a helpful assistant that responds in a cheerful, friendly manner."*  
- **Conversation history:** Includes past prompts and responses to maintain context and enable iterative refinement.  
- **Current prompt:** May be optimized or reworded by the agent, sometimes with grounding data to better scope the response.  

## Prompt Engineering

The term **prompt engineering** describes the process of improving prompts. Both developers designing applications and end-users can enhance the quality of generative AI responses by practicing prompt engineering.


# Create Responsible Generative AI Solutions

The Microsoft guidance for responsible generative AI is designed to be practical and actionable. It defines a **four-stage process** to develop and implement a plan for responsible AI when using generative models.

## Four Stages

1. **Identify** potential harms that are relevant to your planned solution.  
2. **Measure** the presence of these harms in the outputs generated by your solution.  
3. **Mitigate** the harms at multiple layers to minimize their presence and impact, and ensure transparent communication about potential risks to users.  
4. **Operate** the solution responsibly by defining and following a deployment and operational readiness plan.  

These stages should be informed by responsible AI principles. Microsoft has categorized these principles into six key areas.


# Responsible AI Principles

It's important for software engineers to consider the impact of their software on users and society in general, especially when applications involve AI. Due to the probabilistic nature of AI systems and the trust users often place in them, there is potential for harm through incorrect predictions or misuse. Responsible AI ensures fairness, reliability, and adequate protections against discrimination or risks.

## Fairness
AI systems should treat all people fairly.  
- Example: A loan approval model should not discriminate based on gender, ethnicity, or other demographic factors.  
- Fairness requires attention from the start of development: review training data for representativeness and test predictive performance across different subgroups.  

## Reliability and Safety
AI systems should perform reliably and safely.  
- Example: An autonomous vehicle or medical diagnosis model must avoid unreliability, as errors could endanger lives.  
- Apply rigorous testing, deployment processes, and confidence thresholds to account for the probabilistic nature of AI models.  

## Privacy and Security
AI systems should respect privacy and maintain security.  
- Models often rely on sensitive or personal data.  
- Safeguards must protect both training and operational data, ensuring compliance with privacy requirements.  

## Inclusiveness
AI systems should empower and engage everyone.  
- AI should benefit all parts of society, regardless of physical ability, gender, sexual orientation, ethnicity, or other factors.  
- Design, development, and testing should involve diverse groups of people.  

## Transparency
AI systems should be understandable.  
- Users should know the system’s purpose, how it works, and its limitations.  
- Example: Communicate factors influencing predictions (training data size, key features, confidence scores).  
- If personal data is used (e.g., facial recognition), explain how it’s collected, stored, and accessed.  

## Accountability
People are accountable for AI systems.  
- Developers and organizations are responsible for training, validating, and deploying models, as well as ensuring compliance with governance and legal standards.  
- Establish frameworks of governance to uphold responsibility throughout the AI lifecycle.  
