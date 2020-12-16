---
layout: page
title: Datastory
---

# Part 1
# Task A of our creative extension

Our task A consists in two main goals:
1. Compare the predictive performances of the different _outcome variables_ specified in Tables 4 and 6 of the paper to predict whether a household is from the treatment of control group.

To achieve this, we first build a baseline model that only includes control variables, i.e. features from the 2000 census, and features from the 2005 survey that are used either in their regression model 3 or as robustness checks in Table 7 of the paper. 
50 such features were selected.
The task consists in predicting whether an household is from the treatment or control group based on these features. The target feature is therefore the _intention-to-treat_ feature, referred as `dpisofirme` throughout the code.

Our expectation is that the baseline model should not perform well because the authors provided ample evidences that treatment and control households are well balanced over these control variables. As we will see, our models could actually exploit the differences in these control variables and perform very well on this baseline case.

Then we construct a model for each _outcome variable_, i.e. a model that includes the outcome variable as covariate besides the control variables. 
In the paper, the intention-to-treat variable is shown to be important to predict the outcome variables, hence we expect that including an outcome variable in the model should improve our predictions of the intention-to-treat variable. 
As you will see, we could not observe such an effect because the baseline predictions were already perfect.

2. Compare the feature importances of the _outcome variables_ with the regression coefficients from the paper

To achieve this, we perform a feature importance analysis on the models explained above and obtain an importance score for each outcome variable.
Then we compare first qualitatively the importance ranking of the outcome variables using our importance scores with the ranking based on the paper's regression coefficients.
It turns out that they agree relatively well.
We then compare them quantitatively and they agree pretty well too.

Reference to the [paper](https://www.aeaweb.org/articles?id=10.1257/pol.1.1.75):

Matias D. Cattaneo, Sebastian Galiani, Paul J. Gertler, Sebastian Martinez, and Rocio Titiunik. Housing, health, and happiness. American Economic Journal: Economic Policy, 1(1):75–105, February 2009.


## Comparison of the predictive performances

**Table caption**:

Prediction performances on the test set.
The left-most column represents the outcome variable that is included as covariate, besides the control variables. 
The _baseline_ variable means that only the control variables are included.
Columns 2 to 4 show the accuracy of each classifier.
Columns 5 to 7 show the F1-score of each classifier.
The 3 right-most columns represent the feature importance associated to the feature mentioned in the first column for each classifier.

{% include figures/classifiers_performances_table.html %}

Impressively, the baseline models, i.e. the models that only uses the control variables, achieve perfect prediction accuracy and F1-score on the test set using the random forest and XGBoost classifiers.

We did not expect this since the authors provide evidence that households selected in the treatment and control groups are similar in all control variables.

We notice that the results of logistic regressions are not bad as well (all models have an accuracy above 0.8) considering that chance accuracy is 0.51.
The fact that the logistic regression classifier still performs poorly compared to the random forest and XGBoost classifiers suggests that the relationship between the variables and the log odds of the treatment/control target is substantially non-linear.

Let's look at the distribution of the features that are considered the most important according to our classifiers.

Here are the features ranked as most important for each classifier:

{% include figures/coef_and_importance_comparison_table.html %}

We note that several features are considered important by several classifiers, such as `C_households`, `C_blocksdirtfloor`, `C_waterbath`.

Note that the feature importance scores from the logistic regression are calculated differently from the ones of the random forest and XGBoost classifiers and their values cannot be compared. We only look at the relative rankings of the features here.

Let's look at the distribution of all the most important features reported in the above table.


![image](/assets/images/distribution_of_most_important_features_task_A.png)

We see that the distribution for the treatment and control groups are indeed relatively different for these features.
It is remarkable that our classifiers are able to take advantage of these differences to such an extent as to perfectly predict the treatment/control labels on a held-out test set.

## Comparison of our feature importances with their _intention-to-treatment_ coefficients

We now compare the feature importances of the outcome variables obtained in the present task, with the coefficients of the _intention-to-treat_ variable that the authors obtain in their regression analyses.

We expect that our feature importances should loosely agree with their coefficients, meaning that the extent to which the _intention-to-treat_ variable is linearly related to a variable is proportional to the importance of this variable to predict the _intention-to-treat_ variable.

We will use the feature importances from the logistic regression because they range over positive as well as negative values, contrarily to the feature importance scores from random forest and XGBoost classifiers.

**Table caption:**

Features rankings.
The left-most column shows the features ranked by their paper's coefficients. The second column shows the value of the coefficient.
The two right-most columns show the features ranked by their feature importance scores that we computed above, and the value of the scores.

{% include figures/coef_and_importance_comparison_barplot.html %}

We first notice that the ranking of importances is almost the same in the paper and in our task:

The only differences are stated in the 3 following bullet points, in which the first rank mentioned corresponds to the paper's, and the second corresponds to our task's:
- `S_satislife` is ranked 4 instead of 3
- `S_shcementfloor` is ranked 5 instead of 6
- `S_satisfloor` is ranked 7 instead of 4

We also note that the sign of the coefficients agree with the sign of the feature importance scores.

We conclude that our feature importance scores agree qualitatively with the coefficients from the paper's regressions.

Here we show the paper's coefficients transformed in the range \[0, 1\], along with the importance scores that we obtained for the same features and transformed in the same way.

**Observations:**
- The scores are almost the same for 3 of the cement related features (`S_cementfloorbed`, `S_cementfloorkit`, `S_cementfloordin`) and for `S_cesds`, the depression score.
- `S_cementfloorbat`, `S_shcementfloor`, `S_satisfloor` have similar values in terms of the paper coefficients, and also in terms of the feature importance scores. But they all are considered relatively less important by the feature importance scores compared to the paper coefficients.
- The feature for which the two scores differ the most (on a relative scale) is `S_pss`, which represents the perceived stress of the mother of the households.

We conclude that our feature importance scores agree to some extent quantitatively with the paper coefficients. 
But we advise not to rely too much on them, and rather only use the qualitative information of the ranking, which is probably more robust.

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

We computed the mean $$R^2$$ score for all the variables over all the models and choose the five variables for which we can potentially make a meaningful prediction. 

Let's graph the R^2 score for the different models in a barplot again.

{% include figures/mean_r2_performance_chosen_predictions.html %}

This plot offers a very different picture on the performance of the different regressors. The Linear and the Ridge regression perform very similarly. The non-linear methods perform better than the linear techniques. The gradient boosting method is able to deliver the best results. For the Gradient Boosting, the standardization does not improve the score and we will just use the non standardized data in the following.

## Analysis for the selected variables
In the following, we will focus on the selected variables and analyse if we can predict them from the census data from 2000. We will use the Gradient Bosting Regressor as its overall performance was the best. We will also analyse what features are the most important in the prediction of the variable. We will plot the feature importances for the different predictions. We still need to be careful with high cardinality features of the impurity-based feature importance scores (high cardinality tends to have a higher feature importance score). Moreover, the impurity-based feature importance score will not be meaningful if we have correlated features. One correlated feature might have a very high score while the other one is very small. Therefore, we use the permutation feature score on test data to reanalyse the feature importances. The idea of the permutation feature score is to test the influence of a feature immediatly on the model. For unimportant features, the have little effects on the predictive power of the model and the permutation of important feature should have a big negative influence on the predictive power. (Source: We have used some explainations from [datadive](https://blog.datadive.net/selecting-good-features-part-iii-random-forests/) and [sklearn](https://scikit-learn.org/stable/modules/permutation_importance.html#permutation-importance)).

## Overall conclusions
We see that in general the census data from 2000 does not allow us to construct a good model to predict the different features of the survey in 2005. This is not astonishing as the census data has less granularity than the survey data that is specific to every household. The cenus data mostly reports averages over an area that is composed of many households. There are still a few values, where some algorithms are able to make decent predictions. 