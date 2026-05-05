# Modern Systems: AI Agents & RAG

### 1. Overview
Modern software engineering increasingly involves integrating Large Language Models (LLMs) into system architectures. For senior engineers, this isn't just about "Chatbots," but about building resilient **Agentic Systems** that can use tools and **RAG** (Retrieval-Augmented Generation) architectures that provide LLMs with private, up-to-date context.

### 2. Key Concepts
*   **RAG (Retrieval-Augmented Generation)**: Providing the LLM with relevant documents retrieved from a database *before* generating a response. This reduces "Hallucinations."
*   **Vector DB**: Specialized databases (e.g., Pinecone, Weaviate, Milvus) that store data as high-dimensional "Embeddings" for fast similarity search.
*   **Agentic Systems**: LLMs that can "Reason" and "Act" by choosing which external tools (APIs, Python scripts, DB queries) to use to solve a problem.
*   **Tool Orchestration**: The middleware (e.g., LangChain, AutoGen) that manages the loops and state of an agent.
*   **Memory Systems**: How an agent remembers past interactions (Short-term context window vs. Long-term DB memory).

### 3. Real-World Usage
*   **Customer Support**: A RAG-based bot that retrieves "Company Policy PDFs" from a Vector DB to answer user questions with 100% accuracy and citations.
*   **AI Data Analyst**: An Agent that can write SQL, execute it against a DB, and generate a chart using Python—all from a single natural language prompt.
*   **Code Assistants**: Tool-using agents that can read an entire codebase, find a bug, and propose a pull request.
*   **Personalization**: Using Vector embeddings of "User Behaviors" to find similar products for recommendation in real-time.

### 4. Tradeoffs
*   **RAG vs. Fine-tuning**: RAG is cheaper, faster to update with new data, and more accurate for "Facts." Fine-tuning is better for changing the "Tone" or specialized vocabulary of a model.
*   **Agent Complexity vs. Control**: Agents provide massive power but are non-deterministic. A small change in the model's weights can break an agent's ability to call an API correctly.
*   **Latency**: AI responses take seconds. Engineering around this (using Stream responses, optimistic UI, and caching) is a core modern challenge.

### 5. When NOT to Use
*   **Predictable CRUD**: If you can solve a problem with a simple SQL query and a standard UI, do NOT use an LLM. It's too slow, too expensive, and unreliable.
*   **Hard Math/Logic**: LLMs are "Probability Engines," not calculators. For hard math or complex rigid logic, have the LLM call a Python tool rather than doing the calculation itself.

### 6. Interview Focus
*   **The Hallucination Problem**: "How do you design a system that ensures the LLM's response is based ONLY on the provided context?" (Hint: Prompt Engineering + RAG validations).
    * **The "Closed-Book" Architecture**: To ensure the LLM uses *only* the provided context, you need a "Closed-Book" RAG system. This is achieved through strict Prompt Engineering and Validation.
    * **Prompt Engineering (The "Jailbreak" Defense)**:
        * **Strict Context Adherence**: Your system prompt must explicitly forbid the model from using any external knowledge. A good example is the "Llama Guard" instruction set, which forces the model to respond with something like: *"I cannot answer this question as it is outside the scope of the provided documents."*
        * **Instructional Priming**: Start the prompt with: *"You are an AI assistant that ONLY uses the context provided below. If the answer is not in the context, say so."*
    * **Response Validation (The "Guardrail" Check)**:
        * **Citation Verification**: After the LLM generates an answer, your backend code should check if the answer actually references the provided documents (if your RAG system provides citations).
        * **hallucination Detection**: Use a smaller, faster "Toxicity/Safety" model (like Llama Guard or an off-the-shelf API) to classify the LLM's response as "Safe/Relevant" or "Unsafe/Hallucinated." If it fails, reject the response.
*   **Vector DB Selection**: "What is the difference between a 'Global Search' in SQL and a 'Semantic Search' in a Vector DB?"
    * **SQL (Global Search / Lexical Search)**:
        * **Mechanism**: Uses keyword matching and indexing (e.g., B-trees, Inverted Indexes).
        * **How it works**: It looks for exact text matches or patterns using operators like `LIKE '%term%'`.
        * **Example**: `SELECT * FROM products WHERE name LIKE '%apple%'`.
        * **Pros**: Fast, accurate for known terms, works well with structured data, uses less memory.
        * **Cons**: Cannot understand context or synonyms. It won't find "iPhone" if you search for "Apple," nor will it understand "laptop" if you search for "computer."
    * **Vector DB (Semantic Search)**:
        * **Mechanism**: Uses vector embeddings and Approximate Nearest Neighbor (ANN) search.
        * **How it works**: Converts text into high-dimensional numerical vectors. It finds vectors that are mathematically "close" to the query vector in the vector space.
        * **Example**: Searching for "laptop computer" might find documents containing "Apple MacBook" or "Dell XPS" because their vectors are close in meaning, even if the exact words don't match.
        * **Pros**: Understands context, synonyms, and relationships between words. Excellent for unstructured data (documents, images, audio).
        * **Cons**: Slower than SQL for exact matches, requires more memory (and often specialized hardware like GPUs for training embeddings), can have false positives.
*   **Cost Management**: "LLM API calls are expensive. How do you design a caching layer for common AI queries?" (Hint: Semantic Caching).
    * **The Problem**: Traditional caching (based on exact query text) fails because `What is the return policy?` and `How do I return an item?` are different strings but have the same meaning. A naive cache would make two API calls, wasting tokens and money.
    * **The Solution: Semantic Caching**:
        1.  **Generate Embeddings**: When a user asks a question, convert it into a vector embedding using the same embedding model as your RAG system.
        2.  **Vector Similarity Search**: Use a Vector Database (or an in-memory store like FAISS/Annoy) to search for *similar* embeddings, not exact matches.
        3.  **The "Cache Hit" Logic**:
            * **Threshold**: Define a "Similarity Threshold" (e.g., a cosine similarity score of 0.85 or higher).
            * **Hit**: If you find a stored query whose embedding is very close to the new query (above the threshold), you assume the answer will be the same.
            * **Response**: Return the previously cached LLM response *without* calling the expensive LLM API again.
            * **Miss**: If no similar query is found, call the LLM, store the new query and its response in the cache, and then return the result.

### 7. Common Mistakes
*   **Ignoring Token Costs**: Building a RAG system that sends the entire company handbook for every query, resulting in a $10,000 monthly OpenAI bill.
*   **Bad Chunking**: Breaking documents into chunks that are too small, losing the context needed for the LLM to understand the data.
*   **Evaluating "Feel" instead of "Metrics"**: Testing an AI system by just asking it 5 questions and saying "it feels good," rather than using an automated evaluation framework (like RAGAS).
