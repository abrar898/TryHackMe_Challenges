# AI Evasion – Complete Notes (Simple English)

---

## Table of Contents

1. [Introduction to AI Evasion Attacks](#1-introduction-to-ai-evasion-attacks)
2. [The AI Attack Landscape](#2-the-ai-attack-landscape)
3. [Evasion in Traditional ML vs LLMs](#3-evasion-in-traditional-ml-vs-llms)
4. [This Module's Focus – GoodWords](#4-this-modules-focus--goodwords)
5. [The GoodWords Attack](#5-the-goodwords-attack)
6. [Naive Bayes – Theory Refresher](#6-naive-bayes--theory-refresher)
7. [Attack Mechanics – How Probability Gets Manipulated](#7-attack-mechanics--how-probability-gets-manipulated)
8. [Good Word Selection Strategy](#8-good-word-selection-strategy)
9. [Why the Attack Works – Exploiting Independence](#9-why-the-attack-works--exploiting-independence)
10. [Spam Filter Implementation](#10-spam-filter-implementation)
11. [Library Installation and Initial Setup](#11-library-installation-and-initial-setup)
12. [Loading the SMS Spam Dataset](#12-loading-the-sms-spam-dataset)
13. [Text Preprocessing](#13-text-preprocessing)
14. [Training the Naive Bayes Classifier](#14-training-the-naive-bayes-classifier)
15. [GoodWords Attack Implementation (White-Box)](#15-goodwords-attack-implementation-white-box)
16. [GoodWords Extraction and Goodness Scores](#16-goodwords-extraction-and-goodness-scores)
17. [Running the Attack Loop](#17-running-the-attack-loop)
18. [Attack Analysis – Word Impact](#18-attack-analysis--word-impact)
19. [Probability Shift Analysis](#19-probability-shift-analysis)
20. [Black-Box GoodWords Attack – Overview](#20-black-box-goodwords-attack--overview)
21. [Query-Based Function Approximation](#21-query-based-function-approximation)
22. [Exploration vs Exploitation Trade-off (Bandit Framework)](#22-exploration-vs-exploitation-trade-off-bandit-framework)
23. [Upper Confidence Bound (UCB) Strategy](#23-upper-confidence-bound-ucb-strategy)
24. [Black-Box Core Components](#24-black-box-core-components)
25. [Building the Candidate Vocabulary](#25-building-the-candidate-vocabulary)
26. [Query Management and Budget Allocation](#26-query-management-and-budget-allocation)
27. [Black-Box Adaptive Discovery Methods](#27-black-box-adaptive-discovery-methods)
28. [Three-Phase Discovery Algorithm](#28-three-phase-discovery-algorithm)
29. [Black-Box Attack Implementation and Results](#29-black-box-attack-implementation-and-results)
30. [Skills Assessment – GoodWords Challenge](#30-skills-assessment--goodwords-challenge)

---

## 1. Introduction to AI Evasion Attacks

An **evasion attack** is when an attacker tricks a trained AI model at the time it is making a prediction (called **inference time**). The attacker does NOT touch the model itself or its training data – they only change the **input** that is fed to the model. The model then gives a wrong answer. Think of it like a student who has already been taught, but someone hands them a trick question that confuses them. Because the attack only happens when the model is being used (not when it is being trained), it can bypass all the safety checks that were built into the training pipeline. This is why evasion is a very specific and dangerous threat for **deployed AI systems** – systems that are already running in the real world. Anyone who is responsible for securing AI systems, like **AI Red Teamers**, must deeply understand how evasion works.

---

## 2. The AI Attack Landscape

AI systems can be attacked at **two different stages**: during training, or during inference (when predictions are made). These two stages are very different and require very different defenses.

**Training-time attacks** change what the model learns:

- **Data poisoning** – inject fake or altered training samples so the model learns the wrong patterns.
- **Label manipulation** – change the correct answer labels so the model thinks wrong answers are right.
- **Trojan attacks** – hide a secret trigger in the model so it misbehaves only when that trigger appears in the input.

**Inference-time attacks (Evasion)** do not touch training at all. They just send a specially crafted input through the normal interface. The model's brain (its parameters) stays unchanged; only what it sees in that moment is changed. One very important idea is **transferability** – an adversarial example crafted to fool one model can often fool a completely different model too, especially if the two models were trained on similar data. This means an attacker can prepare their attack offline and then use it against a real-world API without ever seeing the model's internals.

---

## 3. Evasion in Traditional ML vs LLMs

In **traditional machine learning** (like spam filters or malware detectors), the model takes structured features as input. So an attacker can make tiny, controlled edits to those features to push a prediction across the decision boundary.

Examples:
- In a **spam filter** using a bag-of-words model, adding some innocent-looking words changes the word counts, which changes the probability the model uses to decide – spam or not spam.
- In a **malware detector**, rearranging sections of a program file changes the byte pattern the model sees, so it no longer looks like malware – even though it still works exactly like malware.

In **Large Language Models (LLMs)**, the model reads free-form text and follows instructions, so the attack pattern is different. The main attack is called **prompt injection** – an attacker hides special instructions inside the text the model reads, tricking the model into following those instructions instead of the original ones. The effect is still inference-time only, but it is more dangerous because the model's output can itself trigger further actions, like running code, executing database queries, or causing API calls.

**The common thread in both cases:** only the input is changed at prediction time; the model's weights stay the same. The difference is just what kind of input is being manipulated.

---

## 4. This Module's Focus – GoodWords

This module focuses on a hands-on evasion technique called the **GoodWords attack**. It targets **Naive Bayes spam filters** by appending carefully chosen legitimate (ham-like) words to a spam message. Because Naive Bayes treats every word as independent, adding enough ham-like words shifts the probability score enough to make the model classify spam as legitimate email. Studying this attack teaches several important ideas that apply beyond spam: how to exploit model assumptions, the difference between **white-box** (full model access) and **black-box** (no model access) attacks, and the trade-off between adding too few words (attack fails) and too many (looks suspicious). These ideas generalize to many other classifiers and adversarial tasks.

---

## 5. The GoodWords Attack

The GoodWords attack was introduced by **Lowd and Meek in 2005** in a paper titled *"Good Word Attacks on Statistical Spam Filters."* The core idea is clever and simple: instead of trying to change or hide the spam content, you **leave the spam message completely unchanged** and just **stick some innocent-looking words at the end**. These extra words are called "good words" – they look like they belong in a legitimate email. The Naive Bayes classifier adds up evidence from every word it sees. By flooding the message with words that strongly say "this is legitimate," the total evidence shifts toward ham (legitimate), and the classifier makes the wrong decision. The spam message is delivered successfully.

**Analogy:** Imagine a bouncer who checks if you belong at a party by looking at your outfit. You're wearing a suspicious hoodie (spam content), but you wrap yourself in a fancy suit jacket on top (good words). The bouncer sees mostly "fancy suit" and lets you in.

---

## 6. Naive Bayes – Theory Refresher

**Naive Bayes** is a type of classifier that uses probability to make decisions. Given a document (message), it calculates the probability it belongs to each class (spam or ham) and picks the one with the higher probability.

The core formula is Bayes' theorem:

```
P(Class | Document) = P(Document | Class) × P(Class) / P(Document)
```

In plain English: "Given this message, what is the probability it is spam?" The classifier compares P(spam | message) vs P(ham | message) and picks the winner.

The **"naive" assumption** is that every word in the message contributes **independently** to the probability. This means the probability of the whole document is just the product of each individual word's probability:

```
P(Document | Class) = P(word1 | Class) × P(word2 | Class) × ... × P(wordN | Class)
```

In practice, these are very small numbers and multiplying many of them together causes numerical errors (the result becomes essentially zero). So everything is done in **log space**, where multiplication becomes addition:

```
Score = log P(Class) + log P(word1 | Class) + log P(word2 | Class) + ...
```

This additive structure is the key weakness the GoodWords attack exploits.

---

## 7. Attack Mechanics – How Probability Gets Manipulated

Think of the classifier as a **weighing scale**. Every word adds weight to either the spam side or the ham side. Spam words like "FREE" and "WINNER" pile weight onto the spam side.

**Before the attack**, a spam message scores:

```
log P(spam | message) = log P(spam) + sum of log P(each word | spam)
log P(ham  | message) = log P(ham)  + sum of log P(each word | ham)
```

The spam score is higher, so the message is classified as spam. ✓

**After appending good words**, the equations gain new terms:

```
log P(spam | augmented) = log P(spam) + [original spam words] + sum of log P(good word | spam)
log P(ham  | augmented) = log P(ham)  + [original ham words]  + sum of log P(good word | ham)
```

Each good word strongly favors ham (it appears much more often in ham than in spam). So the ham log-score grows faster than the spam log-score. Add enough good words and the ham score overtakes the spam score – the classifier now says "ham."

**The reversal condition (attack succeeds when):**

```
Sum of [log P(good word | ham) - log P(good word | spam)]
  > Sum of [log P(spam word | spam) - log P(spam word | ham)] + [log P(spam) - log P(ham)]
```

In plain English: the total "ham pull" from the good words must exceed the total "spam push" from the original spam words plus any built-in class bias.

**Example:** If the original spam words create a combined spam advantage of 8.0, and each good word contributes 3.0 toward ham, you need at least 3 good words (3 × 3.0 = 9.0 > 8.0) to tip the scale.

---

## 8. Good Word Selection Strategy

Not all ham words are equally powerful. We want words that appear **frequently in ham** but are **rare in spam**. To measure this, we compute a **goodness score** for each word:

```
Goodness Score (w) = P(w | ham) / (P(w | spam) + ε)
```

Where `ε` (epsilon) is a tiny number (like 0.0000000001) to prevent division by zero. A high goodness score means the word is much more common in ham than in spam.

**Example:** If "meeting" appears in 45% of ham messages but only 1% of spam messages, its goodness score is very high (around 45). If "FREE" appears in 2% of ham but 40% of spam, its goodness score is very low (around 0.05).

**Selection strategy:**
1. Compute goodness scores for all words in the vocabulary.
2. Sort them from highest to lowest.
3. Pick the top N words to append.

**Important constraint:** Adding too few words may not be enough to flip the classification. Adding too many makes the message look unnatural or too long. Research shows that adding **15 to 30** carefully chosen good words typically achieves evasion rates above **90%**.

---

## 9. Why the Attack Works – Exploiting Independence

The critical flaw the GoodWords attack exploits is the **conditional independence assumption** of Naive Bayes. Because the model treats every word independently, it **cannot penalize the weirdness** of a spam message that suddenly also contains lots of professional-sounding words like "meeting," "tomorrow," and "thanks." A human reading the message would immediately notice the mismatch. The model just keeps adding up word scores.

**Four specific weaknesses that make this possible:**

1. **Additive log-space scoring** – every appended word simply adds to the running sum. More words always shift the score.
2. **Static learned distributions** – the model's word probabilities are fixed after training. An attacker can study them and know exactly which words to add.
3. **No context modeling** – the model has no idea that "FREE WINNER CLAIM" and "meeting tomorrow thanks" are semantically incompatible in one message.
4. **Laplace smoothing** – a common technique that assigns non-zero probability to every word, even rare ones. This means the attacker is never blocked from adding a word because it is "too unusual" for spam.

---

## 10. Spam Filter Implementation

To demonstrate the attack, we first build a real **Naive Bayes spam classifier** trained on real SMS messages (the SMS Spam Collection from UCI, with 5,574 labeled messages). This becomes our "victim" model. Building it ourselves means we understand every part of it, which helps us understand exactly how the attack works. The classifier uses **bag-of-words** features – it simply counts how many times each word appears in a message and uses those counts to calculate spam/ham probabilities. The entire implementation uses Python with standard data science libraries (scikit-learn, pandas, numpy). The trained model achieves about 99% accuracy on the test set, making it a realistic and strong target.

---

## 11. Library Installation and Initial Setup

**Install the custom HTB library:**

```bash
# Install the AI Library (or update it)
pip install --upgrade git+https://github.com/PandaSt0rm/htb-ai-library
```

**Import standard libraries:**

```python
import json
import pickle
import random
import re
import urllib.request
import zipfile
from pathlib import Path
import numpy as np

# Set seeds so results are the same every run
random.seed(1337)
np.random.seed(1337)
```

**Import ML libraries:**

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.model_selection import train_test_split
from sklearn.naive_bayes import MultinomialNB
from sklearn.metrics import classification_report, accuracy_score
```

**Import HTB color/utility library:**

```python
from htb_ai_library import (
    AZURE, HACKER_GREY, HTB_GREEN, MALWARE_RED, NODE_BLACK,
    NUGGET_YELLOW, WHITE, AQUAMARINE,
    load_model, save_model,
)
```

**Why seed `1337`?** Setting random seeds means every time you run the code you get exactly the same results. This is important when testing attacks – you need to know if a difference in evasion rate is due to the attack strategy, not random chance.

**Why `pathlib.Path`?** It automatically handles folder separator differences between Windows (`\`) and Linux (`/`). For example: `Path("data") / "file.txt"` works on both systems without any changes.

---

## 12. Loading the SMS Spam Dataset

```python
print("\n[*] Loading SMS Spam Dataset...")

data_dir = Path("data")
data_dir.mkdir(exist_ok=True)
dataset_path = data_dir / "sms_spam.csv"
```

**Caching logic – check if already downloaded:**

```python
if dataset_path.exists():
    print(f"[+] Using cached dataset: {dataset_path}")
    df = pd.read_csv(dataset_path)
else:
    print("[*] Downloading from UCI repository...")
    url = "https://archive.ics.uci.edu/ml/machine-learning-databases/00228/smsspamcollection.zip"
    zip_path = data_dir / "sms_spam.zip"
    urllib.request.urlretrieve(url, zip_path)
```

**Extract and parse the downloaded zip file:**

```python
    with zipfile.ZipFile(zip_path, "r") as zf:
        with zf.open("SMSSpamCollection") as f:
            lines = [line.decode("utf-8").strip() for line in f]

    data = []
    for line in lines:
        parts = line.split('\t')        # tab-separated: label and message
        if len(parts) == 2:
            data.append({"label": parts[0].lower(), "message": parts[1]})

    df = pd.DataFrame(data)
    df.to_csv(dataset_path, index=False)
    zip_path.unlink()                   # delete the zip to save space
    print(f"[+] Dataset saved to {dataset_path}")
```

**Dataset statistics:**
```
[+] Loaded 5572 messages
    Spam: 747    (13.4%)
    Ham:  4825   (86.6%)
```

**Why cache?** The first download takes 2-3 seconds. Subsequent reads from the saved CSV file take under 50ms – about 40× faster. During development when you run the code many times, this adds up.

**Why check `len(parts) == 2`?** Real-world data files sometimes have blank lines or broken records. This check prevents errors by only processing rows that have exactly a label and a message.

**Class imbalance note:** There are far more ham messages (86.6%) than spam (13.4%). This mirrors real life. It also means the classifier naturally tends to predict "ham" more often, which can be exploited in our attack.

---

## 13. Text Preprocessing

Raw SMS text is messy. We clean it in **two stages**: minimal cleaning (preserve spam indicators) then full cleaning (for the vectorizer).

**Stage 1 – Minimal cleaning:**

```python
import html as html_module
import unicodedata

def minimal_clean(text):
    # Convert HTML entities like &amp; → &
    text = html_module.unescape(text)

    # Normalize unicode (e.g., the ligature "ﬁ" becomes "fi")
    text = unicodedata.normalize('NFKC', text)

    # Collapse multiple spaces/newlines into one space
    text = re.sub(r'\s+', ' ', text)
    text = re.sub(r'\n+', ' ', text)
    text = re.sub(r'\r+', ' ', text)

    return text.strip()
```

**Stage 2 – Full cleaning for the vectorizer:**

```python
def clean_text(text):
    text = text.lower()   # "FREE" and "free" become the same feature

    # Keep numbers, currency symbols (£$€¥), and punctuation – these are spam clues!
    # Only remove characters that cause real problems
    text = re.sub(r'[^\w\s£$€¥!?.,;:\'\"-]', ' ', text)
    text = re.sub(r'\s+', ' ', text)
    return text.strip()
```

**Apply both stages:**

```python
df['preprocessed'] = df['message'].apply(minimal_clean)
df['clean_message'] = df['preprocessed'].apply(clean_text)
```

**Why keep currency symbols and exclamation marks?** These are strong spam indicators. `£900 prize!!!` looks very different from a normal message. Removing these would make the classifier less accurate.

**Why two stages?** The first stage prepares text for human analysis (still readable). The second stage prepares text for the machine (fully normalized and consistent).

**Remove duplicates and empty messages:**

```python
original_size = len(df)
df = df.drop_duplicates(subset=['label', 'clean_message'])
print(f"[+] Removed {original_size - len(df)} duplicates")

df = df[df['clean_message'].str.len() > 0]
```

```
[+] Removed 419 duplicates
[+] Final dataset: 5153 messages  |  Spam: 638 (12.4%)  |  Ham: 4515 (87.6%)
```

Spam messages had a higher duplicate rate – they tend to be repetitive templates. Removing duplicates stops the model from over-fitting to the same spam phrasing.

**Train/test split:**

```python
X = df['clean_message'].values
y = df['label'].values

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

```
Training: 4122 messages
Testing:  1031 messages
```

**Why `stratify=y`?** Without stratification, random splitting could create an unbalanced test set (e.g., 20% spam instead of 12%). `stratify=y` guarantees the same spam/ham ratio in both sets, which gives us fair performance measurements.

---

## 14. Training the Naive Bayes Classifier

**Check for saved model (avoid retraining every run):**

```python
model_dir = Path("models")
model_dir.mkdir(exist_ok=True)
model_path = model_dir / "spam_classifier.pkl"

if model_path.exists():
    with open(model_path, 'rb') as f:
        saved_data = pickle.load(f)
        vectorizer = saved_data['vectorizer']
        classifier = saved_data['classifier']
    X_train_vec = vectorizer.transform(X_train)    # use old vocabulary
    X_test_vec  = vectorizer.transform(X_test)
```

**Train a fresh model if not cached:**

```python
else:
    vectorizer = CountVectorizer(
        max_features=3000,
        # Custom pattern: words + currency symbols + numbers + repeated punctuation
        token_pattern=r'\b\w+\b|[£$€¥]+|\d+|!!+|\?\?+|\.\.+',
        lowercase=True,
        stop_words='english'
    )
    X_train_vec = vectorizer.fit_transform(X_train)
    X_test_vec  = vectorizer.transform(X_test)

    classifier = MultinomialNB()
    classifier.fit(X_train_vec, y_train)

    with open(model_path, 'wb') as f:
        pickle.dump({'vectorizer': vectorizer, 'classifier': classifier}, f)
    print(f"[+] Model saved to {model_path}")
```

**Evaluate performance:**

```python
train_acc = classifier.score(X_train_vec, y_train)
test_acc  = classifier.score(X_test_vec, y_test)
print(f"Training accuracy: {train_acc:.4f}")   # 0.9922
print(f"Testing accuracy:  {test_acc:.4f}")    # 0.9864
```

```
              precision  recall  f1-score
ham               0.99    0.99      0.99
spam              0.95    0.94      0.94
overall accuracy                    0.99
```

**Why `transform()` not `fit_transform()` on the test set?** `fit_transform()` builds a new vocabulary from whatever data it sees. If we called it on the test set, the model would "see" test words it never saw in training. Instead we use `transform()`, which applies the existing training vocabulary – any test word not in that vocabulary is simply ignored. This is correct real-world behavior.

**Why `max_features=3000`?** Limiting to 3000 features prevents the model from memorizing rare words that only appear once or twice, improving generalization.

**Why `stop_words='english'`?** Words like "the," "is," "a" appear equally often in spam and ham – they provide zero useful information for classification. Removing them speeds up training and reduces noise.

---

## 15. GoodWords Attack Implementation (White-Box)

**White-box** means we have **full access** to the model's internals: we can see every word probability, every feature, every weight. This is like having the answer key before the exam. In this setting, we do not need to guess which words are good – we can directly calculate a **goodness score** for every word in the vocabulary and pick the best ones mathematically. The attack then appends the top-scoring words to spam messages and tests whether the classifier now says "ham." This is the strongest version of the attack. Later we will see the black-box version where we have no internal access.

---

## 16. GoodWords Extraction and Goodness Scores

**Step 1 – Get feature names and log probabilities:**

```python
feature_names = vectorizer.get_feature_names_out()
ham_log_probs  = classifier.feature_log_prob_[0]   # log P(word | ham)
spam_log_probs = classifier.feature_log_prob_[1]   # log P(word | spam)
```

`feature_log_prob_` stores `log P(word | class)` for each word and each class. Index `[0]` = ham, `[1]` = spam. We work in log space because multiplying thousands of tiny probabilities would underflow to zero.

**Step 2 – Calculate goodness scores:**

```python
goodness_scores = []
for i, word in enumerate(feature_names):
    ham_prob  = np.exp(ham_log_probs[i])    # convert log → probability
    spam_prob = np.exp(spam_log_probs[i])
    goodness  = ham_prob / (spam_prob + 1e-10)  # 1e-10 prevents division by zero
    goodness_scores.append((word, goodness, ham_prob, spam_prob))
```

**Step 3 – Sort and print top 10:**

```python
goodness_scores.sort(key=lambda x: x[1], reverse=True)
top_good_words = goodness_scores[:100]

for word, score, hp, sp in top_good_words[:10]:
    print(f"{word:15} | goodness: {score:8.2f} | ham_p: {hp:.4f} | spam_p: {sp:.4f}")
```

```
lor             | goodness:    50.22 | ham_p: 0.0047 | spam_p: 0.0001
ü               | goodness:    47.99 | ham_p: 0.0045 | spam_p: 0.0001
...             | goodness:    45.65 | ham_p: 0.0299 | spam_p: 0.0007
da              | goodness:    45.39 | ham_p: 0.0042 | spam_p: 0.0001
later           | goodness:    29.39 | ham_p: 0.0027 | spam_p: 0.0001
doing           | goodness:    25.67 | ham_p: 0.0024 | spam_p: 0.0001
ask             | goodness:    25.30 | ham_p: 0.0024 | spam_p: 0.0001
really          | goodness:    24.55 | ham_p: 0.0023 | spam_p: 0.0001
lol             | goodness:    23.81 | ham_p: 0.0022 | spam_p: 0.0001
```

**Analogy:** Imagine sorting all foods by how much a vegetarian restaurant would serve them vs a steakhouse. "Salad" would score very high (common at vegetarian, rare at steakhouse). "Ribeye" would score very low. That's exactly what goodness score measures – how much more "ham" a word is compared to "spam."

---

## 17. Running the Attack Loop

**The helper function to augment a message:**

```python
def augment_message(message, words_to_add):
    """Append good words to a message"""
    if len(words_to_add) > 0:
        return message + " " + " ".join(words_to_add)
    return message
```

**Test different word counts and measure evasion rate:**

```python
word_counts   = [0, 5, 10, 15, 20, 25, 30, 35, 40]
attack_results = []

for num_words in word_counts:
    selected_words = [w for w, _, _, _ in top_good_words[:num_words]]
    evaded = 0

    for message in spam_test_messages:
        augmented = augment_message(message, selected_words)
        vec   = vectorizer.transform([augmented])
        prob  = classifier.predict_proba(vec)[0]

        # [0] = ham probability, [1] = spam probability
        if prob[0] > prob[1]:    # ham probability wins → attack succeeded
            evaded += 1

    evasion_rate = (evaded / len(spam_test_messages)) * 100
    attack_results.append({'num_words': num_words, 'evasion_rate': evasion_rate})
    print(f"Words: {num_words:2d} | Evasion: {evasion_rate:6.2f}% ({evaded}/{len(spam_test_messages)})")
```

**Results:**

```
Words:  0 | Evasion:   6.25% (8/128)
Words:  5 | Evasion:  41.41% (53/128)
Words: 10 | Evasion:  74.22% (95/128)
Words: 15 | Evasion:  96.09% (123/128)
Words: 20 | Evasion: 100.00% (128/128)
Words: 25 | Evasion: 100.00% (128/128)
Words: 30 | Evasion: 100.00% (128/128)
```

**What the numbers tell us:**

- `0 words` → only 6.25% evasion. These are natural false-negatives (spam the model already gets wrong).
- `5 words` → evasion jumps to 41%. Nearly half of all spam bypasses the filter.
- `15 words` → 96% evasion. Almost complete success.
- `20 words` → **100% evasion**. Every single spam message in the test set fools the classifier.

**The decision boundary is 0.5.** When `prob[0] > prob[1]` (i.e., ham probability > 0.5), the message is misclassified as ham. A message with `prob = [0.501, 0.499]` evades just as well as one with `prob = [0.99, 0.01]`. The model only cares which side wins, not by how much.

**Three attack phases (S-curve pattern):**

- **Phase 1 (0–5 words):** Small gains. Not enough to overcome strong spam signals.
- **Phase 2 (5–15 words):** Rapid rise. This is where most messages cross the decision boundary.
- **Phase 3 (15–20 words):** Saturation. Adding more words gives no extra benefit because 100% of messages already evade.

---

## 18. Attack Analysis – Word Impact

We want to know: which individual words do the most damage to the spam classification?

**Test each word individually on 50 spam messages:**

```python
sample_spam  = spam_test_messages[:50]
word_impacts = []

for word, _, _, _ in top_good_words[:20]:
    total_impact = 0
    for message in sample_spam:
        vec_orig = vectorizer.transform([message])
        prob_orig = classifier.predict_proba(vec_orig)[0][1]    # spam prob before

        vec_aug  = vectorizer.transform([message + " " + word])
        prob_aug  = classifier.predict_proba(vec_aug)[0][1]     # spam prob after

        impact = prob_orig - prob_aug    # reduction in spam probability
        total_impact += impact

    avg_impact = (total_impact / len(sample_spam)) * 100
    word_impacts.append((word, avg_impact))

word_impacts.sort(key=lambda x: x[1], reverse=True)
```

**Sample output:**
```
Word 'lor': 3.98% reduction
Word 'ü':   3.93% reduction
Word '...': 3.89% reduction
Most Effective Single Word: 'lor' (reduces spam prob by ~4.0%)
```

**Key insight:** No single word is a "silver bullet." The best word only reduces spam probability by about 4%. This makes sense – the model learned from thousands of messages, and strong spam signals like "FREE!!!" and "£900 WINNER" accumulate a lot of probability mass. One word cannot overcome all of that alone. Success requires **combination and accumulation**.

**Why does goodness score differ from practical impact?** Goodness score measures how "ham-like" a word is in the training data (a ratio). Practical impact measures how much a word actually shifts the probability when added to a real spam message. A word can score very high on goodness but have low practical impact because the original spam message's evidence is just too strong for one word to budge.

---

## 19. Probability Shift Analysis

This analysis visualizes how spam probability decreases as we keep adding more good words to individual messages.

**Setup:**

```python
sample_messages  = spam_test_messages[:8]
test_word_counts = [0, 5, 10, 20, 30]
```

For each message and each word count, we compute the spam probability and plot grouped bars. When `spam_probability < 0.5`, the message successfully evades (shown with a ✓ checkmark on the bar).

**Key observations from the chart:**

- Messages M7 and M8 start with spam probability below 0.5 even at 0 words – these are natural false negatives.
- M5 needs only 5 words to drop below the 0.5 boundary.
- M2, M3, M4 need 10 words.
- M1 and M6 have very strong spam indicators and need 20 words to finally cross the boundary.

**The pattern confirms:** the stronger the original spam signal in a message, the more good words you need. But the attack always eventually succeeds – there is no message for which it fails when given enough words.

Each added word contributes `log P(word | ham)` to the ham score. When enough of these accumulate, the total ham evidence crosses the decision boundary. This is the additive log-space structure working against the classifier.

---

## 20. Black-Box GoodWords Attack – Overview

In the **white-box** setting we had complete access to the model: feature probabilities, model weights, everything. In the **black-box** setting, we have none of that. We can only:

1. **Send a message** to the classifier.
2. **Observe the output** (label + spam probability).

We cannot see inside the model at all. This is the realistic scenario – most real-world spam filters expose only an API endpoint, not their internal parameters. The challenge becomes: how do we find effective good words without being able to directly calculate goodness scores? The answer: **systematic experimentation**. We send many queries, observe the results, and gradually learn which words work. The core idea stays the same – add ham-like words to shift the probability. The method changes – we must discover the good words empirically instead of computing them directly.

---

## 21. Query-Based Function Approximation

In the black-box setting, we treat the classifier as an unknown function:

```
f(message) → spam probability (a number between 0 and 1)
```

Our goal: find a set of words W to append to the message that minimizes `f(message + W)`.

**Measuring a word's impact without model access (finite differences):**

```
reward(word, message) = f(message) - f(message + word)
```

- Send the original message → get `f(message)`.
- Send the message with the word added → get `f(message + word)`.
- Subtract. If the spam probability dropped by 0.15, the reward is 0.15 → good word.
- If it dropped by only 0.01, the reward is 0.01 → weak word.
- If it increased the spam probability, the reward is negative → harmful word.

This is called a **finite difference** estimate – we approximate the word's effect by measuring the actual difference before and after. It is less precise than the direct probability calculation in the white-box setting, but it works with only query access.

---

## 22. Exploration vs Exploitation Trade-off (Bandit Framework)

Every query costs resources (time, money, detection risk). We have a limited budget (say 1000 queries). How do we use them wisely?

This is the **multi-armed bandit problem**. Imagine a row of slot machines, each with an unknown payout probability. You have a limited number of pulls. Do you:
- Keep pulling the machine that gave the best result so far? (**exploitation**)
- Try machines you have not tested yet to see if they are even better? (**exploration**)

In our attack:
- Each "slot machine" is a candidate word.
- Each "pull" is a query where we test that word on a spam message.
- The "payout" is the reduction in spam probability.

**The dilemma:**
- Pure exploitation: we might keep using the first decent word we found and miss a much better one.
- Pure exploration: we waste all our budget on new words and never have enough tests on good ones to be confident about them.

**The solution:** a strategy that does both intelligently.

---

## 23. Upper Confidence Bound (UCB) Strategy

UCB assigns each word a score combining two components:

```
UCB_score(word) = average_impact(word) + c × sqrt(ln(total_queries) / times_tested(word))
```

- **First term (average_impact):** how well the word has performed historically. High = word is good → exploit it.
- **Second term (exploration bonus):** how uncertain we are about this word. Words tested only a few times have high uncertainty → give them a bonus to encourage testing.

**Breaking down the exploration bonus:**
- `times_tested(word)` is in the denominator → less-tested words get a bigger bonus.
- `total_queries` is in the numerator → as we run more queries overall, all exploration bonuses slowly grow (but shrink per-word as we test each word more).
- `c` (usually 2.0) controls how aggressively we explore.

**Concrete example with numbers (total queries t = 200):**

| Word | Times tested | Average impact | Exploration bonus | UCB score |
|---|---|---|---|---|
| "thanks" | 50 | 0.15 | 2×√(ln(200)/50) = 0.47 | **0.62** |
| "appreciate" | 5 | 0.12 | 2×√(ln(200)/5) = 1.48 | **1.60** ← chosen |
| "wonderful" | 0 | 0 | ∞ | **∞** ← always test first |

Even though "thanks" has a higher observed impact (0.15 vs 0.12), "appreciate" has a much higher UCB score because we've tested it far less. The algorithm wisely chooses to explore "appreciate" because we can't yet be sure our 5-test estimate of 0.12 is accurate. **UCB provides mathematical guarantees:** the total regret (lost effectiveness from exploration mistakes) grows only as log(T), not linearly.

---

## 24. Black-Box Core Components

**The attack simulation setup:**

```python
query_budget = 1000
queries_used = 0
query_log    = []    # optional: track timing/response data
```

The 1000-query budget mirrors real-world API constraints. Many commercial NLP services charge per prediction or enforce daily rate limits. Every query we send must count.

**The budget is divided into three phases: 40% exploration, 40% exploitation, 20% combination.**

```python
def estimate_budget_allocation(total_budget):
    explore = int(0.4 * total_budget)
    exploit = int(0.4 * total_budget)
    combine = total_budget - explore - exploit    # absorb rounding
    return {'exploration': explore, 'exploitation': exploit, 'combination': combine}
```

```
exploration :  400 queries
exploitation:  400 queries
combination :  200 queries
Total: 1000
```

---

## 25. Building the Candidate Vocabulary

Without model access, we can't directly look up goodness scores. We build a **candidate vocabulary** by mining ham (legitimate) messages from the training data.

**Step 1 – Extract word frequencies from ham messages:**

```python
def extract_ham_word_freq(X_train, y_train, sample_size=500):
    ham_msgs = X_train[y_train == 'ham']
    limit = min(sample_size, len(ham_msgs))
    freq = {}
    for msg in ham_msgs[:limit]:
        for w in str(msg).split():
            if 2 < len(w) < 10:     # keep conversational tokens, skip tiny/huge words
                freq[w] = freq.get(w, 0) + 1
    return freq
```

**Why length filter `2 < len(w) < 10`?** Words with 2 or fewer characters ("a", "I") are mostly stop words with no discriminative power. Words longer than 10 characters are often URLs or technical artifacts, not conversational words.

**Step 2 – Select high-frequency words:**

```python
def select_high_frequency_words(word_freq, max_words=100, min_freq=5):
    sorted_by_freq = sorted(word_freq.items(), key=lambda x: (-x[1], x[0]))
    top = [w for w, c in sorted_by_freq if c > min_freq][:max_words]
    return top
```

Only words appearing more than 5 times are kept. This filters out rare words that might be dataset artifacts rather than genuine ham signals.

**Step 3 – Merge with hand-curated conversational words:**

```python
def merge_with_curated(top_words, additional_candidates=None):
    if additional_candidates is None:
        additional_candidates = [
            "ok", "cos", "ill", "thats", "later", "said", "ask", "didnt",
            "dont", "doing", "going", "come", "home", "tomorrow", "today", "sorry",
            "thanks", "yeah", "yes", "sure", "see", "tell", "know", "think",
        ]
    merged = set(top_words) | set(additional_candidates)
    return sorted(merged)
```

The curated list adds informal contractions and temporal words ("tomorrow," "today") that strongly signal legitimate conversation but might be underrepresented in a small ham sample.

**Full combined function:**

```python
candidate_words = build_candidate_vocabulary(X_train, y_train)
# Result: ~112 candidate words
```

```
First 10: you, and, the, have, but, for, that, i'm, all, your
```

---

## 26. Query Management and Budget Allocation

The exploration phase randomly shuffles candidates and spam messages to avoid ordering bias:

```python
np.random.shuffle(candidate_words)
np.random.shuffle(test_spam_samples)
```

**Why shuffle?** Without shuffling, if we run out of budget halfway through, we might have systematically missed certain words (e.g., all words starting with letters T-Z). Shuffling ensures any partial test is a fair sample.

The **40-40-20 split** is a practical heuristic:
- **40% exploration:** cast a wide net across the candidate vocabulary to get rough estimates.
- **40% exploitation:** deeply test the best words from exploration to get reliable estimates.
- **20% combination:** discover synergistic pairs/triplets of words.

---

## 27. Black-Box Adaptive Discovery Methods

**Initialize the adaptive scorer:**

```python
def initialize_adaptive_scorer():
    return {
        'word_scores': {},        # word → effectiveness score
        'word_counts': {},        # word → number of times tested
        'exploration_rate': 0.2   # 20% of selections will explore
    }
```

**Epsilon-greedy word selection:**

```python
def epsilon_greedy_select(scorer, available_words):
    import random
    if random.random() < scorer['exploration_rate']:
        # Explore: prefer words never tested before
        untested = [w for w in available_words if w not in scorer['word_counts']]
        if untested:
            return random.choice(untested)
        else:
            return min(available_words, key=lambda w: scorer['word_counts'].get(w, 0))
    else:
        # Exploit: pick the currently best-scoring word
        return max(available_words, key=lambda w: scorer['word_scores'].get(w, 0))
```

- 20% of the time: explore (try untested or least-tested words).
- 80% of the time: exploit (use the word with the highest known impact).

**Update word scores with Exponential Moving Average (EMA):**

```python
def update_word_score(scorer, word, impact, alpha=0.3):
    if word not in scorer['word_scores']:
        scorer['word_scores'][word] = impact
        scorer['word_counts'][word] = 1
    else:
        old_score = scorer['word_scores'][word]
        scorer['word_scores'][word] = (1 - alpha) * old_score + alpha * impact
        scorer['word_counts'][word] += 1
```

**Alpha = 0.3** means each new observation gets 30% weight, the historical average keeps 70%. This prevents one lucky or unlucky test from wildly changing the score.

**Example:** If "thats" had score 0.10 and its new test shows 0.15:
```
new_score = 0.7 × 0.10 + 0.3 × 0.15 = 0.07 + 0.045 = 0.115
```

**Combinatorial search (finding synergistic word pairs/triplets):**

```python
def discover_word_combinations(message, test_words, max_size=3):
    from itertools import combinations
    combination_scores = {}
    base_score = classifier.predict_proba(vectorizer.transform([message]))[0][1]

    # Test individual words
    for word in test_words[:20]:
        score = classifier.predict_proba(vectorizer.transform([message + " " + word]))[0][1]
        combination_scores[(word,)] = base_score - score

    # Test word pairs
    if max_size >= 2:
        for w1, w2 in combinations(test_words[:15], 2):
            score  = classifier.predict_proba(vectorizer.transform([message + " " + w1 + " " + w2]))[0][1]
            individual = combination_scores.get((w1,), 0) + combination_scores.get((w2,), 0)
            actual = base_score - score
            synergy = actual - individual
            if synergy > 0:    # only keep pairs that work better together than separately
                combination_scores[(w1, w2)] = actual

    # Test triplets (best 5 pairs only, to limit query usage)
    if max_size >= 3:
        top_pairs = sorted([(k,v) for k,v in combination_scores.items() if len(k)==2],
                           key=lambda x: x[1], reverse=True)[:5]
        for pair, _ in top_pairs:
            for word in test_words[:10]:
                if word not in pair:
                    triplet = tuple(sorted(pair + (word,)))
                    score = classifier.predict_proba(vectorizer.transform([message + " " + " ".join(triplet)]))[0][1]
                    combination_scores[triplet] = base_score - score

    return combination_scores
```

**Why look for combinations?** Some word pairs are super-additive. "Meeting" alone might reduce spam probability by 10% and "tomorrow" by 8%. Together they might achieve 25% reduction because the combination strongly signals legitimate scheduling – a pattern the classifier learned from training data.

---

## 28. Three-Phase Discovery Algorithm

```python
def three_phase_discovery(spam_messages, candidate_words, budget=1000):
    scorer = initialize_adaptive_scorer()
    queries_used = 0
    allocation = estimate_budget_allocation(budget)
    exploration_budget  = allocation['exploration']    # 400
    exploitation_budget = allocation['exploitation']   # 400
    combination_budget  = allocation['combination']    # 200
```

**Phase 1 – Broad Exploration (400 queries):**

```python
    while queries_used < exploration_budget:
        test_message = random.choice(spam_messages)
        word = epsilon_greedy_select(scorer, candidate_words)

        prob_orig = classifier.predict_proba(vectorizer.transform([test_message]))[0][1]
        prob_aug  = classifier.predict_proba(vectorizer.transform([test_message + " " + word]))[0][1]

        impact = prob_orig - prob_aug
        update_word_score(scorer, word, impact)
        queries_used += 2    # two queries per test (before and after)
```

Phase 1 output:
```
[P1 100/400] tested_words=10 | top3=need:0.166, say:0.005, thanks:0.000
[P1 200/400] tested_words=17 | top3=happy:0.219, sleep:0.175, say:0.005
[+] Exploration complete. Queries: 400, Words tested: 40
Top5: happy:0.024, sleep:0.001, need:0.001, got:0.001, say:0.001
```

**Phase 2 – Focused Exploitation (400 queries):**

```python
    scorer['exploration_rate'] = 0.1    # reduce exploration to 10%
    top_word_list = [w for w, _ in sorted(scorer['word_scores'].items(),
                                          key=lambda x: x[1], reverse=True)[:30]]

    while queries_used < exploration_budget + exploitation_budget:
        test_message = random.choice(spam_messages[:20])    # focus on fewer messages
        word = random.choice(top_word_list[:15])             # focus on best words
        ...
        update_word_score(scorer, word, impact)
        queries_used += 2
```

Phase 2 refines estimates for the 30 best words from Phase 1 by testing them repeatedly on a focused set of messages. More tests = more reliable average impact estimates.

**Phase 3 – Combination Discovery (200 queries):**

```python
    for i in range(min(3, len(spam_messages))):
        combos = discover_word_combinations(spam_messages[i], top_word_list[:20], max_size=3)
        for combo, score in combos.items():
            if combo not in best_combinations or score > best_combinations[combo]:
                best_combinations[combo] = score
        queries_used += min(remaining_combo, len(combos) * 2)
```

Phase 3 discovers synergistic word pairs and triplets. The best combination found might be something like "later + going" → 25% spam probability reduction together vs 10% + 8% = 18% individually.

**Return values:** sorted word rankings, best combinations, total queries used.

---

## 29. Black-Box Attack Implementation and Results

**Run the full three-phase discovery:**

```python
discovered_words, combination_scores, total_queries = three_phase_discovery(
    spam_test_messages[:50],
    candidate_words,
    budget=query_budget
)
```

**Sample discovered words:**
```
really       | impact: 0.121   (reduces spam prob by 12.1 percentage points)
got          | impact: 0.117
eat          | impact: 0.115
sleep        | impact: 0.078
later        | impact: 0.068
```

**Test discovered words at different word counts:**

```python
for num_words in [0, 5, 10, 15, 20, 25, 30]:
    selected = [w for w, _ in discovered_words[:num_words]]
    evaded = 0
    for msg in eval_messages:
        aug   = msg + " " + " ".join(selected)
        prob  = classifier.predict_proba(vectorizer.transform([aug]))[0][1]
        if prob < 0.5:
            evaded += 1
    rate = (evaded / len(eval_messages)) * 100
    print(f"Words: {num_words:2d} | Evasion: {rate:.2f}%")
```

**Typical black-box results:**
```
Words:  0 | Evasion:  ~6%
Words: 10 | Evasion: ~70-75%
Words: 15 | Evasion: ~90-95%
Words: 20 | Evasion: ~95-100%
```

**Comparison – White-box vs Black-box:**

| Metric | White-Box | Black-Box |
|---|---|---|
| Model access | Full | Query only |
| Best single word impact | ~4% ("lor") | ~12% ("really") |
| Evasion at 20 words | 100% | ~95-100% |
| Queries needed | 0 (direct calc) | 1000 |
| Attack success | Very high | Nearly as high |

**Why does black-box sometimes find "better" words?** White-box goodness scores measure theoretical ham-association from training data (probability ratios). Black-box discovery measures actual probability shifts on real messages. These two metrics can disagree. A word can have high theoretical goodness but low practical impact when added to actual spam. Black-box empirically finds what actually works.

**Final results saved:**

```python
results = {
    'white_box': attack_results,
    'black_box': blackbox_results,
    'top_good_words': [(w, float(s)) for w, s, _, _ in top_good_words[:20]],
    'word_impacts': word_impacts[:10]
}
with open(output_dir / "results.json", 'w') as f:
    json.dump(results, f, indent=2)
```

**Key takeaway for defenders:** Even without any internal access, an attacker who can query the model can discover effective good words and achieve 85-95% of white-box evasion rates. Hiding the model's internals (through an API) provides very limited security against this type of adaptive attack. The root vulnerability is the architecture itself (Naive Bayes additive scoring), not information leakage.

---

## 30. Skills Assessment – GoodWords Challenge

This is the hands-on challenge that tests everything covered in the module. The goal is to apply the GoodWords attack against a live classifier through an API.

**Health check:**

```bash
export BASE_URL="http://your-instance:8080"
curl -s "$BASE_URL/health"
# Expected: {"status": "healthy", "service": "skills_assessment_lab"}
```

**Get the challenge (base spam message + constraints):**

```bash
curl -s "$BASE_URL/challenge" | jq
```

```json
{
  "base_message": "...",
  "max_added_words": 25,
  "target_label": "ham"
}
```

**Query the model for any text (black-box predict):**

```bash
curl -s -X POST "$BASE_URL/predict" \
  -H 'content-type: application/json' \
  -d '{"text": "hello there"}'
```

```json
{"label": "spam", "spam_probability": 0.9123}
```

**Submit your augmented text:**

```bash
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"augmented_text": "<base_message> please thanks meeting hello"}'
```

```json
{
  "result": "success",
  "details": {"label": "ham", "spam_probability": 0.12, "words_added": 4},
  "flag": "HTB{...}"
}
```

**Python minimal scaffold – get baseline:**

```python
import os, requests
host = os.getenv("BASE_URL", "http://127.0.0.1:8080")
ch   = requests.get(f"{host}/challenge", timeout=10).json()
base = ch["base_message"]
res  = requests.post(f"{host}/predict", json={"text": base}, timeout=15).json()
print(res)    # see baseline spam probability
```

**Python scaffold – measure word impacts and attack:**

```python
import os, requests, random, numpy as np
random.seed(1337); np.random.seed(1337)
host = os.getenv("BASE_URL", "http://127.0.0.1:8080")

ch     = requests.get(f"{host}/challenge", timeout=10).json()
base   = ch["base_message"]
budget = int(ch["max_added_words"])

def predict(t):
    return requests.post(f"{host}/predict", json={"text": t}, timeout=15).json()

# Step 1: get baseline spam probability
base_p = predict(base)["spam_probability"]
print(f"Baseline spam probability: {base_p:.4f}")

# Step 2: measure individual word impacts
vocab = ["please","thanks","meeting","tomorrow","coffee","home",
         "support","good","great","safe","ok","later","doing",
         "going","sorry","yeah","sure","know","think","really"]

imp = []
for w in vocab:
    p2 = predict(base + " " + w)["spam_probability"]
    imp.append((w, base_p - p2))    # positive = word helps us evade

imp.sort(key=lambda x: x[1], reverse=True)
print("Top impactful words:")
for w, d in imp[:5]:
    print(f"  {w}: {d:.4f} reduction")

# Step 3: build augmented message greedily under word budget
top_words = [w for w, d in imp if d > 0][: max(2*budget, 20)]
aug = base
for i, w in enumerate(top_words, 1):
    if i > budget:
        break
    aug = aug + " " + w
    lab = predict(aug)["label"]
    print(f"After {i} words: label={lab}")
    if lab == "ham":
        print("Attack succeeded!")
        break

# Step 4: submit
result = requests.post(f"{host}/submit", json={"augmented_text": aug}, timeout=15).json()
print(result)    # look for "flag" key in the response
```

**Success criteria:**
- The augmented message must start with the exact base message (append-only).
- Number of added words must be ≤ `max_added_words`.
- The model must predict "ham" for the augmented message.
- When all conditions are met, the server returns the flag: `HTB{...}`.

**Strategy summary:**
1. Get the base message from `/challenge`.
2. Test individual candidate words using `/predict` to measure their spam-reduction impact.
3. Sort words by impact (biggest reduction first).
4. Greedily add words one at a time, checking label after each addition.
5. Stop as soon as label flips to "ham" (or when budget is exhausted).
6. Submit to `/submit` to receive the flag.

---

## Summary – Key Concepts at a Glance

| Concept | Simple Explanation |
|---|---|
| Evasion attack | Trick a deployed model by changing only the input, not the model itself |
| GoodWords attack | Append innocent-looking words to spam so the classifier thinks it's legitimate |
| Naive Bayes weakness | Treats words independently; more ham words always increase ham score |
| Goodness score | How much more often a word appears in ham vs spam |
| White-box attack | Full model access; calculate goodness scores directly |
| Black-box attack | Only query access; discover good words by testing and observing results |
| Epsilon-greedy | 80% exploit best known words, 20% explore new ones |
| EMA (alpha=0.3) | Smooth word score updates; new result gets 30%, history keeps 70% |
| UCB | Balance exploration/exploitation with mathematical guarantees |
| Synergy | Some word pairs work better together than their individual effects predict |
| Decision boundary | Ham wins when spam probability < 0.5 |
| Attack saturation | After ~20 words, adding more gives no additional evasion benefit |

---

*Notes compiled from: AI Evasion – Foundations course, Sections 1–12.*
