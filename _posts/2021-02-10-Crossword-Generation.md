---
title: "⬜ Crossword Creation"
tag: "KameKurosu"
---

The next step was to create the levels. It turns out that crosswords aren’t easy to make. And I’m definitely not planning on making them all by hand... <!--more-->That said, I looked for programs that could automatically create them for me! After a lot of digging, I found a Japanese crossword program called クロスワード　ギバー (literally, Crossword Giver). Since it uses a lot of difficult Japanese words, it took some time for me to figure it out, but basically its a program where you provide it a clue list and an answer list and it will autogenerate a crossword for you! Here are some screenshots:

![Crossword Giver Program](/assets/images/blog-kamekurosu/crossword-giver.jpg){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">Main screen of Crossword Giver after generating a crossword with clues (to the right)</figcaption>

![Crossword Giver Generation Menu](/assets/images/blog-kamekurosu/crossword-giver-generation.jpg){:class="blogVerticalImage bigImage"}
<figcaption class="blogImageCaption">Menu to generate a crossword using Crossword Giver. You can select # rows, # columns, and a dictionary of clues/answers, among other options.</figcaption>

However, I still had to (a) get a list of clues, i.e. kanji-containing words, (b) get a list of their respective answers, i.e. the readings, and (c) find a way to automate using Crossword Giver to make many many crosswords without me having to click a bunch of buttons over and over again.

## Clue and Answer Generation

For (a) and (b), I started by thinking about the types of words I wanted to include. I knew that I wanted to split up the words by level, but how do I know which words are “easy” and which words are “hard”?

Well, in Japanese, there is a system to measure Japanese ability called JLPT (Japanese Language Proficiency Test). N5 is the easiest level, and it goes up to N1, the hardest level. However, there are also many more advanced words not included even in N1. I decided to roughly group my words based on their JLPT level. However, first I needed a complete list of Japanese words.

For this, after some digging, I came across a program called zkanji. It turns out that this tool lets you export word lists! It also lets you generate word lists based on the JLPT level of the word and/or the kanji in the word. It took me a while to figure out, but eventually I used the program to export a huge word list (around 25,000 words), with each word’s frequency and JLPT level annotated. I used these two criteria to sort the words into my 6 difficulty levels, as specified in the following table:

![Table with Level Descriptions](/assets/images/blog-kamekurosu/kamekurosu-levels-table.jpg){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">Criteria for words included in each difficulty level as well as crossword size</figcaption>

To decide on the cutoffs in this table for each level, I plotted the distribution of word frequencies for each level. Basically, I wanted to know: Are N5 (easiest JLPT level) the most frequent Japanese words, and are N1 the least frequent? Intuition would say yes, but I wanted to make sure.

Here is that plot:

![Histogram of Japanese Words by JLPT Level](/assets/images/blog-kamekurosu/jlpt-dists-stacked.png){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">In the histogram of 25,000 "common" words in Japanese, you can see that N1-N5 words tend to be more common (to the left on the graph) while non-JLPT words (N-) tend to be less common. However, many of the most common words are still dominated by harder JLPT levels (N1, N2, N3), mainly because N4 and N5 just have so few words overall.</figcaption>

Each number on the x-axis represents a single Japanese word, where the higher a number is, the less frequent that word. Therefore, the most common word is at 1, and the least common is at ~25,000. I then plotted the distribution of each JLPT level as a stacked bar chart.

The results were actually a bit surprising. I expected to see all the N5 words (blue bars) mainly be distributed to the left of other JLPT levels. However, it seems like N5-N1 all contains many less frequent words, with N3 (middle JLPT level) actually containing the most high frequency words! Something else that was unexpected is that there is a “bump” at the tail end of each JLPT group around 20k-25k. This is likely due to words being included in JLPT lists that are used for testing language ability but not really used in every day life. Finally, in case you were wondering why each bar doesn't sum to 1,000, that is just because there can be duplicate words of a given frequency and also because many words were filtered out from the frequency list that just had one syllable (e.g. 歯＝は＝ha=tooth).

What did match up with expectations though, is the brown curve, which represents all non-JLPT words. As expected, the vast majority of these non-JLPT words are quite infrequent. Thus, these words will only be used in the final level, Advanced II.

Because many of the most frequent words were found in N4, N3, N2, and N1 (i.e. not just N5), I decided to group words based on whether they are (1) frequent enough for that level OR (2) at an appropriate JLPT level. The cutoffs in the level table above capture this idea.

At this point, I also decided on how big to make the puzzles for each level (also included in the table above). Because the beginner levels have a smaller word bank to choose from to build the puzzles, I had to keep the puzzles smaller or else Crossword Giver would error out.

Once this was complete, I had a defined set of cutoffs (i.e. a list of words) for each level. I then found a way to complete task (c), automating the Crossword Giver program to generate crossword puzzles.

## Automating Crossword Generation

I did this by writing some python code that simulates keystrokes. Because you can use hotkeys to control Crossword Giver, I didn’t have to worry about simulating mouse clicks or cursor movements. I used a python library called pynput for this task. Here is what the code looks like:

![Crossword Generation Code](/assets/images/blog-kamekurosu/xword-generation-code.jpg){:class="blogVerticalImage bigImage"}
<figcaption class="blogImageCaption">Beginning of the python script I wrote to automate crossword generation using the Crossword Giver software, as shown in the GIF below</figcaption>

Even if you have never read code before, you can probably understand most of this! Python is a very expressive programming language, meaning it reads similarly to the English language (at least, compared to older programming languages like C).

Here is a GIF of the python script in action. It requires no input from me. I just type the command to run it and go do something else for a few hours.

![Crossword Generation GIF](/assets/images/blog-kamekurosu/xword-generation.gif){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">Crosswords getting generated and saved without any user input, just based on the script I wrote (see above)</figcaption>

I used my python script to simulate the generation of 100 crosswords at each level (600 total). Sometimes, however, I would get an error so I did have to go back and manually click through Crossword Giver to create a few crosswords for each level, but this was much better than making all 100 that way! It will also make it very easy for me to add more puzzles in the future.

Each puzzle was saved as a JSON file. If you don’t know what JSON is, it is basically just a file type that stores data in a hierarchy. Here is an example puzzle file (in JSON format):

![Puzzle JSON Data from Crossword Giver](/assets/images/blog-kamekurosu/unformatted-json-puzzle-data.jpg){:class="blogVerticalImage bigImage"}
<figcaption class="blogImageCaption">Top part of JSON file for the very first puzzle. You can see that it has the data laid out in a hierarchical format.</figcaption>

While this file contains all the data we need to recreate this puzzle, it’s not in the nicest format. It also has extra information we don’t need in the app, like “notes” and “has_hints”. That said, the next thing I did was process this data (again, using python) into a nicer format!
