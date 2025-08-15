# Phase 1 – Bridge Java to AI (Months 1–2)

**Goal:**  
Quickly build Python proficiency for AI model development, while leveraging your Java expertise for integration into enterprise systems.

---

## Focus Areas (Detailed)

### 1. Python Basics
- Learn syntax, control structures, functions, and error handling.  
- Understand **OOP in Python**: classes, inheritance, polymorphism.  
- Learn Python-specific features like list comprehensions, decorators, generators.  
- 📚 *Recommended:* [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) by Al Sweigart.

### 2. Data Handling
- **NumPy**: array creation, indexing, broadcasting, vectorized operations.  
- **Pandas**: Series, DataFrames, filtering, grouping, merging datasets.  
- **Matplotlib** & **Seaborn**: basic charts, histograms, scatter plots, heatmaps.  
- 📚 *Recommended:* *Python for Data Analysis* by Wes McKinney.

### 3. Machine Learning Basics (Scikit-learn)
- Regression: Linear Regression, Ridge, Lasso.  
- Classification: Logistic Regression, Decision Trees, Random Forest.  
- Clustering: K-means, DBSCAN.  
- Learn about **overfitting/underfitting** and **train/test splits**.

### 4. Model Evaluation Metrics
- **Classification:** accuracy, precision, recall, F1-score, confusion matrix.  
- **Regression:** RMSE, MAE, R².  
- Learn how to perform **cross-validation**.

### 5. Serving Models via REST APIs
- **FastAPI**: async endpoints, request/response models, validation with Pydantic.  
- **Flask**: simple endpoints, JSON responses.  
- Understand serialization of models with `pickle` or `joblib`.

---

## Open-Source Frameworks
- **Python** (core programming)
- **NumPy** (numerical computing)
- **Pandas** (data manipulation)
- **Matplotlib** & **Seaborn** (visualization)
- **Scikit-learn** (machine learning)
- **FastAPI** or **Flask** (API development)

---

## Mini Project (Practical)

**Title:** *Sales Forecasting API with Java Integration*  

**Steps:**
1. **Data Collection:** Use a sample sales dataset (CSV from Kaggle or UCI ML Repository).  
2. **Model Training:** Train a regression model in Python using Scikit-learn.  
3. **Model Serving:** Deploy via a FastAPI endpoint (`/predict`) returning JSON.  
4. **Java Integration:**  
   - Build a **Spring Boot** service that sends data to the Python API and consumes predictions.  
   - Use **RestTemplate** or **WebClient** for API calls.
5. **Version Control:** Push to GitHub with a clear README, API documentation, and setup guide.

---

## Expected Outcome
By the end of Phase 1, you will:
- Be able to read, preprocess, and visualize datasets in Python.
- Train and evaluate basic ML models.
- Deploy models as REST APIs.
- Integrate AI services into Java applications.
