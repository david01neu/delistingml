# Prediction of Delisting using a Machine Learning Ensemble
David Neuhäusler<sup>a</sup>, Jungyeon Yoon<sup>b</sup>, Richard A. Levine<sup>a</sup>, Juanjuan Fan<sup>a</sup>

<sup>a</sup> San Diego State University, San Diego, CA, USA\
<sup>b</sup> Korea Banking Institute, Seoul, South Korea

Thesis project for MSc in Statistics program at [San Diego State University](https://www.sdsu.edu/)

## Abstract
Delisting is the removal of a listed security from a stock exchange. Consequences of delisting are
significant for the company and its shareholders. We develop a data-driven model to predict delisting
using a wide range of features, including company financial ratios and macroeconomic indicators.
To our knowledge, there is no published work yet focusing on the prediction of delistings in the
U.S. Our analysis is based on a data set with quarterly financial ratios of 9,424 companies in the
U.S. from 1970 to 2022. The resulting model combines the predictions of five base machine learning
methods (logistic regression, random forest, gradient boosting, support vector machine and neural
network), which makes the resulting ensemble model a powerful predictor. Performance metrics
evaluate strengths and weaknesses of the five base learners as well as the ensemble model. Our
ensemble model reaches a prediction accuracy of 83.6% on test data. Among the base learners,
the random forest model shows the best performance. Furthermore, we nd that the price-earnings
ratio is the most informative from a forecasting perspective along with cash ratio, return on equity
and in inflation rate.

## R code files

Formatted HTML output of the R Markdown files in this repository available under https://david01neu.github.io/delistingml/.

| Filename                   | Content and model packages                               |
| -------------------------- | -------------------------------------------------------- |
| index.Rmd                  | Load initial data and set constants                      |
| 01_eda.Rmd                 | Exploratory data analysis                                |
| 02_data_preparation.Rmd    | Create predictior data set for training and testing      |
| 03_model_functions.Rmd     | Custom helper functions for model building with `caret`  |
| 04_random_forest.Rmd       | Random forest with `randomForestSRC`                     |
| 05_logistic_regression.Rmd | Logistic regression with `glmnet`                        |
| 06_svm.Rmd                 | Support vector machine with `kernlab`                    |
| 07_neuralnet.Rmd           | Neural network with `keras`                              |
| 08_boosting.Rmd            | Gradient boosting machine with `gbm`                     |
| 09_ensemble.Rmd            | Ensemble model with `caretEnsemble`                      |

## Web Appendix

### Data: Delisting, Financial and Macroeconomic Data
We include U.S. common stocks identified by the CRSP share type code 10 and 11. 
Investment funds and trusts with SIC 6722, 6726, 6798, and 6799 are excluded from the analysis as in Fama and French (2004). The CRSP database provides delisting codes for every share which are summarized in the table below. Following the delisting definition, shares with a delisting code greater than or equal to 400 are considered delisted. In particular, this definition includes voluntary and involuntary delistings.

| Delisting code range | Delisting category            | Example                                                       |
|----------------------|-------------------------------|---------------------------------------------------------------|
| 100 - 199            | Active                        | Share still active and traded                                 |
| 200 - 299            | Mergers                       | Share merged and shareholders receive other property          |
| 300 - 399            | Exchanges                     | Issue exchanged for cash or other property                    |
| 400 - 499            | Liquidations                  | Share stopped trading as a result of company liquidation      |
| 500 - 599            | Dropped                       | Share delisted because of insufficient number of shareholders |
| 600 - 699            | Expirations                   | Expired warrant or right                                      |
| 900 - 999            | Domestics that became Foreign | U.S. share becomes foreign but remains on stock exchange      |

### Data: Exploratory Data Analysis

![Pearson's correlation matrix for 19 observed financial ratios](https://github.com/david01neu/delistingml/blob/dev/images/220_correlation.svg?raw=true)

*Pearson's correlation matrix for 19 observed financial ratios*

The figure above displays Pearson's correlation coefficients for every pair of financial ratios. Most pairs show no linear correlation, a few have a slightly negative correlation and some have a significant positive correlation. The operative profit margin ratios, `opmad` and `opmbd`, as well as the liquidity ratios, `curr_ratio` and `quick_ratio`, are perfectly positively correlated. 
Variables, `opmbd` and `quick_ratio`, are removed from further analysis. 
The correlation coefficient between the debt ratios `debt_assets` and `debt_at` is $0.76$. The remaining correlation coefficients are all lower than $0.64$.

![Histograms of exemplary financial ratios and one-year ahead delisting frequency in the data set described in section 2.1. The gray bars represent the total count.](https://github.com/david01neu/delistingml/blob/dev/images/230_histogram_features.svg?raw=true)
*Histograms of exemplary financial ratios and one-year ahead delisting frequency in the data set described in section 2.1. The gray bars represent the total count.*

The figure above presents histograms for four financial ratio features. Each quarterly data record in the data set from section 2 is counted as one observation. Since financial ratios tend to explode if the denominator is small, we winsorize all displayed features by computing the 5% and 95% quantiles and by setting all values below or above these quantiles to these threshold values, respectively. Hence, the left and right bars contain at least 5% of the observations. The gray bars show the frequencies of the quarterly financial ratio data. 
The orange points display the one-year ahead delisting frequency. The smoothed line is a LOESS smoother (Cleveland (1979)) of the orange dots. Histograms of different financial ratios within the same category (see below) show similar behavior in terms of the observed one-year relative delisting frequency.

*  __Capitalization__ and __Solvency__: `capital_ratio`, `de_ratio`, `debt_assets`, `debt_at`: The higher the ratio of debt to assets or liabilities, the higher the relative delisting frequency in the subsequent year (top left panel).
* __Efficiency__: `at_turn`, `inv_turn`, `rect_turn`, `pay_turn`: These efficiency ratios but `pay_turn` seem to have no observable marginal effect on delisting. However, if the ratio between cost of goods sold plus the change in inventories and accounts payable gets larger, the observed delisting frequency for the subsequent year decreases.
* __Liquidity__: `curr_ratio`, `cash_ratio`, `quick_ratio`: The ratio between cash plus short-term investments (`cash_ratio`) seems to have no marginal predictive power for delisting. However, if the ratio of current assets to current liabilities (`curr_ratio`, `quick_ratio`) increases, the relative frequency of delistings in the subsequent year decreases  (top right panel).
* __Profitability__: `roe`, `roa`, `aftret_eq`, `aftret_equity`, `gpm`, `opmad`, `opmbd`, : As profitability increases, the probability of delisting decreases. The denominators of these profitability ratios contain key assets or sales figures which are positive. So if the income or sales figures in the numerator of the ratios are negative, the company's situation is on average more problematic and hence higher chance of delisting within the year (bottom left panel).
* __Valuation__: `pe_exi`: For positive price-to-earnings ratios (per share), the observed delisting frequency in the subsequent year becomes smaller with increasing ratios. The higher the price-to-earnings ratio, the better a share value. If the price-to-earnings ratio is negative, then the earnings in the denominator of the ratio are negative. If the loss per share is relatively small, then the price-to-earnings ratio is $\ll -1$. Hence, decreasing ratios are associated with a lower relative delisting frequency. (bottom right panel)

### Empirical Results: Training of Base Learners
__Random forest__\
We perform a search of the three random forest hyperparameters - the number of trees $B$, the number of candidate splitting variables $m$ for each split and the minimum terminal node size $n_{min}$ for trees. Hastie et al. (2009) suggest using the out-of-bag (OOB) samples for validation as the OOB error estimates are almost identical to those produced by $k$-fold cross-validation. Since the OOB samples are a byproduct of bootstrap samples used for tree construction, their evaluation is computationally efficient in comparison to $k$-fold cross-validation. This allows us to evaluate many hyperparameter combinations.\
The upper panels of the figure below present four performance metrics of random forests for different minimum terminal node sizes $n_{min}$ with $B = 1000$ and $m = 17$ held constant. For all four performance metrics, higher values signify better performance. 
The solid red lines show the metrics based on in-bag data. Their values keep decreasing for increased $n_{min}$. If $n_{min}$ is near zero, all $B$ trees are nearly fully grown. This entails perfect prediction for in-bag data. However, these overfitted trees fail to predict on new data as the dashed blue lines based on OOB data show. For $n_{min} = 1$, recall and $F_1$ scores are near zero, indicating high false negative rates. That is, many `delisted` (positive) labels are predicted as `active` (negative) incorrectly. 

Note that precision, recall, and $F_1$ scores stabilize for $n_{min} \geq 5$. Therefore, $n_{min} = 5$ is chosen. Repeated fitting shows that $B = 1000$ trees produce stable random forests. 
The bottom panels of Figure \ref{fig:random_forest_hyperparameter_metrics} show that different choices of $m$ barely influence the performance metrics on the OOB data. Therefore, the default value $m = \frac{\text{number of features}}{3} = 12$ is chosen.

![Performance metrics for random forest models with different minimal terminal node sizes $n_{min}$ (top row) and different number of splitting variable candidates $m$ (bottom row). Metrics for in-bag (INB) samples in solid red and for out-of-bag (OOB) samples in dashed blue.](https://github.com/david01neu/delistingml/blob/dev/images/4_random_forest_hyperparameter_metrics.svg?raw=true)
*Performance metrics for random forest models with different minimal terminal node sizes (top row) and different number of splitting variable candidates (bottom row). Metrics for in-bag (INB) samples in solid red and for out-of-bag (OOB) samples in dashed blue.*

__Neural network__\
Neural networks are a statistical tool with several hyperparameters which influence the form of the network and the behavior of the fitting algorithm. The number of nodes in the input layer is predetermined by the number of features which is $36$ in our application. The number of nodes in the output layer is one for binary classification. The aim is to construct a neural net with three hidden layers. After some manual discovery of different node sizes, a grid search evaluates options for the decreasing number of nodes for subsequent hidden layers via $10$-fold cross-validation as listed in the table below. The activation function for hidden layers is set to the ReLu function which is the default recommendation. The sigmoid function activates the output layer.\
We employ dropout, a regularization method that simulates different network architectures by randomly dropping out nodes during training. During fitting, some fractions $p$ of nodes in a hidden layer are randomly dropped, along with all their incoming and outgoing connections. In this manner, dropout can break up situations where network layers co-adapt to correct mistakes from prior layers. Srivastava et al. (2014) find that "the activations of the hidden units become sparse, even when no sparsity inducing regularizers are present". $10$-fold cross-validation suggests a dropout rate $p = 0.1$. The fitted weights of the network will be larger than without dropout of nodes. Therefore, the weights are scaled by $1-p$. The stochastic optimization algorithm Adam introduced by Kingma and Lei Ba (2015) is used to fit the neural network. We observe that a batch size of $16$ and $100$ epochs reduce the loss reliably. The default learning rate is $0.001$.

| Hyperparameter                             | Combinations                                     | Hyperparameter                                         | Combinations                                |
|--------------------------------------------|--------------------------------------------------|--------------------------------------------------------|---------------------------------------------|
|**Random forest**                           |                                                  | **Logistic regression**
| $B$                                        | $\{500,1000,2000\}$                              | $\alpha$                                               | $\{0,0.1,\dots,0.7,0.8,0.9,1\}$             |
| $m$                                        | $\{7,12,17,22,27\}$                              | $\lambda$                                              | $\{10^{-1},10^{-2},\dots,10^{-5},10^{-6}\}$ |
| $n_{min}$                                  | $\{1,2,\dots,15,20,\dots,100\}$                  | **Gradient Boosting Machine**
|**Neural net**                              |                                                  | $B$                                              | $\{100,150,250,500\}$                                  |
| hidden layer $1$ (\#hl1)                   | $\{35,38\}$                                      | $d$                                                    | $\{3,5,8\}$                                 |
| hidden layer $2$ (\#hl2)                   | $\{20,25\}$                                      | $\lambda$                                              | $\{0.001,0.01,0.1,0.2\}$                    |
| hidden layer $3$ (\#hl3)                   | $\{12,18\}$                                      | $n_{min}$                                              | $\{5,10\}$                                  |
| activation function                        | $\{\text{relu}\}$                                | **Support Vector Machine**
| L2 regularization                          | $\{0,0.001,0.01\}$                               | $C$                                                    | $\{1,2,3,4\}$                               |
| dropout rate (dr)                          | $\{0.1,0.2\}$                                    | $d$                                                    | $\{0.001,0.01,1\}$                          |
| batch size                                 | $\{8,16,32,64\}$                                 | $s$                                                    | $\{0.1,0.01,0.001\}$                        

### Empirical Results: Meta model evaluation
The table below presents the correlation between predicted delisting probabilities from the base and ensemble models. Among the base models, predictions from random forest and neural net are most strongly correlated, followed by predictions from random forest and boosting. Multi-correlation between base prediction scores affects the interpretation of the coefficients of the meta-model, but not the precision of the meta-model. The predictions of the ensemble model are nearly perfectly linearly related to the random forest base model with a correlation coefficient of $0.99$. The correlation to gradient boosting and neural net models is $0.82$ and $0.84$, respectively, for the MCC-F1 variant. These three models show higher accuracy than the logistic regression and SVM.

|                     | random forest | logistic regression | SVM         | neural net  | boosting    | ensemble    |
|---------------------|---------------|---------------------|-------------|-------------|-------------|-------------|
| random forest       | 1.00          | 0.51 (0.52)         | 0.60 (0.63) | 0.85 (0.85) | 0.83 (0.83) | 0.99 (0.99) |
| logistic regression | 0.51 (0.52)   | 1.00                | 0.50 (0.46) | 0.49 (0.56) | 0.68 (0.69) | 0.44 (0.46) |
| SVM                 | 0.60 (0.63)   | 0.50 (0.46)         | 1.00        | 0.65 (0.68) | 0.49 (0.65) | 0.60 (0.65) |
| neural net          | 0.85 (0.85)   | 0.49 (0.56)         | 0.65 (0.68) | 1.00        | 0.75 (0.79) | 0.84 (0.85) |
| boosting            | 0.83 (0.83)   | 0.68 (0.69)         | 0.65 (0.65) | 0.75 (0.79) | 1.00        | 0.82 (0.83) |
| ensemble            | 0.99 (0.99)   | 0.44 (0.46)         | 0.60 (0.65) | 0.84 (0.85) | 0.82 (0.83) | 1.00        |

The table below presents the estimated coefficients and standard errors of the meta-model, along with the $95%$ confidence intervals for the regression coefficients. Although the confidence intervals for the SVM and neural net predictors contain zero, they may contribute to reducing the model error together with the significant collinear variables.

| predictor           | Estimate          | Std. Error      | 2.5% CI           | 97.5% CI         |
|---------------------|-------------------|-----------------|-------------------|-------------------|
| (Intercept)         | 2.7275 (2.8708)   | 0.2456 (0.2286) | 2.2465 (2.4238)   | 3.2097 (3.3205)   |
| random forest       | -5.6041 (-5.3360) | 0.6151 (0.6126) | -6.8121 (-6.5385) | -4.4000 (-4.1364) |
| logistic regression | 1.7449 (1.3777)   | 0.4582 (0.4050) | 0.8530 (0.5889)   | 2.6496 (2.1769)   |
| SVM                 | 0.1236 (0.2306)   | 0.2730 (0.2623) | -0.4078 (-0.2799) | 0.6633 (0.7492)   |
| neural net          | -0.0266 (-0.2404) | 0.1725 (0.1646) | -0.3629 (-0.5617) | 0.3136 (0.0839)   |
| boosting            | -1.8881 (-1.8508) | 0.5547 (0.5567) | -2.9814 (-2.9480) | -0.8062 (-0.7647) |
