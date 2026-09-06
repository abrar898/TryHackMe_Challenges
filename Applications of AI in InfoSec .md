# Spam Classification – Complete Notes (Simple English)

---

## Table of Contents

1. [Introduction to Spam and Why It Matters](#1-introduction-to-spam-and-why-it-matters)
2. [Naive Bayes for Spam Detection – The Theory](#2-naive-bayes-for-spam-detection--the-theory)
3. [Applying Bayes Theorem to Spam – Step by Step](#3-applying-bayes-theorem-to-spam--step-by-step)
4. [Full Worked Example – Is This Email Spam?](#4-full-worked-example--is-this-email-spam)
5. [The SMS Spam Collection Dataset](#5-the-sms-spam-collection-dataset)
6. [Downloading and Loading the Dataset](#6-downloading-and-loading-the-dataset)
7. [Inspecting the Dataset](#7-inspecting-the-dataset)
8. [Preprocessing the Spam Dataset](#8-preprocessing-the-spam-dataset)
9. [Step 1 – Lowercasing the Text](#9-step-1--lowercasing-the-text)
10. [Step 2 – Removing Punctuation and Numbers](#10-step-2--removing-punctuation-and-numbers)
11. [Step 3 – Tokenization](#11-step-3--tokenization)
12. [Step 4 – Removing Stop Words](#12-step-4--removing-stop-words)
13. [Step 5 – Stemming](#13-step-5--stemming)
14. [Step 6 – Rejoining Tokens into Strings](#14-step-6--rejoining-tokens-into-strings)
15. [Feature Extraction – Bag of Words](#15-feature-extraction--bag-of-words)
16. [CountVectorizer – How It Works](#16-countvectorizer--how-it-works)
17. [Unigrams vs Bigrams – Worked Example](#17-unigrams-vs-bigrams--worked-example)
18. [Training the Spam Classifier](#18-training-the-spam-classifier)
19. [Pipeline – Chaining Steps Together](#19-pipeline--chaining-steps-together)
20. [GridSearchCV – Finding the Best Alpha](#20-gridsearchcv--finding-the-best-alpha)
21. [Evaluating the Model on New Messages](#21-evaluating-the-model-on-new-messages)
22. [Preprocessing New Messages](#22-preprocessing-new-messages)
23. [Making Predictions and Reading Probabilities](#23-making-predictions-and-reading-probabilities)
24. [Saving and Loading the Model with joblib](#24-saving-and-loading-the-model-with-joblib)
25. [Model Upload for Evaluation (Skills Assessment)](#25-model-upload-for-evaluation-skills-assessment)

---

## 1. Introduction to Spam and Why It Matters

**Spam** is unsolicited bulk messaging – emails or SMS messages sent to many people without their permission, usually for advertising, scams, or phishing attacks. Spam has been a problem since the early days of email and messaging, and it is not going away. It clutters inboxes, wastes time, and can be genuinely dangerous when used for phishing (tricking people into giving up passwords or financial details) or for spreading malware links. Effective spam detection is therefore not just a convenience feature – it is a **security necessity**. Spam filters protect users from fraud and cyber attacks. Building an automated spam classifier using machine learning allows email and messaging systems to automatically separate legitimate messages (called **ham**) from unwanted ones (**spam**) without requiring a human to manually read every message.

---

## 2. Naive Bayes for Spam Detection – The Theory

**Bayes' Theorem** gives us a mathematically principled way to decide if a message is spam, based on the words it contains. The core formula is:

```python
P(A | B) = (P(B | A) * P(A)) / P(B)
```

In plain English: "Given that I have already seen evidence B, what is the probability that A is true?"

**For spam detection, we substitute:**

| Symbol | What it means in our context |
|---|---|
| `A` | The hypothesis that this email **is spam** |
| `B` | The **features** of the email (the words it contains) |
| `P(A\|B)` | Probability that the email is spam, **given** its words |
| `P(B\|A)` | Probability of seeing these words **if** the email is spam |
| `P(A)` | Prior probability that any email is spam (before reading it) |
| `P(B)` | Overall probability of seeing these words in any email |

So the full spam version of the formula is:

```python
P(Spam | Features) = (P(Features | Spam) * P(Spam)) / P(Features)
```

The **"Naive"** part means we assume that all words are **independent of each other** given the class. So instead of computing the joint probability of all words together (which is computationally expensive), we simply multiply individual word probabilities:

```python
P(Features | Spam)     = P(word1 | Spam) * P(word2 | Spam) * ... * P(wordN | Spam)
P(Features | Not Spam) = P(word1 | Not Spam) * P(word2 | Not Spam) * ... * P(wordN | Not Spam)
```

**Why "naive"?** Because in reality, words are not independent. "Free" and "prize" tend to appear together in spam. But ignoring this dependency makes the math much simpler, and the algorithm still works surprisingly well in practice.

---

## 3. Applying Bayes Theorem to Spam – Step by Step

Here is the complete decision process for a new incoming email:

```
Step 1: Calculate P(Spam)     – what fraction of all training emails were spam?
Step 2: Calculate P(Not Spam) – what fraction were legitimate?
Step 3: For each word in the new email, look up:
            P(word | Spam)     – how often did this word appear in spam emails?
            P(word | Not Spam) – how often did it appear in ham emails?
Step 4: Multiply all word probabilities together (independently) for each class:
            P(all words | Spam)
            P(all words | Not Spam)
Step 5: Apply Bayes' Theorem to get:
            P(Spam | this email's words)
            P(Not Spam | this email's words)
Step 6: Pick the class with the HIGHER probability → that is the prediction.
```

**Analogy:** Imagine you are a detective who has read thousands of spam and legitimate emails. Over time, you learn that the word "FREE" appears in 80% of spam but only 5% of legitimate emails. The word "meeting" appears in 60% of legitimate emails but only 2% of spam. When a new email arrives, you check each word's "track record" in your memory, combine all the clues, and make your verdict: spam or not spam.

---

## 4. Full Worked Example – Is This Email Spam?

**Given information:**

| Fact | Value |
|---|---|
| P(Spam) | 0.3 (30% of all emails are spam) |
| P(Not Spam) | 0.7 (70% are legitimate) |
| P(F1 \| Spam) | 0.4 (feature F1 appears in 40% of spam) |
| P(F2 \| Spam) | 0.5 (feature F2 appears in 50% of spam) |
| P(F1 \| Not Spam) | 0.2 (F1 appears in 20% of ham) |
| P(F2 \| Not Spam) | 0.3 (F2 appears in 30% of ham) |

**Step 1 – Apply the Naive assumption (multiply independently):**

```python
P(F1, F2 | Spam)     = P(F1|Spam) * P(F2|Spam)     = 0.4 * 0.5 = 0.20
P(F1, F2 | Not Spam) = P(F1|Not Spam) * P(F2|Not Spam) = 0.2 * 0.3 = 0.06
```

**Step 2 – Calculate P(F1, F2) using the law of total probability:**

```python
P(F1, F2) = P(F1,F2|Spam) * P(Spam) + P(F1,F2|Not Spam) * P(Not Spam)
           = (0.20 * 0.3) + (0.06 * 0.7)
           = 0.060 + 0.042
           = 0.102
```

**Step 3 – Apply Bayes' Theorem:**

```python
P(Spam | F1,F2)     = (0.20 * 0.3) / 0.102 = 0.060 / 0.102 ≈ 0.588  (58.8%)
P(Not Spam | F1,F2) = (0.06 * 0.7) / 0.102 = 0.042 / 0.102 ≈ 0.412  (41.2%)
```

**Step 4 – Make the decision:**

```
P(Spam | F1,F2) = 0.588  > P(Not Spam | F1,F2) = 0.412
→ PREDICTION: SPAM ✓
```

Since 58.8% > 41.2%, the model classifies this email as spam.

---

## 5. The SMS Spam Collection Dataset

This dataset was created by researchers at the University of Campinas (Brazil) and Optenet (Spain) and published at the 2011 ACM Symposium on Document Engineering. It contains **5,574 SMS text messages**, each labeled as either:

- **Ham** – a legitimate message (from a known contact, a useful subscription, etc.)
- **Spam** – an unwanted message that offers no benefit and may pose a risk

The dataset was assembled from multiple sources including the Grumbletext website, the NUS SMS Corpus, and Caroline Tag's PhD thesis. It became a standard benchmark dataset for testing spam classification algorithms. The class distribution is heavily imbalanced:

- About **86.6% Ham** (most messages are legitimate)
- About **13.4% Spam** (a minority are spam)

This mirrors real-world distributions – in real life, most messages you receive are legitimate. This imbalance is important to keep in mind when evaluating classifier performance.

---

## 6. Downloading and Loading the Dataset

**Download the dataset from UCI repository:**

```python
import requests
import zipfile
import io

url = "https://archive.ics.uci.edu/static/public/228/sms+spam+collection.zip"
response = requests.get(url)

if response.status_code == 200:
    print("Download successful")
else:
    print("Failed to download the dataset")
```

`requests.get(url)` sends an HTTP GET request (like typing the URL in a browser). `response.status_code == 200` means the server responded successfully. Any other code (like 404 or 500) means something went wrong.

**Extract the zip file directly from memory:**

```python
with zipfile.ZipFile(io.BytesIO(response.content)) as z:
    z.extractall("sms_spam_collection")
    print("Extraction successful")
```

`response.content` is the raw binary data of the downloaded zip file. `io.BytesIO()` wraps that binary data into a file-like object in memory (so we do not need to write it to disk first). `extractall()` unzips all files into the folder `sms_spam_collection`.

**Verify the extraction:**

```python
import os
extracted_files = os.listdir("sms_spam_collection")
print("Extracted files:", extracted_files)
# Expected: ['SMSSpamCollection', 'readme']
```

**Load into a pandas DataFrame:**

```python
import pandas as pd

df = pd.read_csv(
    "sms_spam_collection/SMSSpamCollection",
    sep="\t",           # tab-separated file
    header=None,        # no column header in the file
    names=["label", "message"]   # we name the columns ourselves
)
```

`sep="\t"` tells pandas the columns are separated by a tab character (not a comma). `header=None` means the file has no header row. `names=["label", "message"]` gives the columns meaningful names: "label" (spam or ham) and "message" (the SMS text).

---

## 7. Inspecting the Dataset

**Three useful inspection commands:**

```python
print(df.head())       # Show first 5 rows – quick look at data
print(df.describe())   # Statistical summary of all columns
print(df.info())       # Column types, non-null counts, memory usage
```

**Check for missing values:**

```python
print("Missing values:\n", df.isnull().sum())
```

`isnull()` creates a True/False table marking which cells are empty. `.sum()` counts the True values per column. If any column shows a number > 0, that column has missing data that needs to be handled.

**Check for and remove duplicate entries:**

```python
print("Duplicate entries:", df.duplicated().sum())
df = df.drop_duplicates()
```

`duplicated()` flags rows that are exact copies of previous rows. `drop_duplicates()` removes them. Duplicates can skew training – if the same spam message appears 50 times, the model will overweight its words.

---

## 8. Preprocessing the Spam Dataset

Raw SMS text is messy. Before training, we must **clean and standardize** it so the model can learn meaningful patterns. Without preprocessing:
- "FREE" and "free" would be treated as two different words.
- "running," "runs," and "ran" would be three separate features instead of variations of the same concept.
- Common words like "the," "is," and "and" would dominate the vocabulary without helping classification.

Preprocessing solves all of these problems through a series of steps. We use the `nltk` library (Natural Language Toolkit) for several of these operations.

**Install NLTK resources first:**

```python
import nltk
nltk.download("punkt")       # tokenization rules
nltk.download("punkt_tab")   # additional tokenization tables
nltk.download("stopwords")   # list of common English stop words
```

**View the raw data before any changes:**

```python
print("=== BEFORE ANY PREPROCESSING ===")
print(df.head(5))
```

---

## 9. Step 1 – Lowercasing the Text

```python
df["message"] = df["message"].str.lower()
print("\n=== AFTER LOWERCASING ===")
print(df["message"].head(5))
```

**Why?** "FREE", "Free", and "free" are the same word semantically. If we do not lowercase, the model treats them as three completely different features, tripling the vocabulary size for no benefit. Lowercasing collapses them into one token: "free".

**Before:** `"FREE Money CLICK NOW!"`
**After:** `"free money click now!"`

---

## 10. Step 2 – Removing Punctuation and Numbers

```python
import re

df["message"] = df["message"].apply(lambda x: re.sub(r"[^a-z\s$!]", "", x))
print("\n=== AFTER REMOVING PUNCTUATION & NUMBERS (except $ and !) ===")
print(df["message"].head(5))
```

**The regex pattern `[^a-z\s$!]` explained:**
- `[^ ... ]` = "everything NOT in this list"
- `a-z` = lowercase letters (kept)
- `\s` = whitespace/spaces (kept)
- `$` = dollar sign (kept – spam indicator!)
- `!` = exclamation mark (kept – spam indicator!)
- Everything else (numbers, periods, commas, etc.) → replaced with empty string (removed)

**Why keep `$` and `!`?** These characters carry meaningful spam signals. `$1000 FREE PRIZE!!!` – the dollar sign and exclamation marks tell the model something important. Removing them would throw away useful information.

**Before:** `"Congratulations! You've won $1000! Call 0800-123456 now."`
**After:** `"congratulations! youve won $ call now"`

---

## 11. Step 3 – Tokenization

```python
from nltk.tokenize import word_tokenize

df["message"] = df["message"].apply(word_tokenize)
print("\n=== AFTER TOKENIZATION ===")
print(df["message"].head(5))
```

**What is tokenization?** It splits a string of text into a list of individual words (tokens). A simple `.split()` would split only on spaces. `word_tokenize` from NLTK is smarter – it handles edge cases like contractions and punctuation more reliably.

**Before tokenization:** `"free money click now!"`
**After tokenization:** `["free", "money", "click", "now", "!"]`

Each message is now a Python list of words instead of one long string. This is necessary for the next steps (stop word removal and stemming), which operate on individual words.

---

## 12. Step 4 – Removing Stop Words

```python
from nltk.corpus import stopwords

stop_words = set(stopwords.words("english"))
df["message"] = df["message"].apply(lambda x: [word for word in x if word not in stop_words])
print("\n=== AFTER REMOVING STOP WORDS ===")
print(df["message"].head(5))
```

**What are stop words?** Very common English words that appear in almost every message regardless of whether it is spam or ham: "the," "is," "a," "and," "or," "but," "in," "for," etc. They appear so often in both classes that they provide no useful signal for discrimination.

**Why remove them?** They:
- Add noise to the feature space.
- Inflate vocabulary size with useless terms.
- Make the model less focused on the actually informative words.

**Before:** `["free", "money", "is", "waiting", "for", "you", "now"]`
**After:** `["free", "money", "waiting", "now"]`

The words "is," "for," and "you" were stop words and got removed. The remaining words are more informative.

**Why `set()`?** Using a set (`stop_words = set(...)`) instead of a list makes the `in` check run in O(1) time (instant lookup) instead of O(n) time (scanning the whole list). Much faster for large datasets.

---

## 13. Step 5 – Stemming

```python
from nltk.stem import PorterStemmer

stemmer = PorterStemmer()
df["message"] = df["message"].apply(lambda x: [stemmer.stem(word) for word in x])
print("\n=== AFTER STEMMING ===")
print(df["message"].head(5))
```

**What is stemming?** It reduces words to their base or root form by stripping suffixes.

**Examples:**
| Original Word | After Stemming |
|---|---|
| running | run |
| runner | runner |
| congratulations | congratul |
| claiming | claim |
| waiting | wait |
| offers | offer |

**Why?** "Running," "runs," and "runner" all refer to the same concept. Without stemming, they are three separate features. With stemming, they collapse into one feature ("run"), reducing vocabulary size and helping the model generalize.

**Trade-off:** Stemming is aggressive and sometimes produces non-words ("congratulations" → "congratul"). A gentler alternative is **lemmatization**, which reduces to the dictionary root form. But stemming is faster and works well enough for spam detection.

---

## 14. Step 6 – Rejoining Tokens into Strings

```python
df["message"] = df["message"].apply(lambda x: " ".join(x))
print("\n=== AFTER JOINING TOKENS BACK INTO STRINGS ===")
print(df["message"].head(5))
```

**Why join back?** Most ML algorithms and text vectorizers (like `CountVectorizer`) expect text as a **single string**, not as a list of words. We split into tokens to apply preprocessing (stop words, stemming), and now we put them back together.

**Before:** `["free", "monei", "wait", "now"]`
**After:** `"free monei wait now"`

At this point, each message is a clean, preprocessed string ready for feature extraction.

**Summary of all preprocessing steps:**

```
Original:  "FREE Money is waiting for you NOW! Claim $1000 today."
Lowercase: "free money is waiting for you now! claim $1000 today."
Remove punct/num: "free money is waiting for you now! claim $ today"
Tokenize:  ["free", "money", "is", "waiting", "for", "you", "now!", "claim", "$", "today"]
Remove stop words: ["free", "money", "waiting", "now!", "claim", "$", "today"]
Stem:      ["free", "monei", "wait", "now!", "claim", "$", "todai"]
Join:      "free monei wait now! claim $ todai"
```

---

## 15. Feature Extraction – Bag of Words

Machine learning models cannot process raw text – they only understand **numbers**. Feature extraction converts each preprocessed message into a **numerical vector** (a list of numbers). The most common approach for text is the **Bag-of-Words** model.

**The Bag-of-Words idea:**
1. Build a vocabulary of all unique words seen in training data.
2. Represent each message as a vector where each position corresponds to one word in the vocabulary.
3. The value at each position = how many times that word appears in this message.

**Example:**

Vocabulary: `["free", "prize", "meeting", "tomorrow", "click"]`

| Message | free | prize | meeting | tomorrow | click |
|---|---|---|---|---|---|
| "free prize click now" | 1 | 1 | 0 | 0 | 1 |
| "meeting tomorrow ok" | 0 | 0 | 1 | 1 | 0 |

Now each message is a row of numbers that the classifier can work with.

**Important limitation:** Bag-of-Words does NOT preserve word order. "Dog bites man" and "Man bites dog" produce the same vector. For spam detection this is usually acceptable because the presence of spam-indicating words matters more than their order.

---

## 16. CountVectorizer – How It Works

`CountVectorizer` from scikit-learn implements the Bag-of-Words model efficiently.

```python
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(
    min_df=1,           # word must appear in at least 1 document
    max_df=0.9,         # exclude words appearing in >90% of documents
    ngram_range=(1, 2)  # include both unigrams and bigrams
)

X = vectorizer.fit_transform(df["message"])
y = df["label"].apply(lambda x: 1 if x == "spam" else 0)
```

**Three key parameters explained:**

| Parameter | What it does | Why it helps |
|---|---|---|
| `min_df=1` | Keep words that appear in at least 1 document | Removes truly unique typos/noise (set higher like 3 in practice) |
| `max_df=0.9` | Remove words appearing in >90% of all documents | Removes ultra-common words that don't help distinguish spam from ham |
| `ngram_range=(1,2)` | Include individual words (unigrams) AND word pairs (bigrams) | Captures context: "free prize" is more informative than "free" alone |

**Three stages CountVectorizer follows internally:**

```
Stage 1 – Tokenization:
  Splits each message into tokens based on ngram_range.
  "free prize" → ["free", "prize", "free prize"]  (unigrams + bigram)

Stage 2 – Build Vocabulary:
  Collects all unique tokens across all messages.
  Applies min_df and max_df filters to remove unhelpful words.

Stage 3 – Vectorization:
  For each message, counts how many times each vocabulary word appears.
  Returns a sparse matrix (most values are 0, so only non-zero values stored).
```

**Output:** `X` is a sparse matrix of shape `(num_messages, vocab_size)`. `y` is a binary vector: 1 = spam, 0 = ham.

---

## 17. Unigrams vs Bigrams – Worked Example

**Unigrams (ngram_range=(1,1)):** Individual words only.

Example 5 documents:
```
1. "The free prize is waiting for you"
2. "The spam message offers a free prize now"
3. "The spam filter might detect this"
4. "The important news says you won a free trip"
5. "The message truly is important"
```

After `max_df=0.9` removes "The" (appears in 100% of documents), the unigram feature matrix looks like:

| Doc | free | prize | is | waiting | spam | message | ... |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 1 | 1 | 1 | 0 | 0 | ... |
| 2 | 1 | 1 | 0 | 0 | 1 | 1 | ... |
| 3 | 0 | 0 | 0 | 0 | 1 | 0 | ... |

**Bigrams (ngram_range=(1,2)):** Individual words PLUS consecutive word pairs.

The bigram `"free prize"` appears in Documents 1 and 2. Adding this bigram as a feature helps because:
- "free" alone might appear in legitimate messages ("free time," "free to talk").
- "free prize" is a much stronger spam signal.

Bigrams capture **local context** – they let the model see which words appear next to each other, not just which words appear.

---

## 18. Training the Spam Classifier

We use **Multinomial Naive Bayes** (`MultinomialNB`) – the version of Naive Bayes designed specifically for word count features (discrete non-negative numbers). It is fast, simple, memory-efficient, and works very well for text classification.

**Multinomial Naive Bayes internally:**
- For each class (spam/ham), calculates the probability of each word given that class: `P(word | spam)` and `P(word | ham)`.
- When a new message arrives, multiplies all word probabilities together for each class and picks the winner.
- Uses log-probabilities internally to prevent numerical underflow (multiplying thousands of small numbers together would reach zero quickly).

**The `alpha` smoothing parameter (Laplace Smoothing):**
- When a word appears in training ham messages but never in spam, its `P(word | spam) = 0`. This is a problem – a zero probability would make the entire product zero, regardless of all other evidence.
- `alpha` adds a small constant to every word count, ensuring no probability is ever exactly zero.
- `alpha=1.0` is the standard Laplace smoothing. Smaller values (0.1, 0.01) give more weight to the actual training data.

---

## 19. Pipeline – Chaining Steps Together

A `Pipeline` combines multiple steps (vectorization + classification) into a single object. This ensures the exact same transformation is always applied consistently.

```python
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ("vectorizer", vectorizer),       # Step 1: convert text to numbers
    ("classifier", MultinomialNB())   # Step 2: classify using Naive Bayes
])
```

**Why use a Pipeline?**

| Without Pipeline | With Pipeline |
|---|---|
| Must manually apply vectorizer then classifier | Single `pipeline.fit()` and `pipeline.predict()` call |
| Easy to forget to apply vectorizer to new data | Automatically handles all steps consistently |
| Hard to tune both steps together | GridSearchCV can tune parameters from any step |
| Risk of data leakage in cross-validation | Pipeline prevents leakage by fitting vectorizer only on training fold |

**Data leakage problem (what Pipeline prevents):** If you fit the vectorizer on ALL data before cross-validation, the vocabulary includes words from test messages. This "leaks" test information into training, giving artificially good results. With Pipeline, the vectorizer is fit only on the training fold each time, giving honest performance estimates.

---

## 20. GridSearchCV – Finding the Best Alpha

`GridSearchCV` automatically tests different hyperparameter values and finds the best one using cross-validation.

```python
param_grid = {
    "classifier__alpha": [0.01, 0.1, 0.15, 0.2, 0.25, 0.5, 0.75, 1.0]
}

grid_search = GridSearchCV(
    pipeline,
    param_grid,
    cv=5,          # 5-fold cross-validation
    scoring="f1"   # optimize for F1 score
)

grid_search.fit(df["message"], y)

best_model = grid_search.best_estimator_
print("Best model parameters:", grid_search.best_params_)
```

**How `GridSearchCV` works:**

```
For each alpha value in [0.01, 0.1, 0.15, 0.2, 0.25, 0.5, 0.75, 1.0]:
    For each fold in 5-fold cross-validation:
        Train on 80% of data (with this alpha)
        Evaluate on remaining 20%
        Record F1 score
    Average F1 score across 5 folds → this alpha's performance score
Pick the alpha with the highest average F1 score → best_model
```

**Parameter name syntax `"classifier__alpha"`:** The double underscore `__` tells GridSearchCV to look inside the pipeline step named `"classifier"` and set its `alpha` parameter. This is how you pass parameters to specific steps inside a Pipeline.

**Why F1 score?** With imbalanced data (86% ham, 14% spam), accuracy is misleading. A classifier that always predicts "ham" would get 86% accuracy but detect zero spam. F1 score balances Precision (when we say spam, are we right?) and Recall (do we catch most spam?), giving a more meaningful measure for imbalanced classification.

```python
F1 = 2 * (Precision * Recall) / (Precision + Recall)
```

---

## 21. Evaluating the Model on New Messages

After training, we test the model on brand new messages it has never seen.

```python
new_messages = [
    "Congratulations! You've won a $1000 Walmart gift card. Go to http://bit.ly/1234 to claim now.",
    "Hey, are we still meeting up for lunch today?",
    "Urgent! Your account has been compromised. Verify your details here: www.fakebank.com/verify",
    "Reminder: Your appointment is scheduled for tomorrow at 10am.",
    "FREE entry in a weekly competition to win an iPad. Just text WIN to 80085 now!",
]
```

**Expected model behavior:**

| Message | Expected Prediction | Why |
|---|---|---|
| "$1000 gift card, claim now" | Spam | Prize/gift language, URL |
| "meeting for lunch today?" | Ham | Casual conversational language |
| "account compromised, verify" | Spam | Urgency + phishing URL |
| "appointment tomorrow 10am" | Ham | Routine reminder language |
| "FREE competition to win iPad" | Spam | "FREE" + competition + prize |

---

## 22. Preprocessing New Messages

**Critical rule:** New messages MUST be preprocessed in EXACTLY the same way as training data. If you lowercase during training, you must lowercase during prediction. If you stem during training, you must stem during prediction.

```python
import re
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer

stop_words = set(stopwords.words("english"))
stemmer = PorterStemmer()

def preprocess_message(message):
    message = message.lower()                                          # lowercase
    message = re.sub(r"[^a-z\s$!]", "", message)                     # remove punctuation/numbers
    tokens = word_tokenize(message)                                    # tokenize
    tokens = [word for word in tokens if word not in stop_words]      # remove stop words
    tokens = [stemmer.stem(word) for word in tokens]                  # stem
    return " ".join(tokens)                                            # rejoin

processed_messages = [preprocess_message(msg) for msg in new_messages]
```

**Vectorize using the SAME vocabulary as training:**

```python
X_new = best_model.named_steps["vectorizer"].transform(processed_messages)
```

Notice we use `.transform()` NOT `.fit_transform()`. We are applying the existing training vocabulary to new messages. Any word in a new message that was not in the training vocabulary is simply ignored.

---

## 23. Making Predictions and Reading Probabilities

```python
predictions = best_model.named_steps["classifier"].predict(X_new)
prediction_probabilities = best_model.named_steps["classifier"].predict_proba(X_new)

for i, msg in enumerate(new_messages):
    prediction = "Spam" if predictions[i] == 1 else "Not-Spam"
    spam_probability = prediction_probabilities[i][1]   # index 1 = spam probability
    ham_probability  = prediction_probabilities[i][0]   # index 0 = ham probability

    print(f"Message: {msg}")
    print(f"Prediction: {prediction}")
    print(f"Spam Probability: {spam_probability:.2f}")
    print(f"Not-Spam Probability: {ham_probability:.2f}")
    print("-" * 50)
```

**Sample output:**

```
Message: Congratulations! You've won a $1000 Walmart gift card...
Prediction: Spam
Spam Probability: 1.00
Not-Spam Probability: 0.00
--------------------------------------------------
Message: Hey, are we still meeting up for lunch today?
Prediction: Not-Spam
Spam Probability: 0.00
Not-Spam Probability: 1.00
--------------------------------------------------
Message: Urgent! Your account has been compromised...
Prediction: Spam
Spam Probability: 0.94
Not-Spam Probability: 0.06
--------------------------------------------------
Message: Reminder: Your appointment is scheduled for tomorrow at 10am.
Prediction: Not-Spam
Spam Probability: 0.00
Not-Spam Probability: 1.00
--------------------------------------------------
Message: FREE entry in a weekly competition to win an iPad...
Prediction: Spam
Spam Probability: 1.00
Not-Spam Probability: 0.00
--------------------------------------------------
```

**How to read the probabilities:**
- `predict_proba()` returns a 2D array: `[[p_ham, p_spam], [p_ham, p_spam], ...]`
- Index `[i][0]` = ham probability for message i
- Index `[i][1]` = spam probability for message i
- The model picks whichever is higher. Default threshold = 0.5.

---

## 24. Saving and Loading the Model with joblib

Once we have a trained, tested model, we want to save it so we do not have to retrain it every time. `joblib` is the recommended way to serialize (save) scikit-learn models.

**Save the model:**

```python
import joblib

model_filename = 'spam_detection_model.joblib'
joblib.dump(best_model, model_filename)
print(f"Model saved to {model_filename}")
```

`joblib.dump()` converts the entire model object (including the vectorizer's vocabulary and the classifier's learned probabilities) into a binary file on disk.

**Load the model later:**

```python
loaded_model = joblib.load(model_filename)

new_data_processed = [preprocess_message(msg) for msg in new_messages]
predictions = loaded_model.predict(new_data_processed)
```

**Why joblib instead of pickle?**
- joblib is optimized for objects containing large NumPy arrays (which scikit-learn models do).
- It uses compression and memory mapping to handle large models efficiently.
- It is the official scikit-learn recommendation for model serialization.

**What gets saved in the `.joblib` file:**
- The CountVectorizer's entire vocabulary (all learned words and their indices)
- The MultinomialNB classifier's learned log-probabilities for every word in every class
- The best alpha value found by GridSearchCV
- The Pipeline structure connecting all steps

When you load the model, it is instantly ready to predict – no retraining needed.

---

## 25. Model Upload for Evaluation (Skills Assessment)

After saving your model, upload it to the evaluation portal on the Playground VM to receive the flag.

**Method 1 – Python script (from Jupyter):**

```python
import requests
import json

url = "http://localhost:8000/api/upload"
model_file_path = "spam_detection_model.joblib"

with open(model_file_path, "rb") as model_file:
    files = {"model": model_file}
    response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))
```

`open(..., "rb")` opens the file in **binary read** mode (needed for non-text files like .joblib). The `files` dict tells requests to send it as a file upload (multipart form data). If your model meets the performance criteria, the server returns a JSON response containing the flag.

**Method 2 – Browser upload:**
Navigate to `http://<VM-IP>:8000/` in your browser and use the web interface to upload your `.joblib` file manually.

**Performance criteria the server checks:**
The server evaluates your model on a held-out test set. Your model needs to achieve strong performance – specifically high F1 score on spam detection. The GridSearchCV with F1 scoring during training should optimize for exactly this metric.

---

## Summary – Full Pipeline at a Glance

```
RAW SMS DATA
    ↓
1. Download + Load dataset (requests, zipfile, pandas)
    ↓
2. Inspect: head(), describe(), info(), isnull(), duplicated()
    ↓
3. Preprocess:
   a. Lowercase
   b. Remove punctuation/numbers (keep $ and !)
   c. Tokenize (word_tokenize)
   d. Remove stop words
   e. Stem (PorterStemmer)
   f. Rejoin tokens into string
    ↓
4. Feature Extraction (CountVectorizer):
   - Build vocabulary from training data
   - Represent each message as word count vector
   - Include unigrams + bigrams
   - Filter rare and ultra-common words
    ↓
5. Training (Pipeline + GridSearchCV):
   - Try multiple alpha values for MultinomialNB
   - 5-fold cross-validation with F1 scoring
   - Select best alpha → best_model
    ↓
6. Evaluation on new messages:
   - Preprocess with same function
   - Vectorize with SAME vocabulary (transform, not fit_transform)
   - predict() → label  |  predict_proba() → confidence scores
    ↓
7. Save model (joblib.dump)
    ↓
8. Upload to evaluation portal → receive flag
```

### Key Concepts Quick Reference

| Concept | Simple Explanation |
|---|---|
| Bayes' Theorem | Update probability of A given new evidence B |
| Naive Bayes | Bayes + assume all features are independent |
| Multinomial NB | Naive Bayes variant for word count features |
| Alpha smoothing | Prevents zero-probability by adding small constant to all counts |
| Bag of Words | Represent text as word frequency counts (ignores word order) |
| Unigram | Single word feature (e.g., "free") |
| Bigram | Two consecutive word feature (e.g., "free prize") |
| CountVectorizer | Converts text to word count matrix |
| min_df | Minimum document frequency for a word to be included |
| max_df | Maximum document frequency – removes too-common words |
| Pipeline | Chain of preprocessing + model steps executed consistently |
| GridSearchCV | Exhaustive hyperparameter search with cross-validation |
| F1 Score | Balanced metric = harmonic mean of Precision and Recall |
| joblib | Library for saving/loading scikit-learn models efficiently |
| Ham | Legitimate (non-spam) message |
| Spam | Unsolicited, unwanted message |

---

*Notes compiled from: Applications of AI in InfoSec – Spam Classification module.*# Network Anomaly Detection – Complete Notes (Simple English)

---

## Network Anomaly Detection

Anomaly detection is the task of finding data points that look very different from everything else. In cybersecurity, these unusual data points often mean something bad is happening – a network intrusion, a denial-of-service attack, or an attempt to gain unauthorized access. The normal approach to finding these threats is to manually write rules (e.g., "flag any connection with more than 1000 packets per second"), but attackers learn to evade rules quickly. A machine learning approach trains the model on what **normal** looks like, so it can flag anything that deviates – even if it has never seen that exact attack before. Random Forests are a strong choice for this problem because they handle high-dimensional data well, are robust to noise, and do not require the data to follow any specific statistical distribution. They also naturally provide feature importance rankings, helping security analysts understand which network characteristics are the biggest red flags.

---

## Random Forests

A **Random Forest** is an **ensemble** algorithm – instead of training one decision tree, it trains **many** trees and combines their answers. Each individual tree is like a junior analyst who makes a judgment call. The forest is the senior team that takes a vote. The majority vote wins for classification; the average wins for regression. This approach almost always outperforms a single decision tree because individual trees make different mistakes, and those mistakes cancel out when you average many of them together.

Three ideas make a Random Forest work:

**Bootstrapping:** Each tree is trained on a different random sample of the training data (sampled with replacement – meaning the same row can appear multiple times in one sample). This is called **bagging** (Bootstrap AGGregating). Because each tree sees slightly different data, each tree learns slightly different patterns.

**Random feature subsets:** At every split in every tree, the algorithm randomly picks a subset of features to consider. It does not always use all features at once. This forces the trees to be different from each other, reducing the correlation between them.

**Voting:** Once all trees are trained, a new data point is sent through every tree. Each tree produces a prediction. For classification: whichever class gets the most votes wins. For regression: the predictions are averaged.

**Analogy:** Imagine you are trying to decide if a piece of network traffic is an attack. Instead of asking one expert, you ask 100 different security analysts who each only see a random subset of the logs. Even if individual analysts make mistakes, the group vote is almost always correct.

---

## Random Forests for Anomaly Detection

When a Random Forest is used for anomaly detection, it is trained on data that represents **normal behaviour**. Then, when a new data point arrives, the model tries to classify it. If the model is very uncertain (votes are split between classes) or the point falls into a class it almost never sees, that point is flagged as a potential anomaly. In the NSL-KDD context, the model is trained on labeled network traffic and learns to distinguish five categories: normal traffic, DoS attacks, Probe attacks, Privilege Escalation attacks, and Access attacks.

This makes the approach more powerful than binary detection alone (normal vs. not-normal) because it tells you not just that something is wrong, but **what kind of attack it might be**. A security analyst responding to an alert needs to know whether they are dealing with a denial-of-service flood or a quiet network probe – those require very different responses.

---

## NSL-KDD Dataset

The **NSL-KDD dataset** is a refined version of the original KDD Cup 1999 dataset, which was the first major network intrusion detection benchmark. The original KDD dataset had serious problems: it contained many duplicate records, and some classes were massively over-represented, making it easy for a model to get high accuracy by just memorizing common patterns. NSL-KDD fixes these issues by removing redundant entries and rebalancing the class distribution.

It was created by researchers at the University of New Brunswick (Tiago Almeida, Akebo Yamakami, and José María Gómez Hidalgo) and published as a standard reference benchmark for intrusion detection research. The dataset contains labeled instances of both normal network traffic and four categories of malicious activity. Each row represents one network connection with 41 features describing its characteristics. Using this dataset, practitioners can build models that detect not just whether traffic is suspicious, but what type of attack it resembles. The modified version used in this module is stored in `KDD+.txt`.

---

## Downloading the Dataset

```python
import requests, zipfile, io

# URL for the NSL-KDD dataset
url = "https://academy.hackthebox.com/storage/modules/292/KDD_dataset.zip"

# Download the zip file and extract it
response = requests.get(url)
z = zipfile.ZipFile(io.BytesIO(response.content))
z.extractall('.')   # extracts files into the current directory
```

`requests.get(url)` sends an HTTP GET request and downloads the file. `response.content` is the raw binary bytes of the downloaded zip file. `io.BytesIO(response.content)` wraps those bytes into a file-like object in memory so we never need to write the zip to disk first. `zipfile.ZipFile(...)` opens the zip from memory, and `extractall('.')` extracts all its files into the current working directory. After this runs, you will have `KDD+.txt` ready to load.

---

## Loading the Dataset

### Importing Libraries

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
                              f1_score, confusion_matrix, classification_report)
import seaborn as sns
import matplotlib.pyplot as plt
```

Each library has a specific role. `numpy` handles fast numerical operations on arrays. `pandas` loads, inspects, and transforms tabular data. `RandomForestClassifier` is scikit-learn's implementation of the Random Forest algorithm. `train_test_split` divides data into training and testing portions. The metrics functions (`accuracy_score`, `precision_score`, etc.) measure how well the model performs. `seaborn` and `matplotlib` create charts and visualizations, including the confusion matrix heatmap.

### Defining Column Names

The NSL-KDD file has no header row – it is just raw comma-separated values. We must tell pandas what each column means by supplying a list of column names:

```python
file_path = r'KDD+.txt'

columns = [
    'duration', 'protocol_type', 'service', 'flag', 'src_bytes', 'dst_bytes',
    'land', 'wrong_fragment', 'urgent', 'hot', 'num_failed_logins', 'logged_in',
    'num_compromised', 'root_shell', 'su_attempted', 'num_root', 'num_file_creations',
    'num_shells', 'num_access_files', 'num_outbound_cmds', 'is_host_login', 'is_guest_login',
    'count', 'srv_count', 'serror_rate', 'srv_serror_rate', 'rerror_rate', 'srv_rerror_rate',
    'same_srv_rate', 'diff_srv_rate', 'srv_diff_host_rate', 'dst_host_count', 'dst_host_srv_count',
    'dst_host_same_srv_rate', 'dst_host_diff_srv_rate', 'dst_host_same_src_port_rate',
    'dst_host_srv_diff_host_rate', 'dst_host_serror_rate', 'dst_host_srv_serror_rate',
    'dst_host_rerror_rate', 'dst_host_srv_rerror_rate', 'attack', 'level'
]
```

These 43 columns cover everything about a network connection. The first group (`duration`, `src_bytes`, `dst_bytes`) describes basic volume and timing. The middle group (rates ending in `_rate`) describes statistical patterns computed over many connections. The last two columns (`attack`, `level`) are the ground-truth labels – what type of traffic this connection actually was.

### Reading into a DataFrame

```python
df = pd.read_csv(file_path, names=columns)
print(df.head())
```

`pd.read_csv()` with `names=columns` reads the file and immediately assigns our column names. `df.head()` shows the first 5 rows as a sanity check. A typical row looks like:

```
0, tcp, ftp_data, SF, 491, 0, 0, 0, ... , normal, 20
0, tcp, private, S0,  0,  0, 0, 0, ... , neptune, 19
```

The second row is a neptune attack (a type of DoS), the first is normal traffic.

---

## Preprocessing and Splitting the Dataset

### Creating a Binary Classification Target

The simplest way to frame the problem is binary: is this traffic normal or not? We add a new column `attack_flag`:

```python
df['attack_flag'] = df['attack'].apply(lambda a: 0 if a == 'normal' else 1)
```

`lambda a: 0 if a == 'normal' else 1` is a tiny inline function. For every row, if the `attack` column says `'normal'`, assign 0. Otherwise assign 1. This gives us a clean binary label for every row. The `apply()` method runs this function on every row in the column efficiently.

This binary target is a good starting point, but it loses information. Knowing that an attack happened is useful; knowing that it is a DoS attack versus a privilege escalation attempt is far more useful for an incident responder.

### Creating the Multi-Class Classification Target

We define five categories (0 through 4) and map every attack type into one of them:

```python
dos_attacks       = ['apache2','back','land','neptune','mailbomb','pod',
                     'processtable','smurf','teardrop','udpstorm','worm']
probe_attacks     = ['ipsweep','mscan','nmap','portsweep','saint','satan']
privilege_attacks = ['buffer_overflow','loadmdoule','perl','ps',
                     'rootkit','sqlattack','xterm']
access_attacks    = ['ftp_write','guess_passwd','http_tunnel','imap',
                     'multihop','named','phf','sendmail','snmpgetattack',
                     'snmpguess','spy','warezclient','warezmaster','xclock','xsnoop']

def map_attack(attack):
    if attack in dos_attacks:           return 1   # DoS
    elif attack in probe_attacks:       return 2   # Probe
    elif attack in privilege_attacks:   return 3   # Privilege Escalation
    elif attack in access_attacks:      return 4   # Access
    else:                               return 0   # Normal

df['attack_map'] = df['attack'].apply(map_attack)
```

The five classes and what they mean:

| Class | Label | Description |
|---|---|---|
| 0 | Normal | Legitimate network traffic |
| 1 | DoS | Denial-of-Service – overwhelm a target with requests until it crashes |
| 2 | Probe | Reconnaissance – scan networks to find vulnerabilities |
| 3 | Privilege | Try to gain unauthorized administrator-level control |
| 4 | Access | Try to break into accounts or access data without permission |

**Why lists instead of a dictionary?** Using Python `in` with a list is readable and clear. For larger lookups a dictionary would be faster, but for 40 attack types a list works fine.

### Encoding Categorical Variables

Machine learning algorithms need numbers. The NSL-KDD dataset has two categorical text columns that must be converted: `protocol_type` (values like `tcp`, `udp`, `icmp`) and `service` (values like `http`, `ftp`, `smtp`).

```python
features_to_encode = ['protocol_type', 'service']
encoded = pd.get_dummies(df[features_to_encode])
```

`pd.get_dummies()` performs **one-hot encoding**. For a column with values `tcp`, `udp`, `icmp`, it creates three new binary columns: `protocol_type_tcp`, `protocol_type_udp`, `protocol_type_icmp`. Each row gets a 1 in the column matching its actual value and 0 in all others.

**Why not just use numbers (0, 1, 2)?** If we assigned `tcp=0`, `udp=1`, `icmp=2`, the model would think `icmp` is "twice as much" as `udp`, which is nonsense. One-hot encoding avoids this false ordering by treating every category as completely separate.

**Why these two columns and not `flag`?** The `flag` column is also categorical but was excluded here. In practice, you would encode it too – but the course example focuses on these two. The same `get_dummies()` call would handle it.

### Selecting Numeric Features

The dataset has many numeric columns describing statistical properties of network connections. We manually select the most useful ones:

```python
numeric_features = [
    'duration', 'src_bytes', 'dst_bytes', 'wrong_fragment', 'urgent', 'hot',
    'num_failed_logins', 'num_compromised', 'root_shell', 'su_attempted',
    'num_root', 'num_file_creations', 'num_shells', 'num_access_files',
    'num_outbound_cmds', 'count', 'srv_count', 'serror_rate',
    'srv_serror_rate', 'rerror_rate', 'srv_rerror_rate', 'same_srv_rate',
    'diff_srv_rate', 'srv_diff_host_rate', 'dst_host_count', 'dst_host_srv_count',
    'dst_host_same_srv_rate', 'dst_host_diff_srv_rate',
    'dst_host_same_src_port_rate', 'dst_host_srv_diff_host_rate',
    'dst_host_serror_rate', 'dst_host_srv_serror_rate',
    'dst_host_rerror_rate', 'dst_host_srv_rerror_rate'
]
```

What these measure in plain English:

| Feature | What it captures |
|---|---|
| `duration` | How long the connection lasted |
| `src_bytes` / `dst_bytes` | How many bytes went in each direction |
| `wrong_fragment` | Malformed network packets (often a DoS indicator) |
| `num_failed_logins` | Failed login attempts (brute-force indicator) |
| `root_shell` | Whether a root shell was obtained (privilege escalation) |
| `serror_rate` | Fraction of connections with SYN errors (SYN flood indicator) |
| `dst_host_count` | Number of connections to the same destination host |

Columns excluded from this list (like `logged_in`, `land`, `is_host_login`) are either binary flags that some implementations handle differently or had less discriminative power for this model.

### Preparing the Final Feature Matrix

```python
train_set = encoded.join(df[numeric_features])
multi_y   = df['attack_map']
```

`encoded.join(...)` horizontally concatenates the one-hot columns and the numeric columns side by side into one big DataFrame. Each row now has all the features the model needs: one-hot protocol/service indicators plus 34 numeric measurements. `multi_y` stores the target labels (0–4) aligned with the rows in `train_set`.

**Result:** `train_set` has shape approximately (125,973 rows × 107 columns). Every value is numeric. This is the input to the Random Forest.

### Splitting the Dataset

We split data into three separate sets so the model is trained, tuned, and evaluated on completely different data:

```python
# Step 1: Split off the final test set (20% of all data)
train_X, test_X, train_y, test_y = train_test_split(
    train_set, multi_y, test_size=0.2, random_state=1337
)

# Step 2: Split the remaining training data into train + validation (30% of training = validation)
multi_train_X, multi_val_X, multi_train_y, multi_val_y = train_test_split(
    train_X, train_y, test_size=0.3, random_state=1337
)
```

After these two splits:

| Set | Used for | Approximate size |
|---|---|---|
| `multi_train_X` / `multi_train_y` | Fitting the model | ~56% of all data |
| `multi_val_X` / `multi_val_y` | Tuning hyperparameters | ~24% of all data |
| `test_X` / `test_y` | Final performance measurement | 20% of all data |

**Why three sets instead of two?** If you tune the model on the test set (e.g., try different numbers of trees and pick the one with best test accuracy), you are implicitly fitting the test set and your reported performance will be optimistic. The validation set lets you tune without touching the test set, so the test result is an honest estimate of real-world performance.

**Why `random_state=1337`?** Setting a fixed seed makes the split identical every time you run the code. This is critical for reproducibility – you want the same rows in training and testing every run, so your experiments are comparable.

---

## Training and Evaluation (Network Anomaly Detection)

### Training the Model

```python
rf_model_multi = RandomForestClassifier(random_state=1337)
rf_model_multi.fit(multi_train_X, multi_train_y)
```

`RandomForestClassifier(random_state=1337)` creates a new Random Forest with default settings. The most important defaults are: 100 trees (`n_estimators=100`), splitting until all leaves are pure or have fewer than 2 samples (`min_samples_split=2`), and considering `sqrt(n_features)` features at each split. `random_state=1337` ensures the tree-building random choices are identical each run.

`.fit(multi_train_X, multi_train_y)` is where the learning happens. The algorithm builds 100 decision trees in parallel, each on a different bootstrap sample and with random feature subsets at each split. On 70,000+ training rows with 107 features, this takes a minute or two. After fitting, `rf_model_multi` contains all 100 trained trees and is ready to make predictions.

### Evaluating the Model on the Validation Set

```python
multi_predictions = rf_model_multi.predict(multi_val_X)

accuracy  = accuracy_score(multi_val_y, multi_predictions)
precision = precision_score(multi_val_y, multi_predictions, average='weighted')
recall    = recall_score(multi_val_y, multi_predictions, average='weighted')
f1        = f1_score(multi_val_y, multi_predictions, average='weighted')

print(f"Accuracy:  {accuracy:.4f}")
print(f"Precision: {precision:.4f}")
print(f"Recall:    {recall:.4f}")
print(f"F1-Score:  {f1:.4f}")
```

`.predict(multi_val_X)` sends every validation row through all 100 trees and returns the majority-vote class for each row. Then we measure four things:

**Accuracy** – the simplest metric. What fraction of all predictions were correct?
```python
Accuracy = correct predictions / total predictions
```

**Precision** – when the model says "this is a DoS attack," how often is it actually right? High precision means few false alarms.
```python
Precision = true positives / (true positives + false positives)
```

**Recall** – of all the actual DoS attacks, what fraction did the model catch? High recall means few missed attacks.
```python
Recall = true positives / (true positives + false negatives)
```

**F1-Score** – the harmonic mean of precision and recall. Balances both. Useful when the classes are imbalanced (many more Normal rows than Privilege rows).
```python
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**Why `average='weighted'`?** With five classes of very different sizes (67,000 Normal vs. 43 Privilege), a simple average would treat the 43-sample Privilege class equally to the 67,000-sample Normal class. Weighted averaging weights each class by its number of samples, giving a metric that reflects overall real-world performance.

### Confusion Matrix

```python
conf_matrix = confusion_matrix(multi_val_y, multi_predictions)
class_labels = ['Normal', 'DoS', 'Probe', 'Privilege', 'Access']

sns.heatmap(conf_matrix, annot=True, fmt='d', cmap='Blues',
            xticklabels=class_labels, yticklabels=class_labels)
plt.title('Network Anomaly Detection - Validation Set')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.show()
```

The confusion matrix is a grid where rows are the actual classes and columns are the predicted classes. The diagonal cells (top-left to bottom-right) show **correct predictions**. Off-diagonal cells show **mistakes** – what the model predicted when it was wrong.

**How to read it:**

```
                 Predicted
                 Normal  DoS  Probe  Privilege  Access
Actual Normal  [  high    0      0          0       0 ]   ← most Normal correctly classified
       DoS     [     0  high     0          0       0 ]   ← most DoS correctly classified
       Probe   [     0    0   high          0       0 ]   ← most Probe correctly classified
       Privilege[    0    0      0       some       0 ]   ← few samples, harder to classify
       Access   [    0    0      0          0    high ]   ← most Access correctly classified
```

Large numbers on the diagonal = good. Any large number off-diagonal = a systematic mistake the model is making. For example, if many DoS attacks are being predicted as Normal, that is a serious security gap.

`annot=True` prints the count inside each cell. `fmt='d'` formats them as integers. `cmap='Blues'` uses blue shading: darker = larger number.

### Classification Report

```python
print(classification_report(multi_val_y, multi_predictions, target_names=class_labels))
```

This prints a human-readable table with precision, recall, F1-score, and support (number of samples) for **each class separately**. Example output:

```
              precision    recall  f1-score   support
      Normal       1.00      1.00      1.00     13498
         DoS       1.00      1.00      1.00      9183
       Probe       1.00      0.99      1.00      2331
   Privilege       0.75      0.33      0.46         9
      Access       0.97      0.90      0.93       174
    accuracy                           1.00     25195
   macro avg       0.94      0.84      0.88     25195
weighted avg       1.00      1.00      1.00     25195
```

The Privilege class has poor recall (0.33) because there are only 9 samples – the model has barely any examples to learn from. This is the **class imbalance problem**: rare classes are always harder to detect. The weighted average is still excellent (1.00) because the Privilege class is so small it barely affects the overall numbers. But from a security standpoint, missing privilege escalation attacks 67% of the time is a real problem that would need to be addressed with techniques like oversampling, class weights, or gathering more labeled data.

### Testing the Model on the Test Set

```python
test_multi_predictions = rf_model_multi.predict(test_X)

test_accuracy  = accuracy_score(test_y, test_multi_predictions)
test_precision = precision_score(test_y, test_multi_predictions, average='weighted')
test_recall    = recall_score(test_y, test_multi_predictions, average='weighted')
test_f1        = f1_score(test_y, test_multi_predictions, average='weighted')

print(f"Accuracy:  {test_accuracy:.4f}")
print(f"Precision: {test_precision:.4f}")
print(f"Recall:    {test_recall:.4f}")
print(f"F1-Score:  {test_f1:.4f}")

# Confusion Matrix for Test Set
test_conf_matrix = confusion_matrix(test_y, test_multi_predictions)
sns.heatmap(test_conf_matrix, annot=True, fmt='d', cmap='Blues',
            xticklabels=class_labels, yticklabels=class_labels)
plt.title('Network Anomaly Detection')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.show()

print(classification_report(test_y, test_multi_predictions, target_names=class_labels))
```

This is identical to the validation evaluation but now uses `test_X` and `test_y` – data the model has **never seen at all**, not even indirectly during hyperparameter tuning. This gives the most honest picture of real-world performance. If test accuracy closely matches validation accuracy, the model generalises well (no overfitting). If test accuracy is significantly lower, the model memorised the training data too much.

**Why run validation AND test evaluation separately?** Think of it this way: the validation set is like a practice exam you use to study. The test set is the real exam. If you used the real exam as your study guide, your reported score would be misleading. By keeping them separate, the test score is a genuine measure of what the model can do on data it has truly never encountered.

### Saving the Model

```python
import joblib

model_filename = 'network_anomaly_detection_model.joblib'
joblib.dump(rf_model_multi, model_filename)
print(f"Model saved to {model_filename}")
```

`joblib.dump()` serialises the entire Random Forest to a binary file on disk. A Random Forest with 100 trees trained on 100,000+ rows stores a lot of information – all the tree structures, all the split thresholds, all the leaf predictions. The resulting file is typically several megabytes (around 9 MB for this model).

**Why joblib and not pickle?** Both work, but joblib is optimised for objects containing large NumPy arrays (which scikit-learn models use internally). It compresses them more efficiently and is the official scikit-learn recommendation for model persistence.

**What gets saved:** Every single decision tree in the forest, including which features each node splits on, the threshold values, and which class each leaf predicts. When you load the model later with `joblib.load()`, it is instantly ready to predict – no retraining required.

### Uploading for Evaluation

```python
import requests, json

url = "http://localhost:8001/api/upload"
model_file_path = "network_anomaly_detection_model.joblib"

with open(model_file_path, "rb") as model_file:
    files = {"model": model_file}
    response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))
```

`open(..., "rb")` opens the file in **binary read** mode – necessary for non-text files like `.joblib`. `files={"model": model_file}` wraps it in a multipart form upload that the server expects. If the model meets the performance criteria, the server returns a JSON response containing the flag. If working from your own machine over VPN, replace `localhost:8001` with `<VM-IP>:8001`.

---

## Key Concepts at a Glance

| Concept | Simple Explanation |
|---|---|
| Random Forest | Ensemble of many decision trees that vote on the final answer |
| Bootstrapping | Each tree trains on a different random sample drawn with replacement |
| Feature subsets | Each split considers only a random subset of features – forces tree diversity |
| Voting | Classification = majority vote across all trees |
| NSL-KDD | Cleaned network intrusion dataset with 5 traffic categories |
| Binary target | 0 = Normal, 1 = Any attack |
| Multi-class target | 0 Normal, 1 DoS, 2 Probe, 3 Privilege, 4 Access |
| One-hot encoding | Turn categorical text (tcp/udp) into binary 0/1 columns |
| `get_dummies()` | Pandas function that does one-hot encoding automatically |
| Numeric features | 34 columns measuring bytes, rates, counts, and ratios |
| Validation set | Used during development for hyperparameter tuning |
| Test set | Never touched until final evaluation – gives honest performance |
| `random_state=1337` | Fixed random seed for reproducible results |
| Accuracy | Fraction of all predictions that are correct |
| Precision | When model says class X, how often is it right? |
| Recall | Of all actual class X instances, how many did model catch? |
| F1-Score | Harmonic mean of precision and recall |
| `average='weighted'` | Weight each class by its sample count when averaging |
| Confusion matrix | Grid showing correct and incorrect predictions per class |
| joblib | Library to save and load scikit-learn models efficiently |
| Class imbalance | When some classes have far fewer samples – makes those classes harder to detect |

---

*Notes compiled from: Applications of AI in InfoSec – Network Anomaly Detection, Sections 15–18.*

# Malware Image Classification – Complete Notes (Simple English)

---

## Malware Classification

Malware is software deliberately created to damage systems, steal data, or perform unauthorized actions. Every distinct "type" of malware is called a **malware family** – groups of programs that share the same code base, behaviour, or author. Famous families include Emotet (a banking trojan that also acts as a delivery vehicle for other malware) and WannaCry (a ransomware worm that encrypted files and demanded Bitcoin payments). Classifying a new malware sample into the correct family traditionally requires a human analyst to reverse-engineer the binary – disassembling machine code, tracing system calls, and comparing behaviour. This is extremely time-consuming. An ML classifier can look at thousands of samples and learn to distinguish families automatically, drastically speeding up the triage process. In this module we use a CNN trained on visual representations of malware binaries, which has the additional safety advantage that we never need to execute or directly handle the dangerous binary files themselves.

---

## Why Classify Malware Using Images?

A Windows executable (PE file) is a binary file – a long sequence of bytes. Each byte can hold a value from 0 to 255. The key insight is: **if you arrange those bytes row by row and treat each byte value as a pixel brightness, you get a grayscale image**. A byte value of 0 becomes a black pixel, 255 becomes white, and everything in between becomes a shade of grey.

This is not just a clever trick. Malware families that share the same code base will produce images with similar visual patterns. The packer sections, the import table, the code section, the resource section – each appears at roughly the same position in the image for samples from the same family, creating recognizable textures and block structures. A human can sometimes visually tell two families apart just by looking at the images. A CNN can learn these visual signatures far more reliably across thousands of samples.

**Advantages of the image approach:**
- No need to execute or handle live malware – just PNG files.
- CNNs are extremely well-optimized for image classification.
- Visual patterns are robust to minor code modifications that would fool signature-based scanners.
- The image can reconstruct the original binary exactly – no information is lost in the conversion.

---

## The Malimg Dataset

The **Malimg dataset** was proposed in a research paper and consists of **9,339 grayscale PNG images** representing **25 different malware families**. The dataset is organized in folders – one folder per family, named after the family. Examples of the 25 families included:

```
Adialer.C    Agent.FYI    Allaple.A    Allaple.L    Alueron.gen!J
Autorun.K    C2LOP.gen!g  C2LOP.P      Dialplatform.B  Dontovo.A
Fakerean     Instantaccess  Lolyda.AA1  Lolyda.AA2  Lolyda.AA3
Lolyda.AT    Malex.gen!J  Obfuscator.AD  Rbot!gen   Skintrim.N
Swizzor.gen!E  Swizzor.gen!I  VB.AT    Wintrim.BX  Yuner.A
```

**Class imbalance** is present – Allaple.A and Allaple.L have far more samples than families like Skintrim.N. This matters because a model trained on imbalanced data will be biased toward the majority class. Visualizing the class distribution with a bar plot before training is therefore an important first step, so you know which families might need more data or special handling.

**Download commands:**
```bash
wget https://www.kaggle.com/api/v1/datasets/download/ikrambenabd/malimg-original -O malimg.zip
unzip malimg.zip
```

---

## Exploring the Dataset – Class Distribution

Before training, always visualize the class distribution to spot imbalances:

```python
import os
import matplotlib.pyplot as plt
import seaborn as sns

DATA_BASE_PATH = "./malimg_paper_dataset_imgs/"

# Count images per malware family
dist = {}
for mlw_class in os.listdir(DATA_BASE_PATH):
    mlw_dir = os.path.join(DATA_BASE_PATH, mlw_class)
    dist[mlw_class] = len(os.listdir(mlw_dir))

# HTB Color Palette
htb_green  = "#9FEF00"
node_black = "#141D2B"
hacker_grey = "#A4B1CD"

classes     = list(dist.keys())
frequencies = list(dist.values())

plt.figure(facecolor=node_black)
sns.barplot(y=classes, x=frequencies, edgecolor="black", orient='h', color=htb_green)
plt.title("Malware Class Distribution", color=htb_green)
plt.xlabel("Malware Class Frequency", color=htb_green)
plt.ylabel("Malware Class", color=htb_green)
plt.xticks(color=hacker_grey)
plt.yticks(color=hacker_grey)
ax = plt.gca()
ax.set_facecolor(node_black)
plt.show()
```

`os.listdir(DATA_BASE_PATH)` returns all folder names inside the base directory – each folder is one malware family. `len(os.listdir(mlw_dir))` counts the PNG files inside that family's folder. The resulting bar chart lets you immediately see which families are over-represented (Allaple.A, Allaple.L) and which are rare. If you find the trained model has poor recall on a specific family, the class distribution plot is the first place to investigate.

---

## Preprocessing the Malware Dataset

### Splitting into Train and Test Sets

Before feeding images to the CNN, we split the dataset. We use the `split-folders` library:

```bash
pip3 install split-folders
```

```python
import splitfolders

DATA_BASE_PATH   = "./malimg_paper_dataset_imgs/"
TARGET_BASE_PATH = "./newdata/"

TRAINING_RATIO = 0.8
TEST_RATIO     = 1 - TRAINING_RATIO    # = 0.2

splitfolders.ratio(
    input=DATA_BASE_PATH,
    output=TARGET_BASE_PATH,
    ratio=(TRAINING_RATIO, 0, TEST_RATIO)
)
```

`splitfolders.ratio()` takes the source folder, the destination folder, and a tuple `(train, val, test)`. The `0` in the middle means we create no validation folder (we skip the validation set for this experiment). After running this once, the `./newdata/` directory contains:

```
./newdata/train/   ← 7,459 images (80%)
./newdata/test/    ← 1,880 images (20%)
./newdata/val/     ← 0 images    (unused)
```

Each subfolder inside `train/` and `test/` is named after a malware family – the exact same structure as the original dataset, just with 80% of each family's images in `train/` and 20% in `test/`.

**Why split-folders instead of doing it manually?** It handles the per-class splitting correctly. If you randomly shuffled all images and then split 80/20, some families might end up entirely in the test set. `splitfolders` ensures every class has proportional representation in both sets.

### Defining Preprocessing Transforms

CNNs require all input images to be the same fixed size, and they train more stably when pixel values are normalized. We define these transforms using PyTorch's `torchvision.transforms`:

```python
from torchvision import transforms

transform = transforms.Compose([
    transforms.Resize((75, 75)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])
```

`transforms.Compose([...])` chains multiple transforms into a single pipeline – each image goes through all transforms in order.

**Step 1 – `transforms.Resize((75, 75))`:** Resizes every image to 75×75 pixels, regardless of its original size. Malware images vary in size depending on how large the binary was. The CNN requires a fixed input size. Some fine detail is lost during resizing (large images are compressed, small ones are stretched), but the overall pattern structure is preserved.

**Step 2 – `transforms.ToTensor()`:** Converts the PIL image (or NumPy array) into a PyTorch tensor and rescales pixel values from the range [0, 255] to [0.0, 1.0]. It also changes the axis order from (Height, Width, Channels) to (Channels, Height, Width), which is what PyTorch expects.

**Step 3 – `transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])`:** Subtracts the mean and divides by the standard deviation per channel. These specific values (0.485, 0.456, 0.406 for mean and 0.229, 0.224, 0.225 for std) are the ImageNet dataset statistics – the dataset ResNet50 was originally trained on. Using the same normalization the model was pre-trained with ensures our input distribution matches what the model expects.

**Why normalize?** Without normalization, pixel values range from 0 to 1 after `ToTensor()`. With normalization they are centred around 0 with unit variance. This prevents large activation values in early layers, helps gradient flow, and makes training faster and more stable.

**Why grayscale images get 3 channels:** Our malware images are greyscale (1 channel), but ResNet50 expects 3-channel RGB images. PyTorch's `ImageFolder` automatically converts greyscale images to 3-channel by repeating the single channel three times, so no extra code is needed.

### Loading Datasets with ImageFolder

```python
from torchvision.datasets import ImageFolder
import os

BASE_PATH = "./newdata/"

train_dataset = ImageFolder(
    root=os.path.join(BASE_PATH, "train"),
    transform=transform
)

test_dataset = ImageFolder(
    root=os.path.join(BASE_PATH, "test"),
    transform=transform
)
```

`ImageFolder` is a PyTorch dataset class that automatically reads images organized in a folder-per-class structure. It scans the `root` directory, finds all subfolders (one per malware family), and creates a mapping from folder name to integer class label. The `transform` parameter applies our preprocessing pipeline to every image as it is loaded. `train_dataset.classes` gives the list of class names in alphabetical order. `train_dataset.class_to_idx` gives the name-to-integer mapping.

### Creating DataLoaders

```python
from torch.utils.data import DataLoader

TRAIN_BATCH_SIZE = 512
TEST_BATCH_SIZE  = 1024

train_loader = DataLoader(
    train_dataset,
    batch_size=TRAIN_BATCH_SIZE,
    shuffle=True,
    num_workers=2
)

test_loader = DataLoader(
    test_dataset,
    batch_size=TEST_BATCH_SIZE,
    shuffle=False,
    num_workers=2
)
```

A `DataLoader` wraps a dataset and handles batching, shuffling, and parallel loading. It is what the training loop actually iterates over.

`batch_size=512` for training: each training step processes 512 images at once. Larger batches use more GPU memory but give more stable gradient estimates. `batch_size=1024` for testing: no gradient computation needed, so we can use a larger batch for faster evaluation.

`shuffle=True` for training: randomly reorders the images before each epoch. This prevents the model from learning the order of the data rather than the content. Important because `ImageFolder` loads images alphabetically by class, so without shuffling the model would see all Adialer.C images first, then all Agent.FYI images, etc.

`shuffle=False` for testing: order does not matter for evaluation, and keeping it consistent makes debugging easier.

`num_workers=2`: uses 2 CPU processes to load and preprocess images in the background while the GPU trains. This overlaps data loading with computation, preventing the GPU from sitting idle waiting for the next batch.

### Complete `load_datasets` Function

```python
from torchvision import transforms
from torch.utils.data import DataLoader
from torchvision.datasets import ImageFolder
import os

def load_datasets(base_path, train_batch_size, test_batch_size):
    transform = transforms.Compose([
        transforms.Resize((75, 75)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])

    train_dataset = ImageFolder(root=os.path.join(base_path, "train"), transform=transform)
    test_dataset  = ImageFolder(root=os.path.join(base_path, "test"),  transform=transform)

    train_loader = DataLoader(train_dataset, batch_size=train_batch_size, shuffle=True,  num_workers=2)
    test_loader  = DataLoader(test_dataset,  batch_size=test_batch_size,  shuffle=False, num_workers=2)

    n_classes = len(train_dataset.classes)
    return train_loader, test_loader, n_classes
```

The function also returns `n_classes` – the number of malware families found in the training folder. Reading this dynamically from the data means the code still works if you add or remove families from the dataset without changing any hardcoded numbers.

---

## The Model – ResNet50 with Transfer Learning

### What is ResNet50?

**ResNet50** (Residual Network, 50 layers deep) was introduced by Microsoft Research in 2015 and became a landmark in computer vision. It has approximately **23 million parameters** and uses a clever design called **residual connections** (skip connections) to solve the vanishing gradient problem that made very deep networks impossible to train before. The "50" refers to 50 weight layers (convolutional + fully connected).

ResNet50 was originally trained on **ImageNet** – a dataset of 1.2 million images across 1000 categories (cats, dogs, cars, furniture, etc.). During that training it learned to detect edges, textures, shapes, and complex visual patterns. These learned features are useful far beyond ImageNet – which is why we reuse them.

### Transfer Learning – Standing on the Shoulders of Giants

Training ResNet50 from scratch on ImageNet took weeks on high-end hardware. **Transfer learning** lets us skip that. We download the already-trained weights and use them as our starting point. The model already knows how to detect visual features. We only need to teach it what malware images look like.

**Analogy:** Imagine hiring a photographer who has spent years learning composition, lighting, and focus. Instead of teaching them photography from scratch, you just show them malware images and say "now apply your existing skills to distinguish these families." They adapt far faster than someone starting with zero photography knowledge.

### Freezing Layers

We **freeze** (lock) all the weights in ResNet50's convolutional layers. Only the final fully-connected layer we add will be trained. This means during backpropagation, gradients do not flow back through the frozen layers and those weights do not change.

**Why freeze?** Training 23 million parameters requires enormous data and compute. By freezing the feature extractor and only training the classifier head, we dramatically reduce training time. The trade-off is slightly lower accuracy than full fine-tuning, but for a proof-of-concept this is an excellent trade-off.

### The MalwareClassifier Class

```python
import torch.nn as nn
import torchvision.models as models

HIDDEN_LAYER_SIZE = 1000

class MalwareClassifier(nn.Module):
    def __init__(self, n_classes):
        super(MalwareClassifier, self).__init__()

        # Load pre-trained ResNet50 with ImageNet weights
        self.resnet = models.resnet50(weights='DEFAULT')

        # Freeze all ResNet parameters – no updates during training
        for param in self.resnet.parameters():
            param.requires_grad = False

        # Replace the final fully-connected layer with our custom classifier head
        num_features = self.resnet.fc.in_features    # 2048 for ResNet50
        self.resnet.fc = nn.Sequential(
            nn.Linear(num_features, HIDDEN_LAYER_SIZE),   # 2048 → 1000
            nn.ReLU(),                                     # non-linearity
            nn.Linear(HIDDEN_LAYER_SIZE, n_classes)        # 1000 → 25
        )

    def forward(self, x):
        return self.resnet(x)
```

**Line by line:**

`models.resnet50(weights='DEFAULT')` downloads and loads the ResNet50 architecture with ImageNet pre-trained weights. `weights='DEFAULT'` means "use the best available pre-trained weights."

`for param in self.resnet.parameters(): param.requires_grad = False` iterates over every weight tensor in ResNet50 and sets `requires_grad=False`. This tells PyTorch not to compute or store gradients for these parameters – they will not be updated during training.

`self.resnet.fc.in_features` gets the size of the input to ResNet50's original final layer. For ResNet50 this is always 2048 – the output of the global average pooling layer.

`nn.Sequential(...)` creates a new classifier head with three layers:
- `nn.Linear(2048, 1000)` – a fully-connected layer reducing from 2048 to 1000 features. These weights ARE trainable.
- `nn.ReLU()` – activation function introducing non-linearity.
- `nn.Linear(1000, n_classes)` – output layer producing one score per malware family. For 25 families this is `nn.Linear(1000, 25)`.

`def forward(self, x)` defines how data flows through the model. `x` is a batch of images. It goes through the entire ResNet50 (feature extraction, pooling) and then through our new classifier head, producing a vector of 25 raw scores (logits), one per class.

**Initializing the model:**
```python
# Static: if you always have 25 classes
model = MalwareClassifier(25)

# Dynamic: read from the dataset (preferred)
train_loader, test_loader, n_classes = load_datasets(DATA_PATH, TRAIN_BATCH_SIZE, TEST_BATCH_SIZE)
model = MalwareClassifier(n_classes)
```

---

## Training and Evaluation

### The Training Function

```python
import torch
import time

def train(model, train_loader, n_epochs, verbose=False):
    model.train()                                    # put model in training mode
    criterion = torch.nn.CrossEntropyLoss()          # loss function
    optimizer = torch.optim.Adam(model.parameters()) # optimizer

    training_data = {"accuracy": [], "loss": []}

    for epoch in range(n_epochs):
        running_loss = 0
        n_total   = 0
        n_correct = 0
        checkpoint = time.time() * 1000

        for inputs, labels in train_loader:          # iterate batches
            optimizer.zero_grad()                    # clear old gradients
            outputs = model(inputs)                  # forward pass
            loss = criterion(outputs, labels)        # compute loss
            loss.backward()                          # backpropagation
            optimizer.step()                         # update weights

            _, predicted = outputs.max(1)            # pick highest-score class
            n_total   += labels.size(0)
            n_correct += predicted.eq(labels).sum().item()
            running_loss += loss.item()

        epoch_loss     = running_loss / len(train_loader)
        epoch_duration = int(time.time() * 1000 - checkpoint)
        epoch_accuracy = compute_accuracy(n_correct, n_total)

        training_data["accuracy"].append(epoch_accuracy)
        training_data["loss"].append(epoch_loss)

        if verbose:
            print(f"[i] Epoch {epoch+1} of {n_epochs}: Acc: {epoch_accuracy:.2f}% "
                  f"Loss: {epoch_loss:.4f} (Took {epoch_duration} ms).")

    return training_data
```

**Key concepts explained one by one:**

`model.train()` switches the model into training mode. This matters for layers like BatchNorm and Dropout that behave differently during training vs. evaluation. ResNet50 uses BatchNorm, so this call is important.

`torch.nn.CrossEntropyLoss()` is the standard loss function for multi-class classification. It combines a softmax (converts raw scores to probabilities) with negative log-likelihood loss. For the correct class, you want high probability; the loss penalizes low probability for the true class. Lower loss = model is more confident and correct.

`torch.optim.Adam(model.parameters())` creates the Adam optimizer. Adam is an adaptive learning rate optimizer – it tracks a running average of gradients and their squares, automatically adjusting the effective learning rate per parameter. `model.parameters()` passes only the trainable parameters (the unfrozen classifier head) to the optimizer.

`optimizer.zero_grad()` clears the accumulated gradients from the previous batch. PyTorch accumulates gradients by default (adding them each time `.backward()` is called). Failing to zero them would corrupt the gradient calculation for the current batch.

`outputs = model(inputs)` runs the forward pass – sends the batch of images through ResNet50 and our classifier head, producing a tensor of shape `(batch_size, 25)` containing one raw score per class per image.

`loss = criterion(outputs, labels)` computes how wrong the predictions are. `outputs` are the raw scores (logits), `labels` are the true class indices. CrossEntropyLoss converts logits to probabilities internally and computes the loss.

`loss.backward()` runs backpropagation – computes the gradient of the loss with respect to every trainable parameter using the chain rule. This is the mathematical heart of learning: it tells us which direction to move each weight to reduce the loss.

`optimizer.step()` updates all trainable weights by taking a small step in the direction that reduces the loss (negative gradient direction). The step size is controlled by the Adam optimizer's adaptive learning rate.

`outputs.max(1)` finds the class with the highest score for each image. The `1` means "take the max along dimension 1 (the class dimension)." Returns two tensors: values (the max scores) and indices (which class). We only need the indices (`predicted`).

### Loss Function – CrossEntropyLoss in Plain English

Imagine the model produces scores `[2.1, 0.3, -1.2, ..., 4.5]` for 25 classes. The true class is class index 3 (Fakerean). CrossEntropyLoss:
1. Converts all 25 scores to probabilities via softmax (all positive, sum to 1.0).
2. Looks at the probability assigned to class 3.
3. Computes `-log(probability_of_correct_class)`. If probability is 0.95 → loss = `-log(0.95) ≈ 0.05` (small loss, good prediction). If probability is 0.01 → loss = `-log(0.01) ≈ 4.6` (large loss, bad prediction).
4. Averages this over the entire batch.

Training minimizes this number across all batches and epochs.

### Saving the Model

```python
def save_model(model, path):
    model_scripted = torch.jit.script(model)
    model_scripted.save(path)
```

`torch.jit.script(model)` compiles the model into TorchScript – a serialized format that can be loaded without the original Python class definition. This is more portable than `torch.save(model.state_dict(), path)` because it bundles both the architecture and the weights together. `model_scripted.save(path)` writes the compiled model to a `.pth` file on disk.

**Why TorchScript instead of pickle/joblib?** For PyTorch models, `torch.jit.script` produces a self-contained file that can be loaded in any PyTorch environment (including C++ applications) without needing the `MalwareClassifier` class to be defined. It also enables optimization passes that can speed up inference.

### The Predict Function

```python
def predict(model, test_data):
    model.eval()

    with torch.no_grad():
        output = model(test_data)
        _, predicted = torch.max(output.data, 1)

    return predicted
```

`model.eval()` switches the model to evaluation mode. BatchNorm layers use running statistics instead of batch statistics. Dropout layers are disabled (all neurons active). This is essential for consistent, deterministic predictions.

`torch.no_grad()` disables gradient computation entirely. During inference we do not need gradients (we are not updating weights), so disabling them saves memory and speeds up computation by about 2x.

`torch.max(output.data, 1)` finds the class with the highest score for each image in the batch, returning the predicted class indices.

### The Evaluation Function

```python
def compute_accuracy(n_correct, n_total):
    return round(100 * n_correct / n_total, 2)


def evaluate(model, test_loader):
    model.eval()
    n_correct = 0
    n_total   = 0

    with torch.no_grad():
        for data, target in test_loader:
            predicted = predict(model, data)
            n_total   += target.size(0)
            n_correct += (predicted == target).sum().item()

    return compute_accuracy(n_correct, n_total)
```

This iterates over every batch in the test loader, makes predictions, and counts how many are correct. `(predicted == target).sum().item()` creates a boolean tensor (True where prediction matches label), sums the True values (counting correct predictions), and `.item()` converts the PyTorch tensor scalar to a Python integer. The final accuracy is `100 × (correct / total)`.

### Plotting Training Progress

```python
import matplotlib.pyplot as plt

def plot(data, title, label, xlabel, ylabel):
    htb_green   = "#9FEF00"
    node_black  = "#141D2B"
    hacker_grey = "#A4B1CD"

    plt.figure(figsize=(10, 6), facecolor=node_black)
    plt.plot(range(1, len(data)+1), data, label=label, color=htb_green)
    plt.title(title, color=htb_green)
    plt.xlabel(xlabel, color=htb_green)
    plt.ylabel(ylabel, color=htb_green)
    plt.xticks(color=hacker_grey)
    plt.yticks(color=hacker_grey)
    ax = plt.gca()
    ax.set_facecolor(node_black)
    ax.spines['bottom'].set_color(hacker_grey)
    ax.spines['top'].set_color(node_black)
    ax.spines['right'].set_color(node_black)
    ax.spines['left'].set_color(hacker_grey)
    legend = plt.legend(facecolor=node_black, edgecolor=hacker_grey, fontsize=10)
    plt.setp(legend.get_texts(), color=htb_green)
    plt.show()

def plot_training_accuracy(training_data):
    plot(training_data['accuracy'], "Training Accuracy", "Accuracy", "Epoch", "Accuracy (%)")

def plot_training_loss(training_data):
    plot(training_data['loss'], "Training Loss", "Loss", "Epoch", "Loss")
```

`range(1, len(data)+1)` creates x-axis values starting from 1 (epoch 1, not epoch 0). The `training_data` dictionary returned by `train()` contains lists of accuracy and loss values, one per epoch. Plotting these over epochs lets you diagnose training behaviour: a steadily rising accuracy curve means healthy training; a flat or oscillating curve might indicate a learning rate problem or insufficient epochs.

---

## Running the Complete Pipeline

```python
# Parameters
DATA_PATH           = "./newdata/"
N_EPOCHS            = 10
TRAINING_BATCH_SIZE = 512
TEST_BATCH_SIZE     = 1024
HIDDEN_LAYER_SIZE   = 1000
MODEL_FILE          = "malware_classifier.pth"

# Step 1: Load datasets
train_loader, test_loader, n_classes = load_datasets(DATA_PATH, TRAINING_BATCH_SIZE, TEST_BATCH_SIZE)

# Step 2: Initialize model
model = MalwareClassifier(n_classes)

# Step 3: Train
print("[i] Starting Training...")
training_information = train(model, train_loader, N_EPOCHS, verbose=True)

# Step 4: Save
save_model(model, MODEL_FILE)

# Step 5: Evaluate
accuracy = evaluate(model, test_loader)
print(f"[i] Inference accuracy: {accuracy}%.")

# Step 6: Plot
plot_training_accuracy(training_information)
plot_training_loss(training_information)
```

**Expected training output (10 epochs):**

```
[i] Epoch 1  of 10: Acc: 57.09%  Loss: 1.4741
[i] Epoch 2  of 10: Acc: 85.01%  Loss: 0.4631
[i] Epoch 3  of 10: Acc: 89.60%  Loss: 0.2880
[i] Epoch 4  of 10: Acc: 91.88%  Loss: 0.2294
[i] Epoch 5  of 10: Acc: 92.97%  Loss: 0.2113
[i] Epoch 6  of 10: Acc: 93.86%  Loss: 0.1744
[i] Epoch 7  of 10: Acc: 95.13%  Loss: 0.1572
[i] Epoch 8  of 10: Acc: 94.81%  Loss: 0.1501
[i] Epoch 9  of 10: Acc: 96.51%  Loss: 0.1188
[i] Epoch 10 of 10: Acc: 96.26%  Loss: 0.1198
[i] Inference accuracy: 88.54%.
```

The training accuracy rises rapidly in the first few epochs then levels off. Test accuracy (88.54%) is lower than final training accuracy (96.26%) – this gap represents overfitting to the training data. For a proof-of-concept with frozen weights and simple preprocessing, 88%+ is acceptable. With more epochs, unfrozen layers, data augmentation, or a larger hidden layer, accuracy would improve further.

**Note for the Playground VM:** Each epoch may take up to 10 minutes. Three epochs are sufficient to meet the required accuracy threshold for the flag. On a machine with a GPU, each epoch takes under 1 minute.

---

## Model Upload for Evaluation

```python
import requests, json

url = "http://localhost:8002/api/upload"
model_file_path = "malware_classifier.pth"

with open(model_file_path, "rb") as model_file:
    files = {"model": model_file}
    response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))
```

If working over VPN from your own machine, replace `localhost:8002` with `<VM-IP>:8002`. Evaluating an uploaded model may take up to 2 minutes on the VM. If the model meets the performance criteria, the server returns a JSON response containing the flag.

---

## Key Concepts at a Glance

| Concept | Simple Explanation |
|---|---|
| Malware family | Group of malware programs sharing the same code base or behaviour |
| PE file | Windows Portable Executable – the binary format of Windows programs |
| Malware image | Grayscale PNG where each pixel = one byte of the binary (0=black, 255=white) |
| Malimg dataset | 9,339 grayscale malware images across 25 families |
| Class imbalance | Some families have far more samples than others |
| split-folders | Python library for splitting image datasets into train/val/test folders |
| ImageFolder | PyTorch class for loading folder-organized image datasets automatically |
| `transforms.Resize` | Scales all images to a fixed size (75×75 here) |
| `transforms.ToTensor` | Converts image to PyTorch tensor, scales pixels from [0,255] to [0.0,1.0] |
| `transforms.Normalize` | Subtracts mean, divides by std to standardize pixel values |
| ImageNet normalization | Mean [0.485, 0.456, 0.406] and std [0.229, 0.224, 0.225] – matches ResNet50 pre-training |
| DataLoader | Handles batching, shuffling, and parallel data loading |
| `shuffle=True` | Randomize image order each epoch to prevent order-based learning |
| `num_workers=2` | 2 CPU processes load data in background while GPU trains |
| ResNet50 | 50-layer CNN with ~23M parameters; state-of-the-art image classifier |
| Transfer learning | Reuse weights trained on one task (ImageNet) as a starting point for another (malware) |
| Freezing layers | Setting `requires_grad=False` – weights don't update during training |
| Classifier head | The new fully-connected layers we add and train on malware data |
| CrossEntropyLoss | Loss function for multi-class classification = softmax + negative log-likelihood |
| Adam optimizer | Adaptive learning rate optimizer; usually faster and more stable than SGD |
| `model.train()` | Switch to training mode (BatchNorm uses batch stats, Dropout active) |
| `model.eval()` | Switch to eval mode (BatchNorm uses running stats, Dropout disabled) |
| `torch.no_grad()` | Disable gradient tracking during inference to save memory and speed up |
| `optimizer.zero_grad()` | Clear accumulated gradients before each batch |
| `loss.backward()` | Run backpropagation – compute gradients of loss w.r.t. all trainable weights |
| `optimizer.step()` | Update weights using computed gradients |
| `torch.jit.script` | Compile model to TorchScript – self-contained portable format |
| `.pth` file | PyTorch model file format |
| Epoch | One complete pass through the entire training dataset |

---

*Notes compiled from: Applications of AI in InfoSec – Malware Image Classification, Sections 20–24.*


