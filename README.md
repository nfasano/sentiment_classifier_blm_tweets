# Sentiment Classifier of BLM Movement Tweets

### Summary of results
Eight binary classifier models were built to predict whether the sentiment of a tweet was positive or negative toward the Black Lives Matter (BLM) movement. It was found that no model performed significantly better than naively assigning all tweets a positive label (which would yield an accuracy of 81%). Logistic regression achieved the best performance on the held out test set with an accuracy of 83% and AUC of 0.79, but it was not substantially better than SVM (accuracy = 83%, auc = 0.75) or KNN (accuracy = 83%, AUC = 0.74). Naive Bayes had a low accuracy of 73%, but the highest precision at 92%, suggesting that an ensemble learning model may improve prediction accuracy. Note that chatGPT had an accuracy of 76% on the test dataset without any preprocessing.

### Introduction
The summer of 2020 saw the rise of support for #BlackLivesMatter after the murder of George Floyd, with in-person protests supporting the #BLM movement being held during the coronavirus epidemic in every major US city. In addition to in-person protests, social media sites — including Twitter — were also awash with opinions about the movement. When Twitter users read tweets about the #BLM movement, and the contemporaneous #BlueLivesMatter movement in support of policing efforts, they might be interested in categorizing tweets based on their attitudes toward the #BLM and #BlueLivesMatter movements. In this work, we use a dataset of 10,000+ labeled tweets pertaining to the #BLM movement to build a sentiment classifier that can predict whether a tweet is positive or negative toward the #BLM. After pre-processing of the dataset and exploratory data analysis, which provided insights on how to prune the vocabulary (see details below), eight classifier models were built and compared against one another (see comparison criteria below), including KNN, Multinomial Naive Bayes, Log. Reg., SVM (Lin), SVM (rbf), Complement Naive Bayes, LDA, and Random Forest. 5-fold cross-validation was used for all hyperparameter selection.

### Dataset Overview 
|   DataSet   | Num. Negative | Num. Positive | Num. Neither | Totals   | 
|-------------|---------------|---------------|--------------|----------|
|  Train.csv  |    1347       |    5891       |   871        |   8109   |  
|  Test.csv   |    337        |    1474       |   218        |   2029   |  
|   Totals    |    1684       |    7365       |   1089       |   10138  |  

#### Train/Test csv column headers
  - created_at: the date of the tweet
  - hashtags: the hashtags used in the tweet
  - text: text of the tweet
  - BLM: label for the tweet (positive, negative, or neither toward the BLM movement)
     - Refer to the 'tweetLabelCriteria.pdf' document for details on how labels were constructed
     
### Data cleaning and feature extraction
#### Preprocessing of training and testing data
The following preprocessing steps were taken for each tweet in the training and testing dataset (see preprocessing_text(tweet) function in classifier.ipynb notebook)
  - Change emojis to words (import emoji)
  - Remove 'RT' from tweet
  - Remove capitalization
  - Remove urls & user mentions from tweet
  - Remove punctuation
  - Remove stopwords (from nltk.corpus import stopwords)
  - Perfom lemmatization on tokens (from nltk.stem import WordNetLemmatizer)
  - Removed Tweets that were labelled as neither

### Exploratory Data Analysis
The following figure presents word clouds for negatively and positively labelled tweets. The size of the word indicates the amount of times the word appeared in the training dataset conditioned on that label. Prior to making these word clouds, the dataset was cleaned of common stop words and the phrase blacklivesmatter, which was the most frequent word in both datasets.

In this figure, we observe that some words are frequently used in both positive and negative tweets such as the word "black". Other words are only found in one of the two word clouds, such as the words "EricGarner" and "ICantBreath" which are commonly found in positively labelled tweets but not in negatively labelled tweets. Similarly, the word "BlueLivesMatter" is commonly found in negatively labelled tweets but not positively labelled tweets.

<picture> <img src="https://github.com/nfasano/sentimentClassifier_blmTweets/blob/main/images/wordCloud.jpg" alt="drawing" width="80%"/> </picture> 

*Figure caption: Word clouds for negatively and positively labelled tweets.*

Building on the observation that some words are more suggesstive of a positive or negative label, we constructed histograms of the tweet labels for each word in the dataset. These individual word histograms are formed by first filtering the training dataset to keep only tweets that contained the word of interest and then calculating the frequency of these tweets that were labelled as positive or negative. We then compared these histograms against the histogram of tweet labels constructed using the entire training dataset. The main idea here is that if a single word's histogram significantly defers from the training dataset's histogram, then it may be a strong predictor of the tweet label. The following figure plots a few of these histograms against the histogram of the entire dataset. Here we see that some words, such as "make" which occurred in 183 tweets, has an identical histogram to the training dataset, so it will not be a strong predicitor of the tweets label. Other words, such as "alllivesmatter" and "justice" which occurred in 611 and 190 tweets, respectively, have histograms which strongly deviate from that of the training datasets histogram, indicating that they are strong predictors of a tweet being negative or positive.


<picture> <img src="https://github.com/nfasano/sentimentClassifier_blmTweets/blob/main/images/histograms.jpg" alt="drawing" width="99%"/> </picture> 

*Figure caption: (a) Histogram of tweet labels for the entire training dataset. Here we see that 81.4% of tweets were labelled as positive. (b-d) Histogram of tweet labels for only the tweets that containes the words (b) "make", (c) "alllivesmatter", and (d) "justice". In all histograms, the horizontal dashed red lines are plotted at 0.814 for the positive column and at 0.186 for the negative column (i.e. the training datasets histogram values). The title contains the word used to construct the histogram and the number of tweets that the word appeared in.*

### Forming the dataset for classification - extracting features and pruning the vocabulary
The training dataset of tweets was transformed using a bag of words representation, where the features (columns in the dataframe) are the unique words in the training dataset and each instance (rows of the dataframe) is one of the tweets from the dataset. Each entry of the dataframe then contains the number of times a particular word occurred in a particular tweet. This yields a large and sparse matrix. The entire vocabulary after running the preprocessing_text function described above was 11,272.

To prune the vocabulary, we used the insights obtained from the exploratory data analysis to remove any words that did not strongly deviate from the training dataset's histogram. This analysis is provided in "Section IV: Hypothesis testing of word count to trim vocabulary" of the classifier.ipynb Python notebook.

Formaly, we performed a hypothesis test comparing the histogram of tweet labels for each word in the vocabulary against the histogram of labels for the entire training dataset. The null hypothesis is that the histogram of the word is identical to the histogram of the training dataset, i.e. the word appears in 81.4% of positive tweets and 18.6% negative tweets. To compare the word's histogram against the training dataset histogram, we used a binomial test to compute a p-value and rejected the null hypothesis if the p-vlaue was less than 0.05. Note that a standard z-test or t-test would not be appropriate here, especially for infrequent words and words with histograms that did not substantially deviate from the null hypothesis.

Finally, there is an option in the code to remove tweets with too few words/characters (e.g. one could drop any tweets with less than 3 words or 10 characters). In what follows, I only removed a tweet from the dataset if it had 0 words after pruning the vocabulary. 

### Model building and comparison of classifiers
Eight models were built on the training dataset (see "Section VI" of classifiers.ipynb for implementation): KNN, Multinomial Naive Bayes, Log. Reg., SVM (Lin), SVM (rbf), Complement Naive Bayes, LDA, and Random Forest. 5-Fold cross validation was used to determine any hyperparameters at train time. These models were then used to predict the labels of 1811 tweets from the held out testing set. The following table summarizes the accuracy, precision, recall, F1-score, confusion matrix values, and area under curve (AUC) of all eight classifiers. 

|   Classifier   | Accuracy | Precision | Recall | F1-score |   TN   |   FP   |   FN   |   TP   |   AUC  |
|----------------|----------|-----------|--------|----------|--------|--------|--------|--------|--------|
|       KNN      |    0.83  |    0.85   |   0.97 |   0.90   |    76  |   261  |    44  |  1430  |  0.74  |
| M. Naive Bayes |    0.82  |    0.84   |   0.97 |   0.90   |    59  |   278  |    47  |  1427  |  0.78  |
|    Log. Reg.   |    0.83  |    0.85   |   0.96 |   0.90   |    90  |   247  |    66  |  1408  |  0.79  |
|    SVM (Lin)   |    0.82  |    0.83   |   0.98 |   0.90   |    45  |   292  |    26  |  1448  |  0.77  |
|    SVM (rbf)   |    0.83  |    0.85   |   0.96 |   0.90   |    84  |   253  |    58  |  1416  |  0.75  |
| C. Naive Bayes |    0.73  |    0.92   |   0.73 |   0.82   |   243  |    94  |   395  |  1079  |  0.78  | 
|       LDA      |    0.82  |    0.86   |   0.94 |   0.90   |   109  |   228  |    94  |  1380  |  0.78  |
|  Random Forest |    0.80  |    0.85   |   0.91 |   0.88   |   103  |   234  |   127  |  1347  |  0.72  |

Here we see that no model performed significantly better than naively assigning all tweets a positive label (which would yield an accuracy of 81%). Logistic regression achieved the best performance on the held out test set with an accuracy of 83% and AUC of 0.79, but it was not substantially better than SVM (accuracy = 83%, auc = 0.75) or KNN (accuracy = 83%, AUC = 0.74). Naive Bayes had a low accuracy of 73%, but the highest precision at 92%, suggesting that an ensemble learning model may improve prediction accuracy.

<picture> <img src="https://github.com/nfasano/sentimentClassifier_blmTweets/blob/main/images/roc_curve.jpg" alt="drawing" width="85%"/> </picture> 

*Figure caption: Precision-recall curve for all eight trained classifiers.*


### How well do large language models work on sentiment analysis?
Note that chatGPT had an accuracy of ~76% when the testing tweets were fed into a chatGPT prompt without any preprocessing. Although somewhat lower than the accuracy obtained in the trained models discussed above, this is still quite impressive given that this approach required no effort on my part - just copy and paste tweets into a chatGPT prompt (see image below for the prompt used). Moreover, chatGPT will provide some level of reasoning for its response which can sometimes be quite coherent (but, of course, always taken with a grain of salt). In summary, the use of large language models for sentiment analysis seems promising and worthy of additional exploration.


<picture> <img src="https://github.com/nfasano/sentimentClassifier_blmTweets/blob/main/images/chatGPT_prompt.png" alt="drawing" width="95%"/> </picture> 

*Figure caption: An example of a chatGPT prompt used to classify a list of tweets as being positive or negative toward the #BLM movement.*


<picture> <img src="https://github.com/nfasano/sentimentClassifier_blmTweets/blob/main/images/chatGPT_prompt_individual_tweets.png" alt="drawing" width="85%"/> </picture> 

*Figure caption: Examples of chatGPT prompts used to classify a tweet as being positive or negative toward the #BLM movement and provide justification for its answer.*

### Future Work
Build ensemble classifiers, tune a pre-trained large language model (e.g. chatGPT) for sentiment classication, bi-gram, and tf-idf instead of bag of words.
