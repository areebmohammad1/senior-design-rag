# Project Constraints Essay
**Project Title:** Local Context-Aware Knowledge Copilot with Epistemic Tracing  
**Team Identifier:** EpistemicRAG  
**Team Members:** Mohammad Areeb (Computer Science), Patrick Caudill (Geosciences)  
**Faculty Advisor:** Dr. Andre Curtis-Trudel  

### Economic
The budget constraint that our project faces is that of no external investment being available at all for any paid service. This means we cannot have access to any third-party LLM APIs that may charge fees, like OpenAI or Anthropic. Therefore, in order to fulfill the requirement of this budget constraint, we must design our system in such a way that it uses only free open-source software, such as Ollama for model generation, Hugging Face for sentence transformers, and ChromaDB for indexing on our local systems. Accordingly, our deployment environment would be limited to consumer-based personal computers with 16 GB of RAM and not professional-grade computing clusters from the cloud.

### Security
Access for scholarly and business researchers shall strictly follow institutional data governance policies, thus ensuring that no proprietary documents and unpublished papers are accessible outside the local host environment. It is possible through the running of ingestion, vector indexing, and inference pipelines completely offline within the localhost environment. No information will be sent online; therefore, there will be no chance of API key interception or data logging by vendors.

### Ethical
Generative hallucination poses an explicit ethical dilemma in educational and scientific synthesis, as it includes the generation of non-existent scientific articles, which means deceiving the user with false evidential claims. In order to address the ethical issue, we have developed the deterministic epistemic tracing algorithm, which will link each claim that was generated with a certain piece of verified knowledge, providing a specific confidence score. There is an ethical fallback policy in place, according to which the model will not generate any claims unless there is a relevant indexed piece of information available.

### Design Trade-Off
There is a direct conflict between the economic assumption of zero costs and the latency of system performance because using quantized local models on the consumer CPU slows down inference compared to cloud APIs with multiple GPUs. The problem was overcome through the use of lightweight 3-billion-parameter quantized models (e.g., Llama 3.2 3B) along with the asynchronous fetching pipeline, where we sacrifice latency in favor of cost independence and data privacy.
