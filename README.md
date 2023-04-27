# LLM in Applications

This is a summary of https://huyenchip.com/2023/04/11/llm-engineering.html

# ToC

- [Challenges in Productionizing LLms](#part-i-challenges-in-productionizing-llms)
- [Composability of multiple tasks](#part-ii-composability-of-multiple-tasks)


# Part I: Challenges in productionizing LLMs

### Ambiguity
A natural language is way more ambiguous than a programming language. Instead of programming specific instructions, natural language specifies rather an intend and can thus lead to different interpretations. This is esp challenging when the LLM will be integrated with other applications. 

Examples for problems with ambiguity are:
- Variety in output format: even though the model can be instructed to use a specific output format, it not always will.
- Stochastic nature of the model: the "same" input can lead to different outputs (setting `temperature=0` helps but not always).
- Other important aspects of ambiguity in LLMs are:
    1. Prompt evaluation: Prompts are commonly used to "fine tune" the model, i.e. to act in a specific way (fewshot learners). However, its not clear if the model can understand it and also generalizes sufficiently from these prompts. This needs to be evaluated, e.g. with a test set. 
    2. Prompt versioning: As in software development, prompts should be versioned to track the performance of the prompts (eval prompts) and track changes over time.
    3. Prompt optimization: There are multiple ways of optimizing prompts. E.g. Chain-of-that approach that instructs the model to explain how it arrives at an answer, generate multiple outputs for the same input (variants), break big prompts into small and simple prompts.

### Cost and Latency
- Cost: the tradeoff is simple. The more examples you put in the prompt and the more detailed explanation you request, the better the output but this increases the number of tokens used in the API request / response which leads to higher costs. For **LLMOps**, cost is esp. inference related. 
> As a thought exercise, in 2021, DoorDash ML models made 10 billion predictions a day. If each prediction costs $0.004, that’d be[with openAI pricing] $40 million a day!
- Latency: Output length significantly affects latency, as output can't be generated in parallel. For 1 token the output latency is around 0.5 s and with 20 tokens already > 1 s. Hosting your own model will in turn (depending on the no. of paramters) need a lot of time and compute reources and probably even have worse latency.

### Prompting vs. Fine Tuning vs. Alternatives
- Definitions: 
    1. Prompting: give samples and tell the model how to respond (act like)
    2. Finetuning: actually train an existing LLM with your own data.
- Factors to be considered when making the decision: data availability, costs and performance. In general you can say the more examples you have, the more likely it is that the model will perform better when re-trained on these examples than just prompted. The author mentioned 100 examples but thats not a hard threshold. Also costs of prediction can be reduced on fine tuned models since the payload can be less bc less instructions are needed in any prompt.
- Alternatives: 
    1. Prompt tuning: instead of changing a prompt, you'd change the embedding for it (via code). You need to be able to input prompt embeddings into your model to do this -> thus you must use open source LLMs.
    2. Finetuning with distillation: distillation can be defined as training a small model to imitate a bigger one. E.g. fine tune a model based on the examples of a bigger one. 
    3. Embeddings and vector DBs: you can use LLMs to generate embeddings, store them in a vector DB and use these for your applications. When a new item is requested which you didn't create an embedding yet, you can create it in real time with (e.g.) the openAI API. Its like a cache. 
    4. Backward and forward compatibility: models have to be retrained on new data from time to time. However when this happens, its not guaranteed that the model will behave the same with old prompts (backwards compatibility). Old prompts should be covered with prompt evaluation in a unit test manner.  


### Useful links

- [Tips and tricks for prompt engineering by OpenAI](https://github.com/openai/openai-cookbook/blob/main/techniques_to_improve_reliability.md#how-to-improve-reliability-on-complex-tasks)
- [Github repo for Embeddings with GPT](https://github.com/Muennighoff/sgpt)
- [Performance and costs of GPT3 embeddings article](https://medium.com/@nils_reimers/openai-gpt-3-text-embeddings-really-a-new-state-of-the-art-in-dense-text-embeddings-6571fe3ec9d9
)

# Part II: Composability of multiple tasks

Multiple tasks can be found if you need to include other systems as just the LLM. For example if you need to query a database with a LLM generated sql the tasks are: 
1. Convert natural language to sql query (LLM)
2. Execute sql query on database (SQL executor)
3. Convert SQL results to natural language (LLM)

LLMs, such as ChatGPT, can use tools / plugins to perform tasks. These are for example search, bash execution, 3rd party APIs, etc. You can instruct ChatGPT, for example, to generate a picture using a 3rd Party API Plugin (Dall-E API) and post it on twitter (Twitter API).

Tasks can be performed in different ways (control flows). Sequentially, parallel, for-loop, if-/while-condition. 
The model can be instructed to use different tools with different control flows by using a prompt. Or the model API is integrated in an external control flow, that is programmed.

When using a control flow, the developer should implement tests (see prompt evaluation) for it in order to ensure it works correctly. 
 