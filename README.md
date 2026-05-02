# ai-cybersecurity-labs

---

## Labs

| # | Lab | Technique | Dataset | Key Finding |
|---|-----|-----------|---------|-------------|
| 1 | IoT Malware Behavior Detection | Random Forest + Neural Network | CTU-IoT-Malware-Capture-1 (Zeek network logs) | Single-class subset exposed data leakage risk from label columns; RF outperforms NN on small tabular datasets |
| 2 | SMS Spam Detection | Naive Bayes (MultinomialNB) | UCI SMS Spam Collection (5,574 messages) | Vectoriser must be fit on training data only; accuracy is misleading under 87/13 class imbalance |
| 3 | Metamorphic Malware Detection | Hidden Markov Model (hmmlearn) | IDAN1/2/3 (x86 disassembly opcode sequences) | Sequence structure is critical — each binary is one sequence, not thousands of isolated opcodes |
| 4 | Synthetic QR Code Generation | Generative Adversarial Network (Keras) | Programmatically generated (80 train / 20 val) | GANs learn statistical texture, not structural constraints — outputs are visually plausible but not decodable |

---

## Lab Summaries

### 01 — IoT Malware Behavior Detection

**Techniques:** Random Forest, Neural Network, PCA, StandardScaler, OneHotEncoder

The CTU-IoT-Malware-Capture-1 dataset contains Zeek/Bro connection logs from an IoT device. The pipeline preprocesses mixed categorical and numerical features through a sklearn `Pipeline` (scaling + one-hot encoding + PCA) before training both a Random Forest and a Keras neural network classifier.

The provided subset contains only Benign-labelled traffic, making the classification task trivial and the results misleading. Key corrections included removing label columns from the feature matrix to prevent data leakage, dropping non-generalising identifier columns (UID, IP addresses, timestamps), and fixing an incorrect target variable assignment.

---

### 02 — SMS Spam Detection with Naive Bayes

**Techniques:** MultinomialNB, CountVectorizer (BoW), TfidfVectorizer, NLTK stop-word removal

The UCI SMS Spam Collection is a well-known benchmark with 5,574 messages (4,827 ham, 747 spam). The pipeline preprocesses raw SMS text through lowercasing, punctuation removal, tokenisation, and stop-word filtering before vectorising with both Bag-of-Words and TF-IDF representations.

The key bug identified was fitting the vectoriser on the full dataset before the train/test split — a data leakage issue that inflates reported performance. The correct pattern is split first, then fit the vectoriser on training data only and use `transform()` on the test set.


---

### 03 — Metamorphic Malware Detection with HMMs

**Techniques:** Hidden Markov Model (MultinomialHMM), LabelEncoder, opcode sequence analysis

Metamorphic malware rewrites its own code on each infection cycle, defeating signature-based detection. HMMs attack this differently — they model statistical transitions between instruction types, which are harder to mutate away than specific byte patterns.

The IDAN1/2/3 datasets are x86 assembly opcode sequences from disassembled binaries. The critical implementation decision is treating each CSV file as a single sequence rather than a collection of individual opcodes. Feeding thousands of length-1 sequences to an HMM destroys all transition information and the model learns nothing meaningful.


---

### 04 — Synthetic QR Code Generation with a GAN

**Techniques:** DCGAN architecture, Conv2DTranspose generator, Conv2D discriminator, adversarial training loop

A GAN consists of two networks trained in opposition: a generator that maps random noise to synthetic images, and a discriminator that classifies images as real or fake. The generator improves by learning to fool the discriminator; the discriminator improves by learning to detect fakes.

With only 80 training images over 5,000 epochs, the generated outputs are visually QR-like but not decodable — an expected result. QR codes are rigidly structured binary signals with finder patterns, alignment markers, and Reed-Solomon error correction at precise pixel positions. A convolutional GAN has no mechanism to enforce these constraints.

---

## Environment

```
Python        3.10
tensorflow    2.x
scikit-learn  1.x
hmmlearn      0.3.x
pandas        2.x
numpy         1.x
qrcode        7.x
nltk          3.x
```
