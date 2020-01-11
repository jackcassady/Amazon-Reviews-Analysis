# Amazon-Reviews-Analysis

Analyzing reviews of products on Amazon

## Data

The data consists of reviews of 13 different Amazon products. There are 13 different files for the reviews, 5 sets of reviews, a set of sample reviews used to test functions, and a json file to store mappings of products to their asin's. Each set of reviews has a csv file containing the majority of the data for the review itself, including review id, review title, review description referred to review text, rating of the product, whether or not it is recommended by the reviewer, number of people that found the review helpful, and the date of the review. Each set of reviews also has a json file that contains mappings of review ids to reviewer username and the asin of the product being reviewed. When analyzing the data, I also found that the json file for review 4 was corrupted, these reviews were then recovered by the broken_file folder. 

## Functions Used

Read json files by their name and parse out data:
```python
def read_json(filename):
    
    file = open(os.path.join("data", filename+".json"), encoding="utf-8")
    json_str = file.read()
    file.close()

    data = json.loads(json_str)
    return data
```

Read json files by their path and parse data:
```python
def read_json_path(path):
    
    file = open(path, encoding="utf-8")
    json_str = file.read()
    file.close()

    data = json.loads(json_str)
    return data
```

Read csv files and output specified data from reviews:
```python
def read_csv(filename, number, category):
    
    f = open(os.path.join('data', filename+'.csv'), encoding='utf-8') # opening csv
    f_reader = csv.reader(f) # read file
    f_data = list(f_reader) # convert data to list
    f.close() # close file
    header = f_data[0] # specify headers
    data = f_data[1:] # specify data
    for review in data: # loop through data
        if review[header.index('review id')] == number: # find review in data with the input review number 
            return review[header.index(category)] # return data of the input category
```

Get review data from respective json and csv files and combining data from both to output a list of reviews formatted in the Review named tuple:
```python
def get_reviews(file_json, file_csv):
    reviews = [] # initialize list of reviews
    
    with open(os.path.join("data", file_json), encoding="utf-8") as file: # open json file
        try: # error-handling in case of broken files
            json_str = file.read() # read json file
            data_json = json.loads(json_str) # load json data
            file.close() # close file
        except:
            return None # if file is broken, function skips it
    

    f = open(os.path.join('data', file_csv), encoding='utf-8') # open csv file
    f_reader = csv.reader(f) # read csv
    f_data = list(f_reader) # convert data to list
    f.close() 
    header = f_data[0] # specify header
    data_csv = f_data[1:] # specify data
    
    
    rv = {} # initialize review dict
    for review in data_csv: # loop through data from csv
        try: # error-handling for broken files
            rv['review_id'] = int(review[0]) # set review id as int for review dict
            rv['review_title'] = review[header.index('review title')] # set review title in review dict
            rv['review_text'] = review[header.index('review text')] # set review text in review dict
            rv['review_rating'] = int(review[header.index('review rating')]) # set review rating as int in review dict
            if review[header.index('review do_recommend')] == 'True': # setting and converting recommendations to boolean values
                rv['review_recommend'] = True
            elif review[header.index('review do_recommend')] == 'False':
                rv['review_recommend'] = False
            rv['review_helpful'] = int(review[header.index('review num_helpful')]) # set number helpful as int
            rv['review_date'] = review[header.index('review date')] # set review date in review dict
            for i in data_json: # loop through data from json
                if int(i) == rv['review_id']: # find current iteration review in json file
                    rv['uname'] = data_json[i][0] # set username to dict
                    rv['asin'] = data_json[i][1] # set product asin to dict
            review = Review(rv['review_id'], # organize dict into review named tuple
                        rv['uname'], 
                        rv['asin'], 
                        rv['review_title'], 
                        rv['review_text'], 
                        rv['review_rating'], 
                        rv['review_recommend'], 
                        rv['review_helpful'], 
                        rv['review_date'])
            reviews.append(review) # append current iteration review to list
        except:
            continue # if review is broken, skip it
    return reviews # return list of reviews from review files
```

Function used to sort by reivew id:
```python
def review_id(review):
    return review.id
```

Function used to sort by number helpful:
```python
def most_helpful(review):
    return review.num_helpful
```

Creates a scatter plot of a pandas dataframe:
```python
def scatter(x, y, xlabel="please label me!", ylabel="please label me!"):
    df = pd.DataFrame({"x":x, "y":y})
    ax = df.plot.scatter(x="x", y="y", color="black", fontsize=16, xlim=0, ylim=0)
    ax.set_xlabel(xlabel, fontsize=16)
    ax.set_ylabel(ylabel, fontsize=16)
    ax.get_xaxis().get_major_formatter().set_scientific(False)
    ax.get_yaxis().get_major_formatter().set_scientific(False)
```

Used to sort by number of reviews:
```python
def get_key(item):
    return item[1]
```

Returns average rating of a product:
```python
def avg_rating(asin):
    rating = 0
    count = 0
    r = []
    for review in reviews:
        if review.asin == str(asin):
            if review not in r:
                r.append(review)
    for review in r:
        rating += review.rating
        count = len(r)
    avg_rating = rating/count
    return avg_rating
```

Returns the number of reviews for a product:
```python
def get_num_reviews(asin):
    rev = []
    for review in reviews:
        if review.asin == str(asin):
            if review not in rev:
                rev.append(review)
    for review in rev:
        number = len(rev)
    
    return number
```

Returns the average length of review text for reviews in each rating:
```python
def average_text(values):
    
    averages = {}
    for rating in values:
        count = 0
        total = 0
        for review in values[rating]:
            if review.rating == rating:
                total += len(review.text)
                count += 1
                avg = total/count
            averages[rating] = avg
    return averages
```

Finding the percentage of reviews that were found helpful:
```python
def percent_help(values):
    
    percents = {}
    for rating in values:
        count = 0
        total = 0
        for review in values[rating]:
            if review.rating == rating:
                if review.num_helpful >= 1:
                    total += 1
                count += 1
                percent = total/count
            percents[rating] = percent
    return percents
```

Finding the percentage that recommended the product by rating:
```python
def recommend(values):
    
    do_recommend = {}
    for rating in values:
        count = 0
        total = 0
        for review in values[rating]:
            if review.rating == rating:
                if review.do_recommend == True:
                    total += 1
                count += 1
                percent = total/count
            do_recommend[rating] = percent
    return do_recommend
```

Takes a path and looks recursively at anything contained in that directory:
```python
def explore_directory(path):
    paths = [] # list of paths in path
    if os.path.isfile(path) == True:
        paths.append(path) # if it's a file add to paths
    else: # otherwise, explore that directory
        for i in os.listdir(path):
            if i[0] == '.': # skip any files beginning with '.'
                continue
            paths += explore_directory(os.path.join(path, i)) # begin sequence again with new path
    return paths
```

Writing json file:
```python
def write_json(data, filename):
    with open(filename, 'w') as file:
        json.dump(data, file)
```

## Data Analysis

I'll be working with 13 products for the following analysis, looking at `products.json` we can see the names of each of these products as well as their asin. The average ratings are determined using the `avg_rating` function listed above, which utilizes all of the reviews in the dataset. The asin is the primary way that the reviews are linked to their products. 

Asin | Product | Average Rating
---- | ------- | --------------
B00QFQRELG | Amazon 9W PowerFast Official OEM USB Charger and Power Adapter for Fire Tablets and Kindle eReaders | 4.7272727272727275
B01BH83OOM | Amazon Tap Smart Assistant Alexa enabled (black) Brand New | 4.690909090909090
B00ZV9PXP2 | All-New Kindle E-reader - Black, 6" Glare-Free Touchscreen Display, Wi-Fi - Includes Special Offers | 4.590163934426229
B0751RGYJV | Amazon Echo (2nd Generation) Smart Assistant Oak Finish Priority Shipping | 5.0
B00IOY8XWQ | Kindle Voyage E-reader, 6 High-Resolution Display (300 ppi) with Adaptive Built-in Light, PagePress Sensors, Wi-Fi - Includes Special Offers | 4.666666666666667
B0752151W6 | All-new Echo (2nd Generation) with improved sound, powered by Dolby, and a new design Walnut Finish | 5.0
B018Y226XO | Fire Kids Edition Tablet, 7 Display, Wi-Fi, 16 GB, Pink Kid-Proof Case | 4.603448275862069
B01ACEKAJY | All-New Fire HD 8 Tablet, 8 HD Display, Wi-Fi, 32 GB - Includes Special Offers, Black | 4.583333333333333
B01AHB9CYG | All-New Fire HD 8 Tablet, 8 HD Display, Wi-Fi, 32 GB - Includes Special Offers, Magenta | 4.574468085106383
B01AHB9CN2 | All-New Fire HD 8 Tablet, 8 HD Display, Wi-Fi, 16 GB - Includes Special Offers, Magenta | 4.6
B00VINDBJK | Kindle Oasis E-reader with Leather Charging Cover - Merlot, 6 High-Resolution Display (300 ppi), Wi-Fi - Includes Special Offers | 4.866666666666666
B01AHB9C1E | Fire HD 8 Tablet with Alexa, 8 HD Display, 32 GB, Tangerine - with Special Offers | 3.8333333333333335
B018Y229OU | Fire Tablet, 7 Display, Wi-Fi, 8 GB - Includes Special Offers, Magenta | 4.490408673894913

After defining the products, I was working with, I went about trying to access reviews. I wanted to be able to check every review file for the review number I was looking for and then access the part of the review I needed. I used a couple cells in the jupyter notebook to test this function. 

```python
# review text of review 69899
# checking all files in data for review 69899
for file in files_in_data: 
    if file.split('.')[1] == 'csv':
        if read_csv(file.split('.')[0], '69899', 'review text') != None:
            review_69899 = read_csv(file.split('.')[0], '69899', 'review text')
review_69899
```

The program starts by looping through all the files in the `data` file, then it looks for the csv files that have all the review data and inputs each of the csv files into the read csv function from above to find the review number I am looking for.

My next task was to organize each of the reviews into a named tuple which would make it easier for me to access each part of the review. The initialization of the named tuple is:

```python
Review = namedtuple('Review', ['id', 'username', 'asin', 'title', 'text', 'rating', 'do_recommend', 'num_helpful', 'date'])
```

It takes categories from both the csv and json files of each review set and creates a named tuple for each individual review.

In order to get each of these reviews from all of the review files I used the `get_reviews` function from above. Each step of the function is described in comments within the code itself. 

The function returns a list containing all of the individual reviews in a list with all of the data. The first ten reviews are as follows:

ID | Username | Asin | Title | Text | Rating | Recommend | Num Helpful | Date
-- | -------- | ---- | ----- | ---- | ------ | --------- | ----------- | ----
10101 | Mikey123456789 | B00QFQRELG | A charger | It seems to work just like any other usb plug in charger. | 5 | True | 0 | 2017-01-02
99904 | diamond | B00QFQRELG | amazon power fast usb charger | got this for my kindle 7 tablet. Does an excellent job charging the kindle fire 7 a lot faster than the one it came with the kindle fire | 5 | True | 2 | 2016-06-03
89604 | Pat91 | B00QFQRELG | Amazon powerfast wall charger | Best kindle charger ever. Took 30 minutes to being my kindle back to life. | 5 | True | 0 | 2016-11-21
58704 | Frank | B00QFQRELG | correct plug for kindle | Quickly charges kindle so son can use it. Worked great right out of the package | 5 | True | 0 | 2016-10-14
38104 | LADYD92 | B00QFQRELG | Fast Charger | Bought this charger for the Kindle voyage and its great. | 5 | True | 0 | 2016-09-30
76407 | RobT | B00QFQRELG | Good charger | This wall charger works exactly as described for the Kindle Paperwhite. | 5 | True | 0 | 2016-07-22
83810 | Iodine | B00QFQRELG | Great item | Have been using this item and it seems to be working quite well. | 5 | True | 0 | 2017-03-15
32310 | Akki | B00QFQRELG | Nice one | Good one and working without any issues. Slim and portable | 5 | True | 0 | 2016-06-24
22010 | STRIPYGOOSE | B00QFQRELG | not any faster | it does not charge any faster than regular charger. | 3 | False | 0 | 2016-08-18
1410 | Jk60 | B00QFQRELG | Satisfied | It does what it is suppose to. No problems with it... | 4 | True | 0 | 2016-12-07
 
Looking at all of the reviews in the data, I found that the number of reviews for Amazon Tap Smart Assistant Alexa enabled (black) Brand New is 165 and the number of reviews for All-New Fire HD 8 Tablet, 8 HD Display, Wi-Fi, 32 GB - Includes Special Offers, Black is 12.
 
However, the product with the highest number of reviews was the Fire Tablet, 7 Display, Wi-Fi, 8 GB - Includes Special Offers, Magenta. 
 
I moved to focus on the reviews themselves. The review with the highest number of people that found the review helpful was review 85969 on 2016-02-14 by user Benikc on the product Fire Kids Edition Tablet, 7 Display, Wi-Fi, 16 GB, Pink Kid-Proof Case. Their review titled, '5 star device crippled by amazon' said of the device, 'This device would be the best possible tablet for the money if it had Google Play. However Amazon chose to block access to it. This took their well made tablet with a beautiful screen and great performance from an amazing value to a waste of money. This is my last amazon branded product.If you use a lot of apps or want specific apps shop for another device.'. They gave it a rating of 1 and did not recommend it. The review had 20 people who found it helpful.  

In total, there are 3,798 unique users who wrote reviews in the dataset. Of these users, the 10 most prolific users (users with the highest number of reviews were:

User | Reviews
---- | -------
Dave | 5
Missy | 4
1234 | 4
Steve | 4
Chris | 4
Angie | 4
Mike | 4
Susan | 4
Frank | 3
Manny | 3

The users who have been found at least 5 times were Beninkc, Roberto002007, Dick, CarlosEA, EricO, Junior, TerrieT, LadyEsco702, safissad, Quasimodo, iMax, mysixpack, Deejay, stephfasc22, AshT, fenton, Rodge, Ellen, Karch, FrankW, Kime, Mark, 1Briansapp, trouble, Stuartc, and Earthdog

Now looking at the relationships between different parts of the reviews.

#### Average rating vs Number of Reviews
I developed a graph relating the average rating of the products with the number of reviews the product has:

<img src="Graph 1.png" width="400">

This graph had a few outliers that made it difficult to see some of the plots in detail so if the product has more than 800 reviews I removed it from consideration, giving me a new graph of:

<img src="Graph 2.png" width="400">

We can see from the graph that when products have fewer reviews their ratings are much more sensitive to change compared to products with more reviews.

#### Average Rating vs Length of Review text

I got the average length of review text for each product using the `average_text` function, then created a list of ratings and average text length as so:

```python
rating_avgtext = average_text(review_values)
ratings = []
avg_text = []
for rating in rating_avgtext:
    ratings.append(rating)
    avg_text.append(rating_avgtext[rating])

```

Giving me the scatter plot:

<img src="Graph 3.png" width="400">

This graph reveals that products with a rating of 1 have a much higher number of characters used in their reviews with a steep drop off from 1 to 2. The review length of the higher ratings are on the lower end with ratings 4 and 5 having around 100 characters per review.

#### Average Rating vs Likelihood of Review Being Found Helpful

In order to find the percentage of reviews that were found helpful I used the `percent_helpful` function from above. 

<img src="Graph 4.png" width="400">

The graph shows that reviews that gave the product a rating of 1 or 2 were more likely to be found helpful by other users. The rating with the lowest number of people that found it helpful was 3 with 4 and 5 being slightly above it. 

#### Average Rating vs Likelihood of the Product Being Recommended

The average rating for all the reviews in which the reviewer recommended the product was 4.607549120992761

<img src="Graph 5.png" width="400">

The graph shows that products rated 4 or 5 by their reviewers were recommended disproportionally more than products with lower ratings which is to be expected.

#### Words Found in Reviews

Looking at the words used in either the title or text of the reviews in the dataset we can find which words are most commonly used to describe products based on rating.

For products with a rating of 5, the most common words found in the review titles were:

* great
* tablet
* fire
* for
* the
* my
* kindle
* to
* good
* price
* gift
* it
* a
* love
* kids
* awesome
* product

Looking at some of the words used we see a couple things:

* The Kindle and Amazon tablets are commonly reviewed by their users
* Products with a rating of 5 were described using more powerful emotional words such as great, love, or awesome
* These products are commonly given as gifts primarily for their great value for price and their popularity with kids

While looking at the titles of reviews for products with a rating of 1 we see the words:

* not
* very
* disappointed
* poor 
* does 
* amazon 
* a 
* great 
* 5 
* work 
* to 
* use 
* and 
* kindle 
* tablet 
* for 
* good 
* really 
* with

Based on these words we can see that the reviews for these products are much more negative than those for the higher rated products.

Finally, with reviews for products with a rating of 3 we see words like:

* tablet 
* the 
* great 
* good 
* for 
* price 
* ok 
* a 
* not

These words convey that their products are fairly average with a mix of positive and negative words used in their reviews and less words used to describe the products than reviews with ratings of 1 or 5.

#### Broken File 4

With the file for `review4.json` being broken it's important to recover the data from this file in order to get a more accurate analysis of the products.

The `broken_file` directory holds the fragments of the `review4.json` so I created the `explore_directory` function in order to access all of the files in the directory. Doing this I found that review 4 covers 1 product, Fire Tablet, 7 Display, Wi-Fi, 8 GB - Includes Special Offers, Magenta and that the total number of reviews in the dataset is 4,992. When I looked at how the new file effected the rating of the product, I found that the average rating of the product changes by 31.19% when the new reviews are added.
