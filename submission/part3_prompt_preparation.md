# Part 3: Prompt Preparation

**Selected PR:** MetaGPT #1172 (Make RAG embedding configurable and add gpt-4-turbo in token_counter)

### 3.1.1 Repository Context
MetaGPT is an advanced multi-agent framework designed to simulate a software development company. Users provide a single-line software requirement, and MetaGPT deploys specialized AI agents—acting as Product Managers, Architects, Project Managers, and Engineers—to collaboratively generate Product Requirements Documents (PRDs), system designs, task lists, and the final repository code. 

The intended users are software developers, AI researchers, and engineering teams looking to automate repetitive parts of the software engineering lifecycle. It primarily addresses the problem domain of LLM-based autonomous coding and complex multi-agent collaboration. To achieve this effectively, the framework requires high configurability, allowing users to leverage various Large Language Models and embedding providers depending on their cost constraints, latency requirements, and data privacy policies.

### 3.1.2 Pull Request Description
This PR addresses the rigidity of the framework's Retrieval-Augmented Generation (RAG) feature. Previously, the embedding model used for RAG was tightly coupled to either OpenAI or Azure through default configurations, and utilizing other providers required manual, cumbersome workarounds. 

This change introduces an abstract and configurable embedding mechanism, allowing seamless integration with an extended list of providers, specifically adding built-in support for Gemini and Ollama. Additionally, it updates the framework's token counter to support the newly released `gpt-4-turbo` model. The new behavior allows users to specify their preferred embedding provider directly in the standard `llm_config` file. If no configuration is explicitly provided, it safely defaults to the previous OpenAI behavior, ensuring seamless backward compatibility.

### 3.1.3 Acceptance Criteria
✓ When the `llm_config` specifies `gemini` as the embedding provider, the system should successfully initialize the Gemini embedding model without raising a missing configuration error.
✓ The implementation should handle the initialization of `ollama` embeddings by routing the configuration to the local Ollama client instance.
✓ When text is passed to the token counter with the model string `gpt-4-turbo`, the system should return an accurate token count using the appropriate base encoding.
✓ When no explicit embedding provider is defined, the system should gracefully default back to the previous OpenAI/Azure behavior to ensure backward compatibility.
✓ The RAG engine should correctly generate embeddings for a sample text document using any of the four supported providers (openai, azure, gemini, ollama).

### 3.1.4 Edge Cases
1. **Missing Credentials:** The system must gracefully handle scenarios where the user specifies a provider (e.g., `gemini`) but fails to supply the required API key in the configuration.
2. **Unrecognized Provider:** If an unsupported embedding type string (e.g., `unknown-provider`) is provided, the system should raise a clear, descriptive `NotImplementedError` rather than failing silently.
3. **Tokenizer Fallback:** If `gpt-4-turbo` is passed to the token counter but the underlying tokenizer library is outdated and lacks the specific mapping, it should fall back to a standard encoding (like `cl100k_base`) to avoid application crashes.

### 3.1.5 Initial Prompt
"You are tasked with enhancing the MetaGPT multi-agent framework by making its Retrieval-Augmented Generation (RAG) embedding module configurable and updating the token tracking utilities. Currently, MetaGPT defaults to using OpenAI or Azure for embeddings based on the `llm_config`. We need to expand this to support a wider array of providers, specifically adding built-in configuration support for Gemini and Ollama. 

First, refactor the embedding initialization logic within the RAG engine (`metagpt/rag/engines/`). Instead of hardcoding the embedding provider, implement a factory pattern or a configuration parser that reads the embedding type from the standard `llm_config`. It should dynamically instantiate the correct embedding client (OpenAI, Azure, Gemini, or Ollama) based on this config. Ensure that if no provider is explicitly set, the system defaults to the existing OpenAI/Azure behavior to prevent breaking changes for current users.

Second, navigate to the `metagpt/utils/token_counter.py` module. Update the internal token mapping dictionary to include support for the `gpt-4-turbo` model identifier. Ensure it utilizes the correct underlying encoding standard (e.g., `cl100k_base`). 

When implementing these features, please handle critical edge cases. Specifically, ensure that missing API keys for newly supported providers throw an actionable configuration error, and ensure that specifying an unsupported provider string raises a clear `NotImplementedError`. The final implementation must pass tests validating that `gpt-4-turbo` tokens are counted correctly and that the RAG pipeline can successfully generate embeddings using Ollama locally."

---

**Integrity Declaration:**
I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.
