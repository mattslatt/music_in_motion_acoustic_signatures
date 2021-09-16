# Spotify Genre Analysis
Spotify is the leading music streaming service in the world, home to more than 70 million tracks, and at least 1.2 million artists. [This dataset](https://www.kaggle.com/yamaerenay/spotify-dataset-19212020-160k-tracks) collected data from the Spotify API, resulting in 1.1 million artists rows (including artist name, number of followers, genre tags, and popularity columns) along with 586k tracks (including 11 numerical features describing the acoustic signature, along with categorical data such as the affiliated artists and release date). The data was collected in April 2021, but track release dates span to 1922.

The primary goal is to orient this data around musical genres, and determine how genres change over time. First, we'll evaluate the current state of Spotify genres (from April 2021 when the data was collected). Then, by linking track release dates to artist genre tags, we can examine how genres have changed over time -- both in terms of their relative popularity and their acoustic signatures. 

# EDA
### Modern genre popularity
This analysis is reliant on Spotify's "popularity" metric for artists, which is primarily based on the total number of plays and how recent they are -- so there is a temporal bias for plays in April 2021. Popularity ranges from 0 to 100, and scales logarithmically with artist follower counts (total number of plays is not available through the Spotify API).

Artists are, on average, highly unpopular -- 21% of all artists are at 0 popularity!

![Popularity exploration](./img/popularity_metric_hist_scat.png)

Genre data is associated with artists (but not tracks -- we'll deal with that later). Among the 1,104,349 artists, only 27% (298,616) have any genres associated with them.

After removing the empty genre tags, there are 5,365 unique genres that constitute 48,787 genre combinations. Some of the rarest genres include 'whale song', 'iowa hip hop', and 'albanian iso polyphony'.

Half of all genres are observed 61 or fewer times, while the most common genres have nearly 600 tagged artists.

![Tagged artist distribution](./img/genre_count_histogram.png)

The average popularity of a genre is strongly related to the number of artists tagged. Some top genres, such as 'classical performance' are unpopular in the context of their artist abundance.

Interestingly, the most rare genres have relatively high popularity -- their uniqueness probalby helps them to stand out -- but still pale in comparison to the popularity of the dominant genres.

![Top 10 genres](./img/top_10_genres_count_and_pop_bar.png)
![Bottom 10 genres](./img/bottom_10_genres_count_and_pop_bar.png)

How can this relationship between genre and popularity be useful? Imagine a record label is interested in signing an up-and-coming artist, but isn't sure how their content fits in with the current interests of the Spotify community.

Here I calculated a 'demand' metric, calcluated by dividing average popularity by the number of tagged artists for that genre. A genre garnering lots of popularity, yet lacking in artists producing that content, would recieve a high 'demand' score.

There is a heavy skew in the demand distribution, given that uncommon genres seem to attract disproportional popularity. If we assume a record label isn't interested in taking a chance on a relatively unknown genre, the distribution and data below excludes the bottom 50% of genres (61 or fewer artists tagged) along with genres that have a popularity of 0 (for which demand cannot be calculated).

![Genre demand distribution](./img/genre_demand_hist.png)

At the bottem end of the distrubtion, genres such as 'neo-proto' and 'vintage western' are extremely low demand (~0.003), suggesting listeners have plenty of artist options for those genres. On the other hand, 'viral rap', 'melodic rap', and 'girl group' are in high demand (0.90, 0.78, 0.77 respectively), given their popularity relative to the low number of artists producing such content.


### Using high demand genres to recommend other artists

The artist_recommender.py script takes the name of an artist, and returns their tagged genres, the demand of each of those genres, and 5 relatively unknown artists that play the highest demand genre.

Those 5 relatively unknown artist are selected based on a 'rising' metric, calculated by their popularity divided by their followers. This helps to bias artists for recent plays, especially if not many people follow them yet. A default minimum number of followers is set to 10,000 so that recommended artists aren't completely underground.

For example, by entering "Crooked Colours" into the script, the following information is returned:

"Crooked Colours is in the top 0% of rising artists

Their 'indie soul' music is the most in-demand"

![Artist recommendation](./img/artist_recommender.png)

# Genres over time
### Linking song release dates to artist genres

Genres are not provided with track data, only artist data. I merged the 586k tracks and with the artist dataframe, resulting in 2,222,219 rows (on average, there are about 4 genres associated with each track).

![Top 7 genre abundance across time](./img/top_10_genre_time_scatter.png)

Interesting to see that 'classical performance' and 'rock' peaked substantially earlier than the other genres, which still seem to be on the climb.

How do specific track features, such as danceability and acousticness, change within a genre as time goes on?

# Genre analysis - Rock example

With more than 5,000 unique genres, let's focus on one example. These graphs and statitics can be easily replicated with functions found in the Jupyter notebook, including 

![Rock features](./img/rock_features_scatter.png)

Rock as a genre has a rich history that includes a formative period in the late 60s to early 80s. You can see a clear upward trend for 'energy' and a downward trends for 'acousticness'. Because this data is a time series, we must transform the data to a rate of change in order to make our samples i.i.d. and therefore valid with a t-test.

![Select features over time](./img/classic_rock_change.png)

The "genre_features_test('rock', 1967, 1982)" function will calculate the p-values for each of the track features for a given timeframe. Below is the output 'rock' from 1967 to 1982:

"Is the mean of the annual rate of change different from 0?

danceability:  
p-value = 0.71

energy:  
p-value = 0.006

speechiness:  
p-value = 0.81

acousticness:  
p-value = 0.031

instrumentalness:  
p-value = 0.958

liveness:  
p-value = 0.629

valence:  
p-value = 0.723"


We can see that in this case, there are two significant categories -- the energy and acousticness are changing substantially on a year-by-year basis for the given period.
