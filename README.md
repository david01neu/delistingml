# Prediction of Delisting using a Machine Learning Ensemble
David Neuhäusler, Jungyeon Yoon, Richard A. Levine, Juanjuan Fan

Thesis project for MSc in Statistics program at [San Diego State University](https://www.sdsu.edu/)

## Abstract
Delisting is the removal of a listed security from a stock exchange. Consequences of delisting are significant for the company and the shareholders. We develop a data-driven model to predict delisting using a wide range of features including company financial ratios and macroeconomic indicators. To our knowledge, there is no published work yet focusing on the prediction of delistings in the U.S. Our analysis is based on a data set with around 390,000 rows of quarterly financial ratios from numerous listed companies in the U.S. since 1970. The resulting model combines predictions of five base machine learning methods (logistic regression, random forest, gradient boosting, support vector machine and neural network), which makes the resulting ensemble model a powerful predictor. Performance metrics evaluate strengths and weaknesses of the five base learners as well as the ensemble model. Our ensemble model reaches a prediction accuracy of 83.6%, outperforming every base learner. Among the base learners, the random forest model shows the best performance. Furthermore, we find that the price-earnings ratio is the most informative from a forecasting perspective along with cash ratio, return on equity and inflation rate.

## R code files

Formatted output including all files available on https://thesis.edv-neuhaeusler.de

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