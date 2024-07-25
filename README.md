# Semantic Search in articles using NLP

This repo started as a part of the technical assessment for Cyshield, they sent 3 tasks with 14 days time limit! What a fun time it is to be an AI engineer! Anyway, the rest of the tasks will be published as GH repos too for anyone who's interested in looking into them :)

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

## Dataset

Like any other good NLP project, I started this one by hunting for a suitable dataset, the criteria that I was aspiring for in that dataset was to have queries corresponding tied to some article/s.

The first purpose of finding such a dataset for this project is testing, I believe there are plenty of out-of-the-box models that can do this task very well, but the questions here were: 1- which one is better? 2- Do we need to train/fine-tune our own model? 

There were multiple types of datasets that I found during my hunt and although most of them don't really meet the criteria that I already started with! I still like to mention some them right here for future use

Dataset Name | link | desc | size | language
--- | --- | --- | --- | --- 
arXiv Dataset | https://www.kaggle.com/datasets/Cornell-University/arxiv | That dataset contains lots of interesting features including: article titles, authors, categories, abstracts, full-text PDFs | 1.7 million articles | eng
Wikipedia SQLITE Portable DB | https://www.kaggle.com/datasets/christernyc/wikipedia-sqlite-portable-db-huge-5m-rows/data | Huge dataset organized in a SQLITE DB of wiki article, the dataset contain the tiltle of the article and the labels assoiated with it whic I can belive can be used as quires | over 5 million rows of data | eng
Movies Similarity | https://www.kaggle.com/datasets/devendra45/movies-similarity | I belive the movie title and genere can be used as a query while the wiki plot and the imdb plot can be used as the articles assoitated with those queries<br><br> Isn't there any similar data for Arabic language | 100 samples | eng
