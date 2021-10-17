---
layout: post
title:  "The Cat-Dog Polarity in Airbnb Prices"
date:   2021-10-14 12:35:09 +0200
categories: airbnb datascience
---
<html>
<head>
<style>

#img1 {
display: block;
width: 60%;
margin-left: 5%;
margin-right: auto;
}

#img2 {
display: block;
width: 60%;
margin-left: 24%;
margin-right: auto;
}

#table1 {
display: block;
width: 60%;
margin-left: 22%;
margin-right: auto;
}
</style>
</head>
<body>

<img src="\images\CatDog-Featured-Cropped.jpg" />

<p>
<h2>Analysis of Airbnb Data</h2>
</p>

<p>Airbnb lodging serves all possible tastes, from mountain cottages to city apartments, from shared rooms to entire homes. The lodges come with many different amenities, from simple to fully equiped, and you might even come across pets on some of the properties.</p>

<p>Airbnb hosts do annote all kind of features about their properties. Airbnb makes this data available together with interesting information such as lodging prices and review ratings, information that can be analyzed in various ways. How do property features influence the price? Do cats and dogs influence the price differently?</p>

<p>To answer such questions, I have performed a regression analysis with the aim to predict the price from property features using Airbnb data of <a href="https://www.kaggle.com/airbnb/seattle/data">Seattle</a> and <a href="https://www.kaggle.com/airbnb/boston">Boston</a>. The figure below shows the 20 best indicators for price, with negative and positive price-indicators shown in red and blue, respectively. Hence, the top indicator for a high price is 'bedrooms', whereas the top indicator for a low price is 'review scores value'. It is not surprising that the price scales with the number of bedrooms. The value scores on the other hand relate to the review question  "did the guest feel that the listing provided good value for the price?". A negative correlation for price and value rating is <a href="https://bnbfacts.com/airbnb-value-rating-how-hosts-can-improve-it">well-known</a> for Airbnb data.</p>

<div id="img1">
    <img src="\images\features_price.png" />
</div>

<p>Next, I wanted the see if dogs and cats on the property can indicate lodging prices. Here, a linkage is much less obvious than for variables like the number of bedrooms. The analysis surprisingly reveals a cat-dog polarity. Whereas dogs are weak positive indicators (coefficient = 0.6), cats clearly are negative indicators for price (coefficient = -11.3).</p>

<div id="img2">
    <img src="\images\dogs_cats_price.png" />
</div>

<p>Still the question remains why properties with cats relate to low prices even when considering a total of 90 different property features. Do cats lower the value of a property, whereas dogs don't? This question is difficult to answer. It might however help to have a look at features that have the largest difference in correlation between cats and dogs.</p>

<div id="table1">
<table style="width: 400px;">
<colgroup>
<col width="30%" />
<col width="30%" />
</colgroup>
<thead>
<tr  class="header">
<th>Cat(s)</th>
<th>Dog(s)</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">Otherpet(s)</td>
<td markdown="span">PetsAllowed</td>
</tr>
<tr>
<td markdown="span">property_type_Yurt</td>
<td markdown="span">Petsliveonthisproperty</td>
</tr>
<tr>
<td markdown="span">room_type_Shared room</td>
<td markdown="span">IndoorFireplace</td>
</tr>
<tr>
<td markdown="span"><b>SuitableforEvents</b></td>
<td markdown="span">property_type_Tent</td>
</tr>
<tr>
<td markdown="span">room_type_Private room</td>
<td markdown="span">room_type_Entire home/apt</td>
</tr>
<tr>
<td markdown="span"><b>SmokingAllowed</b></td>
<td markdown="span">property_type_Camper/RV</td>
</tr>
<tr>
<td markdown="span">property_type_House</td>
<td markdown="span"><b>Family/KidFriendly</b></td>
</tr>
</tbody>
</table>
</div>

<p>Interesting features that correlate more with cats are 'Suitable for Events' and 'Smoking allowed', whereas we find the feature 'Family friendly' for dogs. It is therefore likely that the porperties with cats are different in average from the properties with dogs and that the variable 'Cat(s)' indirectly expresses this difference. In this case it wouldn't be the cats that cause the low prices, the variable rather catches a hidden feature that more often comes together with cats.</p>

<p>How well can we predict prices? About 58% of the variability in the price is explained by explanatory variables such as amenities, room/property type or pets. Property features thus explain the variability of the price only moderately well. It would be beneficial to collect additional features and analyse data from additional locations to better understand a putative cat-dog polarity in Airbnb prices.</p>
</body>
</html>