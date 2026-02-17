#  Phishing URL Detection System (Post-Mortem Analysis)

> **Status:** *Archived / Retrospective Analysis* > **Model:** Logistic Regression  
> **Focus:** Cybersecurity, Feature Engineering, Machine Learning

##  Overview
This project explores the application of **Machine Learning (Logistic Regression)** to cybersecurity, specifically for detecting malicious (phishing) URLs based purely on their **Lexical Features**. 

Unlike traditional blacklisting methods which require constant updates, this approach attempts to classify URLs dynamically by analyzing their text structure (e.g., length, character counts, path depth) to identify suspicious patterns common in phishing attacks.

**Note:** This repository serves as a documentation of the research, implementation, and a critical analysis of the "Training-Serving Skew" challenges encountered during deployment.

---

##  Methodology & Architecture

The system was designed to operate without external dependencies (like DNS or Whois) to ensure speed and privacy. The pipeline consists of three stages:



### 1. Data Collection
We utilized a labeled dataset of approximately **[Insert Number, e.g., 10,000]** URLs, split between:
* **Benign (0):** Legitimate websites (Google, YouTube, etc.).
* **Phishing (1):** Confirmed malicious links from repositories like PhishTank.

### 2. Feature Engineering (The Core Logic)
Since raw text cannot be fed directly into Logistic Regression, we extracted **20+ numerical features** from each URL string. Key features included:

* **`urlLen` (URL Length):** Phishing URLs are statistically longer to hide the true domain.
* **`host_letter_count`:** Quantifies the randomness of the domain name (high randomness often indicates algorithmic generation).
* **`pathurlRatio`:** The ratio of the path length to the total URL length.
* **`delimeter_path`:** Count of special characters (`-`, `_`, `?`, etc.) in the path, often used to obfuscate malicious intent.
* **`subDirLen`:** The length of sub-directories, checking for deep nesting used to mimic legitimate file structures.

### 3. Model Selection
**Logistic Regression** was chosen for:
* **Interpretability:** Ability to see *which* features (weights) contributed most to a "Phishing" classification.
* **Efficiency:** extremely low latency for real-time inference (O(1) prediction time).

---

##  Retrospective: The "Training-Serving Skew" Challenge

While the model achieved a **Training Accuracy of ~96.3%**, the project encountered a critical issue during the real-time inference phase.

### The Problem
When testing the model on live URLs (e.g., `google.com`), the model consistently predicted **False Positives (Probability ~1.0)**.

### Root Cause Analysis
A deep dive revealed a classic **Training-Serving Skew**:
1.  **Feature Logic Mismatch:** The code used to generate the *training dataset* (likely from a pre-processed CSV) used a slightly different logic than our custom `feature_extractor.py` script.
2.  **Normalization Errors:** The live data was not normalized (scaled) in the same way as the training data.
3.  **Result:** The model received input vectors that were drastically different in scale from what it learned, causing the intercept term to dominate the decision boundary.

###  Lessons Learned
* **Pipeline Consistency:** The feature extraction code must be identical for both training and inference. Ideally, this logic should be encapsulated in a shared library or class.
* **Unit Testing for Data:** "Data Unit Tests" are necessary to verify that a URL produces the exact same feature vector in the training pipeline as it does in the production pipeline.

---

## 🛠️ Tech Stack
* **Language:** Python 3.x
* **Libraries:** `pandas`, `scikit-learn`, `numpy`, `seaborn` (for visualization)
* **Algorithm:** Logistic Regression (Binary Classification)

---
