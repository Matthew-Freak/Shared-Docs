# Trade Hub chat bot potential options

| Ref | Stack Type                 | Stack                                                                                                                                                               | Data Protection                                                                                                                                                                                              | Ouput Quality                                                                                                           | Pros                                                                                                                                                              | Cons                                                                        | Dev effort* and cost       | Run cost <br>(monthly estimated - see below for calculations)                                            |
| --- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------- | -------------------------------------------------------------------------------------------------------- |
| A   | OSS / self hosted / hybrid | • Ingestion / retrieval: **Haystack 2**<br>• Embedding: **OpenAI `text-embedding-3-small`**  <br>• Vector store: **Qdrant**<br>• LLM / Generator: **OpenAI GPT-4o** | **Good**<br>• Only time it leaves our org is for the OpenAI API.<br>• OpenAI does not use this data for training and only stores for 30 days for abuse monitoring. Can request **Zero-Data Retention (ZDR)** | **Excellent**<br>Open AI are generally pretty close to SOTA                                                             | • Quickest OSS path (Haystack pipelines premade)  <br>• Small footprint – no GPU needed for retrieval <br>• Highly module, can switch out LLM or other components | • Every answer leaves VPC -> OpenAI  <br>• Hard latency ceiling on API call | ~4wk<br>Embedding: >A$1.00 | **A$86.71 - A$112.38/mo**<br>• Semantic search: A$32.04 - A$57.71/mo<br>• Gen answers: A$54.67/mo        |
| B   | OSS / self hosted          | • Ingestion / retrieval: **LangChain**<br>• Embedding: **MiniLM/s-BERT**  <br>• Vector store: **OpenSearch k-NN**<br>• LLM / Generator: **Llama-3 8 B**             | **Excellent**<br>We can choose where everything is hosted and where our data goes and is stored                                                                                                              | **Good**<br>Not as good as OpenAI unless highly tuned                                                                   | • Completely OSS (no SaaS)<br>• Can fine-tune Llama locally on RTX 4060**                                                                                         | • Requires GPU hosting<br>• Lowest answer quality unless highly tuned       | ~6wk                       | **A$586.44/mo*****<br>• Semantic search: A$32.04 - A$57.71/mo<br>• Gen answers: A$554.40/mo              |
| C   | Managed                    | **Google Vertex AI “Search & Chat”**                                                                                                                                | **Ok**<br>• Data sits in google cloud but we can enable Customer-managed encryption keys (CMEK).<br>• They don't train on our data and we can enable ZDR                                                     | **Excellent**<br>Gemini models are generally pretty close to SOTA                                                       | • No infrastructure to manage<br>• Simplest and quickest to get running<br>• Drop-in UI                                                                           | • Not much tuning ability<br>• Vendor lock in                               | ~1.5wk                     | **A$24.47 - A$67.76/mo**<br>• Basic: A$33.88 - A$67.76/mo<br>• Alt: A$24.47 - A$45.65/mo                 |
| D   | Managed                    | **Amazon Kendra**                                                                                                                                                   | **Good**<br>• Uses VPC endpoint and KMS encryption<br>• Does not train on our data                                                                                                                           | **Excellent**<br>Claude models are generally pretty close to SOTA                                                       | • Best access controls<br>• Best ingestion<br>• Changeable LLM                                                                                                    | • Hourly billing<br>• No drop-in UI                                         | ~3wk                       | **A$432.43/mo**<br>• Kendra (semantic search): A$354.81/mo<br>• Bedrock (generative answers): A$77.62/mo |
| E   | Managed                    | **Azure AI Search + Copilot**                                                                                                                                       | **Good**<br>• Uses CMEK<br>• No training but unclear if they have ZDR. Standard is 30 day retention.                                                                                                         | Should be **Excellent**<br>Personally haven't liked copilots output as much but it uses GPT-4o now so it should be good | • Drop-in UI<br>• Deep Microsoft stack                                                                                                                            | • Not much tuning ability <br> • Limited to AUS-East region                 | ~3wk                       | **A$210.68**<br>• Azure AI search: A$156.12/mo<br>• Azure OpenAI: A$54.56/mo                             |

>**Disclaimer:** The stacks outlined in this document represent the most logical configurations at the time of writing. Each stack’s components (Semantic search and Generative answers for managed, everything for OSS) can be mixed and matched to varying degrees based on requirements such as cost, compliance, and performance. Final implementation choices may evolve as vendor capabilities and project priorities change.

>*Estimate based off preliminary research and is liable to change  
>**Hopefully has enough memory and isn't too slow  
>***At minimum
## Pricing calculations
### Users: 1,100/mo
Based on a 12 month, calendar month by month, average of the pervious 12 months as recorded by GA4 active users report  
### Queries: 5500/mo
based on an average 5 queries per person per month  
### Tokens: 6.4m/mo
#### Input tokens: 3.8m/mo
common approximation **1 token ≈ 0.75 words**  
**My assumptions**
- User prompt ≈ **20 words** = ~**27 tokens**
- Retrieved context (from semantic search) ≈ **350 words** = ~**467 tokens**
- Input overhead ≈ **200 tokens**
**Per request Input tokens** ≈ 27 + 467 + 200 = **~694 tokens**  
>**Per month (5,500 queries) Input tokens:** 694 × 5,500 ≈ **~3.81 million tokens**  
### Output tokens: 2.6m/mo
common approximation **1 token ≈ 0.75 words**  
**My assumptions**
- Model reply ≈ **350 words** = ~**467 tokens**
**Per request Output tokens** ≈ **~467 tokens**  
>**Per month (5,500 queries) output tokens:** 467 × 5,500 ≈ **~2.57 million tokens**  

## Option A (Hybrid)
Using the conversion of 1 USD = 1.54 AUD (at time of writing)
#### Semantic Search
We can potentially reuse our existing Oracle VPS for ingestion, embedding and vector store.  
If we need more resources we can buy more oracle VPS' or upgrade our existing one.  
**Plan:** Oracle VPS VM.Standard.E5.Flex, 1-2 OCPU, 8-16GB  
**Price: A$32.04 - A$57.71/mo**
#### Generative Answers
**Plan:** Open AI gpt-4o  
Input tokens: 3.8m @ A$3.85 per 1m = A$14.63/mo (assuming no cached input)  
Output tokens: 2.6m @ A$15.40 per 1m = A$40.04/mo  
**Total: A$54.67/mo**
#### **Grand total: A$86.71 - A$112.38/mo**
## Option B (Pure OSS)
Using the conversion of 1 USD = 1.54 AUD (at time of writing)
#### Semantic Search
We can potentially reuse our existing Oracle VPS for ingestion, embedding and vector store.  
If we need more resources we can buy more oracle VPS' or upgrade our existing one.  
**Plan:** Oracle VPS VM.Standard.E5.Flex, 1-2 OCPU, 8-16GB  
**Price: A$32.04 - A$57.71/mo**
#### Generative Answers
This will require GPU hosting  
Due to the nature of the service and the relatively high cold start time, it is likely we will require always on hosting rather than on demand, which adds significant cost.  
Lowest always on GPU hosting: Hugging face T4 16GB @ A$0.77/hr x 720 hours/month = **A$554.40/mo**
#### **Grand total: A$586.44/mo***
*At a minimum
## Option C (Google)
Using the conversion of 1 USD = 1.54 AUD (at time of writing)

**Plan:** Vertex AI Search Enterprise Edition  
Base: 5500 queries @ A$6.16 per 1000 = A$33.88 (we get a 10k queries free trial as well)  
Data indexed: >10GB = A$0  
Optional Advanced Generative Answers: 5500 queries @ A$6.16 per 1000 = +A$33.88  
**Total: A$33.88 - A$67.76/mo**  

***Alternative - direct call LLM - better control***  
**Plan:** Vertex AI Search Standard Edition + Gemini 2.5 Flash  
Base: 5500 queries @ A$2.31 per 1000 = A$12.70  
Input tokens: 3.8m @ A$0.462 per 1m = A$1.76  
Output tokens: 2.6m @ A$3.85 per 1m = A$10.01  
Data indexed: >10GB = A$0  
Grounded Generation (Maybe required): 5500 queries @ A$3.85 per 1000 = A$21.18  
**Total: A$24.47 - A$45.65/mo**  
## Option D (AWS)
Using the conversion of 1 USD = 1.54 AUD (at time of writing)  
#### Amazon Kendra (semantic search)
**Plan:** GenAI Enterprise Edition  
Base: A$0.49/hr x 720 hours/month = **A$354.81/mo***  
*Assuming >=200MB of extracted text from tech docs
#### Amazon Bedrock (generative answers)
**Plan:** Anthropic Claude Sonnet 4, On-Demand, ap-southeast-2  
Input tokens: 3.8m @ A$0.00462 per 1000 = A$17.56/mo  
Output tokens: 2.6m @ A$0.0231 per 1000 = A$60.06/mo  
**Total: A$77.62/mo**  
#### **Grand total A$432.43/mo**
## Option E (Azure)
Using the conversion of 1 USD = 1.54 AUD (at time of writing)  
Using MCA licensing program  
#### Azure AI Search
**Plan:** Basic, 1 unit, 5.5k semantic queries  
base: A$149.21/mo  
Semantic ranker: 5500 queries @ A$1.54 per 1000 = A$6.92/mo  
**Total: A$156.12/mo**  
#### Azure OpenAI
**Plan:** GPT-4o-2024-08-06, Standard (o3 is slightly cheap for most likely better answers but longer response time)  
Input tokens: 3.8m @ A$0.0038 per 1000 = A$14.60/mo (assuming no cached input)  
Output tokens: 2.6m @ A$0.0154 per 1000 = A$39.96/mo  
**Total: A$54.56/mo**  
#### **Grand total A$210.68/mo**