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
margin-left: 10%;
margin-right: auto;
}

#img2 {
display: block;
width: 60%;
margin-left: 27%;
margin-right: auto;
}

#img3 {
display: block;
width: 60%;
margin-left: 11%;
margin-right: auto;
}

</style>
</head>
<body>

<img src="\images\CatDog-Featured-Cropped.jpg" />

<p>
<h2>Analysis of Airbnb Data</h2>
</p>

<p>Airbnb lodging serves all possible tastes, from mountain cottages to city apartments, from shared rooms to entire homes. The lodges come with many different amenities, from simple to fully equiped, and you might even come across pets on some properties.</p>

<p>Airbnb hosts do annote all kind of features about their properties. Airbnb makes this data available together with interesting information such as lodging prices and review ratings, information that can be analyzed in various ways. How do property features influence the price? Do cats and dogs influence the price differently?</p>

<p>To answer such questions, I have performed a regression analysis with the aim to predict the price from property features using Airbnb data of <a href="https://www.kaggle.com/airbnb/seattle/data"> Seattle </a> and <a href="https://www.kaggle.com/airbnb/boston"> Boston </a>. The figure below shows the 15 best indicators for price, with negative and positive price-indicators shown in red and blue, respectively. Hence, the top indicator for a low price is 'shared room', whereas the top indicator for high price is 'entire home/apartment'. This is not surprising, as most people would certainly pay a higher price for their own apartment than sharing a bedroom with other people.</p>

<div id="img1">
    <img src="\images\features_price.png" />
</div>

<p>Next, I wanted the see if dogs and cats on the property can indicate lodging prices. Here, a linkage is much less obvious than for sharing your room with other people. The analysis surprisingly reveals a cat-dog polarity. Whereas dogs are positive indicators (coefficient: 3.8), cats are negative indicators for price (coefficient: -4.8).</p>

<div id="img2">
    <img src="\images\dogs_cats_price.png" />
</div>

<p>Still the question remains why properties with dogs relate to higher prices even when considering a total of 77 different property features. Do dog owners charge higher prices? Are guests willing to pay more money for the presence of dogs? Let's take a look if the presence of cats and dogs correlates with other features in the dataset. The figure below shows the top indicators for price and whether they correlate more with cats or dogs, as indicated by the cat or dog image, respectively.</p>

<div id="img3">
    <img src="\images\features_price_emojis.png" />
</div>

<p>Obviously most positive indicators for price correlate more with dogs and the same is true for negative indicators with cats (shown by the ticks to the right). For instance, properties with dogs correlate more with location scores. There may be additional features that are not accounted for but correlate with the variables cats and dogs. In that case the presence of dogs may not be the direct causation for higher prices.</p>

<p>How well can we predict prices? About 57% of the variability in the price is explained by explanatory variables such as amenities, room/property type or pets. Property features thus explain the variability of the price only moderately well. All in all, despite the polarity found, it is difficult to conclude that cats and dogs truely influence lodging prices in opposite ways. It would be beneficial to collect additional features and analyse data from additional locations to better understand a putative cat-dog polarity in Airbnb prices.</p>

</body>
</html>