---
layout: post
title: Consuming JSON API Data with Haskell
readtime: true
tags: [haskell, tutorial, functional programming, api]
---

Recently I stumbled upon the [The Odds API](https://the-odds-api.com/) which provides betting odds for all types of sporting events from baseball to cricket to rugby. This is the first site I've seen that actually has an API for retrieving the odds (for free). Most places I've seen force you to download an excel sheet everyday. As part of my journey to learn Haskell, I thought it would be fun to write a simple client for this API. In this post I'll walk through how we can write a bit of Haskell code to consume JSON data from this API.

As you read, please keep in mind that I'm assuming a basic knowledge of Haskell. The code snippets are not guaranteed to compile right away. For example, I've omitted the imports for all code that we write in this tutorial - you can add the imports based on how you structure your code. The goal for this post is not to provide a drop-in snippet to get data but rather to explain a process for writing your own Haskell code to consume data from your favorite API!

### Setup

First, you'll have to get an API key [here](https://the-odds-api.com/). Thankfully you can get one for free if you make less than 500 requests per month. The only other thing you need before we get started is a brand new Haskell project. I'm using [stack](https://docs.haskellstack.org/en/stable/README/) so I just ran `stack new odds-api` to create a fresh project.

### Understanding the Data

One of the reasons I enjoy Haskell is it forces you to think about your data up front before we can even parse the JSON string. In our case, we will be using a library called [aeson](https://github.com/haskell/aeson) to parse the JSON response into the type we'll use. In order to do that we have to define our data and then write a `FromJSON` instance. `FromJSON` tells aeson how to decode a JSON string representation into the data type we have defined. This is in contrast to a language like JavaScript where we can just make a GET request and we get a generic object back (JavaScript Object Notation, if you will). JavaScript allows you to get something up and running quickly, but I prefer thinking about the data beforehand and ensuring that it is well thought out before I start using it.

So, let's say we want to get the upcoming head to head (moneyline) odds for the US region (these terms are described on the odds api's site). We have to send a GET request that looks something like this: `https://api.the-odds-api.com/v3/odds/?sport=upcoming&region=us&mkt=h2h&apiKey=<api-key>` (You'll have to plug in the api key you were sent earlier). The response will look like this:

```json
{
  "success": true,
  "data": [{
    "sport_key": "basketball_ncaab",
    "sport_nice": "NCAAB",
    "teams": [
    	"Boston College Eagles",
    	"Notre Dame Fighting Irish"
    ],
    "commence_time": 1610830800,
    "home_team": "Notre Dame Fighting Irish",
    "sites": [
      {
        "site_key": "fanduel",
        "site_nice": "FanDuel",
        "last_update": 1610836634,
        "odds": {
          "h2h": [
          	26,
         	1.01
          ]
        }
      },
      {
        "site_key": "williamhill_us",
        "site_nice": "William Hill (US)",
        "last_update": 1610837230,
        "odds": {
          "h2h": [
              2.75,
              1.48
            ]
        }
      },
      {
        "site_key": "mybookieag",
        "site_nice": "MyBookie.ag",
        "last_update": 1610837236,
        "odds": {
          "h2h": [
              5.09,
              1.06
            ]
        }
      },
      {
        "site_key": "caesars",
        "site_nice": "Caesars",
        "last_update": 1610837239,
        "odds": {
          "h2h": [
              31,
              1.01
            ]
      	}
      }
    ],
    "sites_count": 4
  },
  ...

]}
```

There are a couple of things we need to keep in mind before defining our data types. First, our moneyline odds have an optional field `h2h_lay`, which is in the same format of `h2h` (this is explained in the odds api docs). Secondly, the `odds` value will look different depending on what market we request. The choices are `h2h`, `spreads`, and `totals`. We'll have to parameterize our data based on which type of odds we expect.

### Defining our Data Types

Now, we can start defining our data types. I like to start from the top-down. If we play with the API for a bit, we'll find that we always get a response with a `success` field, and a `data` field. Armed with this knowledge we can write our first Haskell type:

```haskell
import Data.Aeson (FromJSON, parseJSON, (.:))
import Data.Aeson.Types (withObject)
import Data.Text (pack)

data ApiResponse b = ApiResponse
    { success :: Bool,
      body :: [b]
    } deriving (Show, Eq)

instance (FromJSON b) => FromJSON (ApiResponse b) where
    parseJSON = withObject "ApiResponse" $ \v ->
      ApiResponse
        <$> v .: pack "success"
        <*> v .: pack "data"
```

We are saying here that we have a type `ApiResponse` parameterized by some type `b`, which contains a boolean field `success` and a field called `body` that contains an array of type `b`. In our `FromJSON` instance, we first ensure that our parameterized type `b` is an instance of `FromJSON` (that's the `(FromJSON b) =>` part), and then we parse the JSON string. Note that the field we get back from the API is `data`, but that is a reserved keyword in Haskell so we changed the field name to `body` in our type.

{: .box-note}
**Note:** If you're interested in learning more about parsing JSON in Haskell, checkout [aeson's docs](https://hackage.haskell.org/package/aeson-1.5.5.1/docs/Data-Aeson.html).

Next in our response is an array of sporting events and their associated odds. Let's define the `SportingEvent` type:

```haskell
import Data.Aeson (FromJSON, parseJSON, (.:))
import Data.Aeson.Types (withObject)
import Data.Site (Site (..))
import Data.Text (Text, pack)

type SportKey = Text
type TeamName = Text
type Timestamp = Integer

data SportingEvent odds = SportingEvent
  { sportKey :: SportKey,
    sportName :: Text,
    teams :: [TeamName],
    commenceTime :: Timestamp,
    homeTeam :: TeamName,
    sites :: [Site odds],
    sitesCount :: Integer
  } deriving (Show, Eq)

instance (FromJSON o) => FromJSON (SportingEvent o) where
  parseJSON = withObject "SportingEvent" $ \v ->
    SportingEvent
      <$> v .: pack "sport_key"
      <*> v .: pack "sport_nice"
      <*> v .: pack "teams"
      <*> v .: pack "commence_time"
      <*> v .: pack "home_team"
      <*> v .: pack "sites"
      <*> v .: pack "sites_count"
```

This type is pretty straightforward. I defined a couple of type aliases for stronger type safety, and then we manually write the `FromJSON` instance again since some of the keys are not exactly the same.

We can see that each `SportingEvent` has a list of sites which are listing odds for that event, so let's define the `Site` type as well.

```haskell
import Data.Aeson (FromJSON, parseJSON, (.:))
import Data.Aeson.Types (withObject)
import Data.Text (pack, Text)

type SiteKey = Text

data Site o = Site
  { siteKey :: SiteKey,
    siteName :: Text,
    lastUpdate :: Timestamp,
    odds :: o
  } deriving (Show, Eq)

instance (FromJSON o) => FromJSON (Site o) where
  parseJSON = withObject "Site" $ \v ->
    Site
      <$> v .: pack "site_key"
      <*> v .: pack "site_nice"
      <*> v .: pack "last_update"
      <*> v .: pack "odds"
```

The `Site` is pretty straightforward as well. Now we can move onto the interesting part: defining our odds data types!

Our example request above asks for `mkt=h2h` which means we want to see head-to-head (aka moneyline) odds. However as I mentioned earlier we will want to handle other markets like spreads and totals (over/under). I'll go over how we can define the moneyline odds and the spreads. Below we define our `H2HResponse` and `MoneylineOdds` type.

```haskell
import Data.Aeson (FromJSON, parseJSON, (.:), (.:?))
import Data.Aeson.Types (withArray, withObject)
import Data.Text (pack)

type OddsValue = Float
type OddsList = [OddsValue]

data MoneylineOdds = MoneylineOdds
  { team1Odds :: OddsValue,
    team2Odds :: OddsValue,
    drawOdds :: Maybe OddsValue
  }
  deriving (Show, Eq)

instance FromJSON MoneylineOdds where
  parseJSON j = do
    oddsList <- parseJSON j
    return $
      MoneylineOdds
        { team1Odds = head oddsList,
          team2Odds = head $ tail oddsList,
          drawOdds = extractDrawOdds oddsList
        }
    where
      extractDrawOdds l = case length l of
        3 -> Just (head $ tail $ tail l)
        _ -> Nothing

data H2HResponse = H2HResponse
  { h2h :: MoneylineOdds,
    h2hLay :: Maybe MoneylineOdds
  }
  deriving (Show, Eq)

instance FromJSON H2HResponse where
  parseJSON = withObject "H2HResponse" $ \v ->
    H2HResponse
      <$> v .: pack "h2h"
      <*> v .:? pack "h2h_lay"
```

This one is not as straightforward. Each `Site` might have an `H2HResponse` for its `odds` field. Each `H2HResponse` contains `MoneylineOdds` as defined by the odds api documentation. `MoneylineOdds` are a list of at least 2 and at most 3 numbers. First, are the odds that team 1 will win. Second are the odds that team 2 will win. If there are three items in the list the third value is the odds that a draw will occur. Instead of making the `MoneylineOdds` data type the same as the list we get back from the api, it seemed advantageous to explicitly encode the odds for each outcome (team1Odds, team2Odds, drawOdds). The last thing of note is that we've introduced a new operator from `aeson`: `.:?`. The `.:?` operator is the same as `.:` except it doesn't fail if the key is not present. This is perfect since `h2hLay` is of type `Maybe MoneylineOdds`.

- One could argue I should model the `teams` field in the `SportingEvent` data type in a similar fashion to how I've modeled `MoneylineOdds` and I would agree. However, I think that's a nice to have for now. I'm sure there's a better way to parse `MoneylineOdds` so feel free to let me know how a more experienced Haskeller would do it!

The last data types we need to define are `SpreadsResponse` and `SpreadOdds` types.

```haskell
{-# LANGUAGE DeriveGeneric #-}

import Data.Aeson (FromJSON)
import GHC.Generics (Generic)

data SpreadOdds = SpreadOdds
  { odds :: [OddsValue],
    points :: [String]
  }
  deriving (Show, Eq, Generic)

instance FromJSON SpreadOdds

newtype SpreadsResponse = SpreadsResponse
  { spreads :: SpreadOdds
  }
  deriving (Show, Eq, Generic)

instance FromJSON SpreadsResponse
```

I could have explicitly encoded `team1Odds` and `team2Odds` like I did for `MoneylineOdds` but I wanted to show how `aeson` can automatically define the `FromJSON` instances for you.

Bringing this all together, every time we request odds from the API, our response will be parameterized by the type of odds we expect back. Our initial request will return data of type `ApiResponse (SportingEvent H2HResponse)` for example. That might not be clear just yet, so let's write some code to make our first request in Haskell!

### Making HTTP Requests

Now that we have defined the data we expect to receive, we can make a request to the api. This is what I meant about being forced to think about your data before anything else. Many people (including myself) who attempt to do this in non-statically typed languages would start by making the http request and not think as much about the data. In the long run, I am in favor of spending time on your data model for any serious project.

Anyway, Haskell has a library for sending http requests called `http-conduit`. If you created your project using `stack`, you can add it to your project by updating the dependencies section of `package.yaml` like so:

```yaml
dependencies:
- http-conduit
```

Using `http-conduit` we can just say we expect to receive JSON data back, and give the type that it should parse the JSON into. The client code can look something like this:

```haskell
import Data.Aeson (FromJSON)
import Data.Text (Text, append, pack, toLower, unpack)
import Network.HTTP.Client.Conduit (Request, parseRequest)
import Network.HTTP.Simple (getResponseBody, httpJSON)

baseUrl :: Text
baseUrl = pack "https://api.the-odds-api.com/"

version :: Text
version = pack "v3/"

callApi :: FromJSON b => Request -> IO (ApiResponse b)
callApi r = do
  response <- httpJSON r
  pure $ getResponseBody response

getUpcoming :: (FromJSON o) => String -> String -> IO [SportingEvent o]
getUpcoming token mkt =
  let path = append (pack "odds?sport=UPCOMING&region=us&mkt=") $ append (pack mkt) (pack ("&apiKey=" ++ token))
      url = append baseUrl $ append version $ append path
   in do
        request <- parseRequest (unpack url)
        body <$> callApi request
```

We can focus on two functions here: `callApi` and `getUpcoming`. `callApi` is a helper function that calls a given endpoint and extracts the `ApiResponse` that we are interested in. `getUpcoming` takes in our api token and the market we wish to see (head-to-head, spreads, etc), builds the request, and calls the api. If you setup the project with `stack`, there should be a `Main.hs` file in the `app` folder. In there we can use these functions like this (don't forget to import your code!):

```haskell
main :: IO ()
main = do
  spreads <- getUpcoming "<token>" "h2h" :: IO [SportingEvent H2HResponse]
  print spreads
  return ()
```

Notice how we have to explicitly tell Haskell what type we expect so it knows how to parse the response. Once you have that, you should be able to type in `stack run` in your terminal and get some odds!

{: .box-note}
One thing I don't like is that right now it's possible to say `spreads <- getUpcoming "token" "spreads" :: IO [SportingEvent H2HResponse]`. This will cause a runtime error, as we are telling `http-conduit` to parse the response into `IO [SportingEvent H2HResponse]` type, but we actually get back `IO [SportingEvent SpreadsResponse]` based on the parameters we passed into the function ("spreads"). It would be nice to change this to a compilation error. There is probably a way to do it, but I haven't figured it out yet.

### Wrapping Up

Connecting to an API of some sort is a great project for anyone looking to learn a language. For me, it was also enabling as now I am looking for a project that uses the data returned from the API! I've described a high-level process that I think applies to many software projects:

1. Understand your Data
2. Define your Data
3. Use your Data

Notice how for this project, the bulk of the work was done in steps 1-2. This was a trivial example, but as projects get more complex the integrity of the underlying data model becomes more and more important. Investing time up front to design your data model is of paramount importance for successful projects!

Hopefully you learned a bit about Haskell, aeson, http-conduit, and how to use all three to get data from an API! The code from this post lives [here](https://github.com/bradsherman/odds-api). Let me know if this helps you use Haskell to get data from your favorite API 🙂

Thanks for reading!
