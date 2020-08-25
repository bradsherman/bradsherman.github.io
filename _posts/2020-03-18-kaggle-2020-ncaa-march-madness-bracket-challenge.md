---
layout: post
title: Shooting my Shot in the 2020 Kaggle March Madness Bracket Challenge
---

One of my goals this year was to become more fluent in the language of data. I
think it's only going to become more useful in my professional career as a
software engineer. Another motivating factor is that in the fall of 2019 I
enrolled in the Introduction to Analytics Modeling course given through the
[Georgia Tech Online Master of Science in
Analytics](https://pe.gatech.edu/degrees/analytics). I took this class mostly to
see if getting a master's degree in data science might be something that would
interest me. It also offered the chance to revisit some topics I learned in
school that I have not thought about much since! My point is, even though I was
able to complete some class projects in that course, I wanted something a little
more real world, dealing with a field that I am interested in. If you know me, I
naturally gravitated towards sports and lucky for me March Madness was right
around the corner! (Or so I thought 😔) So I decided to take on the challenge
and try to create a model that would predict the winners and losers of the 2020
NCAA Basketball tournament.

Unfortunately, as I'm sure most of you know, the NCAA tournament was cancelled
this year due to the coronavirus. However, I was able to get a model up and
running before that decision was made. The goal of this post is to document what
I did in order to get acquainted with Kaggle, what I will do the same next time,
and what I will do differently. Hopefully you can take some lessons with you!

When I first decided to join the competition, I wanted to set some realistic
expectations. I saw that there was a cash prize, but I didn't want to
participate in this competition for the money. I knew that I was doing this to
strengthen my knowledge of the data science/analytics process and work through
some of the issues that an analytics professional might have. I mention this
simply because I do think it's important to set realistic goals and expectations
before beginning a new project. With that being said, that doesn't mean I wasn't
trying to create the best model! Hope for the best, but expect the worst 🙂

### Part 0: Introduction and Setup

Before I could dive into building a model, I had to get familiar with Kaggle's
interface, the competition rules, the data available to me, and how submissions
would be scored. It is very important to take note of this information because
obviously it will inform many decisions later on. For example, are you being
asked to give a percentage chance that team A might beat team B? Maybe you would
use a logistic regression model because it outputs a probability between 0
and 1. Or maybe you are simply being asked to predict who will win, and you can
use a simple linear regression to predict how many points a team will score
given statistics about that team and their opponent. As you can see, it is very
important to have a solid grasp on the context and the domain in which you are
creating a model before you even write any code. After I became familiar with
the problem at hand, I was able to use one of the starter notebooks on Kaggle to
quickly set up my environment and get down to business!

### Part 1: Exploratory Data Analysis (EDA)

One of the first things you should do when trying to model some process or event
is explore the data at your disposal. I'll be the first to admit, that this is
something I didn't give as much attention to in this project as I should have,
and I'll make sure to fix that in my next project! The reason I mention this is
because it helps give you an even better understanding of the domain and the
context in which you are working. Quite often you will be surprised with what
you find! Finding maxes, mins, medians, and means to name a few of your
dataset's features can nudge you to think about the data in ways you might not
have thought before. For example, if you create a [pairs
plot](https://www.quora.com/What-are-pair-plots?share=1) of your data, you might
discover that two seemingly uncorrelated features are actually highly
correlated! This can help down the road in feature selection when you are trying
to make your model as simple as possible (click
[here](https://en.wikipedia.org/wiki/Occam's_razor) to read about why simple is
desirable). Here is an example pair plot from some of the features in the
dataset I used:

<img src="/assets/img/ncaa-pair-plot.png" alt="NCAA Stats Pair Plot" />

Based on this image (which is just a slice of all the statistics in play for my
purposes), I can tell that 'FGP' (Field Goal Percentage) and 'ATO'
(Assist/Turnover Ratio) are somewhat correlated which makes sense. I can also
tell that 'FGP' and 'OppScore' have almost no correlation, and you can see how
this would get interesting as you add more features. I should note, that
software exists that will automatically prune correlated features for you before
using them to train your model, so you won't have to manually inspect a pair
plot like this for hundreds of features. However, it is always one of the more
interesting visuals to me so I like to pick some features and check my
assumptions using this visual.

<!-- TODO maybe don't need this --> I also found that as I became more familiar
with the data, I was more comfortable manipulating it and came up with more
efficient ways to slice and dice the data later on when tweaking the model.

### Part 2: Data Cleaning

I put data cleaning second, but it really can (and probably will to some extent)
be done while you are exploring the data. You'll almost never be giventhe data
in the exact format that you want. Whether it's missing data, incorrect data, or
the data just isn't in the most usable format you'll probably have to write some
code to [impute](https://en.wikipedia.org/wiki/Imputation_%28statistics%29) the
missing values or remove them. If you don't have a lot of data you might want to
impute the missing data points rather than remove them and lose some of the
little information you have. In my case, there was no data missing (phew!) but
it wasn't ready to be used right away. Here is a snippet of how the data is
formatted originally:

| Season | WTeamID | WScore | LTeamID | LScore |
|:------:|:-------:|:------:|:-------:|:------:|
|   1985 |    1228 |     81 |    1328 |     64 |
|   1985 |    1106 |     77 |    1354 |     70 |
|   1985 |    1112 |     63 |    1223 |     56 |
|   1985 |    1165 |     70 |    1432 |     54 |
|   1985 |    1192 |     86 |    1447 |     75 |

Now, if I wanted to get team 1128's average score for the 1985 season, it's not
so easy. From this format, there's no easy way to group on season and 'WTeamID'
to include all of 1128's games, because whenever they lose, they show up in the
'LTeamID' column. Because of this, I have to split the data up into two halves
and merge them back together into something that might look like this:

| Season | TeamID | Score | OppTeamId | OppScore |
|:------:|:------:|:-----:|:---------:|:--------:|
|   1985 |   1228 |    81 |      1328 |       64 |
|   1985 |   1106 |    77 |      1354 |       70 |
|   1985 |   1112 |    63 |      1223 |       56 |
|   1985 |   1165 |    70 |      1432 |       54 |
|   1985 |   1192 |    86 |      1447 |       75 |

That way when I group on season and 'TeamID' I get all of team 1128's games.
This is just one example of reshaping the data into a more usable format. A
large majority of the initial process is spent working through this process so
you can produce a clean, informational dataset that will be used to train your
models!

### Part 3: Model Creation and Validation

Now, on to the easy part! Well, maybe it's not easy but there are so many good
libraries out there that it is becoming increasingly trivial to stand up a
statistical model of any kind. The trick is knowing which one to pick, and how
to make sure you've picked the right one! This is called model validation. As I
mentioned before, knowing your data and the type of output you are being judged
on/expected to produce will affect which model you choose. For simplicity's
sake, I chose a Logistic Regression model to start. This is because my
submissions were being evaluated based on a log loss function, which essentially
penalizes you more if you're more confident about a team winning when they
actually lose. So, if my model predicts that Duke has a 95% chance of beating
North Carolina and North Carolina wins, that model will get penalized more than
if it would have predicted that Duke had a 55% chance of winning. Knowing this
and knowing that logistic regression models can produce a binary output (0 or 1)
or a probability I made my decision. You may wonder, well how am I supposed to
know that a logistic regression model produces a binary output or a probability?
It's a valid question, and the answer is you shouldn't! In my case, that sort of
knowledge is gained through studying different analytics models (like the online
class), and retained through applying that knowledge in real world projects
(like this one!). I certainly do not know every characteristic of every model,
but the goal is to have a strong fundamental base to build off of and reference
when you encounter new models, just like I did with XDGBoost! After I chose my
model, it was time to start training.

#### Scaling

Generally, it's good practice to scale the data before training a regression
model if your features have different ranges (click
[here](https://medium.com/@swethalakshmanan14/how-when-and-why-should-you-normalize-standardize-rescale-your-data-3f083def38ff)
for more on why) and since I have features like 'Score' that can roughly range
from 50-100 and 'Assists' that can range from about 3-15, I chose to scale the
data. There are multiple ways to scale data, such as a `MinMaxScaler`, which
will transform all values to be from [0,1], where values of 0 correspond to the
original minimum and values of 1 correspond to your original maximum. Once your
data is normalized, you can move on to the next step in training a model!

#### Training & Testing Datasets

You've probably heard about the `train_test_split` numerous times if you've ever
done a data science project before. The objective is to make sure your model
doesn't over-fit to your training data. This would prevent it from generalizing
to unknown data that you would like to use your model on in the future!
([Here](https://elitedatascience.com/overfitting-in-machine-learning) is a good
article that discusses over-fitting more in depth) Clearly this is a big
problem, but luckily the solution is very simple: split up our dataset! Any data
manipulation library worth using will have a simple way to split up the entire
dataset you have into one that should be used to _train_ your model and one that
should be used to _test_ your model. This way, you can train the model on one
set of data, and then you can report it's performance using the testing set,
which simulates what the model will do in the real world.

#### Validation & Model Selection

One thing to note is that if we have enough data at our disposal, it may be
beneficial to split the initial dataset up into _three_ distinct sets: train,
test, and validation. I will explain the purpose of the validation data set with
an example. Let's say we are trying to choose between 4 different models. First,
we train each model using the training dataset. Then, in order to choose a model
we "test" each model on the validation set. Whichever model performs the best is
the model we will choose as our winner! Once we have selected a winner, the
accuracy we report for our model comes from the test dataset. This extra step
provides another chance for use to filter out any models that happen to score
highly because of the specific dataset used.

Once you complete this process for a new project, it's time to have some fun!
This is where you can basically rinse & repeat, choosing different input data
and models while tweaking model parameters to see which combinations yield the
best results.

### Conclusion

In my opinion, it's very beneficial to first setup a pipeline that includes all
of these steps with a simple dataset and a simple model. Once the process is
setup and you know what is happening during each step, you can go back and
examine each step individually and optimize it. Think a new feature would give a
better score? Add it to the original dataset, put it through the ringer and see
how it compares to previous versions of your model! This is where inquisitive
minds prevail, and domain knowledge really helps!

I had a great time finally participating in my first Kaggle competition, even if
I didn't actually get to submit a solution since the competition was cancelled.
I learned a lot and gained a ton of confidence in my ability to undertake the
data science process. Hopefully you learned something in this post or were
inspired to partake in a competition of your own!
