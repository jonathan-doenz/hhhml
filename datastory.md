---
layout: page
title: Datastory
---

If you are reading this and you are unfamiliar with the [Housing, Health & Happiness (2009) paper](https://www.aeaweb.org/articles?id=10.1257/pol.1.1.75), here's the paper's abstract:

>We investigate the impact of a large-scale Mexican program to replace dirt floors with cement floors on child health and adult happiness. We find that replacing dirt floors with cement significantly improves the health of young children measured by decreases in the incidence of parasitic infestations, diarrhea, and the prevalence of anemia, and an improvement in children's cognitive development. Additionally, we find significant improvements in adult welfare measured by increased satisfaction with their housing and quality of life, as well as by lower scores on depression and perceived stress scales.

In a nutshell, the authors found two neighboring cities in Mexico that formed a single metropolitan area. This area was divided aministratively, so they had a single area with very similar demographic attributes differing, where (by law) only half of the residents were part of the program. They split the households into clusters using data from a **2000 census**. Then, in **2005**, the authors **surveyed** these households about a variety of welfare and happiness indicators. Finally, the authors use least-squares regression techniques to measure the impact of Piso Firme, a government program to replace dirt floors with cement, on several regressor variables. For copyright purposes we can't link to the main paper, but you can find a description of all of the variables the author's investigated [here](https://github.com/epfl-ada/ada-2020-project-milestone-p3-p3_the-social-impacters/blob/master/README.pdf).

From a high level, we determined that the paper thoroughly analyses the data provided and rigorously shows using linear regressors that the implementation of the Piso Firme program had a statistically significant impact on the dependent variables shown. We aim to extend this analysis in two distinct ways. First, we use classifiers and the associated feature importances to further confirm or reject the conclusions found by the original authors' regression analyses. Second, we aim to build regressors that predicts the features collected in the 2005 survey that most generalizeable to other areas of public policy. Importantly, we aimed to build these regressors *using only features from the 2000 census*. The ability to successfully derive the survey features from the census data would indicate that executing the 2005 survey was not necessary, and that time and funding allotted to similar future public policy surveys could be saved.

Put concretely, we aimed to answer the following 2 research questions:
1.  **[Task A]** Do Machine Learning-based classification approaches find similar positive treatment effects as the regression analyses of the paper? In other words, do we reach the same conclusions as the researchers if we try to classify households as `treatment` or `control` based on the dependent variables?
2.  **[Task B]** Is it possible to predict the most important variables from the 2005 survey using only data from the 2000 census?

Here's a table of contents for the rest of this article:
* TOC
{:toc}

# Task A

Our task A consists in two main goals:
- 1.Compare the predictive performances of the different _outcome variables_ specified in Tables 4 and 6 of the paper to predict whether a household is from the treatment of control group.

To achieve this, we first build a baseline model that only includes control variables, i.e. features from the 2000 census, and features from the 2005 survey that are used either in their regression model 3 or as robustness checks in Table 7 of the paper. 
50 such features were selected.
The task consists in predicting whether a household is from the treatment or control group based on these features. The target feature is therefore the _intention-to-treat_ feature, referred as `dpisofirme` throughout the code.

Our expectation is that the baseline model should not perform well because the authors provided ample evidences that treatment and control households are well balanced over these control variables. As we will see, our models could actually exploit the differences in these control variables and perform very well on this baseline case.

Then we construct a model for each _outcome variable_, i.e. a model that includes the outcome variable as covariate besides the control variables. 
In the paper, the intention-to-treat variable is shown to be important to predict the outcome variables, hence we expect that including an outcome variable in the model should improve our predictions of the intention-to-treat variable. 
As you will see, we could not observe such an effect because the baseline predictions were already perfect.

- 2.Compare the feature importances of the _outcome variables_ with the regression coefficients from the paper

To achieve this, we perform a feature importance analysis on the models explained above and obtain an importance score for each outcome variable.
Then we compare first qualitatively the importance ranking of the outcome variables using our importance scores with the ranking based on the paper's regression coefficients.
It turns out that they agree relatively well.
We then compare them quantitatively and they agree pretty well too.
## Comparison of the predictive performances

{% include figures/classifiers_performances_table.html %}

**Caption for the above table**:
Prediction performances on the test set.
The left-most column represents the outcome variable that is included as covariate, besides the control variables. 
The _baseline_ variable means that only the control variables are included.
Columns 2 to 4 show the accuracy of each classifier.
Columns 5 to 7 show the F1-score of each classifier.
The 3 right-most columns represent the feature importance associated to the feature mentioned in the first column for each classifier.

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

## Comparison of our feature importances with original _intention-to-treat_ coefficients

We now compare the feature importances of the outcome variables obtained in the present task, with the coefficients of the _intention-to-treat_ variable that the authors obtain in their regression analyses.

We expect that our feature importances should loosely agree with their coefficients, meaning that the extent to which the _intention-to-treat_ variable is linearly related to a variable is proportional to the importance of this variable to predict the _intention-to-treat_ variable.

We will use the feature importances from the logistic regression because they range over positive as well as negative values, contrarily to the feature importance scores from random forest and XGBoost classifiers.

{% include figures/coef_and_importance_comparison_barplot.html %}

We first notice that the ranking of importances is almost the same in the paper and in our task:

The only differences are stated in the 3 following bullet points, in which the first rank mentioned corresponds to the paper's, and the second corresponds to our task's:
- `S_satislife` is ranked 4 instead of 3
- `S_shcementfloor` is ranked 5 instead of 6
- `S_satisfloor` is ranked 7 instead of 4

We also note that the sign of the coefficients agree with the sign of the feature importance scores.

We conclude that our feature importance scores agree qualitatively with the coefficients from the paper's regressions.

Here we show the paper's coefficients transformed in the range \[0, 1\], along with the importance scores that we obtained for the same features and transformed in the same way.

{% include figures/coef_and_importance_comparison_barplot.html %}

**Observations:**
- The scores are almost the same for 3 of the cement related features (`S_cementfloorbed`, `S_cementfloorkit`, `S_cementfloordin`) and for `S_cesds`, the depression score.
- `S_cementfloorbat`, `S_shcementfloor`, `S_satisfloor` have similar values in terms of the paper coefficients, and also in terms of the feature importance scores. But they all are considered relatively less important by the feature importance scores compared to the paper coefficients.
- The feature for which the two scores differ the most (on a relative scale) is `S_pss`, which represents the perceived stress of the mother of the households.

# Task B
## Survey of Different Algorithms
We predict the different 2005 survey values from the census data from 2000. We calculate and report the $$R^2$$ scores for the different targets of the survey data and the different models.
We use the following models:
- Linear models
    - Linear regression: A simple linear regression is always a good baseline to see how the more advance machine learning techniques perform compared to the linear regressor.
    - Ridge regression: The Ridge regression use a L2 regularization term that penalizes the size of the coefficients and 'shrinks' these coefficients. Therefore, the model becomes more robust to collinearity.
- Non-linear models
    - DecisionTreeRegressor: A decision tree is build from the data and can have a very high depth.
    - GradientBoostingRegressor: Builds a certain number of weak learners (decsion trees with a shallow depth) and combine these in order to have a strong prediction. 
    - MLPRegressor: The multi-layer perceptron is a fully connected neural network that is able to build complex non linear models.

We calculate the $$R^2$$ score with a standardized and non-standardized dataset. Different algorithms improve in performance when standardizing and others have in general no need for standardization like the Gradient Boosting Regressor. We will see how the different algorithms behave in the following heatmap. 

Each row corresponds to a different algorithm, and each column corresponds to the dependent variable of the regression. Lighter colors indicate a regressor's ability to successfully predict the corresponding variable.

{% include figures/taskb_R2_scores_heatmap.html %}

When we compare the results between the different average performances of our five regressors, we notice that the overall performance of all the regressors are not very good. The Linear and Ridge Regressor seem to have better performance than the non-linear regressors. This is mostly explained by the fact that they don't have a very bad performance on variables that are difficult to predict. The linear techniques perform far better than the non-linear ones on some variables, where a good prediction seems  to be feasible. We will in the following analysis focus on the variables for which we can make a meaningful prediction and compare the different models for these. 

{% include figures/taskb_mean_r2_performance_all_predictions.html %}

We computed the mean $$R^2$$ score for all the variables over all the models and choose the five variables for which we can potentially make a meaningful prediction. 

Let's graph the $$R^2$$ score for the different models in a barplot again.

{% include figures/mean_r2_performance_chosen_predictions.html %}

This plot offers a very different picture on the performance of the different regressors. The Linear and the Ridge regression perform very similarly. The non-linear methods perform better than the linear techniques. The gradient boosting method is able to deliver the best results. For the Gradient Boosting, the standardization does not improve the score and we will just use the non standardized data in the following.

## Analysis for the selected variables
In the following, we will focus on the selected variables and analyse if we can predict them from the census data from 2000. We will use the Gradient Bosting Regressor as its overall performance was the best. We will also analyse what features are the most important in the prediction of the variable. We will plot the feature importances for the different predictions. We still need to be careful with high cardinality features of the impurity-based feature importance scores (high cardinality tends to have a higher feature importance score). Moreover, the impurity-based feature importance score will not be meaningful if we have correlated features. One correlated feature might have a very high score while the other one is very small. Therefore, we use the permutation feature score on test data to reanalyse the feature importances. The idea of the permutation feature score is to test the influence of a feature immediatly on the model. For unimportant features, the have little effects on the predictive power of the model and the permutation of important feature should have a big negative influence on the predictive power. (Source: We have used some explainations from [datadive](https://blog.datadive.net/selecting-good-features-part-iii-random-forests/) and [sklearn](https://scikit-learn.org/stable/modules/permutation_importance.html#permutation-importance)).

### S_garbage
*Definition: This is a binary variable that tells us if this households have access to garbage collection service.*

This is a binary value, which tells us if we use garbage collection. In this case, we can use a classification algorithm and see how easily we can predict this value. Most of the households dispose of garbage collection, therefore we should not use the naive accuracy score to judge the quality of our prediction. The naive prediction of saying that we predict that all the households have garbage collection leads to an accuracy score of $$\frac{1617}{1617+336}\ =\ 0.830$$. This score would give the impression that our predictions is good even if it does not provide is any additional information. In order to better judge the quality of the predictor, we will plot the Receiver Operating Characteristic (ROC) curve and provide the Area Under Plot (AUC) score. The ROC curve is able to illustrate the quality of a predictor by ploting the False positive rate ($$\text{FP rate} = \frac{\text{FP}}{\text{FP}+\text{TN}}$$, $$\text{TP rate} = \frac{\text{TP}}{\text{FP}+\text{FN}}$$) against the True positive rate for different threshold levels. The AUC score summarizes this plot in a meaningful way by calculating the area under this curve. The closer it is to 1, the better the prediction is. A score of 0.5 means, that the performance of the regressor is randomly guessing.

{% include figures/taskb_AUC_curve_S_garbage.html %}.

The Area Under ROC curve is 0.846. Recall that an AUC value of 0.5 corresponds to a random estimator, and an AUC value of 0.1 corresponds to a perfect estimator. Therefore, our model performs in a decent way on this variable and can often predict what households will get garbage collection from the census data. In the following, we will analyse how important the different features were in the prediction.

{% include figures/taskb_feature_importance_S_garbage.html %}

Both plots we produce give a very different picture of the feature importance. Some of the features are important in both plots, but some are also overestimated in the impurity-based feature importance analysis. Examples for features that are overestimated are `C_blocksdirtfloor` and `C_gasheater`. These two variables have a high cardinality of 93 for the `C_blocksdirtfloor` and 109 for `C_gasheater`. 

Good indicators to predict if the household has garbage collection 5 years into the future are `C_poverty` and `C_headeduc`. We can only hypotheszise why these two features are the most important:
- `C_poverty`: Most of the household that are economically better of have already garbage collection in 2000. Therefore the interesting part is to predict if the poorest of the households will have garbage collection. Government programs might also focus the most on the poorest households. This makes this variable quite important. We need to notice that the confidence interval for this variable is very large.
- `C_headeduc`: The average number of schooling years of an household might be predictive in a sense that an educated person will know about the importance of garbage collection service for the general hygiene and will also be able to articulate their needs to the institution that takes care of the garbage collection. 
We are very aware that these kind of hypothesis need to be taken with a grain of salt. The predictions are not very good and tree learners have a tendency of overfitting.

### S_instcement
*Definition: Indicator equal to one if the household reports having installed a cement floor since 2000.*

{% include figures/taskb_AUC_curve_S_instcement.html %}

The Area Under ROC curve is 0.797. Again the predictions of our model are decent. We will turn our focus towards the feature importance of our prediction.

{% include figures/taskb_feature_importance_S_instcement.html %}

We will only consider the second subplot based on the permutation importances for our analysis. `C_waterland` and `C_people` seem to be the greatest predictor if a household will get cement floor in between 2000 and 2005. `C_waterland` is the proportion of household that has no water connection outside the house and `C_people` is the cumulated number of people. 
We can again hypothezise why these two values are the most important ones:
- We could not come up with an intuitive explaination for the importance of the `C_waterland` variable.
- If a lot of people live in a certain area, the government might turn its attention more easily towards these households. Therefore, they might have higher probabilities of profiting from such a program. 

### S_cementfloordin and S_cementfloorkit
*Definition: Cement floor in dining room and in the kitchen.*

We will treat both predictions in this subsection because we find it interesting to compare the results for both. 

{% include figures/taskb_AUC_curve_S_cementfloordin.html %}
{% include figures/taskb_AUC_curve_S_cementfloorkit.html %}

The Area Under ROC curve is 0.722 and 0.710, respectively: Both predictions are acceptable and there quality is quite similar.

`S_cementfloordin` does not really have features that are much more important than other when we consider the permutation importance scores. So an analysis is rather difficult. For `S_cementfloorkit`, we have `C_HHdirtfloor` and a little less important `C_refrigerator`. We hypothesize:
- `C_HHdirtfloor`: We don't really know why this feature would be more important particularly to predict the presence of cement floor in the kitchen. By the little analysis just above, we could show that often the kitchen is one of the last rooms that gets a concrete floor. Therefore, the information what proportion of dirtfloors are in the households might be particularly important.
- `C_refrigerator`: This value seems to influence the prediction also in a meaningful way. this might make sense, because the presence of a refrigerator might indicate if a kitchen is often used and how the hygienic standards are in this particular household. If the kitchen is often used and standards are high, the investment in a concrete floor is more probable.

### S_shcementfloor
*Definition: Share of rooms with cement floors.*

This is a continuous variable, so we use regression to attempt to predict the value. Using a sklearn `GradientBoostingRegressor`, we were only able to achieve an $$R^2$$ value of 0.103.  From census data it seems to be difficult to infer the proportion of concrete floors 5 years later.
# Conclusions

In determining whether machine learning techniques strengthen or weaken the original authors' hypothesis, we conclude that our feature importance scores agree to some extent quantitatively with the original paper coefficients. But, we advise not to rely too much on them, and rather only use the qualitative information of the ranking, which is probably more robust.

We see that in general the census data from 2000 does not allow us to construct a good model to predict the different features of the survey in 2005. This is not astonishing as the census data has less granularity than the survey data that is specific to every household. The census data mostly reports averages over an area that is composed of many households. There are still a few values, where some algorithms are able to make decent predictions, but overall, our results indicate that in general the 2005 survey data **cannot** be predicted from the 2000 census data.