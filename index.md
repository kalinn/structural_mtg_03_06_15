--- 
title       : Multivariate Pattern Analysis and Confounding in Neuroimaging
subtitle    : 
author      : Kristin Linn
job         : March 6, 2015
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : prettify      # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

## Roadmap
========================================================

- Motivate and define the goals of Multivariate Pattern Analysis (MVPA)
- Demonstrate the consequences of confounding
- Inverse probability weighting to address confounding in MVPA
- Simulated and real data examples

--- 

## How does disease affect brain structure?
========================================================

### "Mass-univariate" approaches test for group differences at each voxel 
- e.g., Statistical Parametric Mapping, Voxel-based morphometry

<img src="assets/fig/unnamed-chunk-1-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" style="display: block; margin: auto;" />

--- 

## Multivariate Pattern Analysis (MVPA)
========================================================

### Any tool that can identify joint disease patterns across the brain.

- SVM 
- Logistic regression
- Scalar on image regression

### __MVPA goals__:

1. Study structural patterns in the brain that characterize disease
2. Develop image-based disease biomarkers

--- &twocoltop

## Support Vector Machine (SVM)
========================================================

*** {name: left}

<img src="assets/fig/unnamed-chunk-2-1.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

*** {name: right}

<img src="assets/fig/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

*** {name: bottom}

### In neuroimaging each point is a brain, classes are cases/controls

^http://docs.opencv.org/doc/tutorials/ml/introduction_to_svm/introduction_to_svm.html

--- 

## MVPA using the SVM
========================================================

### The SVM returns a vector of <span style="color:red">weights</span>, one per voxel/region 
- Weights indicate relative contribution to the discrimative rule
  
<img src="assets/fig/unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />

--- 

## Marginal vs Joint MVPA
========================================================

### Non-imaging variables can be informative about disease process

- Investigators must decide whether or not to include non-image variables as features 

- We define <span style="color:red">Marginal MVPA</span> to be any MVPA that uses only the image

- We define <span style="color:red">Joint MVPA</span> to be any MVPA that uses image and non-image variables



--- 



## Confounding in Neuroimaging
========================================================

### A <span style="color:red">confounder</span> is a non-image variable that affects both disease and image
- e.g., In Alzheimer's disease <span style="color:red">age</span> is associated with diagnosis and atrophy

### Recall the goals of MVPA:
1. Study structural patterns in the brain that characterize disease
2. Develop image-based disease biomarkers

### It is important the analysis reflect the target MVPA goal!
- This talk focuses on Goal 1 with brief comments about Goal 2


--- 

## Confounding and MVPA Goal 1
========================================================

### Goal: estimate the stuctural pattern that characterizes disease
- Ideally, observe each patient with and without disease
- In causal inference terms: want to see both potential outcomes

### Several problems: 
- Can't observe both potential outcomes
- Can't randomize disease
- Matching on all confounders may not be feasible
- Not addressing confounding may lead to a biased estimate of the disease pattern

---

## Confounding Toy Example
========================================================

<img src="assets/fig/unnamed-chunk-5-1.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />

--- &twocoltop

## Voxel-wise "adjustment"
========================================================

*** {name: left}

<center>"Adjusted SVM"</center>

*** {name: right}

<center>"CN-Adjusted SVM"</center>

*** {name: bottom}
<img src="assets/fig/unnamed-chunk-6-1.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" style="display: block; margin: auto;" />

### Note:
- Both methods are "conditional" on A
- Changes interpretation of features, i.e., now <span style="color:red">joint</span> with A

--- &twocol

## Inverse Probability Weighting
========================================================

<img src="assets/fig/unnamed-chunk-7-1.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" style="display: block; margin: auto;" />

---

## Inverse Probability Weighting
========================================================

<img src="assets/fig/unnamed-chunk-8-1.png" title="plot of chunk unnamed-chunk-8" alt="plot of chunk unnamed-chunk-8" style="display: block; margin: auto;" />

---

## IPW-MVPA
========================================================

### <span style="color:red">Idea</span>: perform MVPA in the pseudo-population created by IPW 

- Define $W_{i} = I(D_{i}=1)\mbox{pr} (D_{i}=1 \mid {\boldsymbol A}_{i}) + I(D_{i}=0)\mbox{pr} (D_{i}=0 \mid {\boldsymbol A}_{i})$

- Weight each subject by $W_{i}^{-1}$ 

### Usually IP weights are unknown and must be estimated 
- Logistic regression: 
$$
\mbox{logit}[ \mbox{pr} (D=1 \mid {\boldsymbol A})] = \beta_{0} + \beta_{1}^{T}{\boldsymbol A}
$$
- Obtain $\widehat{W}_{i}$ by plugging in 
$$\widehat{\mbox{pr}} (D_{i}=1 \mid {\boldsymbol A}_{i}) = \widehat{\beta}_{0} + \widehat{\beta}_{1}^{T}{\boldsymbol A}
$$


--- &twocol

## IPW-SVM
========================================================

### Soft-margin SVM

- Weight the slack variables $\xi_{i}$ by IP weights
- Extension of class imbalance weighting 

*** {name: left}

<img src="assets/fig/unnamed-chunk-9-1.png" title="plot of chunk unnamed-chunk-9" alt="plot of chunk unnamed-chunk-9" style="display: block; margin: auto;" />

*** {name: right}

<img src="assets/fig/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" style="display: block; margin: auto;" />


--- 

## IPW-SVM in R
========================================================

### To our knowledge, no SVM package allows subject-level weights 

- Approximate IPW-SVM:
  1. Estimate the IP weights and define $\widehat{T}_{i}$ to be the estimated weight for subject $i$ rounded to the nearest integer.
  2. Create a new dataset that contains $\widehat{T}_{i}$ copies of subject $i$'s data vector, $(Y_{i}, X_{i}, A_{i})$.
  3. Train the SVM in the new dataset.

---

## Simulations
========================================================

<img src="assets/fig/unnamed-chunk-11-1.png" title="plot of chunk unnamed-chunk-11" alt="plot of chunk unnamed-chunk-11" style="display: block; margin: auto;" />

- Generate 2,000 samples, save half for testing
- Learn SVM on 1,000 training samples to get "true" weights
- Take a biased subsample (n=200 per class) from the 1,000 training samples
- Compare methods on the biased subsample  

---

## Marginal MVPA Results
========================================================

<img src="assets/fig/unnamed-chunk-12-1.png" title="plot of chunk unnamed-chunk-12" alt="plot of chunk unnamed-chunk-12" style="display: block; margin: auto;" />

--- 

## Joint MVPA Results
========================================================

<img src="assets/fig/unnamed-chunk-13-1.png" title="plot of chunk unnamed-chunk-13" alt="plot of chunk unnamed-chunk-13" style="display: block; margin: auto;" />

---

## Joint MVPA Results 
========================================================

- $X_{1} = 5 - 3D + \epsilon_{1}$
- $X_{2} = -3A - 4D + 3AD + \epsilon_{2}$

<img src="assets/fig/unnamed-chunk-14-1.png" title="plot of chunk unnamed-chunk-14" alt="plot of chunk unnamed-chunk-14" style="display: block; margin: auto;" />

--- 

## Caveat
========================================================

- Can't weight slack variables when the data are linearly separable!
- Data become more separable as $p$ increases and are always perfectly linearly separable when $p > n$. 

---

## Caveat
========================================================

<img src="assets/fig/unnamed-chunk-15-1.png" title="plot of chunk unnamed-chunk-15" alt="plot of chunk unnamed-chunk-15" style="display: block; margin: auto;" />

---

## Current solution: PCA-IPW-SVM
========================================================

  1. Estimate the IP weights and create the new dataset that contains subject-level copies
  2. Do PCA on the image features in the new dataset
  3. Train SVM on the first several PC scores

---

## Results from ADNI data
========================================================

- 137 ROI volumes
- Used an age-matched sample to estimate the "true" weights for classifying Alzheimer's vs. controls
- Performed PCA-IPW-SVM on a biased subsample of the full age-matched sample

<img src="assets/fig/unnamed-chunk-16-1.png" title="plot of chunk unnamed-chunk-16" alt="plot of chunk unnamed-chunk-16" style="display: block; margin: auto;" />

---

## Thank you!
========================================================

### Questions?
