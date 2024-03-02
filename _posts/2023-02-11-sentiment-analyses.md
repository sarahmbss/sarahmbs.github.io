---
layout: post
title: Sentiment Analysis of the Brazilian Elections
subtitle: Analysis of sentiments expressed on Twitter in relation to the presidential election candidates of 2022
tags: [data-science, python]
author: Sarah Silva
--- 

## Objective
This project aims to analyze feelings expressed by Twitter users in relation to
candidates for the presidency of the 2022 election, with the aim of verifying whether the performance
of candidates in the presidential election is related to their popularity on social networks
social.

## Data Extraction

To collect data from the Twitter platform, the snscrape tool was used. It is
available as a Python library, which allows you to extract tweets from a specific period with key words.
On this study, the filtered locations were the capitals of the Brazilian states, as well as the capital of
country. The key-words used were the names of the candidates.

```python
import snscrape.modules.twitter as sntwitter
import pandas as pd
import itertools

dat_ini = '2022-10-01'
dat_fim = '2022-10-02'

def search_bolsonaro_sudeste(text = 'bolsonaro', start = dat_ini, end = dat_fim):
    tweets =[]
    for i, tweet in enumerate(sntwitter.TwitterSearchScraper(f'{text} since:{start} until:{end} near:"Belo Horizonte" lang:pt').get_items()):
        tweets.append([tweet.date, tweet.id, tweet.content, tweet.username])
    tweets_bolsonaro_bh = pd.DataFrame(tweets, columns = ['date', 'tweet id', 'content', 'username'])
    
    for i, tweet in enumerate(sntwitter.TwitterSearchScraper(f'{text} since:{start} until:{end} near:"São Paulo" lang:pt').get_items()):
        tweets.append([tweet.date, tweet.id, tweet.content, tweet.username])
    tweets_bolsonaro_sp = pd.DataFrame(tweets, columns = ['date', 'tweet id', 'content', 'username'])
    
    for i, tweet in enumerate(sntwitter.TwitterSearchScraper(f'{text} since:{start} until:{end} near:"Rio de Janeiro" lang:pt').get_items()):
        tweets.append([tweet.date, tweet.id, tweet.content, tweet.username])
    tweets_bolsonaro_rj = pd.DataFrame(tweets, columns = ['date', 'tweet id', 'content', 'username'])
    
    for i, tweet in enumerate(sntwitter.TwitterSearchScraper(f'{text} since:{start} until:{end} near:"Vitória" lang:pt').get_items()):
        tweets.append([tweet.date, tweet.id, tweet.content, tweet.username])
    tweets_bolsonaro_vt = pd.DataFrame(tweets, columns = ['date', 'tweet id', 'content', 'username'])
    
    df_final_bolsonaro_sudeste = pd.concat([tweets_bolsonaro_bh, tweets_bolsonaro_sp, tweets_bolsonaro_rj, tweets_bolsonaro_vt], ignore_index=True)
```

After each search, the tweets found were saved in a file containing the
publication date, id, text, user and region. At the end of the collection, each candidate listed
had an archive of tweets for each region of the country, therefore totaling five.

## Train database

In order to classify each text as positive, negative or neutral,
the Naive Bayes and SVM algorithms were used. As they are supervised models, it was
necessary to build a base of labeled tweets, in order to obtain the training database for the models.

| Sentiment | Number of tweets | 
| :------ |:--- | :--- |
| Positive | 320 |
| Neutral | 297 |
| Negative | 383 |
| Total 1000 |

## Preprocessing

To the data cleaning, the following procedures were carried out:

- Removal of special characters, hyperlinks, punctuation, markings
users and accents: do not add value to the construction of the model;

```python
for i in range(len(Corpus['content'])):
    palavras = Corpus['content'][i].split()
    palavras = [x for x in palavras if "@" not in x]
    palavras = [x for x in palavras if "kk" not in x]
    entry = ' '.join(palavras)
    Corpus['content'][i] = entry

Corpus['content'] = Corpus['content'].apply(lambda x: re.sub('[0-9]|,|\.|/|$|\(|\)|-|\+|:|•', ' ', x))

Corpus['content'] = [re.sub(r'^rt[\s]+', '', entry) for entry in Corpus['content']]
    
Corpus['content'] = [re.sub(r'https?:\/\/.*[\r\n]*http', '', entry) for entry in Corpus['content']]
Corpus['content'] = [re.sub(r'https', '', entry) for entry in Corpus['content']]
Corpus['content'] = [re.sub(r'http', '', entry) for entry in Corpus['content']]

Corpus['content'] = [re.sub(r'\n', '', entry) for entry in Corpus['content']]
```

- Removal of the word used in the search;
- Word correction: incorrectly written words and slang words were corrected;
- Standardization of text in lowercase letters.

```python
Corpus['content'] = [entry.lower() for entry in Corpus['content']]
```

- Tokenization: extraction of minimum text units from tweets;

```python
Corpus['content']= [word_tokenize(entry) for entry in Corpus['content']]
```

- Stopwords removal: use of Python's NLTK library to remove
words that are considered irrelevant. A list was created
with additional words to complete the library;
- Lemmatization: reduction of words to their root, removing the inflections present.

```python
tag_map = defaultdict(lambda : wn.NOUN)
tag_map['J'] = wn.ADJ
tag_map['V'] = wn.VERB
tag_map['R'] = wn.ADV
for index,entry in enumerate(Corpus['content']):
    Final_words = []
    for word, tag in pos_tag(entry):
        if word not in stopwords and word.isalpha():
            if word in correcao:
                word = correcao[word]
            word = unidecode(word)
            word_Final = [token.lemma_ for token in nlp(word)]
            Final_words.extend(word_Final)
    Corpus.loc[index,'text_final'] = str(Final_words)
```