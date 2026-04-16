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
*   **Vector DB Selection**: "What is the difference between a 'Global Search' in SQL and a 'Semantic Search' in a Vector DB?"
*   **Cost Management**: "LLM API calls are expensive. How do you design a caching layer for common AI queries?" (Hint: Semantic Caching).

### 7. Common Mistakes
*   **Ignoring Token Costs**: Building a RAG system that sends the entire company handbook for every query, resulting in a $10,000 monthly OpenAI bill.
*   **Bad Chunking**: Breaking documents into chunks that are too small, losing the context needed for the LLM to understand the data.
*   **Evaluating "Feel" instead of "Metrics"**: Testing an AI system by just asking it 5 questions and saying "it feels good," rather than using an automated evaluation framework (like RAGAS).
