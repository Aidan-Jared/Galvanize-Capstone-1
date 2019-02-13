# Galvanize Capstone 1

## Table of Contents

- [Project Explanation and Goals](#project-explanation-and-goals)    
- [Baysian A/B testing](#baysian-ab-testing)        
    - [Fantasy and Fiction](#fantasy-and-fiction)        
    - [Science-Fiction and Fantasy](#science-fiction-and-fantasy)        
    - [Fiction and Vampire](#fiction-and-vampire)        
    - [Top 10 tags](#top-10-tags)    
- [Linear Regresion](#linear-regresion)        
    - [Ridge](#ridge)        
    - [Lasso](#lasso)

## Project Explanation and Goals

For this poject I decided to look at book genres and the "bleeding" between genres (the enjoyment of one genre given the enjoyment of another genre). The Goodreads 10k book dataset from kaggle seemed perfect because it countained 10k books, user reviews and user given tags. The data sets was made of four csv files, books, book_tags, ratings, and tags. My end plan is to be able to predict the raitings that someone might give any genre given their enjoyment of one genre. 

My first step was to fiugre out how to put all of the data together so I could search through the data frame. The main problem though is the identifyer I am using are the user generated tags. (tag csv file) When just loading up theses tags, the first thing I noticed was that the majority of the tags are worthless such as to-read, favorite, --31-, and some tags writen in japanese. To deal with this I just did a quick visual search and made a function that removed the most common words I did not care about and would have messed with my analysis.
```python
def dataClean(df):
    for i in ['read', 'book', 'favorite', 'own', 'audio', 'wish-list', '--', 
    'library','buy','kindle','finish','have','audi','borrowed',
    'favourites','default']:
        mask1 = df['tag_name'].str.contains(i)
        df= df[~mask1]
    return df
```

After this I needed to find out how group the tags, books, and users so I could select specific genres and mean users reviews. My solution was to join the book_tags, and tag dataframes so that I would have tag names tied with book_ids. I then put this dataframe through my cleaning function and removed all repeate values so that I would only have one instance of every book and the tag that most people games this book. Then end result looked soemthing like this:

| Genre        | Books in Genre           |
| ------------- |:-------------:|
|fiction            |      1639|
|fantasy            |      1104|
|young-adult        |       695|
|mystery            |       640|
|non-fiction        |       597|
|romance            |       464|
|historical-fiction |       428|
|classics           |       419|
|childrens          |       294|
|science-fiction    |       269|
|horror             |       201|
|graphic-novels     |       180|

After finishing all this up it was off to the races.

## Baysian A/B testing

After cleaning and orginizing my dataset, I decided that the beset way to compare genres would be through Baysian A/B testing because it can show the mean rating that users give to other genres and can give me a numerical values for how much better does one genre fan like another genre. For these experments my Null is that individuals give the same rating to any genre and my Alt is that they give differnt values.

### Fantasy and Fiction
The first two genres I compared were fantasy and fiction being the two most common book types.

```python
df_fantasy = isolate_tag(df_tags_books, 'fantasy', min_books_read)
df_fic = isolate_tag(df_tags_books, 'fiction', min_books_read)
fantasy_fic = Beta.Beta(df_fantasy, df_fic, 'user_rating').compile_analysis('Fantasy', 'Fiction', Plot=True)
plt.show()
print('A/B test, Difference in Mean Raiting', fantasy_fic)
'A/B test, Difference in Mean Raiting' [0.5153, -0.012692577030812365]
```

![alt text](images/fig_Fantasy.png)

From running the A/B test and the Beta function produced I found that acording to the data there is almost no difference in the ratings that fans of fantasy give to fiction and vice-versa. I ended up getting that model B was better than model A about 50% of the time which once again shows no difference between the two models which makes it so I can't reject the null. This does make some sense because these two genres tend to have a lot of overlap in intrests and some individuals might consider a fantasy novel to be fiction which would definetly skew the data one way or another.

To double check the distribution of user raitings I took the difference between the average fantasy rating and the average fiction and found that on average, there is only a -.01 point difference between how individuals rate fantasy and fiction. To me this shows that if you like fiction or fantasy, you will mostlikly like the other genre equally as much.

### Science-Fiction and Fantasy

I then compared were science and religion.
![alt text](images/fig_Science-Fiction.png)
These results suprised me 49%

### Fiction and Vampire

After seeing the results for fiction and fantasy, I decided to take a look at the genre called Vampire, which due to the vitrial around twilight and its ripoffs I though would produce intresting results, and I was right.
![alt text](images/fig_Fiction.png)
Acording to my code there is not a single person who has read 4 books of the fiction genre and the vampire genre. I can take this two ways, 1 that people who like either genre avoid the other, but I cannot accept this to be true. the most likley result is that my dataset and data cleaning methods are reducing the total amount of books and people in each genre. For example lets think of the fictional book, <b>Mary Popins and the Vampire Apoclypse</b>. How my code works is that it takes a look at the most used tag for each book and makes that the genre of the book, well people who might have read <b>Mary Popins and the Vampire Apoclypse</b> gave it the tags, fiction, fantasy, vampire, ya, and post-apocolyptic. If fiction was the most used tag my code would result in <b>Mary Popins and the Vampire Apoclypse</b> becoming part of the fiction genre. When I then compare ficiton and vampire, this book only counts as a fiction book and all the readers of this book count as having read a fiction book so if they have only read 2 other fiction books, or no other vampire books, they are not included when I compare the two genres against eachother.

### Top 10 tags

After having this relisations, I decided to look at how the ten most used tags compare against eachother. In order to acomplish this I seleceted the ten most used tags and then ran the resulting list through the combinations tool to produce all the combinations.
```python
def massTagComb(df, num_tag):
        df_user = df[["tag_name"]].sort_values(by=['tag_name'])
        tag_10 = df_user['tag_name'].value_counts().reset_index()
        tag_10 = tag_10.iloc[:num_tag,0]
        tag_comb = combinations(tag_10, 2)
        return tag_comb, tag_10.tolist()
```
I then ran these combinations through my code and resulted with the following values.
## temp head
|Combinations | Bleed_Over | Avg_Raiting_diff|
| ------------- |:-------------:|:-------------|
|(fiction, fantasy)|0.4903|1.269258e-02|
|(fiction, young-adult)|0.0000|NaN|
|(fiction, mystery)|0.0000|NaN|
|(fiction, non-fiction)|0.4287|-1.535714e-01|
|(fiction, romance)|0.0000|NaN|
|(fiction, historical-fiction)|0.0000|NaN|
|(fiction, classics)|0.5076|-5.630631e-03
|(fiction, childrens)|0.0000|NaN|
|(fiction, science-fiction)|0.4956|4.229421e-17|
|(fantasy, young-adult)|0.0000|NaN|
|(fantasy, mystery)|0.0000|NaN|
|(fantasy, non-fiction)|0.5587|3.174603e-03|
|(fantasy, romance)|0.0000|NaN|
|(fantasy, historical-fiction)|0.0000|NaN|
|(fantasy, classics)|0.3292|-2.366667e-01|
|(fantasy, childrens)|0.0000|NaN|
|(fantasy, science-fiction)|0.4114|-2.096693e-01|
|(young-adult, mystery)|0.0000|NaN|
|(young-adult, non-fiction)|0.0000|NaN|
|(young-adult, romance)|0.0000|NaN|
|(young-adult, historical-fiction)|0.0000|NaN|
|(young-adult, classics)|0.0000|NaN|
|(young-adult, childrens)|0.0000|NaN|
|(young-adult, science-fiction)|0.0000|NaN|
|(mystery, non-fiction)|0.0000|NaN|
|(mystery, romance)|0.0000|NaN|
|(mystery, historical-fiction)|0.0000|NaN|
|(mystery, classics)|0.0000|NaN|
|(mystery, childrens)|0.0000|NaN|
|(mystery, science-fiction)|0.0000|NaN|
|(non-fiction, romance)|0.0000|NaN|
|(non-fiction, historical-fiction)|0.0000|NaN|
|(non-fiction, classics)|0.0000| NaN|
|(non-fiction, childrens)|0.0000|NaN|
|(non-fiction, science-fiction)|0.0000|0.000000e+00|
|(romance, historical-fiction)|0.0000|NaN|
|(romance, classics)|0.0000|NaN|
|(romance, childrens)|0.0000|NaN|
|(romance, science-fiction)|0.0000|NaN|
|(historical-fiction, classics)|0.0000|NaN|
|(historical-fiction, childrens)|0.0000|NaN|
|(historical-fiction, science-fiction)|0.0000|NaN|
|(classics, childrens)|0.0000|NaN|
|(classics, science-fiction)|0.4981|-3.916667e-01|
|(childrens, science-fiction)|0.0000|NaN|

## Linear Regresion

### Ridge

### Lasso