# The Bin Chicken in Italy
My favourite bird in Australia is without a doubt the Ibis bird (scientific name: Bin Chicken). After leaving Australia, I was sad that I leaving this majestic bird behind. But then news have reached me that the bin chickens are enjoying Euro Summer like many Australians do. And what better place to go than Italy.

In this blog post, I will use Species Distribution Modelling to find out where are the best places to go if you want to see an Ibis in Italy. Following the brilliant [tutorial](https://developers.google.com/earth-engine/tutorials/community/species-distribution-modeling) on the Google Earth Engine website, but applying it to bin chickens in Italy. If you want to see the code behind this work, or you want to run this on another species in another country, then have a look at the [Github Repo](). Read this blog post for a fun read about Species Distribution.

<figure>
    <img src="bin_chicken_vespa.png" alt="Ibis occurence animation" width="500">
</figure>

## Species Distribution Modelling
Species Distribution Modelling (SDM) tries to predict the distribution of a species across a geographical space. To do so, we combine environmental information about our area of interest and occurence data of our species of interest. Environmental information, we can get from remote sensing, I used the Google Earth Engine and their huge data collection. Occurence data, to my surprise, was not that hard to come by. There is [GBIF](https://www.gbif.org/) where you can find occurence data on basically any species.

**What can Species Distribution Modelling be used for?**
Here, I will focus a bit more on invasive species such as the Ibis. SDM is used to derive a habitat suitability map that shows in which areas the environmental conditions are suitable for the species and where it would thrive. At the same time, it also shows unsuitable areas. This information can then be used for example in 4 ways:

1) **Forecasting Future Spread**: When adding additional information on climate projections and urban expension scenarios, the model can predict how the habitat of the species changes in time. This helps conservation planners anticipate future threats to biodiversity and ecosystem function.
2) **Impact on Native Species**: Overlaying the ibis's predicted range with sensitive habitats or known locations of native birds allows us to identify where competition or predation may occur. This can support evidence-based conservation decisions.
3) **Rapid Response and Containment**: In case of species that are newly observed in an area, the model can highlight other high-risk areas nearby. This supports targeted response strategies before the species becomes firmly established. (The ibis is well established in North Italy and rapid response is too late. Similarly to France. Spain, on the other hand, acted fast and culled the bird immediately to prevent damage to the local wildlife and crops.)
4) **Management and Communication**: Finally, suitability maps are compelling visual tools. They can help communicate risk to land managers, decision-makers, and the public.

Alright, so let me walk you through the steps to derive a habitat suitability map.

## Occurence Data
First, we need data on where the species has spotted. This is usually the biggest problem on any ecological project. If we had detailed information about where and how many animals of a species are, we could do so many cool things with it. Instead, we have to rely on whatever data we have. For invasive species or rare species, the data is generally sparse and most importantly opportunistic. The data from GBIF come from birdwatchers, citizen scientists, or researchers during other studies. These reports vary in accuracy and coverage, but when aggregated, they paint a powerful picture of the species’ spread.

For my Ibis model, I used occurrence records from GBIF, which allowed me to map out where the Sacred Ibis has been recorded in Italy. See a short clip on the sightings and locations over the last 20 years.

<figure>
    <img src="ibis_spread_quarters.gif" alt="Ibis occurence animation" width="500">
</figure>

## Environmental Predictors
Now that we know *where* to find the Ibis, the next question is *why*. This is where the environmental predictors come into play. Variables such as the climate, landscape and the vegetation all help to understand the ecological preferences of the species. Knowledge of the species can help with identifying important variables. The ibis loves wetlands (elevation and slope data) and is a bird (Tree Canopy Cover). I decided to go with the following parameters:

1) **Climatic Variables from WorldClim**: I used bioclimatic variables (e.g. annual mean temperature, precipitation of driest quarter), which capture seasonality and extremes—not just averages.
2) **Terrain Information**: Elevation, slope, and aspect help identify low-lying wetlands or flat, open areas where ibises tend to forage.
3) **Tree Canopy Cover (TCC)**: This is generally an important feature for birds because birds like trees for protection from preditors

Before doing the modelling, we reduce the number variables. Most of the climatic variables will be strongly correlated which can be tricky when training our prediction model. We sort out the strongly correlated variables using Variance Inflation Factor. This reduces the multi-colinearity.

Below I show the remaining 6 features to get a rough idea of Italy. From left to right, top to bottom, we have: *Elevation, Mean temperature of the driest month, minimum temperature of the coldest month, slope, percipitation, TCC)
<figure>
    <img src="predictors6.png" alt="Ibis occurence animation" width="1000">
</figure>


## Performing the SDM
Alright, let's move on to the modelling. There are generally 2 approach that we can take. The presence-background method requires only presence data and applies Maximum Entropy Modelling to derive a occurence probability. The presence-absence data need both presence and absence data. Absence data is hard to come by because it is hard to be sure where a species is absent. But the presence-absence method has the advantage that besides the habitat suitability it also gives a feedback how important each environmental variable is.

As we do not have absence, we need to simulate it and create what is called pseudo-absence data. This is a crucial part in the modelling. Basically, we use the presence data to train a KMeans algorithm to cluster the area into 2 groups: The ones that are similar to the presence data location and the the areas that are dissimilar. To avoid overfitting, we only choose a small number of sightings (100ish) to train the model. The points are chosen semi-randomly. Generally random, but we want to ensure that they are somewhat spaced "equally" across the area.

The actual modelling is pretty easy. We have labelled data, i.e., presence and pseudo absence data. Now we pick a number of random points from the presence and absence data to train the model and validate it. We use the Random Forest algorithm which is fast and commonly used in the literature (Gradient Boosting algorithms, e.g. XGBoost, which are popular in the Machine Learning Community could be used as well, but in most cases the upsides of potentially higher accuracy is not worth the additional parameter tuning).

While training, we have to take into account spatial autocorrelation: Nearby points are generally stronger correlated than the ones further away. This can mess with the model. To reduce the impact, we use an approach called spatial block cross-validation. We make sure that points that are used for training have a minimum distance from the testing points. Thhis reduces spatial autocorrelation and makes our model more robust.

Without further ado, let's present the results. Below, you can see the habitat suitability of the Ibis in Italy. Green means high suitability and red bad suitability. In black, the sightings are shown.
<figure>
    <img src="habitat.png" alt="Ibis occurence animation" width="500">
</figure>
We can see that the Ibis likes flat areas and does not like the mountains. The northern flats of Italy is very popular with Ibis. With the most sightings and great suitability. Generally, the coastal areas are popular with the Ibis. Somehow, similar with Australians during Euro Summer. Generally, they prefer low altitudes without slopes. The tree canopy cover is not as important for the Ibis as it is for other birds. As the Ibis is quite big, most predatators avoid it.

## Final Thoughts

This project was really fun as it included my favourite bird in my neighbourhood. Turned out there was already similar research being done and published in [Nature](https://www.nature.com/articles/s41598-020-79137-w). So, we now have the technical skills to publish in Nature in 2021. Of course, the paper goes into much more detail about the Ibis, it's history, how it got to Italy and more. But this is the great thing about a blog post. It does not need to be this extensive.

The biggest reveal was the GBIF platform for wildlife data. I thought it would have been really hard to come along presence data. Now with the basics of SDM, I can think about some more involved topics. The plan was to have an additional part on how the climate will affect the habitat of the Ibis, but as the most important factor for the habitat was the elevation and slope, I think climate projections will not affect the habitat much. But maybe I will do this in a future project.







 ## Species Distribution Modelling
Species Distribution Modelling (SDM) tries to predict the distribution of a species across a geographical space. To do so, we require environmental information about our area of interest and occurance data of our species of interest.



# Write Along

### Occurence Data

1) We start by downloading he data from [GBF API](https://api.gbif.org/v1/occurrence/search). All we need to do is input the name of the species and country. Unfortunately, the data retrieval did not work via the API. It only got me 300 data points, when the total data set should be around in the 1,000-10,000 range. So we download the data by hand and then extract the information.

2) Next, we turn the dataframe into a geodataframe which just means we add a geometry and combine out longitude and lattitude into coordinates. We also drop all irrelevant information.

3) Now we can start with some exploration of the occurance data. We can check the number of sightings vs the time. Note that unfortunately when working with sightings data, there are a lot of uncertainties. More sightings does not mean necessarily mean more birds. I think generally the number of sightings increased over time as more people are reporting. But when there is a bird sighting that means there has been bird sighted. This will be useful later on for mapping the spatial distribution of the animals. But let's start with a simple animation showing the sightings for each year with a colour scale for the markers. Plotting this on a map, we can already see that these birds prefer the flat lands and avoid the mountains. I am surprised that they have not been able to enter cities. In Australia, these birds have adapted to the city life using their long beaks to dig deep into rubbish bins leading to their name of bin chickens. In Sydney, I would say together with pidgeons and ravens they are the most common birds in urban areas.

4) In the species distribution modelling, we will not consider the time component of the sighting, which could show some interesting insights. Initially, my idea was to model the change in habitat due to climate change, but you will see later that this actually not interesting in this case. I thought the Ibis arrived in Italy because of the changing climate, but it most likely escaped from captivity and is now thriving here. For the modelling, we divide the map into a grid with pixel size of 1km and resample the sighting data so that there is only 1 sighting per grid.



### Environmental Data
Moving on to the environmental data. We combine terrain information (remember these guys are usually found in flat lands) and combining it with the bioclimatic variables from WorldClim. There is actually an article in Nature covering what I am doing and even more. So, one could say that we are using cutting-edge techniques used for publication in probably the most renown science journal. Definitely check it out, there is a lot of additional research in this paper. Unfortunately, that means I can't claim this to be some ground-breaking research, but we are fact checking the paper I guess and giving a tutorial on how to use the techniques of a paper actually implemented. There are also tutorials on Google Earth Engine following the approach of the paper very closely (both in Pyton and Javascript). We will do a pure Python application of this instead of relying on the GEE toolkit. The advantage of the GEE toolkit is mainly the interactiveness of the maps with the downside being the JavaScript like Syntax, limitations to the tools of the toolkit and having to deal with Google Earth Engine Objects. This is published on a static website, so no interactive maps, thus, I am happy to use other Python libraries.


1) Our next step is to collect the environmental data of area of interest. We get elevation data, bioclimatic data from WorldClim and also land usage information. These birds are found in the "northern valley" of Italy where a lot of agriculture is happening, so there is probably some correlation between the birds habitat and the land usage. They probably love rice fields which are similar to their original habitat of wetlands where their beak can be used to dig. For birds, often Tree coverage or canopy heights are also used as an indicator. We add this as well, but I think it won't be very relevant for these kind of birds. They are pretty big, so most predators are discouraged from attacking them, hence, they don't need much tree coverage.

2) Take care of multicolinearity. If certain variables show a too strong correlation between each other, they can screw with our analysis. We take some random samples from our area of interest and see how our variables are correlated. We filter out the variables that are too strongly correlated. For one they do not give much additional information, but most importantly, they will mess up our regression analysis. The go-to method is to use the Variance Inflation Factor (VIF). And only keep variables with a VIF smaller than 5 (10). We should end up with around 5-10 variables, which is perfect should make the modelling run reasonable fast.

### Creating Pseudo-Absence Data
This is probably the trickiest bit of the modelling. If we had Presence and Absence data for the Ibis, this would be easy. But all we have is sightings aka presence data. We do not have absence data. Generally, absence data is really difficult to come by. The band-aid solution is to generate pseudo-absence data. Depending on how skillfull we do this, we get better results. Unfortunately, there is no one-fits-all solution to this and the results strongly depend on this approach. The simplest way is to take random samples from our area of interest that are not presence data. This can be quickly improved by adding a buffer area around the presence data. The problems are that if the buffer area is too small, we risk sampling inside the actual presence area. If the buffer area is too large, the border area between presence and absence could be modelled incorrectly. We try this method anyways.

The other method we are trying is to use environmental profiling. We use a similar approach to above, but then cluster our the area of interest into either similar to presence data or similar to the pseudo-absence points. I think this could be the best approach, but we still need to decide how many points we choose for the random samples. I was hoping to find some clear guidance on this, but let's see how things go. This could actually be a good intro to Machine Learning. But let's see how things go.

### Training the Model
The next part is quite straightforward. We have all the data prepped and ready. We add one more trick frequently used with spatial data that deals with spatial autocorrelation. Points that spatial close to each other tend to have similar attributes. A good example for this would be elevation. The points surrounding the peak of a mountain are usually similarly high in elevation. Commonly used is the *spatial block cross validation*. We divide the study area into larger blocks containing a large number of pixels. The blocks are then divided into training and test data, i.e., 70% of the blocks are used for training and 30% for validation. This method has been shown to effectively reduce the effect of spatial autocorrelation.

Besides this little extra step, this step will be business as usual. The most popular algorithm is Random Forest. The Kaggle favourites XGBoost or LightGBM are not used commonly because they need a bit of parameter tuning and the generally small presence-absence data there is not big difference between the using these heavier models. I think for large models where the training with Random Forests will slow down these models could be considered. Maybe I will see how the Gradient Boosters work.

### Model Validation
Now, we can evaluate the performance of the model. 

Not quite sure how this works, but I think there are a couple of metrics that are usually compared.

### Visualisation
Visualisation in this project is quite straight forward. The results of the modelling gives us a map with a habitat suitability index ranging from 0 to 1 where 0 means you will never see a bin chicken and 1 means perfect conditions for our ibis. So below we see the results.

The shape of Italy allows us for nice side-by-side visualisations. Here I plotted the elevation on the right and we can see that our bin chickens prefer the flats, low evaluation and small slope. Let's see if we can combine this into one plot. I am guessing that we can use a outline of where the elevation is above a certain level to enclose most of the habitat. 

Finally, we can use the results of the RF to plot which parameters are important. And we see that the elevation and slope are important and the birds also like warmer winters. They do not particularly care about tree coverage or aspect.

### Final Words
I had a lot more planned for this project, but the results were quite obvious. My initial plan was to do some climate modelling on how the habitat will grow given some climate projection. But the main driver of the locations was the terrain which is not really affected by climate modelling. Most likely, the Ibis did not make its own way into Italy, but broke out of captivity from Zoos and the habitat has always been suitable for the Ibis.

The more interesting question to ask is how to deal with Ibis. While this is one of my favourite birds that looks just majestical. It is also a very smart bird that can have huge impact on the surroundings. The bin chicken do not only have superior looks, but they are also superior in many ways to local birds and wildlife. I read a really nice book discussing the difficult subject of conversation. I am wondering if there is a way to model the impact of not intervening or culling the ibis population and show the effects on the wildlife. Ultimately, the question is what the goal of conservation is. I think the modelling might be a bit too complex, read as mathematically not solveable, but i will need to do more research.
