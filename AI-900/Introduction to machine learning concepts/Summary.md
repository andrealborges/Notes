# Machine Learning — Summary

## Introduction
- ML sits at the intersection of **data science** (prepare/explore data) and **software engineering** (deploy models for **inferencing**).
- Core idea: learn from **past observations** to predict **unknown outcomes**.
- Examples: forecast ice-cream sales from weather; predict diabetes risk from clinical data; classify penguin species from measurements.

---

## ML Models: Training → Inferencing
- A model encapsulates a function **y = f(x)** learned during **training**.
- **x** = feature vector `[x1, x2, ...]`; **y** = label; **ŷ** (“y-hat”) = predicted label at inference time.
- An **algorithm** fits a function that maps features to labels; resulting **model** is used to predict new **ŷ**.

---

## Types of Machine Learning

### Supervised Learning
- Training data includes **features and labels**.

**Regression** (numeric **y**):
- Examples: sales, house price, fuel efficiency.

**Classification** (categorical **y**):
- **Binary** (two classes): e.g., diabetic vs. not.
- **Multiclass** (many classes): e.g., penguin species, movie genre.
- **Multilabel**: multiple valid labels (e.g., sci-fi + comedy).

### Unsupervised Learning
- Training data has **features only**; discover structure/relationships.

**Clustering**:
- Groups similar observations without prior labels.
- Often used to **discover segments** before building a supervised classifier.

---

## Regression: Evaluation & Iteration

### Common Metrics
- **MAE**: mean absolute error (average |y − ŷ|).
- **MSE**: mean squared error (emphasizes larger errors).
- **RMSE**: √MSE (back in label units).
- **R²**: proportion of variance explained (closer to 1 is better).

### Iterative Training Cycle
- Adjust **features**, **algorithm choice**, and **hyperparameters** to improve metrics; pick the best acceptable model.

---

## Binary Classification: Concepts & Metrics

**Training**
- Learn **f(x) = P(y=1 | x)** (e.g., logistic regression’s sigmoid).
- Apply a **threshold** (commonly 0.5) to turn probabilities into class predictions.

**Confusion Matrix Terms**
- **TP**, **TN**, **FP**, **FN**.

**Metrics**
- **Accuracy** = (TP + TN) / all.
- **Recall (TPR)** = TP / (TP + FN).
- **Precision** = TP / (TP + FP).
- **F1** = 2 · (Precision · Recall) / (Precision + Recall).
- **ROC & AUC**: plot TPR vs. FPR across thresholds; **AUC** near 1.0 = strong model (0.5 ≈ random).

---

## Multiclass Classification

**Approaches**
- **One-vs-Rest (OvR)**: one sigmoid/binary model per class; pick highest probability.
- **Multinomial (e.g., Softmax)**: single model returns probability vector `[P(y=0|x), P(y=1|x), …]`; pick argmax.

**Evaluation**
- Confusion matrix extended to multiple classes.
- Accuracy/precision/recall/F1 can be computed **per class** and aggregated (macro/micro/weighted).

---

## Clustering

**K-Means (example)**
1. Vectorize features (n-D space).
2. Choose **k** clusters; initialize **centroids**.
3. Assign points to nearest centroid.
4. Recompute centroids; reassign; repeat until stable or max iterations.

**Evaluation (no labels)**
- **Avg/Max distance to centroid**, **avg distance to other centroids**.
- **Silhouette** (−1 to 1; higher is better separation).

---

## Deep Learning (Neural Networks)

**What**
- **Artificial Neural Networks (ANNs)** with multiple layers (**DNNs**) solve regression/classification and power **NLP**/**CV**.

**How**
- Forward pass computes **ŷ**; loss compares **ŷ** vs **y**.
- **Backpropagation + optimizer** (e.g., gradient descent) updates **weights (w)** to reduce loss over **epochs**.

**Example (Penguins)**
- Input features: bill length/depth, flipper length, weight → network → **softmax** output `[p_Adelie, p_Gentoo, p_Chinstrap]`; pick highest probability.

---
