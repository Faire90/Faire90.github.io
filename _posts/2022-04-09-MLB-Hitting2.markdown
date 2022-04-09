---
layout: post
title:  "DH Stats Part 2"
date:   2022-04-09 16:5:00 -0700
categories: New
---

As I discussed in my [last post,][last-post] I am attempting to get batting stats alongside games and positions.  I left off having acquired the team_id for every team and put them into a list and a dictionary.  This time, I vowed to get to work on examining the boxscore_data() method.

boxscore_data() takes "gamePk" as a parameter.  Which I learned I could acquire from one of the fields returned by the schedule() method.  Schedule(), in turn, takes a team_id, which I retrieved with the lookup_team() method.  Previously, I had used September 6, 1999, but this time I'm going to look for a more recent game.  So I use:

{% highlight python %}
print(statsapi.schedule(team=136, date="04/02/2021"))
{% endhighlight %}

To get game_id 634578 printed to the console. This is a home game against the Giants.  Next, I use:

{% highlight python %}
print(statsapi.boxscore_data(634578))
{% endhighlight %}

This produces a long and confusing JSON-looking output dumped to the console.  This isn't very useful.  For one thing, I'll have to keep making calls to the MLB API everytime I want to access it and for another, it is unreadable without paying very close attention.  

To prevent needing to make too many API calls, I could copy and paste it into a file, but Python provides us with another option: the "json" library.

Since the output is in something similar to JSON format, we can use the json.dump() method to save it to a new .txt file that we can then access later.  I write the following code:

{% highlight python %}
import json

score = statsapi.boxscore_data(634578)
with open('score.txt', 'w') as boxscore:
    json.dump(score, boxscore)
{% endhighlight %}

This should pull the boxscore_data into the "score" variable and then write it to a .txt file for later use.  According to the Python docs (https://docs.python.org/3/tutorial/inputoutput.html), using "with open()" is good practice that ensures the file properly closes even if an exception is raised.

Okay, so now I have the boxscore_data() output, but I need to know what is in it.  I briefly look at the data and see that, like many JSON files, it is basically a series of embedded dictionaries.  I run:

{% highlight python %}
with open('score.txt') as boxscore:
    score = json.load(boxscore)
    for key in score:
        print(key)
{% endhighlight %}

In Python, a 'for' loop can iterate through each key (You can get a similar result using the keys() method, but this works fine), which is not possible in many languages because keys are typically unordered.  For us, this should print out all of the keys in the first dictionary.  It outputs:

{% highlight python %}
gameId
teamInfo
playerInfo
away
home
awayBatters
homeBatters
awayBattingTotals
homeBattingTotals
awayBattingNotes
homeBattingNotes
awayPitchers
homePitchers
awayPitchingTotals
homePitchingTotals
gameBoxInfo
{% endhighlight %}

So that looks promising.  This is a home game for the Mariners, so I probably want to look at one of the keys that contains "home."  Since I'm eventually going to be looking at every game, I'll also want to figure out where to look to see whether the Mariners are home or away.

I could probably write a series of nested "for" loops to parse out the contents of each key, but first I just tried using the Ctrl+f word search function in Pycharm (or, honestly, any text editor) and found that most of these keys are relatively easy to understand.

It turns out that the "home" key contains a dictionary with all of the basic home team information, so that should make things simple when looking at multiple games.  We could just check whether this section contains information for the Mariners.

"homeBatters" looks to contain everything I need.  It contains a list of dictionaries.  The first element (index "0") appears to be headers, but the others contain batting data as well as positions played.  For example, Mitch Haniger has the following data:

{% highlight python %}
{"namefield": "1 Haniger  RF", "ab": "4", "r": "1", "h": "1", "doubles": "0", "triples": "0", "hr": "0", "rbi": "0", "sb": "0", "bb": "0", "k": "2", "lob": "1", "avg": ".222", "ops": ".444", "personId": 571745, "battingOrder": "100", "substitution": false, "note": "", "name": "Haniger", "position": "RF", "obp": ".222", "slg": ".222"}
{% endhighlight %}

You can see he played right field, went 1 for 4, and managed to score a run after getting on base with his one hit (I assume this because he has no bases on balls).

Next time, I will put together a function for downloading all of the game data we need and parsing that data.

[last-post]: https://faire90.github.io/new/2022/03/28/new-post.html

