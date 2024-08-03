# Semantic Search in articles using NLP

This repo started as a part of the technical assessment for Cyshield, they sent 3 tasks with 14 days time limit, -What a fun time it is to be a junior AI engineer!- Anyway, the rest of the tasks will be published as GH repos too for anyone who's interested in looking into them :)

That was the task body:

```
Description
▪ Write a pipeline of searching some of words in English articles then extract hot keywords.
▪you can split your data to testing , training, validation . .
▪Do pre-processing on the data just to be able to run it.
▪Do optimization time for article to extract the relevant words.
▪Feel free to use any technique of feature extraction .
✓ U can use Deep Learning Methods and show the diff and which one the best.(optional)
```
I wrote an email to the company for more clarification, and received this response:
```
You can assume that the task is general, so people could search using keywords or using a sentence that represents what he wants to retrieve. you can implement any one of them and we will discuss other possible solutions for the problems. 

You can implement it in Arabic or english. 
```
Given those descriptions, this problem can be described as an **asymmetric semantic search** problem.
## Dataset

Like any other good NLP project, I started this one by hunting for a suitable dataset, the criteria that I was aspiring for in that dataset was to have queries corresponding tied to some article/s.

The first purpose of finding such a dataset for this project is testing, I believe there are plenty of out-of-the-box models that can do this task very well, but the questions here were: 1- which one is better? 2- Do we need to train/fine-tune our own model? 

There were multiple types of datasets that I found during my hunt and although most of them don't really meet the criteria that I already started with! I still like to mention some of them right here for future use

Dataset Name | link | desc | size | language
--- | --- | --- | --- | --- 
arXiv Dataset | https://www.kaggle.com/datasets/Cornell-University/arxiv | That dataset contains lots of interesting features including article titles, authors, categories, abstracts, full-text PDFs | 1.7 million articles | eng
xlsum | https://huggingface.co/datasets/csebuetnlp/xlsum | The dataset contains various articles from the BBC news, each sample include the title which can be used a query and the article which can be used as the answer of that query | 1.35 million samples | 45 languages, ar, eng
Wikipedia SQLITE Portable DB | https://www.kaggle.com/datasets/christernyc/wikipedia-sqlite-portable-db-huge-5m-rows/data | Huge dataset organized in an SQLite DB of a wiki article, the dataset contains the title of the article and the labels associated with it which I believe it can be used as queries | over 5 million rows of data | eng
Wikipedia Movie Plots | https://www.kaggle.com/datasets/jrobischon/wikipedia-movie-plots | I believe the movie title and genre can be used as queries while the wiki plot and the IMDb plot can be used as the articles associated with those queries<br><br> Isn't there any similar data for Arabic language | 34,886 samples | eng
mr-tydi | https://huggingface.co/datasets/castorini/mr-tydi | this dataset is designed for monolingual retrieval, specifically to evaluate ranking with learned dense representations, each sample of this dataset contains a query, a positive message associated with that query and multiple negative passages<br><br> if instead of passages we had full articles, that would be the perfect dataset for this task :) | 48,729 samples | 11 languages in total, ar, eng

I was asked to answer this question by Cyshield: **What was the biggest challenge you faced when carrying out this project?** and I want to say that the biggest challenge of that task was to find a suitable testing dataset, every dataset got its own issue, for example, if I used the movies dataset to test, assuming that: query = movie title and answer = movie plot, I would be ignoring the fact that some movies got a name that's not that relevant to its plot or maybe worse, may the name of the movie is mentioned inside the plot directly, Same thing for the articles datasets, if every article's title = the query and the article = the answer.<br>
Also, I think that each article can be associated with multiple queries, for example, it would make much sense for an article about chatGPT titled "chatGPT and the effect of AI on the modern world" to have multiple queries highlighting different keywords like (machine learning, statistical learning, Artificial intelligence, Natural language processing, transformers) not a single query that's just the title of the article!<br>
I tried to go for free automatic open-source tools like [GenQ](https://www.sbert.net/examples/unsupervised_learning/query_generation/README.html) but the results weren't really satisfying especially in the Arabic dataset.

I chose to go with the **xlsum** dataset -English-, I used the title as the query and the corresponding article as the answer.

## Embedding model
That part is the actual fun part :)<br>
Why?<br>
Cause I already knew what I was going to do right away :) all that I had to do was to determine 1 thing: **The embedding model**, I had a very simple strategy to do that:
1. Use the model to get the embeddings of all the articles and store them -usually in a vector database but for the start I'll go with the regular pandas column-
2. Use the same model to get the embeddings of each query then measure the similarity score between that query embeddings and all the articles' embeddings.
3. Get the article with the highest similarity score with the query and see if it matches the query's original answer.
4. Count how many answers were correct using that model and compare that count with the rest of the models.

The data that I used to test those different models were the first 1000 queries of the validation XLSUM dataset, while the answers that the model had to choose from were the entire no. of samples of validation XLSUM dataset which is around 11,000 samples. That was a tough test for the models and if you tried to lower the number of answers that the model had to choose from, that model would -in most cases- score higher accuracies.

Model | accuracy | size | embedding length | distance metric
--- | --- | --- | --- | ---
https://huggingface.co/sentence-transformers/msmarco-distilbert-base-dot-prod-v3 |  64.0% | 266 MB | 768 | Dot Product
https://huggingface.co/sentence-transformers/msmarco-roberta-base-ance-firstp | 72.39% | 499 MB | 768 | Dot Product
https://huggingface.co/SeyedAli/Multilingual-Text-Semantic-Search-Siamese-BERT-V1 | 71.39% | 90.9 MB | 384 | Dot Product
https://huggingface.co/flax-sentence-embeddings/multi-QA_v1-mpnet-asymmetri | 78.10% | 438 MB for each model | 768 for each | Cosine Similarity
https://huggingface.co/intfloat/e5-base-v2 | 85.3% | 438 MB | 768 | Dot Product
https://huggingface.co/sentence-transformers/distiluse-base-multilingual-cased-v2 | 54.2% | 539 MB | 512 | Cosine Similarity

Given the above results, it's obvious that **e5** is the best model among all of them with a clear margin, but given its size and the fact that **Siamese-BERT** is much lighter and has an accuracy that's only 10% less than e5's, I decided to go with **Siamese-BERT** as my embedding model.

## Storage

I was asked to answer this question by Cyshield: **What do you think you have learned from the project?** And even though I've got to use new and different models in that project, I also got to use a vector database for the very first time! I chose to go with **FAISS**  because there are a ton of resources explaining how to deal with it, and it seemed very easy to deal with.<br>
I stored the test dataset -assuming that in our case this test dataset represents the production data while the validation dataset that I used to help me pick the embedding model is the development data- in the FAISS database.

## Useful references:
- https://subirverma.medium.com/semantic-search-with-s-bert-is-all-you-need-951bc710e160
- https://sbert.net/examples/applications/semantic-search/README.html
- https://www.sbert.net/docs/pretrained-models/msmarco-v3.html
- https://huggingface.co/learn/nlp-course/chapter5/6?fw=tf
- https://myscale.com/blog/mastering-data-science-cosine-similarity-vs-dot-product-insights/


## To do:
1 - Build/find a better testing dataset. 
2- Fine-Tune Siamese-BERT.

