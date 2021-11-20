---
layout: post
title:  "Predicting Customers from Demographics"
date:   2021-11-16 12:35:09 +0200
categories: demographics avrato datascience
---

<html>
<head>
<style>

#img1 {
display: block;
width: 60%;
margin-left: 18%;
margin-right: auto;
}

#img2 {
display: block;
width: 60%;
margin-left: 18%;
margin-right: auto;
}

#img3 {
display: block;
width: 60%;
margin-left: 18%;
margin-right: auto;
}

p1 {
  font-size: 14px;
}
</style>
</head>
<body>

<img src="\images\packages.jpg" />
<br>
<br>
<h3>Analysis of Demographic Data</h3>


<p>Many companies have a core customer base that reflects a segment of the general population only. This segment can be very different for each company and/or the products and services offered. If it is possible to segment the customer base from demographic information, this would allow to identify potential targets for a marketing campaign. Given the limited resources of a company it would be beneficial to only target the relevant demographic segment. Additionally, it may be beneficial to not bother parts of the population that are unlikely to become customers.
</p>

<p>The aim of this project was to analyze demographic data in order to identify potential customers of a mail-order sales company. This involves analyzing attributes of established customers and contrasting with the general population using unsupervised learning techniques. The gained knowledge about the demographic differences was then applied to optimize supervised learning techniques, with a model being trained and tested on data from a previous marketing campaign. The data for this project was provided by Bertelsmann Arvato Analytics, as part of the Udacity Data Scientist Nanodegree program.</p>


<h3>Data Understanding</h3> 

<p>As is usual in Data Science, the first step was to get to know the data. The general population data and customers data have the same number of attributes, with some additional attributes for the customers. The information level of these attributes reaches from personal information to group information, such as household, community or district based information. The personal level includes attributes such as age, financial typology, social status, or preferred information and buying channels. The group based representation includes attributes such as the type of building, transactional activity, share of small cars, distance to the city center, or the share of unemployed persons in a community. In the figure below you can see a histogram plot of the number of adult persons in a household for the general population data, which is only one of 366 attributes.</p>

<div id="img1">
    <img src="\images\numb_persons.png" />
</div>

<p1><b>Figure 1.</b> Histogram plot for the number of adult persons in a household. Shown is the density of appearance in the general population dataset.</p1>
<br> 
<br> 

<h3>Data Cleaning</h3> 

<p>Before looking at how the customers and general population data differ, the datasets were cleaned in several steps. Besides custom cleaning steps, this involved many standard procedures such as removing duplicated data, removing columns with no variability, removing columns with many NaN values, parsing strings with digits into floats, and creating dummy variables for object type data. Generally, as many attributes as possible were kept, where after the cleaning steps a total of 600 attributes existed.
</p>

<h3>Customer Segmentation Report</h3>

<p>After data cleaning, we are ready to identify parts of the population that best describe the customer base of the company. Therefore a Mann-Whitney U test was applied on each attribute of the customer and general population datasets. The Mann-Whitney U test is a non-parametric test for statistical differences in the distribution of two datasets. The resulting p-value allowed to sort the attributes accordingly. In the table below the top attributes with the largest statistical difference between the datasets are described (only attributes where a description was available are listed).</p>

<div id="table1">
<table style="width: 400px;">
<colgroup>
<col width="30%" />
</colgroup>
<thead>
<tr  class="header">
<th>Customers...</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">...are less consumption minimalists</td>
</tr>
<tr>
<td markdown="span">...are less consumption traditionalists</td>
</tr>
<tr>
<td markdown="span">...are more money savers</td>
</tr>
<tr>
<td markdown="span">...care less about financial security</td>
</tr>
<tr>
<td markdown="span">...have lower financial interests</td>
</tr>
<tr>
<td markdown="span">...are more advertising interested</td>
</tr>
<tr>
<td markdown="span">...invest more money</td>
</tr>
<tr>
<td markdown="span">...are younger</td>
</tr>
<tr>
<td markdown="span">...have higher household incomes</td>
</tr>
</tbody>
</table>
</div>

<p1><b>Table 1.</b> The table describes some of the main differences between customers and the general population.</p1>
<br> 
<br> 

<p>Below the distributions are shown for the attribute 'household income', with customers clearly tending towards higher incomes compared to the general population.</p>

<div id="img2">
    <img src="\images\hh_income.png" />
</div>

<p1><b>Figure 2.</b> Comparison of the attribute household income for customers and the general population. The distributions reveal that the customers tend towards higher household incomes.</p1>
<br> 
<br> 

<h3>Supervised Learning Model</h3>

<p>At this point a third dataset comes into play. It is the dataset from a previous marketing campaign that includes the same demographic information together with a response column, that states whether or not an individual has become a customer of the company. This dataset was used to train and test a supervised learning model, identifying potential customers.</p>

<p>Therefore, a pipeline was used for a list of transforms and a final estimator, performing a grid search to optimize the parameters. This is based on scikit-learn's Pipeline, and GridSearchCV. The full pipeline is as follows: Scaler, Imputer, Classifier. To be used in a pipline, the Scaler was coded as a class with fit and transform methods. It can apply 'min-max' or 'standard' scaling, whereas the scikit-learn's Imputer can insert the 'mode' or the 'mean' of the attribute for missing values.</p>

<p>We compare three different classifiers: Logistic Regression, Random Forest, and Gradient Boosting. The Random Forest and Gradient Boosting classifiers are regularized using the parameter 'min_weight_fraction_leaf', indicating the minimum number of samples a leaf node must have (expressed as a fraction of the total number of instances). For Logistic Regression the parameter 'C' is used with L2 regularization. Below the code for the Random Forest classifier model is shown.</p>

{% highlight ruby %}
# build model for the RandomForestClassifier
def build_model():
    pipeline = Pipeline([
        ('scaler', Scaler(mode=None)),
        ('imputer', Imputer()),
        ('clf', RandomForestClassifier())
    ])

    parameters = {
        'imputer__strategy': ['mean','median'],
        'scaler__mode': ['min_max','standard'],
        'clf__min_weight_fraction_leaf': [0.005, 0.01, 0.02, 0.03, 0.04, 0.05],
        'clf__n_estimators': [100]
    },
    
    cv = GridSearchCV(pipeline, param_grid = parameters, cv=3, scoring='roc_auc', refit=True)
    
    return cv
{% endhighlight %}

<p1><b>Codebox.</b> Learning model using scikit-learn's Pipeline and GridSearchCV.</p1>
<br> 
<br> 

<p>Importantly, during the training of the estimator, we also optimize the data by including different numbers <i>n</i> of features. The features included are always the <i>n</i> most significant features as obtained from the Mann-Whitney U test. We find that the estimators perform the best with <i>n</i>=10-22 features included out of a total of 600 features. As a metric for training we use the area under the curve (AUC) of the receiver operating characteristic (ROC). Below the ROC curve is shown for the best estimator found for a Random Forest classifier.</p>

<div id="img3">
    <img src="\images\roc_curve_RF.png" />
</div>

<p1><b>Figure 3.</b> ROC curve of the Random Forest classifier for predicting customers from demographic information.</p1>
<br> 
<br> 

<p>The performance of the classifiers tested is as follows: Logistic Regression < Random Forest < Gradient Boosting. The AUC score for cross-validation is 0.69 for Logistic Regression, 0.77 for Random Forest, and 0.78 for Gradient Boosting. The AUC scores obtained for cross-validation and on a test dataset for the different classifiers are summarized in the table below.</p>

<div id="table1">
<table style="width: 400px;">
<colgroup>
<col width="30%" />
<col width="30%" />
<col width="30%" />
<col width="30%" />
</colgroup>
<thead>
<tr  class="header">
<th></th>
<th>AUC (cross-validation)</th>
<th>AUC (test dataset)</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span"><b>LogisticRegression</b></td>
<td markdown="span">0.69</td>
<td markdown="span">0.70</td>
</tr>
<tr>
<td markdown="span"><b>RandomForestClassifier</b></td>
<td markdown="span">0.77</td>
<td markdown="span">0.75</td>
</tr>
<tr>
<td markdown="span"><b>GradientBoostingClassifier</b></td>
<td markdown="span">0.78</td>
<td markdown="span">0.75</td>
</tr>
</tbody>
</table>
</div>

<p1><b>Table 2.</b> The table compares the AUC scores for the Random Forest, Logistic Regression, and Gradient Boosting classifiers.</p1>
<br> 
<br> 

<h3>Improvements</h3>

<p>One possible improvement for predicting customers would be to perform more data cleaning steps. The data structure is very diverse and requires custom cleaning steps, for instance there are different values representing 'unknown' for the different attributes. Using digits for unknown values biases the statistical testing as well as the supervised learning. Finally, not all of the categorical attributes have ordered categories and one-hot encoding should be used consequently for those categories. All of these points were addressed but may not have been exhaustively applied.</p>

<p>The Mann-Whitney U test performed here is only one possibility to obtain a reduced dataset, other techniques for feature selection and dimensionality reduction could be tested such as scikit-learn's SelectKBest and PCA, respectively. For the supervised learning, other techniques could be tested as well, such as Support Vector Machines or Neural Networks for classification.</p>

<p>Finally, the training dataset is strongly imbalanced with the two classes not represented equally. There is only a tiny fraction (1.2%) of customers in the training dataset. In classification tasks this is not optimal, as good metric scores may be obtained by ignoring the minor class. One could therefor try to resample the dataset into several balanced subsets for better training. Additionally, due to the imbalance in the data, stratified splitting should be considered when creating train and test datasets.</p>

<h3>Conclusions</h3>

<p>We obtain a ROC-AUC score of 0.78 for the Gradient Boosting classifier. This score lies in between a perfect (AUC=1) and a random classifier (AUC=0.5). The Gradient Boosting classifier performs substantially better than the Logistic Regression classifier but only slightly better than the Random Forest classifier. The ROC-AUC scoring is adequate for an imbalanced dataset where the positive class is rare. The project also reveals the importance of data preparation and selection. As there are many features in the dataset, it is important to reduce the dimensionality of the data using unsupervised learning techniques. With the prediction performance achieved here, the company may target 20% of the population in a marketing campaign and gain as many customers as selecting ~55% of the population randomly.</p>

</body>
</html>