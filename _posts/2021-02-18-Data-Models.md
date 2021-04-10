---
title: "Data Wrangling"
tag: "KameKurosu"
---

![Ropes and Knots on a Ship](/assets/images/blog-kamekurosu/headers/ship-ropes.jpg){:class="blogHorizontalImage"}
<figcaption class="blogImageSourceCaption"><a href="https://unsplash.com/photos/u-qKOduhSqg"><u>Image Source</u></a></figcaption>

So, at this point I have 6 levels x 100 JSON files per level = 600 JSON files filled with raw data, not in a super nice format, and also containing data we don’t need. We need a way to organize all of this data for our app! <!--more-->

The first thing I did was convert each JSON into a more nicely formatted JSON for my app’s purposes. To decide on what this “nice format” would be, I designed a “data model” for my app in Xcode, Apple's open-source software for iOS development:

![Data Model](/assets/images/blog-kamekurosu/data-model.png){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">Data model for crossword data - each table represents a specific piece of data and has relationships with the other tables</figcaption>

The data model basically shows how I want my data to be organized in my app. For example, each Level (Beginner I, Beginner II, etc.) contains information about its current ID, name, number of puzzles for that level, etc. In addition, it has a relationship with Puzzle. Specifically, each Level contains many Puzzles (double arrowhead), but each Puzzle only has one Level it’s associated with (single arrowhead).

I also have Words and Cells in my data model, which are exactly what you think they are. Finally, I have UserInfo which I will use to store any user-specific information. It is also connected to all the Completed Puzzles that the user has finished. These two tables are separate from the other four because I want to store them in the cloud (while all others are only stored locally). This ensures that only a small amount of the user’s iCloud storage space is used while also allowing them to sync which puzzles have been completed across devices.

Now that I know how I want my app’s data to be structured, I should reformat my raw JSON data into a format that looks similar. This will make importing the data into the app from these files more painless.

To do this, I wrote a few python scripts to format and merge the 600 raw data files (JSON) into a single big JSON file that has the data formatted similarly to how it will be in my app. Here is a part of that final JSON file:

![Formatted JSON Puzzle Data](/assets/images/blog-kamekurosu/formatted-json-puzzle-data.jpg){:class="blogVerticalImage bigImage"}
<figcaption class="blogImageCaption">Final complete JSON file containing puzzle data for 600 puzzles</figcaption>

You can see that the structure in this file nicely tracks that of my data model (e.g. level_data -> puzzles -> words). Note that I didn’t include “Cells” in this data file (even though it's in my data model). I'll just create each Cell data object within the app, while initially loading the list of words for each puzzle.

If you are curious about the code I used for all this data processing and crossword generation, see my [<u>GitHub repository here</u>](https://github.com/jonstrutz11/KameKurosuPuzzleGeneration).

Now that I have my final data file for all the puzzles which I will import in my app, I can start to actually build my app!
