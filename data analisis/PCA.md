Principal Component Analysis (PCA) is essentially a "data compressor." It takes a dataset with many variables and simplifies it by finding new, uncorrelated variables called **Principal Components** that capture the most information (variance).

Think of it like taking a 3D shadow puppet: you’re projecting a complex shape onto a flat 2D wall while trying to make sure the shadow still looks as much like the original object as possible.

---

## 1. The Core Concept

The goal is to transform our original coordinates into a new coordinate system where:

1. The first axis (**PC1**) lies along the direction of the greatest spread in the data.
    
2. The second axis (**PC2**) is perpendicular to the first and captures the next highest spread.
    

---

## 2. A Mathematical Example

Let’s say we have a tiny 2D dataset representing the height and weight of three people (simplified for the math).

|**Person**|**Height (x1​)**|**Weight (x2​)**|
|---|---|---|
|A|2|3|
|B|4|5|
|C|6|7|

### Step 1: Mean Centering

We subtract the mean of each column so the data is centered at the origin (0,0).

- Mean of $x_1 = 4$; Mean of $x_2 = 5$.
    
- Adjusted Data: $A(-2, -2), B(0, 0), C(2, 2)$.
    

### Step 2: Calculate the Covariance Matrix

The covariance matrix $C$ shows how the variables move together. For our centered data $X$:

$$C = \frac{1}{n-1} X^T X$$

For this specific set, the matrix looks like this:

$$C = \begin{pmatrix} 4 & 4 \\ 4 & 4 \end{pmatrix}$$

### Step 3: Find Eigenvalues and Eigenvectors

This is where the magic happens. We solve the characteristic equation $det(C - \lambda I) = 0$.

- **Eigenvalues ($\lambda$):** These tell us the "strength" of each component. Here, $\lambda_1 = 8$ and $\lambda_2 = 0$.
    
- **Eigenvectors ($v$):** These tell us the "direction." For $\lambda_1 = 8$, the eigenvector is $v_1 = [0.707, 0.707]$.
    

### Step 4: Project the Data

Since $\lambda_1$ is 8 and $\lambda_2$ is 0, the first principal component explains 100% of the variance. We can drop PC2 entirely. Our new 1D data point for Person C would be:

$$(2 \times 0.707) + (2 \times 0.707) = 2.828$$

---

## 3. Why do we do this?

- **Noise Reduction:** Small eigenvalues usually represent random noise. By dropping them, we clean the data.
    
- **Visualization:** We can’t "see" 10D data, but we can see a 2D PCA plot of it.
    
- **Efficiency:** Models run much faster on 50 principal components than on 5,000 raw features.
    

> **Keep in mind:** PCA assumes linear relationships. If your data is shaped like a horseshoe or a circle, PCA might struggle to represent it accurately without some extra help (like Kernel PCA).


### example : 

```python
import numpy as np

# Original Data
X = np.array([[4, 5],
              [6, 7],
              [8, 0]])

# Step 1: Mean Centering
mean_X = np.mean(X, axis=0)
X_centered = X - mean_X

# Step 2: Covariance Matrix
# np.cov treats columns as variables, rows as observations
# bias=True divides by n, False (default) divides by n-1
cov_matrix = np.cov(X_centered, rowvar=False)

# Step 3: Eigenvalues and Eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# Sorting to get PC1 and PC2
idx = eigenvalues.argsort()[::-1]
eigenvalues = eigenvalues[idx]
eigenvectors = eigenvectors[:, idx]

print("Original Data:\n", X)
print("\nMean of columns:", mean_X)
print("\nCentered Data:\n", X_centered)
print("\nCovariance Matrix:\n", cov_matrix)
print("\nEigenvalues:", eigenvalues)
print("\nEigenvectors (as columns):\n", eigenvectors)


```
```R
> X <- data.frame(
  X1 = c(4, 6, 7),
  X2 = c(5, 7, 0)
)
> X
  X1 X2
1  4  5
2  6  7
3  7  0
> pca <- prcomp(X, scale. = FALSE)
> pca
Standard deviations (1, .., p=2):
[1] 3.712931 1.243978

Rotation (n x k) = (2 x 2):
          PC1        PC2
X1  0.2534013 -0.9673612
X2 -0.9673612 -0.2534013
> pca$center
      X1       X2
5.666667 4.000000
> pca$rotation
          PC1        PC2
X1  0.2534013 -0.9673612
X2 -0.9673612 -0.2534013
> pca$sdev^2
[1] 13.78585  1.54748
> pca$x
           PC1        PC2
[1,] -1.389697  1.3588674
[2,] -2.817617 -1.0826577
[3,]  4.207313 -0.2762097
```


Let’s break down the Principal Component Analysis (PCA) process step-by-step using your data.

### 1. Our Dataset

We have two variables, $x_1$ and $x_2$, representing three data points:

$$X = \begin{pmatrix} 4 & 5 \\ 6 & 7 \\ 8 & 0 \end{pmatrix}$$

---

### Step 1: Mean Centering

PCA requires the data to be centered around zero. We calculate the mean of each variable and subtract it from the original values.

- Mean of $x_1$ ($\bar{x}_1$): $\frac{4+6+8}{3} = 6$
    
- Mean of $x_2$ ($\bar{x}_2$): $\frac{5+7+0}{3} = 4$
    

**Centered Data Matrix ($X_{centered}$):**

$$X_{centered} = \begin{pmatrix} 4-6 & 5-4 \\ 6-6 & 7-4 \\ 8-6 & 0-4 \end{pmatrix} = \begin{pmatrix} -2 & 1 \\ 0 & 3 \\ 2 & -4 \end{pmatrix}$$

---

### Step 2: Calculate the Covariance Matrix

The covariance matrix $C$ shows how $x_1$ and $x_2$ vary together. We use the formula $C = \frac{1}{n-1} X^T X$, where $n=3$:

$$C = \frac{1}{3-1} \begin{pmatrix} -2 & 0 & 2 \\ 1 & 3 & -4 \end{pmatrix} \begin{pmatrix} -2 & 1 \\ 0 & 3 \\ 2 & -4 \end{pmatrix}$$

$$C = \frac{1}{2} \begin{pmatrix} (-2)^2 + 0^2 + 2^2 & (-2 \times 1) + (0 \times 3) + (2 \times -4) \\ (1 \times -2) + (3 \times 0) + (-4 \times 2) & 1^2 + 3^2 + (-4)^2 \end{pmatrix}$$

$$C = \frac{1}{2} \begin{pmatrix} 8 & -10 \\ -10 & 26 \end{pmatrix} = \begin{pmatrix} 4 & -5 \\ -5 & 13 \end{pmatrix}$$

---

### Step 3: Find Eigenvalues ($\lambda$)

We solve the characteristic equation $\det(C - \lambda I) = 0$ to find the "strength" of the new components:

$$\det \begin{pmatrix} 4 - \lambda & -5 \\ -5 & 13 - \lambda \end{pmatrix} = 0$$

$$(4 - \lambda)(13 - \lambda) - (-5)(-5) = 0$$

$$\lambda^2 - 17\lambda + 52 - 25 = 0 \Rightarrow \lambda^2 - 17\lambda + 27 = 0$$

Using the quadratic formula:

$$\lambda_1 = 15.227 \quad \text{and} \quad \lambda_2 = 1.773$$

- **Interpretation:** $PC1$ (the first component) explains $\frac{15.227}{15.227 + 1.773} \approx 89.6\%$ of the variance.
    

---

### Step 4: Find Eigenvectors ($v$)

Now we find the direction of the first component by solving $(C - \lambda_1 I)v_1 = 0$ for $\lambda_1 = 15.227$:

$$\begin{pmatrix} 4 - 15.227 & -5 \\ -5 & 13 - 15.227 \end{pmatrix} \begin{pmatrix} v_{11} \\ v_{12} \end{pmatrix} = 0$$

$$-11.227 v_{11} - 5 v_{12} = 0 \implies v_{12} = -2.245 v_{11}$$

After normalizing (so the vector has a length of 1), we get:

$$v_1 = [0.407, -0.914]$$

This is the direction of the "main line" that captures the most spread in your data.

---

### Step 5: Project Data onto Principal Components

To get the new values (scores), we multiply the centered data by the eigenvector $v_1$.

For our third point ($2, -4$):

$$Score = (2 \times 0.407) + (-4 \times -0.914) = 0.814 + 3.656 = 4.47$$

**Summary of transformation:**

Instead of tracking two coordinates ($x_1, x_2$), you can now describe most of your data using just one coordinate (the PC1 score). Your 2D data is now effectively 1D.