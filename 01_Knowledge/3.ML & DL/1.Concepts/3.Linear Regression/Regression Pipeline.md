# Regression Pipeline

## What is it?

A **regression pipeline** is the end-to-end sequence of steps for building a working [[Regression]] system — from raw data to a deployed prediction function.

---

## The Full Pipeline

```
1. Problem Definition
        ↓
2. Data Collection
        ↓
3. Exploratory Data Analysis (EDA)
        ↓
4. Data Preprocessing
        ↓
5. Feature Engineering
        ↓
6. Model Selection
        ↓
7. Training
        ↓
8. Evaluation
        ↓
9. Hyperparameter Tuning
        ↓
10. Final Evaluation on Test Set
        ↓
11. Deployment (Inference)
        ↓
12. Monitoring
```

---

## Step-by-Step Details

### 1. Problem Definition
- Is the output continuous? → Regression ✅
- What is the target variable $y$?
- What features $x$ are available at prediction time?
- What is the acceptable error? (RMSE of \$10,000 on house prices is different from RMSE of \$1,000)

### 2. Data Collection
- Gather raw data from source (CSV, database, API)
- Check: do you have enough examples? (Rule of thumb: $n \gg d$)

### 3. Exploratory Data Analysis (EDA)
- Distribution of $y$ (histogram): is it skewed? Log-transform?
- Correlation of each $x_j$ with $y$ (scatter plots, correlation matrix)
- Missing values: how many? Which features?
- Outliers: extreme values in $x$ or $y$?

**Useful correlations to compute:**
$$r_{x_j, y} = \frac{\sum_i (x_j^{(i)} - \bar{x}_j)(y^{(i)} - \bar{y})}{\sqrt{\sum_i(x_j^{(i)}-\bar{x}_j)^2 \cdot \sum_i(y^{(i)}-\bar{y})^2}}$$

High $|r|$ → strong linear relationship between $x_j$ and $y$.

### 4. Data Preprocessing
- **Handle missing values:** impute (mean/median) or drop
- **Handle outliers:** cap, remove, or use robust methods
- **Encode categoricals:** one-hot encoding
- **Train/Val/Test split** (do this FIRST, before any scaling)

### 5. Feature Engineering
- **Standardise/normalise** features (fit on train set only, apply to val/test)
  $$x_j' = \frac{x_j - \mu_j^{\text{train}}}{\sigma_j^{\text{train}}}$$
- **Log-transform** skewed features: $x_j' = \log(x_j + 1)$
- **Polynomial features** if non-linear relationship expected
- **Interaction terms:** $x_i \cdot x_j$ if interaction effect expected
- **Target transformation:** if $y$ is skewed, predict $\log(y)$ and exponentiate prediction

### 6. Model Selection
Start simple, then increase complexity as needed:
1. **Linear regression** — fast, interpretable, baseline
2. **Ridge / Lasso** — if multicollinearity or many features
3. **Polynomial regression** — if relationship is curved
4. **Random Forest / Gradient Boosting** — if non-linear and tabular
5. **Neural Network** — if very complex patterns or large data

### 7. Training
- Fit on training set only
- For linear regression: solve Normal Equations or run gradient descent
- Monitor training loss: should decrease steadily

### 8. Evaluation on Validation Set
Report:
$$\text{RMSE}_{\text{val}} = \sqrt{\frac{1}{n_{\text{val}}}\sum_{i \in \text{val}}(y^{(i)} - \hat{y}^{(i)})^2}$$
$$R^2_{\text{val}} = 1 - \frac{\sum_{i \in \text{val}}(y^{(i)}-\hat{y}^{(i)})^2}{\sum_{i \in \text{val}}(y^{(i)}-\bar{y}_{\text{val}})^2}$$

**Compare** train RMSE vs. val RMSE:
- Train RMSE ≈ Val RMSE (both high): underfitting → more complex model
- Train RMSE ≪ Val RMSE: overfitting → regularise or get more data

### 9. Hyperparameter Tuning
Grid search or random search over:
- Regularisation strength $\lambda$
- Polynomial degree $p$
- Learning rate $\eta$ (if using gradient descent)

Use validation set performance to select hyperparameters.

### 10. Final Test Set Evaluation
- Apply the best model (best hyperparameters, trained on train+val) to the **test set once**.
- Report final RMSE and R².
- Never tune further after seeing test set results.

### 11. Deployment
- Serialise model (save $\hat{\theta}$ and preprocessing statistics $\mu, \sigma$).
- Preprocessing must be **identical** at inference time.
- Wrap in an API endpoint.

### 12. Monitoring
- Track prediction distribution over time.
- Alert if distribution shift detected.
- Schedule periodic retraining.

---

## Common Pitfalls

| Pitfall | What happens | Fix |
|---|---|---|
| Scaling after splitting | Test statistics leak into train | Always split first, then fit scaler on train only |
| Looking at test set early | You overfit to the test set | Touch test set only once |
| No baseline | No reference point | Baseline: always predict $\bar{y}$ (gives R²=0) |
| Ignoring outliers | MSE blown up by a few examples | Inspect and handle outliers |
| Feature selection on all data | Data leakage | Feature selection must be inside CV loop |

---

## Connections

- [[Linear Regression]] — the core algorithm
- [[Feature Engineering]] — step 5
- [[Feature Scaling]] — normalisation/standardisation
- [[Gradient Descent]] — optimisation in step 7
- [[Overfitting]] — what step 8 diagnoses
- [[Regularization]] — solution to overfitting
- [[Inference]] — step 11

---

## One-line Summary

> The regression pipeline is the full end-to-end workflow from raw data to deployed predictions — its correctness (especially data leakage prevention, proper splitting, and identical preprocessing at train and inference time) determines whether model evaluation metrics are trustworthy.
