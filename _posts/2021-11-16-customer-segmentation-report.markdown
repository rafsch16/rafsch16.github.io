---
layout: post
title:  "Predicting Customers from Demographics"
date:   2021-11-16 12:35:09 +0200
categories: demographics arvato datascience
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
width: 36%;
margin-left: 30%;
margin-right: auto;
}

#img3 {
display: block;
width: 60%;
margin-left: 18%;
margin-right: auto;
}

#img4 {
display: block;
width: 60%;
margin-left: 18%;
margin-right: auto;
}

#table1 {
display: block;
width: 60%;
margin-left: 20%;
margin-right: auto;
}

#table2 {
display: block;
width: 80%;
margin-left: 19%;
margin-right: auto;
}

#table3 {
display: block;
width: 100%;
margin-left: 0%;
margin-right: auto;
}

table.myFormat tr td { font-size: 13px; }
p1 {
  font-size: 14px;
}
</style>
</head>
<body>

<img src="\images\packages.jpg" />
<br>
<br>

<p>This blog post is about my Capstone project for the Udacity Data Scientist Nanodegree program.</p>

<h3>Project Definition</h3>
<h4>Project Overview</h4>

<p>Many companies have a core customer base that reflects a segment of the general population only. This segment can be very different for each company and/or the products and services offered. If it is possible to segment the customer base from demographic information, this would allow to identify potential targets for a marketing campaign. Given the limited resources of a company it would be beneficial to only target the relevant demographic segment. Additionally, it may be beneficial to not bother parts of the population that are unlikely to become customers.
</p>

<p>The aim of this project is to analyze demographic data in order to identify potential customers of a mail-order sales company. This involves analyzing attributes of established customers and contrasting them with the general population using unsupervised learning techniques. The gained knowledge about the demographic differences should then be applied to optimize supervised learning techniques.</p>

<p>The project is a Udacity specific project that was developed in collaboration with Arvato Financial Services. All the data for this project was provided by Bertelsmann Arvato Analytics and AZ Direct GmbH.</p>

<h4>Problem Statement</h4> 

<p>In a first part of the project, two datasets with demographic information are provided, one for the customers of a mail-order sales company and one for the general population of Germany. Unsupervised learning techniques should be used to describe the relationship between the demographics of the company's existing customers and the general population. This should allow to describe segments of the population that are more likely to become customers of the company.</p>

<p>The knowledge gained in part one should then be applied on a dataset with demographic information for targets of a marketing campaign, by training a supervised learning model and predicting which individuals are most likely to convert into customers. Having a predictive mode would allow to target segments of the population more specifically than was done in previous campaigns.</p>

<p>To describe the company's customer base using unsupervised learning techniques, I decided to use a Mann-Whitney U test on the demographic features of the datasets. The Mann-Whitney U test is a non-parametric test for statistical differences in the distribution of two datasets. This will allow to outline the relationship between customers and the general population and will then also allow to select subsets of features in order to boost supervised learning techniques. Several Machine Learning methods will be tested to optimize supervised learning.</p>

<h4>Metrics</h4> 

<p>The dataset provided for supervised learning is imbalanced with the two classes, i.e., customers and non-customers, represented unequally with only 1.2% of customers. I therefore use the area under the curve (AUC) of the receiver operating characteristic (ROC), which is an optimal metric for datasets where the positive class is rare.</p>

<h3>Data Exploration</h3> 

<p>As is usual in Data Science, the first step is to get to know the data. The general population data and customers data have the same number of attributes, with some additional attributes for the customers. The information level of these attributes reaches from personal information to group information, such as household, community or district based information. The personal level includes attributes such as age, financial typology, social status, or preferred information and buying channels. The group based representation includes attributes such as the type of building, transactional activity, share of small cars, distance to the city center, or the share of unemployed persons in a community. In the figure below you can see a histogram plot of the number of adult persons in a household for the general population data, which is only one of a total of 366 features.</p>

<div id="img1">
    <img src="\images\numb_persons.png" />
</div>

<p1><b>Figure 1.</b> Histogram plot for the number of adult persons in a household. Shown is the density of appearance in the general population dataset.</p1>
<br> 
<br> 

<h3>Methodology</h3>
<h4>Data Preprocessing</h4> 

<p>Before looking at how the customers and general population data relate, the customers and general population datasets were cleaned in several steps. Besides custom cleaning steps, this involved many standard procedures such as removing duplicated data, removing columns with no variability, removing columns with many <i>nan</i> values, parsing strings with digits into floats, and creating dummy variables for object type data. Generally, as many attributes as possible were kept, where after the cleaning steps a total of 600 attributes existed. The cleaning steps were implemented using the function 'clean_df', the docstring of this function is shown in the figure below.
</p>

{% highlight ruby %}
def clean_df(df, customer_in = True, frac = 0.9):
    '''
    Function that applies cleaning steps:
    1. Custom cleaning
    2. Remove and store extra columns for customers data
    3. Remove duplicate data
    4. Remove columns with frac nan values
    5. Remove columns with no variability
    6. Parse strings with digits into floats
    7. Extract year from date
    8. Create dummy variables for obj columns
    
    INPUT
    df - data (DataFrame)
    customer_in - set 'True' for customer data, 'False' for azdias data (boolean)
    frac - fraction of nan values allowed in a column (float)

    OUTPUT
    df - cleaned data (DataFrame)
    df_extra - extra data in customer data (DataFrame)
    '''
{% endhighlight %}

<p1><b>Codebox 1.</b> Docstring of the cleaning function used for data preprocessing.</p1>
<br> 
<br> 

<h4>Implementation</h4> 

<p>After data cleaning, I was ready to identify parts of the population that best describe the core customer base of the company. Therefore a Mann-Whitney U test was applied on each attribute of the customer and general population datasets. To do so, the <i>mannwhitneyu( )</i> function from the <i>scipy.stats</i> module was implemented.</p>

{% highlight ruby %}
p_val = stats.mannwhitneyu(x, y, alternative='two-sided')
{% endhighlight %}

<p1><b>Codebox 2.</b> The <i>mannwhitneyu( )</i> function from the <i>scipy.stats</i> module.</p1>
<br> 
<br> 

<p>Here, <i>x</i> and <i>y</i> are the datasets to be tested. The resulting p-value allowed to sort the attributes according to their statistical difference.</p>

<p>At this point the third dataset comes into play. It is the dataset from a previous marketing campaign that includes the same demographic information together with a response column, that states whether or not an individual has become a customer of the company. This dataset was used to train and test a supervised learning model, identifying potential customers.</p>

<p>Therefore, a pipeline was used with a list of transforms and a final estimator, performing a grid search to optimize the hyper parameters of the model. This is based on scikit-learn's Pipeline and GridSearchCV classes. The full pipeline is as follows: Scaler, Imputer, Classifier. To be used in a pipeline, the Scaler was coded as a class with fit and transform methods. It can apply 'min-max' or 'standard' scaling, whereas scikit-learn's Imputer class can insert the 'mode' or the 'mean' of an attribute for missing values. The docstring of the Scaler class is shown below. Note that the Scaler class inherits from scikit-learn's BaseEstimator and TransformerMixin classes.</p>

{% highlight ruby %}
class Scaler(BaseEstimator, TransformerMixin):
    '''
    Scaling transform with min-max or standard scaling.
    
    Attributes:
    -----------
    mode : 'min-max' or 'standard' (str)
    params : for mode = 'min_max': list of (x_min, x_max) tuples 
                            for each column in data (float)
             for mode = 'standard': list of (mean, std) tuples 
                            for each column in data (float)
    '''
{% endhighlight %}
<p1><b>Codebox 3.</b> Docstring of the Scaler class used for data scaling.</p1>
<br> 
<br> 

<p>Three different classifiers were tested: Logistic Regression, Random Forest, and Gradient Boosting. The Random Forest and Gradient Boosting classifiers are regularized using the parameter <i>min_weight_fraction_leaf</i>, indicating the minimum number of samples a leaf node must have (expressed as a fraction of the total number of instances). For Logistic Regression the parameter <i>C</i> is used with L2 regularization. Below the code for the Random Forest classifier model is shown.</p>

{% highlight ruby %}
def build_model():
    '''
    This function builds the ML model.
    
    INPUT
    None
    
    OUTPUT
    cv - GridSearchCV object
    '''
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
    
    cv = GridSearchCV(pipeline, param_grid=parameters, cv=3,
                                        scoring='roc_auc', refit=True)
    
    return cv
{% endhighlight %}

<p1><b>Codebox 4.</b> Learning model using scikit-learn's Pipeline and GridSearchCV classes.</p1>
<br> 
<br> 

<p>Importantly, during the training of the model, we also optimize the data by including a different number <i>n</i> of features. The features included are always the <i>n</i> most significant features as obtained from the Mann-Whitney U test. As a metric for training we use ROC-AUC scoring. The code for finding the best estimator and the optimal number of features is shown below.</p>

{% highlight ruby %}
def main():
    '''
    This function finds the best model.
    
    INPUT
    None
    
    OUTPUT
    fpr - false positive rates for the test data for the best model (ndarray)
    tpr - true positive rates for the test data for the best model (ndarray)
    best_params - parameter settings for the best estimator (dict)
    best_estim - estimator which gave highest score (object)
    num_features - number of features for the best model (int)
    best_auc - mean cross-validated auc-score for the best model (float)
    auc_test_select - auc-score for the best model on the test data (float)
    '''
    # load explanatory and response variables
    X, y = load_data()
    best_auc = 0
    # make loop over features
    for f in range(2,30,2):
        features = df_p_values['features'][:f]
        
        # clean explanatory variables
        cleaner_obj_X = TrainingDataCleaner(features)
        X_1 = cleaner_obj_X.fit_transform(X)
        
        # split the data in train and test datasets
        X_train, X_test, y_train, y_test = train_test_split(X_1, y, 
                                            test_size=0.33, random_state=42)
        
        # train model and predict on test data
        model = build_model()
        model.fit(X_train, y_train)
        auc = model.best_score_
        y_probas = model.predict_proba(X_test)
        y_scores = y_probas[:,1]
        auc_test = roc_auc_score(y_test, y_scores)
        
        # store best estimator, parameters, number of features and auc scores
        if auc > best_auc:
            fpr, tpr, threshold = roc_curve(y_test, y_scores)
            best_params = model.best_params_
            best_estim = model.best_estimator_
            results={'num_features': len(features)}
            auc_test_select = auc_test
            best_auc = auc
            
    return fpr, tpr, best_params, best_estim, num_features, best_auc, auc_test_select
{% endhighlight %}

<p1><b>Codebox 5.</b> Function to find the best model by looping over a different number of features.</p1>
<br> 
<br> 

The steps for finding the best estimator are as follows:
<ol>
  <li>Load explanatory and response variables.</li>
  <li>Perform a loop for an increasing number of features.</li>
  <li>Clean the explanatory variables. The cleaning is similar to the cleaning in the preprocessing step but was implemented as a class with <i>fit</i> and <i>transform</i> methods.</li>
  <li>Split the data in train and test datasets using scikit-learn's <i>train_test_split</i>.</li>
  <li>Train model and predict on test dataset.</li>
  <li>Store the best estimator, parameters, number of features and AUC score.</li>
</ol>

<h4>Refinement</h4> 

<p>I want to point out some important steps in the process that were required to get a proper performance.</p> 

<p>First, preprocessing of the data is very important. The data structure is diverse and requires custom cleaning steps, for instance, there are different values representing 'unknown' for the different attributes. Using digits for unknown values biases the statistical testing as well as the supervised learning. Additionally, not all of the categorical attributes have ordered categories and one-hot encoding should be used consequently for those categories.</p> 

<p>Second, in order for the Mann-Whitney U test to perform properly, it is important to remove <i>nan</i> values as they will influence the testing. Also, the p-values decrease if the size of the datasets increase and will become zero at some point. In order to obtain non-zero p-values it was necessary to work with a subsample of the dataset. I initially also made the mistake to min-max scale the data before the test, which resulted in a biased test as the minimum and maximum values were sometimes different in the two datasets.</p> 

<p>In the process of refinement it is very important to periodically print or visualize aspects of the data. The pandas method <i>value_counts()</i> has proven to be very useful in this process.</p>

<p>For the supervised learning part several classifiers were tested. Here it is important to regularize a classifier properly to avoid overfitting. To avoid data leakage the Scaler and Imputer classes should be part of the pipeline, such that they are only fit to the train set in each fold of cross-validation.</p>

<h3>Results</h3>
<h4>Customer Segmentation Report</h4>

<p>As outlined above, a Mann-Whitney U test was performed to find statistical differences in the customers and general population datasets. The head of the resulting DataFrame with ascending p-values and corresponding attributes is shown in the figure below.</p>

<div id="img2">
    <img src="\images\p_vals.png" />
</div>

<p1><b>Figure 2.</b> DataFrame with sorted attributes. The attributes are sorted with ascending p-values, ranking them according to their statistical difference between the two datasets.</p1>
<br> 
<br> 

<p>The attributes may have non-intuitive names. For attributes where a description was available, it was used to get a better understanding of customer characteristics. In the table below the top attributes with the largest statistical difference between the datasets are described.</p>

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

<p>We can have a closer look at one of the attributes in the table above. If the Mann-Whitney U test performed properly, we would expect the distribution of values to be visually different for customers and the general population. Below the distributions are shown for the attribute 'household income'.</p>

<div id="img3">
    <img src="\images\hh_income.png" />
</div>

<p1><b>Figure 3.</b> Comparison of the attribute household income for customers and the general population. The distributions reveal that the customers tend towards higher household incomes.</p1>
<br> 
<br> 

<p>The two distributions reveal clearly that the customers tend towards higher incomes compared to the general population. Overall, we can describe the customers as a segment of the population that is price sensitive but doesn't hesitate to spend money for buying many goods, and of course having the necessary means to do so.</p>

<h4>Supervised Learning Model</h4> 

<p>As outlined above, I use <i>n</i> of the top attributes from the Mann-Whitney U test to perform feature selection for the training of a model that can predict customers from demographic data. This is done on a dataset for the targets of a marketing campaign, that comes with a response column that states whether an individual has become a customer of the company. We find that the learning models perform best with <i>n</i>=10-22 features included out of a total of 600 features. Below the ROC curve is shown for the best estimator found for the Gradient Boosting classifier.</p>

<div id="img4">
    <img src="\images\roc_curve_GB.png" />
</div>

<p1><b>Figure 4.</b> ROC curve for the Gradient Boosting classifier for predicting customers from demographic information.</p1>
<br> 
<br> 

<p>The performance of the classifiers tested is as follows: Logistic Regression < Random Forest < Gradient Boosting. The AUC score for cross-validation is 0.69 for Logistic Regression, 0.77 for Random Forest, and 0.78 for Gradient Boosting. The AUC scores obtained for cross-validation and on a test dataset for the different classifiers are summarized in the table below.</p>

<div id="table2">
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

<p>The estimator and hyper parameters found for the best model are shown in the table below. The hyper parameters were found using scikit-learn's GridSearchCV class with cross-validation.</p>

<div id="table3">
<table style="font-size: 14px;">
<colgroup>
<col width="30%" />
<col width="30%" />
<col width="30%" />
<col width="30%" />
<col width="30%" />
<col width="30%" />
</colgroup>
<thead>
<tr  class="header">
<th>classifier</th>
<th>min_weight_fraction_leaf</th>
<th>imputer__strategy</th>
<th>scaler__mode</th>
<th>number of features</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">GradientBoostingClassifier</td>
<td markdown="span">0.04</td>
<td markdown="span">mean</td>
<td markdown="span">min_max</td>
<td markdown="span">22</td>
</tr>
</tbody>
</table>
</div>

<p1><b>Table 3.</b> The estimator and hyper parameters found for the best model.</p1>
<br> 
<br> 

<h4>Justification</h4> 

<p>When optimizing the models, the number of features reduces from 600 down to 10-22, showing that the Mann-Whitney U test is a useful tool for feature selection. As already mentioned, the AUC metric is well suited for imbalanced datasets, where the positive class is rare. Ensemble learning techniques are then amongst the most powerful Machine Learning algorithms available today. I made use of them and achieved good scores, with Gradient Boosting performing the best. The models further performed similarly on test data and the validation data (using cross-validation), which assures the models capability to predict customers.</p>

<h3>Conclusions</h3> 
<h4>Reflection</h4> 

<p>We obtain a ROC-AUC score of 0.78 for the Gradient Boosting classifier. This score lies in between a perfect (AUC=1) and a random classifier (AUC=0.5). The Gradient Boosting classifier performs substantially better than the Logistic Regression classifier but only slightly better than the Random Forest classifier. The ROC-AUC metric is adequate for an imbalanced dataset where the positive class is rare. The project also reveals the importance of data preparation and selection. As there are many features in the dataset, it is important to perform feature selection using unsupervised learning techniques. The Mann-Whitney U test has proven to be capable of feature selection, while also allowing to get a statistical description of the customer base of the company. With the performance achieved here, the company may target 20% of the population in a marketing campaign and gain as many customers as targeting ~60% of the population randomly.</p>

<p>I was certainly impressed how well demographic information allows to identify potential customers of a company. The most challenging part of the project was to perform data preprocessing on datasets with a large number of features and then to select the most relevant features. This process had to be refined repeatedly in order to achieve a good performance.</p>

<p></p>

<h4>Improvement</h4> 

<p>One possible improvement for predicting customers would be to further optimize the data cleaning steps. The data structure is very diverse and requires custom cleaning steps. This has largely been addressed but there may still be space for improvement.</p>

<p>The Mann-Whitney U test performed here is only one possibility to obtain a reduced dataset, other techniques for feature selection or dimensionality reduction could be tested such as scikit-learn's SelectKBest or PCA, respectively. For the supervised learning, other techniques could be tested as well, such as Support Vector Machines or Neural Networks for classification.</p>

<p>The training dataset is strongly imbalanced with the two classes not represented equally. There is only a tiny fraction (1.2%) of customers in the training dataset. In classification tasks this is not optimal, as good metric scores may be obtained by ignoring the minor class. One could therefor try to resample the dataset into several balanced subsets for better training. Additionally, due to the imbalance in the data, stratified splitting should be considered when creating train and test datasets.</p>

<p>Finally, I was using a small test dataset that was split from the train partition. This is not best practice for such a small number of positive classes. Rather, the entire train partition should have been used for training the model. Initially, the model was supposed to be tested on a separate test partition, which was however not possible, as the Kaggle competition providing the response variables was already closed by the end of this project.</p>
</body>
</html>