import tweepy
import pandas as pd
import config
import pymongo
from pymongo import MongoClient

cluster = MongoClient(config.MongoCode)
db = cluster["tweepy"]
collection = db["tweepy_data"]


auth = tweepy.OAuthHandler(config.consumer_key, config.consumer_secret)
auth.set_access_token(config.access_token, config.access_token_secret)
api = tweepy.API(auth)

number_of_tweets = 500
tweets = []
searched_value = set()
likes = []
retweets = []
tweets_split = []
flatten_tweets_split = []
tweets_length = []
tagged_counter = {}
hashtag_counter = {}


class ProgramDone(Exception):
    pass


def new_or_stored_data():
    data_choice = input("Would you like to search for a New(N) or Stored(S) data? Quit(Q): ").upper()
    if data_choice == "N":
        search_choice()
    elif data_choice == "S":
        data_pull_mongodb()
    elif data_choice == "Q":
        print("Ending tweepy analyser")
    else:
        print("Sorry, the value you entered is not a valid option")
        new_or_stored_data()


def show_available_search_value():
    results = collection.find({})
    for result in results:
        searched_value.add(result["searched_value"])
    if len(searched_value):
        print(f"The accounts with stored data are: {searched_value}")
    else:
        print("There is currently no stored data")
        new_or_stored_data()


def data_pull_mongodb():
    show_available_search_value()
    data_pull_searched_value = input("what search value would you like to pull?: ")
    if data_pull_searched_value in searched_value:
        results = collection.find({"searched_value": f"{data_pull_searched_value}"})
        for result in results:
            tweets.append(result["tweet"])
            likes.append(result["likes"])
            retweets.append(result["retweets"])
            tweets_split.append(result["tweet"].split())
    else:
        print("Sorry, the value you entered does not exist in the database.")
        data_pull_mongodb()


def search_choice():
    user_choice = input("Would you like to search for account(A) or quit(Q)?: ").upper()
    if user_choice == "A":
        get_account_data()
    elif user_choice == "Q":
        print("Ending tweepy analyser")
    else:
        print("Sorry, the value you entered is not valid. Please try again.")
        search_choice()


def get_account_data():
    search_user = input("Which twitter account would you like to analyse?: ")
    for tweet in tweepy.Cursor(api.user_timeline, id=f"{search_user}", tweet_mode="extended").items(number_of_tweets):
        post = {"searched_value": f"{search_user}", "tweet": tweet.full_text, "likes": tweet.favorite_count, "retweets": tweet.retweet_count}
        collection.insert_one(post)
    data_pull_mongodb()


def average(x):
    return sum(x) / len(x)


def average_text():
    print("-" * 73)
    print("Tweet averages:\n")
    print("Retweets: ", round(average(retweets), 0), "per tweet")
    print("Likes: ", round(average(likes), 0), "per tweet")
    print("Words: ", round(average(tweets_length), 0), "per tweet")
    print("-" * 73)


def word_splitter():
    for tweet in tweets_split:
        for item in tweet:
            flatten_tweets_split.append(item.lower())


def tweet_length():
    for word in tweets_split:
        tweets_length.append(len(word))


def most_frequent(character, category):
    res = [x for x in flatten_tweets_split if x.lower().startswith(character.lower())]
    for item in res:
        if category == "tagged":
            if item in tagged_counter:
                tagged_counter[item] += 1
            else:
                tagged_counter[item] = 1
        else:
            if item in hashtag_counter:
                hashtag_counter[item] += 1
            else:
                hashtag_counter[item] = 1
    try:
        if category == 'tagged':
            if tagged_counter:
                max_key = max(tagged_counter, key=tagged_counter.get)
                max_value = tagged_counter[max_key]
        else:
            if hashtag_counter:
                max_key = max(hashtag_counter, key=hashtag_counter.get)
                max_value = hashtag_counter[max_key]
        print(f"Most frequent {category}: {max_key}: {max_value} times")
    except UnboundLocalError:
        print("not enough data for most frequent data")


def most_engagement():
    print("-" * 73)
    print("Most engagement:\n")
    most_frequent('@', 'tagged')
    most_frequent('#', 'hashtag')
    print("-" * 73)


def most_liked():
    most_likes = df.loc[df.likes.nlargest(1).index]
    print("-" * 73)
    print("The most liked tweet was: ")
    print(most_likes)
    print("-" * 73)


def most_retweeted():
    most_retweets = df.loc[df.retweets.nlargest(1).index]
    print("-" * 73)
    print("The most retweeted tweet was: ")
    print(most_retweets)
    print("-" * 73)


def clear_data_lists():
    tweets.clear()
    likes.clear()
    retweets.clear()
    tweets_split.clear()
    flatten_tweets_split.clear()
    tweets_length.clear()
    tagged_counter.clear()
    hashtag_counter.clear()


# Delete all in database
#results = collection.delete_many({})

try:
    while True:
        # User picks what they want to search
        new_or_stored_data()
        word_splitter()
        tweet_length()

        # Setting up Dataframe
        df = pd.DataFrame({'tweets': tweets, 'likes': likes, 'retweets': retweets})
        pd.set_option('display.max_colwidth', 50)
        df = df[~df.tweets.str.contains("RT")]
        df = df.reset_index(drop=True)

        # Data for user
        print(df)
        most_retweeted()
        most_liked()
        average_text()
        most_engagement()
        clear_data_lists()
except ProgramDone:
    pass

except AttributeError:
    pass
