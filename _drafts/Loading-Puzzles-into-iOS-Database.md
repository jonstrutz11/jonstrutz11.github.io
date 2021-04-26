---
title: "Loading Puzzles into our App"
tag: "KameKurosu"
---

Now that we have our puzzles created and formatted, we can figure out how to store these puzzles in our app!<!--more-->

### Choosing a Database Solution

There are many ways you can store data for an iOS app. You can store it in a local database (on the actual iPhone), in a flat file (like our JSON file), or you can store it externally in a database on some server somewhere. While I have the data currently in a JSON file, I don't want to load that whole file every time I start the app - it is really big! In addition, I want to store values that **can change**, like inputted characters in a crossword or a list of completed levels.

I decided to use one of Apple's built-in options to store all this data for a few reasons: (1) better performance using a local database than using one hosted on a server somewhere, (2) good documentation on how to set it up, (3) easy-to-setup iCloud syncing between devices, and (4) as a beginner, I just wanted to use the most barebones approach so that I can understand iOS internal storage systems better.

In particular, I am using iOS's built-in data storage solution called **Core Data**. This is the framework Apple uses themselves when they need to allow the user to store data in their apps (the Photos app is one prominent example). Core Data uses SQLite behind the scenes, but also does much more on top of that. You can save property lists, sync the database between Apple devices (e.g. an iPhone and and iPad), and easily edit the database from an interface in XCode (the program used to create iOS apps).

I had also learned a good bit about how to use Core Data in my iOS development course, so this was a perfect chance to try it out!

### Initial App Code

### Model to Database

### Database Initialization

### CloudKit

### 

