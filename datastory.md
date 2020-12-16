---
layout: page
title: Datastory
---

# Part 2
## Predict different survey targets
We predict the different survey values from the census data from 2000. We create a data array of the results for the different models. We calculate and report the $R^2$ scores for the different targets of the survey data and the different models.
We use the different models:
- Linear models
    - Linear regression: A simple linear regression is always a good baseline to see how the more advance machine learning techniques perform compared to the linear regressor.
    - Ridge regression: The Ridge regression use a L2 regularization term that penalizes the size of the coefficients and 'shrinks' these coefficients. Therefore, the model becomes more robust to collinearity.
- Non-linear models
    - DecisionTreeRegressor: A decision tree is build from the data and can have a very high depht.
    - GradientBoostingRegressor: Builds a certain number of weak learners (decsion trees with a shallow depht) and combine these in order to have a strong prediction. 
    - MLPRegressor: The multi-layer perceptron is a fully connected neural network that is able to build complex non linear models.

We calculate the $$R^2$$ score with a standardized and non-standardized dataset. Different algorithms improve in performance when standardizing and others have in general no need for standardization like the Gradient Boosting Regressor. We will see how the different algorithms behave in the following.

{% include figures/taskb_R2_scores_heatmap.html %}

When we compare the results  between the different average performances of our five regressors, we notice that the overall performance of all the regressors are not very good. The Linear and Ridge Regressor seem to have the best performance than the non-linear regressor. This is mostly explained by the fact that they don't have a very bad performance on variables that are difficult to predict. The non-linear techniques perform far better than the non-linear ones on some variables, where a good prediction seems  to be feasible. We will in the following analysis focus on the variables for which we can make a meaningful prediction and compare the different models for these. 

{% include figures/taskb_mean_r2_performance_all_predictions.html %}

We computed the mean $R^2$ score for all the variables over all the models and choose the five variables for which we can potentially make a meaningful prediction. 

Let's graph the R^2 score for the different models in a barplot again.

{% include figures/mean_r2_performance_chosen_predictions.html %}

This plot offers a very different picture on the performance of the different regressors. The Linear and the Ridge regression perform very similarly. The non-linear methods perform better than the linear techniques. The gradient boosting method is able to deliver the best results. For the Gradient Boosting, the standardization does not improve the score and we will just use the non standardized data in the following.

## Analysis for the selected variables
In the following, we will focus on the selected variables and analyse if we can predict them from the census data from 2000. We will use the Gradient Bosting Regressor as its overall performance was the best. We will also analyse what features are the most important in the prediction of the variable. We will plot the feature importances for the different predictions. We still need to be careful with high cardinality features of the impurity-based feature importance scores (high cardinality tends to have a higher feature importance score). Moreover, the impurity-based feature importance score will not be meaningful if we have correlated features. One correlated feature might have a very high score while the other one is very small. Therefore, we use the permutation feature score on test data to reanalyse the feature importances. The idea of the permutation feature score is to test the influence of a feature immediatly on the model. For unimportant features, the have little effects on the predictive power of the model and the permutation of important feature should have a big negative influence on the predictive power. (Source: We have used some explainations from https://blog.datadive.net/selecting-good-features-part-iii-random-forests/, https://scikit-learn.org/stable/modules/permutation_importance.html#permutation-importance)

## Overall conclusions
We see that in general the census data from 2000 does not allow us to construct a good model to predict the different features of the survey in 2005. This is not astonishing as the census data has less granularity than the survey data that is specific to every household. The cenus data mostly reports averages over an area that is composed of many households. There are still a few values, where some algorithms are able to make decent predictions. 