# cocktail_app
creating a better cocktail book

see https://www.thecocktaildb.com/api.php

## todo:
[ ] potentially incorporate this dataset https://gist.github.com/jgorsett/6492ab1253c04167d1639c4ced71b3bf#file-hotaling_cocktails-cocktails-csv

## Notes for medium post:
__Something about the intention of the project and audience__

I'm not a web dev and this was meant to be a quick and dirty fun project to mess around with serving up some data that I collect from outside sources in a coherent way. I've made a few websites before with Node, bootstrap, materialUI, and React. One of them had an Express back end that was used for serving up API data from square in real time.

Those projects were hosting mostly static information and didn't take full advantage of having a modern web stack. This project is meant to focus more on serving up cocktail recipe data in a coherent and user friendly format.

This writeup is intended to focus on the implementation and design decisions rather than be a step-by-step guide on reproducing my results. If you're interested in creating a similar project though and have questions you can find details on how to contact me on my website: ryanlewisengineering.com

__"Scraping" through thecocktaildb.com__

started off by querying http://www.thecocktaildb.com/api/json/v1/1/filter.php?c=Cocktail using an online REST API query testing tool. This could also have been pasted directly into the browser to get the JSON data

Created the main_list.json file and used a jupyter notebook to populate a pandas dataframe with the values (could have just as easily kept it a dict or list but this step isn't too important). Then iterated through the cocktail IDs to query the ingredients and details for each cocktail from the other API endpoint. After pulling these values I noticed that the format for the cocktails included a list of 15 ingredients which mostly served as placeholders containing null values. I'm guessing that means that the data was stored in some sort of sql database like SQLite withh a fixed amount of columns. 

The fully populated dict of all the cocktail recipes was saved as cocktail_dump. This will need to be cleaned later to remove all of the null values.

__Hosting__

Next I needed to find a way to store and host my semi-structured data. I've used Firebase for free site hosting before so Firestore Database was the obvious choice.

I started by creating a new firebase project which is tied to my primary google account. 

Next I chose to start a free database in production mode using the default datacenter location of us-central. This was chosen because my json file was well under the 10GB limit for the free tier (~180kb), I won't be updating the data very frequently, and I'm located in the US.

__Database Design__

Now to populate the database I need to create a collection. This is where I need to look at my existing data and make design decisions that will affect how querying is structured. I want to be able to use this site several different ways:
- looking up drinks by their base liquor and:
  - how common/popular they are
  - what type of cocktail they are (i.e. Apéritifs or digestifs)
  - additional ingredients that I select based on what I have at home
- putting in a small list of whatever mixers/ingredients I have and getting a list back of drinks I could make

I'm sure there's other ways I could query the data but those are the main ways I imagine sifting through the entries.

With those use-cases in mind I want to structure the entries like so:
155_belmont:
{
"type" : ["something like Apéritifs, digestifs, highball etc."],
"liquor" : {
  "dark rum" : "1 shot",
  "light rum",
  "vodka"},
"ingredients" : {
  "orange juice" : "1 shot"},
"garnish" : ["carrot"],
"instructions" : "Blend with ice. Serve in a wine glass. Garnish with carrot sliver.",
"image" : "URL to stock photo of drink",
"video" : "empty for now, might add embeddable YouTube links",
"glass" : "white wine glass",
"rating" : 0,
"numRatings" : 0
}

^^ could also switch to likes and dislikes instead of ratings. This'd probably be cleaner

The next step is to create a new collection called `drinks` which uses `155_belmont` as the Document ID and fill out the fields with the appropriate data types specified.

<put `firestore entry screenshot.png` here>

