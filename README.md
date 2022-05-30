AniList.co offers a free GraphQL API with loads of cool info about anime.

As a anime fan, I have a few questions about trends in anime popularity that the pre-canned stats on these website don't answer such as:

* Are certain anime relatively more popular in some seasons than others
* Are there any interesting trends in popularity of certain genres especially since the coronavirus pandemic
* Some anime are based on manga, and some are original. Are there certain source material types for anime that do better than others?

Let's see if we can find some cool insights!

On the AniList API, the `media` page which has basic info about virtually every anime.

With a little time looking at the documentation i was able to set up a request filtered to:

* Anime Only
* Japanese Made Only
* Start Date after Dec 31, 2012
* Aired on TV


```python
import requests
def make_request(page_num,media_type):
    query = '''
    query ( $page: Int, $perPage: Int
  , $startDate_greater: FuzzyDateInt
,$countryOfOrigin:CountryCode
,$type:MediaType) {
        Page (page: $page, perPage: $perPage) {
            pageInfo {
                total
                currentPage
                lastPage
                hasNextPage
                perPage
            }
            media (startDate_greater:$startDate_greater,
            countryOfOrigin:$countryOfOrigin,
            type:$type) {
                id
                title {
                    english
                    native
                }
                season
                seasonYear
                type
                format
                status
                episodes
                duration
                chapters
                volumes
                isAdult
                averageScore
                popularity
                source
                countryOfOrigin
                isLicensed
                genres
                startDate {
                    day
                    month
                    year
                  }
                endDate {
                    day
                    month
                    year
                  }
            }
        }
    }
    '''
    variables = {
        "startDate_greater": 20121231 # Fuzzy Date for Dec 31, 2012
  ,"countryOfOrigin": "JP"
  ,"type": media_type
  ,"page": page_num
  ,"perPage": 500
  ,"format": "TV"
    }
    
    url = 'https://graphql.anilist.co'

    response = requests.post(url, json={'query': query, 'variables': variables})
    # response = requests.post(url, json={'query': query, 'variables': variables},verify=False)
    return response
```

Testing that function, I was getting some weird responses with anything more than 50 records per page. So I'll limit `perPage` to 50.

Let's then loop that 9999 times use the handy `hasNextPage` attribute to break the loop. And throw in an `if` statement in case we get a bad response that's not `=200`


```python
import json
import pandas as pd
from IPython.display import clear_output
df_media=pd.DataFrame()

hasNextPage=True
for i in range(1,9999):
    clear_output()
    print(i)
    response=make_request(i,"ANIME")
    json_data = json.loads(response.text)
    if response.status_code==200:
        df_new=pd.json_normalize(json_data['data']['Page']['media'], meta  =[['english', 'native']])
        if len(json_data['data']['Page']['media'])>0:
            df_media=pd.concat([df_media,df_new])
        if json_data['data']['Page']['pageInfo']['hasNextPage']==False:
            break
df_media.reset_index(inplace=True,drop=True)
df_media.sort_values(['startDate.year','startDate.month','startDate.day'])
display(df_media)
```

    178
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>season</th>
      <th>seasonYear</th>
      <th>type</th>
      <th>format</th>
      <th>status</th>
      <th>episodes</th>
      <th>duration</th>
      <th>chapters</th>
      <th>volumes</th>
      <th>...</th>
      <th>isLicensed</th>
      <th>genres</th>
      <th>title.english</th>
      <th>title.native</th>
      <th>startDate.day</th>
      <th>startDate.month</th>
      <th>startDate.year</th>
      <th>endDate.day</th>
      <th>endDate.month</th>
      <th>endDate.year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3786</td>
      <td>WINTER</td>
      <td>2021.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>155.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Drama, Mecha, Psychological, Sci-Fi]</td>
      <td>Evangelion: 3.0+1.0 Thrice Upon a Time</td>
      <td>シン・エヴァンゲリオン劇場版:||</td>
      <td>8.0</td>
      <td>3.0</td>
      <td>2021</td>
      <td>8.0</td>
      <td>3.0</td>
      <td>2021.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9260</td>
      <td>WINTER</td>
      <td>2016.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>64.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Drama, Ecchi, Mystery, Psychological,...</td>
      <td>Kizumonogatari Part 1: Tekketsu</td>
      <td>傷物語〈Ⅰ鉄血篇〉</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>2016</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>2016.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9479</td>
      <td>FALL</td>
      <td>2013.0</td>
      <td>ANIME</td>
      <td>TV</td>
      <td>FINISHED</td>
      <td>13.0</td>
      <td>24.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Sci-Fi]</td>
      <td>Coppelion</td>
      <td>コッペリオン</td>
      <td>3.0</td>
      <td>10.0</td>
      <td>2013</td>
      <td>25.0</td>
      <td>12.0</td>
      <td>2013.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9488</td>
      <td>SPRING</td>
      <td>2019.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Sci-Fi]</td>
      <td>None</td>
      <td>センコロール2</td>
      <td>29.0</td>
      <td>6.0</td>
      <td>2019</td>
      <td>29.0</td>
      <td>6.0</td>
      <td>2019.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9851</td>
      <td>SUMMER</td>
      <td>2013.0</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>6.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Comedy, Hentai, Mecha]</td>
      <td>None</td>
      <td>オーパーツ　オーマン</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>2013</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>2013.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>115918</td>
      <td>WINTER</td>
      <td>2018.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>120.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Action, Comedy, Mecha, Sci-Fi]</td>
      <td>None</td>
      <td>フルメタル・パニック！イントゥ・ザ・ブルー</td>
      <td>20.0</td>
      <td>1.0</td>
      <td>2018</td>
      <td>20.0</td>
      <td>1.0</td>
      <td>2018.0</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>115925</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>ONA</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Comedy, Slice of Life]</td>
      <td>None</td>
      <td>かくしごとPV</td>
      <td>16.0</td>
      <td>6.0</td>
      <td>2016</td>
      <td>16.0</td>
      <td>6.0</td>
      <td>2016.0</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>115927</td>
      <td>WINTER</td>
      <td>2019.0</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Comedy]</td>
      <td>None</td>
      <td>劇場版 幼女戦記 マナー映像</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>2019</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>2019.0</td>
    </tr>
    <tr>
      <th>4498</th>
      <td>115931</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>MUSIC</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[]</td>
      <td>None</td>
      <td>ヒトガタ</td>
      <td>20.0</td>
      <td>2.0</td>
      <td>2019</td>
      <td>20.0</td>
      <td>2.0</td>
      <td>2019.0</td>
    </tr>
    <tr>
      <th>4499</th>
      <td>115955</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>True</td>
      <td>[Comedy]</td>
      <td>None</td>
      <td>劇場版K MISSING KINGS予告映像</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>2014</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>2014.0</td>
    </tr>
  </tbody>
</table>
<p>4500 rows × 25 columns</p>
</div>


Okay awesome, that looks good.

I want to see if certain genres are more popular in different seasons of the year.
Let's see what those columns look like.


```python
display(df_media[["title.english","title.native",'genres']])
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title.english</th>
      <th>title.native</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Evangelion: 3.0+1.0 Thrice Upon a Time</td>
      <td>シン・エヴァンゲリオン劇場版:||</td>
      <td>[Action, Drama, Mecha, Psychological, Sci-Fi]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kizumonogatari Part 1: Tekketsu</td>
      <td>傷物語〈Ⅰ鉄血篇〉</td>
      <td>[Action, Drama, Ecchi, Mystery, Psychological,...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Coppelion</td>
      <td>コッペリオン</td>
      <td>[Action, Sci-Fi]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>None</td>
      <td>センコロール2</td>
      <td>[Action, Sci-Fi]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>None</td>
      <td>オーパーツ　オーマン</td>
      <td>[Action, Comedy, Hentai, Mecha]</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>None</td>
      <td>フルメタル・パニック！イントゥ・ザ・ブルー</td>
      <td>[Action, Comedy, Mecha, Sci-Fi]</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>None</td>
      <td>かくしごとPV</td>
      <td>[Comedy, Slice of Life]</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>None</td>
      <td>劇場版 幼女戦記 マナー映像</td>
      <td>[Comedy]</td>
    </tr>
    <tr>
      <th>4498</th>
      <td>None</td>
      <td>ヒトガタ</td>
      <td>[]</td>
    </tr>
    <tr>
      <th>4499</th>
      <td>None</td>
      <td>劇場版K MISSING KINGS予告映像</td>
      <td>[Comedy]</td>
    </tr>
  </tbody>
</table>
<p>4500 rows × 3 columns</p>
</div>


We've got multiple genres for each row in the `genres` column, so let's one hot encode them.


```python
from sklearn.preprocessing import MultiLabelBinarizer
mlb = MultiLabelBinarizer()
df_hot = df_media.join(pd.DataFrame(mlb.fit_transform(df_media.pop('genres')),
                          columns=mlb.classes_,
                          index=df_media.index))

df_hot['averageScore']=pd.to_numeric(df_hot['averageScore'])
display(df_hot[["title.english","title.native",'Mahou Shoujo','Horror','Mecha']])
print('Original Dataframe has '+str(df_media.shape[1])+' columns')
print('One Hot Encoded Dataframe has '+str(df_hot.shape[1])+' columns')
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title.english</th>
      <th>title.native</th>
      <th>Mahou Shoujo</th>
      <th>Horror</th>
      <th>Mecha</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Evangelion: 3.0+1.0 Thrice Upon a Time</td>
      <td>シン・エヴァンゲリオン劇場版:||</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kizumonogatari Part 1: Tekketsu</td>
      <td>傷物語〈Ⅰ鉄血篇〉</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Coppelion</td>
      <td>コッペリオン</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>None</td>
      <td>センコロール2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>None</td>
      <td>オーパーツ　オーマン</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>None</td>
      <td>フルメタル・パニック！イントゥ・ザ・ブルー</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>None</td>
      <td>かくしごとPV</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>None</td>
      <td>劇場版 幼女戦記 マナー映像</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4498</th>
      <td>None</td>
      <td>ヒトガタ</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4499</th>
      <td>None</td>
      <td>劇場版K MISSING KINGS予告映像</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>4500 rows × 5 columns</p>
</div>


    Original Dataframe has 24 columns
    One Hot Encoded Dataframe has 43 columns
    

That added about 20 columns so lets get rid of some where there weren't too many records so our plot isn't too messy

First let's find how many of each genre there are. The One Hot Coded Genre columns start at index=24


```python
genre_list=df_hot.iloc[:, list(range(24, df_hot.shape[1]))].sum().sort_values(0,ascending=False)
display(genre_list)
```

    C:\Users\Spruce\AppData\Local\Temp\ipykernel_10520\1044468406.py:1: FutureWarning: In a future version of pandas all arguments of Series.sort_values will be keyword-only.
      genre_list=df_hot.iloc[:, list(range(24, df_hot.shape[1]))].sum().sort_values(0,ascending=False)
    


    Comedy           1656
    Action           1134
    Slice of Life     892
    Fantasy           875
    Drama             696
    Romance           647
    Sci-Fi            546
    Adventure         500
    Supernatural      480
    Hentai            419
    Ecchi             313
    Music             285
    Mystery           254
    Psychological     237
    Sports            228
    Mecha             208
    Mahou Shoujo      126
    Horror            110
    Thriller           64
    dtype: int64


Wow **Comedy** and **Action** apply to a lot of titles.

Okay let's drop the genres from our dataframe that have less than 500 titles so we're not looking at outliers and our charts aren't too cluttered.


```python
genre_list=genre_list.where(genre_list<500).dropna()
df_hot.drop(genre_list.keys(),axis=1,inplace=True)
display(df_hot)
print('After dropping niche genres, Dataframe has '+str(df_hot.shape[1])+' columns')
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>season</th>
      <th>seasonYear</th>
      <th>type</th>
      <th>format</th>
      <th>status</th>
      <th>episodes</th>
      <th>duration</th>
      <th>chapters</th>
      <th>volumes</th>
      <th>...</th>
      <th>endDate.month</th>
      <th>endDate.year</th>
      <th>Action</th>
      <th>Adventure</th>
      <th>Comedy</th>
      <th>Drama</th>
      <th>Fantasy</th>
      <th>Romance</th>
      <th>Sci-Fi</th>
      <th>Slice of Life</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3786</td>
      <td>WINTER</td>
      <td>2021.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>155.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>3.0</td>
      <td>2021.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9260</td>
      <td>WINTER</td>
      <td>2016.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>64.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>1.0</td>
      <td>2016.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9479</td>
      <td>FALL</td>
      <td>2013.0</td>
      <td>ANIME</td>
      <td>TV</td>
      <td>FINISHED</td>
      <td>13.0</td>
      <td>24.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>12.0</td>
      <td>2013.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9488</td>
      <td>SPRING</td>
      <td>2019.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>6.0</td>
      <td>2019.0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9851</td>
      <td>SUMMER</td>
      <td>2013.0</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>6.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>7.0</td>
      <td>2013.0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>115918</td>
      <td>WINTER</td>
      <td>2018.0</td>
      <td>ANIME</td>
      <td>MOVIE</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>120.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>1.0</td>
      <td>2018.0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>115925</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>ONA</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>6.0</td>
      <td>2016.0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>115927</td>
      <td>WINTER</td>
      <td>2019.0</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>2.0</td>
      <td>2019.0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4498</th>
      <td>115931</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>MUSIC</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>2.0</td>
      <td>2019.0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4499</th>
      <td>115955</td>
      <td>None</td>
      <td>NaN</td>
      <td>ANIME</td>
      <td>SPECIAL</td>
      <td>FINISHED</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>7.0</td>
      <td>2014.0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>4500 rows × 32 columns</p>
</div>


    After dropping niche genres, Dataframe has 32 columns
    

Perfect, we went from 43 to 34 columns, so we should have a more focused list of genres in our dataset.

Lets look at two variables:

1. **Viewership** - We'll want to divide out the number of titles released from the popularity (which represents total views) to get to an average viewership for each title in a genre.
2. **Ranking** - Average User Ranking for each genre

Let's see what it looks like!


```python
df_season=pd.DataFrame(columns=['index','Season','Viewership'])
for x in range(24,df_hot.shape[1]):
    colname = df_hot.columns[x]

    df_season_genre=df_hot[df_hot.iloc[:, x] == 1]

    series_plot_me_count=df_season_genre.groupby('season')['id'].count()
    series_plotme_sumPopularity=df_season_genre.groupby('season')['popularity'].sum()
    series_plotme_avgRanking=df_season_genre.groupby('season')['averageScore'].mean()
    df_season_genre=series_plotme_sumPopularity/series_plot_me_count

    df_season_genre=pd.DataFrame({ 'Season': df_season_genre.keys(), 'Viewership': df_season_genre.values, 'Ranking': series_plotme_avgRanking.values })
    season_order = ["SPRING", "SUMMER", "FALL", "WINTER"]
    df_season_genre=df_season_genre.reset_index().set_index("Season").loc[season_order]
    df_season_genre['Genre']=colname
    if df_season.empty:
        df_season=df_season_genre.copy(deep=True)
    else:
        df_season=pd.concat([df_season,df_season_genre])

df_season.index.name = 'Season'
df_season.reset_index(inplace=True)
df_season.drop(['index'],axis=1,inplace=True)
display(df_season.head(12))
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Viewership</th>
      <th>Ranking</th>
      <th>Genre</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SPRING</td>
      <td>38993.264493</td>
      <td>65.260223</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>1</th>
      <td>SUMMER</td>
      <td>35643.308943</td>
      <td>65.100418</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FALL</td>
      <td>32506.825397</td>
      <td>65.573290</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>3</th>
      <td>WINTER</td>
      <td>35529.513725</td>
      <td>64.912698</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SPRING</td>
      <td>42188.500000</td>
      <td>64.051282</td>
      <td>Adventure</td>
    </tr>
    <tr>
      <th>5</th>
      <td>SUMMER</td>
      <td>39584.674797</td>
      <td>65.593220</td>
      <td>Adventure</td>
    </tr>
    <tr>
      <th>6</th>
      <td>FALL</td>
      <td>39064.129032</td>
      <td>66.379310</td>
      <td>Adventure</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WINTER</td>
      <td>46110.980000</td>
      <td>67.577320</td>
      <td>Adventure</td>
    </tr>
    <tr>
      <th>8</th>
      <td>SPRING</td>
      <td>26481.609813</td>
      <td>63.787402</td>
      <td>Comedy</td>
    </tr>
    <tr>
      <th>9</th>
      <td>SUMMER</td>
      <td>24924.911227</td>
      <td>65.116343</td>
      <td>Comedy</td>
    </tr>
    <tr>
      <th>10</th>
      <td>FALL</td>
      <td>24012.271357</td>
      <td>64.225201</td>
      <td>Comedy</td>
    </tr>
    <tr>
      <th>11</th>
      <td>WINTER</td>
      <td>25908.562667</td>
      <td>65.255014</td>
      <td>Comedy</td>
    </tr>
  </tbody>
</table>
</div>


Okay great, got the dataframe, now to plot...


```python
import seaborn as sns
from seaborn import lineplot
sns.set(rc={'figure.figsize':(11.7,8.27)})

lineplot(data=df_season,x="Season",y="Viewership",hue="Genre",style='Genre',size='Genre',sizes=(4,4)).set(title='Anime Viewership By Season And Genre')
```




    [Text(0.5, 1.0, 'Anime Viewership By Season And Genre')]




    
![png](README_files/README_15_1.png)
    


Okay, well that's a hot mess. We can see which genres have higher average Viewership, but that's about it with this viz.

**Drama**, and **Supernatural** are the winners in terms of Viewership, and **Hentai** is the biggest loser

However, it's pretty hard to make out any interesting seasonal trends by genre

To see those, let's normalize each Genre by its average Viewership and then represent the season-to-season differenes as percents instead of absolute values. (That way the higher absolute Viewerships don't look skewed just because they have larger seasonal swings on an absolute scale)


```python
viewership_offset_genre=df_season.groupby('Genre')['Viewership'].mean()
ranking_offset_genre=df_season.groupby('Genre')['Ranking'].mean()

df_norm=df_season.merge(viewership_offset_genre,on='Genre')
df_norm=df_norm.merge(ranking_offset_genre,on='Genre')

df_norm['Viewership']=(df_norm['Viewership_x']-df_norm['Viewership_y'])/df_norm['Viewership_y']
df_norm['Ranking']=(df_norm['Ranking_x']-df_norm['Ranking_y'])/df_norm['Ranking_y']

df_norm.drop(['Viewership_x','Viewership_y','Ranking_x','Ranking_y'],axis=1,inplace=True)
```


```python
lineplot(data=df_norm,x="Season",y="Viewership",hue="Genre",style='Genre',size='Genre',sizes=(4,4)).set(title='Normalized Anime Viewership By Season And Genre')
```




    [Text(0.5, 1.0, 'Normalized Anime Viewership By Season And Genre')]




    
![png](README_files/README_18_1.png)
    


What's interesting here is in *Summer*, some of the realistic genres like **Romance**, **Slice of Life**, and **Drama** gain more viewership, whereas **Fantasy**, **Adventure**, and **Supernatural** tend to have less relative Viewership. I wonder if that is due to the regular things in life like sunshine, hot dogs, and warm weather making real life seem not so bad. Whereas when it's cold, you'd rather choose something more escapist.

The clearest trend is generally all genres have an dip in Viewership in *Fall* and increase in *Winter*. That makes sense just with people staying inside more with the cold weather as well as New Years holidays.

The exception to that is **Supernatural** anime that come out in *Winter* have much worse viewership than any other time. 

I wonder if Rankings show the same trend


```python
lineplot(data=df_norm,x="Season",y="Ranking",hue="Genre",style='Genre',size='Genre',sizes=(4,4)).set(title='Normalized Anime Viewership By Season And Genre')
```




    [Text(0.5, 1.0, 'Normalized Anime Viewership By Season And Genre')]




    
![png](README_files/README_20_1.png)
    


In terms of Ratings, **Adventure** is rated very well in *Winter* relative to *Spring*. Not surprising that people would want to go on an adventure when they're stuck inside all **Winter**

We also see **Supernatural** again doing poorly in Ranking in *Winter* as well as in Viewership. I wonder if people have a hard time believing in gods in *Winter* when everything outside is cold and dead

**Romance** does especially well in the *Spring* in terms of Viewership and Rankings. I guess thats Why they call it the *"Season of Love"*

Let's look at change in popularity or ranking over the years


```python
df_year=pd.DataFrame(columns=['index','Season','Viewership'])
for x in range(24,df_hot.shape[1]):
    colname = df_hot.columns[x]

    df_year_genre=df_hot[df_hot.iloc[:, x] == 1]

    series_plot_me_count=df_year_genre.groupby('seasonYear')['id'].count()
    series_plotme_sumPopularity=df_year_genre.groupby('seasonYear')['popularity'].sum()
    series_plotme_avgRanking=df_year_genre.groupby('seasonYear')['averageScore'].mean()
    df_year_genre=series_plotme_sumPopularity/series_plot_me_count

    df_year_genre=pd.DataFrame({ 'Year': df_year_genre.keys(), 'Viewership': df_year_genre.values, 'Ranking': series_plotme_avgRanking.values })
    # season_order = ["SPRING", "SUMMER", "FALL", "WINTER"]
    # df_year_genre=df_year_genre.reset_index().set_index("Season").loc[season_order]
    df_year_genre['Genre']=colname
    if df_year.empty:
        df_year=df_year_genre.copy(deep=True)
    else:
        df_year=pd.concat([df_year,df_year_genre])

df_year.reset_index(inplace=True)
df_year.drop(['index'],axis=1,inplace=True)
display(df_year.head(12))
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Viewership</th>
      <th>Ranking</th>
      <th>Genre</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2013.0</td>
      <td>28148.125000</td>
      <td>64.981132</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014.0</td>
      <td>38944.248000</td>
      <td>65.208000</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015.0</td>
      <td>39432.744526</td>
      <td>65.781022</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016.0</td>
      <td>30368.227545</td>
      <td>64.746988</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017.0</td>
      <td>28901.411765</td>
      <td>65.661654</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018.0</td>
      <td>34072.834356</td>
      <td>63.761006</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019.0</td>
      <td>41999.711111</td>
      <td>65.338462</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020.0</td>
      <td>44603.746667</td>
      <td>66.739726</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021.0</td>
      <td>48274.441176</td>
      <td>68.666667</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2022.0</td>
      <td>24329.125000</td>
      <td>60.200000</td>
      <td>Action</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2013.0</td>
      <td>21806.729730</td>
      <td>67.029412</td>
      <td>Adventure</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2014.0</td>
      <td>48164.338983</td>
      <td>67.875000</td>
      <td>Adventure</td>
    </tr>
  </tbody>
</table>
</div>



```python
lineplot(data=df_year,x="Year",y="Ranking",hue="Genre",style='Genre',size='Genre',sizes=(4,4)).set(title='Anime Viewership By Year And Genre')
```




    [Text(0.5, 1.0, 'Anime Viewership By Year And Genre')]




    
![png](README_files/README_26_1.png)
    


Wow! Since 2020 the **Romance**, **Comedy** and **Slice of Life** animes that have come out have garnered more viewership in 2021 and 2022 than they used to. That may be a sad story of people wanting a return to normal life and normal romance in the wake of the pandemic... or to just have a normal laugh.

No increase in the average viewership for a **Hentai** anime as people spent more time at home during the pandemic. Slighlty surprising, but promising.

Next let's take a look see if anime based on manga are more popular than original anime.

First let's see what all types of source material exist in this dataset. The column is called `source`


```python
df_media.groupby('source').size().sort_values(ascending=False)
```




    source
    ORIGINAL        1490
    MANGA           1454
    OTHER            409
    LIGHT_NOVEL      377
    VIDEO_GAME       342
    VISUAL_NOVEL     267
    dtype: int64



Interesting, there's a few more options for source material than just manga. 

Let's see the average ranking of each


```python
import matplotlib.pyplot as plt

df_media['averageScore']=pd.to_numeric(df_media['averageScore'])

summarize_popularity=df_media.groupby('source')['averageScore'].mean()

fig = plt.figure(figsize = (10, 5))
plt.bar(summarize_popularity.keys(), summarize_popularity.values, color ='red',
        width = 0.4)
plt.title('Anime Rankings By Source Material Type')
```




    Text(0.5, 1.0, 'Anime Rankings By Source Material Type')




    
![png](README_files/README_31_1.png)
    


Surprisingly, not a huge difference in ranking between any genre. `LIGHT_NOVEL` and `MANGA` are slightly ahead but probably not significantly.

What about... Viewership?


```python
series_plot_me_count=df_media.groupby('source')['id'].count()
series_plotme_avgPopularity=df_media.groupby('source')['popularity'].sum()

df_plotme=series_plotme_avgPopularity/series_plot_me_count

import matplotlib.pyplot as plt

fig = plt.figure(figsize = (10, 5))
plt.bar(df_plotme.keys(), df_plotme.values, color ='green',
        width = 0.4)
plt.title('Anime Viewership By Source Material Type')
```




    Text(0.5, 1.0, 'Anime Viewership By Source Material Type')




    
![png](README_files/README_33_1.png)
    


Wow okay, we've got a clear winner here as anime based on `LIGHT_NOVEL` have way more viewership than other source material types. Also we have a clear 2nd place in `MANGA`

I wonder if that's entirely fair to say or if there's more to that story. Let's trend these over the years and see...


```python
series_plot_me_count=df_media.groupby(['source','seasonYear'],as_index=False)['id'].count()
df_plotme=df_media.groupby(['source','seasonYear'],as_index=False)['popularity'].sum()
df_plotme['Viewership']=df_plotme['popularity']/series_plot_me_count['id']

pd.DataFrame({'Source':df_plotme['source']})


import seaborn as sns
sns.set_theme(style="whitegrid")

# df_media['avgPopularity']=df_media['avgPopularity']/df_media['avgPopularity']

# Draw a nested barplot by species and sex
g = sns.catplot(
    data=df_plotme, kind="bar",
    x="seasonYear", y="Viewership", hue="source",
    ci="sd", palette="dark", alpha=.6, height=6
)
g.set(title='Yearly Anime Viewership Grouped By Genre')
g.despine(left=True)
g.set_axis_labels("Year", "avgPopularity")
g.legend.set_title("Anime Populartiy by Genre and Year")
```


    
![png](README_files/README_35_0.png)
    


Wow, year over year, Anime based on `LIGHT_NOVEL` and `MANGA` out-perform others consistently, no outliers. That's very interesting.

In conclusion, I think we definitely were able to get some cool insights.

This was really fun to make as I love anime

Here is my myanimelist in case you want to know which anime I really like:

https://myanimelist.net/animelist/sprucemister

Thank you for reading!
