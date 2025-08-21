# Medical_Asst_Bot
### Objective
The probelem is to create a Medical assistant bot that can be trained on prior question and answer set provided and be used to answer new medical related questions.

### Infra:
Was modeled on Google Colab notebook
Used A100GPU for training

### Data Preprocessing
Creation of Data pipeling with the following steps
1. Data Cleaning
2. removing Na's and dropping duplicates
3. Answers are are huge size hence need to chunk them into multiple answers
4. Split the data into Training , validation and testing set with configured splits
5. add extra fields in data like original qa_id - quesiton and answer id, chunk_idx -> each answer chunk is an id
6. Combine the question and answers as f"question: {question}\n Answer: {answer_chunk}" format into a new field called text

### Model Training
#### Approach:
-The first step is to convert each text into vector embeddings using "sentence-transformers/all-MiniLM-L6-v2" model, index them using FAISS and store it as a vector database object.
-When the application receives new query, FAISS does a Similarity search of the vecor with the new query and returns the top k similar candidates.
-Further these k candidates are reranked using the HuggingFace CrossEncoder and choosing RoBERTa model "cross-encoder/stsb-roberta-base".
-Now from the top k, we consolidate them into bigger context and feed them into Generator model (google) in our RAG pipeling, with appropriate and precise prompt.
-The Generator model generates results

### Model Evaluation Technique
For this purpose, I have worked on evaluating the candidate retrieval model.
Metrics used:
Precision@k = \frac{number of relevant answers in top k, k}
Recall@k = \frac{number of relevant answers in top k,number of relevant answers in the entire pool}

Challenge:
- since we cannot have exact match on answers, Used a second sentence embedding model to compare the ground truth answer of validation data vs the candidate solutions
- After embedding, cosign similarity search was used to look for similarities between ground truth answer vs candidate search answers

Here we are not concerned about the exact rank of the candidate answers but we are looking for the fact that relevant answers need to appear in the top k
Hence the choice of Precision and Recall @ k

### Example Interaction:
1). What are the symptoms of Breast Cancer ?
Answer: lumps or anything else that seems unusual. An unusual (higher or lower than normal) amount of a substance can be a sign of disease. Tests that examine the breasts are used to detect (find) and diagnose breast cancer. The doctor will carefully feel the breasts and under the arms for lumps or anything else that seems unusual. 

2). What are the genetic changes related to infantile systemic hyalinosis ?
Answer: mutations. The parents of an individual with an autosomal recessive condition each carry one copy of the mutated gene, but they typically do not show signs and symptoms of the condition.

3). What is (are) Pain ?
Answer: a feeling set off in the nervous system to alert you to possible injury and the need to take care of yourself.


### Model Performance
- The model seem to perform pretty poorly on the retrieval of candidates as the precision score is lingering around 38%
- As as result and poor performance of the generative model, final responses do hallucinate or repeats the exact answer from the retrieved candidates.
Given time, I would have deployer or research a better generator in order to curate the context and provide better answers. Some steps I am take in future are working with better and precise prompts, curating the context in a better and consice way and experimenting with different models.

### Disclaimer
I, hearby confirm that I have not used OpenAI, Caude Code or any other AI tools directly to code my application. I, however used Youtube videos(tutorials), google search, hugging face, machine learning blogs, FAISS, tutorials, my previous practice code to augment my knowledge and used them in this application.
