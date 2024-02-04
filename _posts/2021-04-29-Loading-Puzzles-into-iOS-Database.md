---
title: "📦 Loading Puzzles into our App"
tag: "KameKurosu"
---

Just as we can prepare pasta from scratch and have it then ready to be cooked, we can prepare our data to be available for use by our app!<!--more-->

Okay, I admit that was a terrible analogy - I think I'm just hungry... also note that this is quite a long and technical blog post - primarily because it took me so long to figure all of this out!

### Choosing a Database Solution

There are many ways you can store data for an iOS app. You can store it in a local database (which resides on the actual iPhone), in a flat file (like our JSON file), or you can store it externally in a database on some server somewhere. While I have the data currently in a JSON file, I don't want to load that whole file every time I start the app - it is really big! In addition, I want to store values that **can change**, like inputted characters in a crossword or a list of completed levels.

I decided to use one of Apple's built-in options to store all this data for a few reasons: (1) better performance using a built-in database than using one hosted on a server somewhere, (2) good documentation provided by Apple on how to set it up since it is commonly used, (3) easy-to-setup iCloud syncing between devices, (4) it is completely free to use, and (5) as a beginner, I just wanted to use the most barebones approach so that I can understand iOS internal storage systems better.

In particular, I am using iOS's built-in data storage solution called **Core Data**. This is the framework Apple uses themselves when they need to allow the user to store data in their apps (the Photos app is one prominent example). Core Data uses SQLite behind the scenes, but also does much more on top of that. You can save property lists, sync the database between Apple devices (e.g. an iPhone and and iPad) using a service called **CloudKit**, and easily edit the database structure from an interface in **XCode** (the program used to create iOS apps).

I had also learned a good bit about how to use Core Data in my iOS development course, so this was a perfect chance to try it out!

### Initial App Code

Before importing any data into our app, I first needed to create the barebones of the app! When you start a new iOS project in XCode, it automatically generates the following code so you have a functional barebones app right out of the gate!

This first file creates the app and says we have one view, the ContentView (currently the only view in our app).

{% highlight swift %}
//
//  KameKurosuApp.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/18/21.
//
import SwiftUI

@main
struct KameKurosuApp: App {
    let persistenceController = PersistenceController.shared

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
        }
    }
}
{% endhighlight %}
<figcaption class="blogImageCaption">KameKurosuApp.swift: This creates our app and is where we specify the overall settings for our app. For example, we can add settings here to tell iOS what to do when a user enters/exits our app.</figcaption>

This next file creates said ContentView. The top section is where we would begin designing our UI and anything you see on the screen (I just added some text saying "Test" so far). Eventually we will have many view files (one for the menu screen, one for the title screen, etc.) and they will all be hooked up to one another so you can navigate between them. The bottom section is just for XCode to create a preview, like in the image below.

{% highlight swift %}
//
//  ContentView.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/18/21.
//
import SwiftUI
import CoreData

struct ContentView: View {
    @Environment(\.managedObjectContext) private var viewContext

    var body: some View {
        Text("Test")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView().environment(\.managedObjectContext, PersistenceController.preview.container.viewContext)
    }
}
{% endhighlight %}
<figcaption class="blogImageCaption">ContentView.swift: This is the default view file that gets generated when you start building a new app.</figcaption>

![Initial App Preview with Test Text](/assets/images/blog-kamekurosu/preview-initial-test.png){:class="blogVerticalImage"}
<figcaption class="blogImageCaption">The ContentView when the app is running. It currently just writes "Test" to the screen.</figcaption>

Now that we have our (super basic) app up and running, we can write some code to load puzzle data into our app!

### Model to Database

If you remember back to the last blog post, we discussed data models, like the one I created for this app:

![Data Model](/assets/images/blog-kamekurosu/data-model.png){:class="blogHorizontalImage"}
<figcaption class="blogImageCaption">Data model for crossword data - each table represents a specific piece of data and has relationships with the other tables</figcaption>

Well, now it is time to tell our app about this data model so it can create a database for us. Then we can tell the app to load the big JSON file we created with all our puzzle data and have the app store that into the database according to the data model.

To tell our app about our data model, we have to create a Core Data **Context** object and a **DataStore** object. This **Context** object allows us to interact with a database built based on our data model, represented by our **DataStore**.

Within my **DataStore**, I needed two individual **Stores**, one for the local data we save only on the device, and the other for data both saved locally and to the cloud (for syncing across devices).

It took me multiple days to figure out how to do this, but in the end I figured it out. The following is the main bit of code I wrote to solve this problem.

{% highlight swift %}
//
//  Persistence.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/18/21.
//
import CoreData

struct DataStore {
    static let shared = DataStore()
    
    /// A Core Data container with two stores, one for local data and one for cloud data
    let persistentContainer : NSPersistentCloudKitContainer = {
        let container = NSPersistentCloudKitContainer(name: "KameKurosu")
        let defaultDirectoryURL = NSPersistentCloudKitContainer.defaultDirectoryURL()

        // Create a store description for the local store
        let localStoreURL = defaultDirectoryURL.appendingPathComponent("Local.store")
        let localStoreDescription = NSPersistentStoreDescription(url: localStoreURL)
        localStoreDescription.configuration = "Local"

        // Create a store description for the CloudKit-backed store
        let cloudStoreURL = defaultDirectoryURL.appendingPathComponent("Cloud.store")
        let cloudStoreDescription = NSPersistentStoreDescription(url: cloudStoreURL)
        cloudStoreDescription.configuration = "Cloud"
        
        // Set the container options on the cloud store
        cloudStoreDescription.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(containerIdentifier: "iCloud.com.Kamesama.KameKurosu")

        // Update the container's list of store descriptions
        container.persistentStoreDescriptions = [localStoreDescription, cloudStoreDescription]
        
        // Load both stores
        container.loadPersistentStores { _, error in
            guard error == nil else {
                fatalError("Could not load persistent stores. \(error!)")
            }
        }
        
        return container
    }()
}
{% endhighlight %}

Walking through this, line by line:

{% highlight swift %}
static let shared = DataStore()
{% endhighlight %}

This guy just ensures that we create a single (i.e. static) DataStore object that gets shared by any other entities in our app that want to interact with it, i.e. we can just do DataStore.shared wherever we need to use it in our app.

{% highlight swift %}
/// A Core Data container with two stores, one for local data and one for cloud data
let persistentContainer : NSPersistentCloudKitContainer = {
    let container = NSPersistentCloudKitContainer(name: "KameKurosu")
    let defaultDirectoryURL = NSPersistentCloudKitContainer.defaultDirectoryURL()
{% endhighlight %}

Here, we are creating our persistentContainer, which you can just think of as the actual database object. I give it a name and also get its location in the file system for the next step.

{% highlight swift %}
/ Create a store description for the local store
let localStoreURL = defaultDirectoryURL.appendingPathComponent("Local.store")
let localStoreDescription = NSPersistentStoreDescription(url: localStoreURL)
localStoreDescription.configuration = "Local"

// Create a store description for the CloudKit-backed store
let cloudStoreURL = defaultDirectoryURL.appendingPathComponent("Cloud.store")
let cloudStoreDescription = NSPersistentStoreDescription(url: cloudStoreURL)
cloudStoreDescription.configuration = "Cloud"
{% endhighlight %}

In the first part, I'm specifying the location in the filesystem for the local part of the database, and letting the app know this is for local data only. In the second part, I do the same for the cloud-based store.

{% highlight swift %}
// Set the container options on the cloud store
cloudStoreDescription.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(containerIdentifier: "iCloud.com.Kamesama.KameKurosu")
{% endhighlight %}

And here I just give the cloud-based store a link to an online version of the database to sync any data to. I had to set up this link and database beforehand using the online CloudKit Dashboard.

{% highlight swift %}
// Load both stores
container.loadPersistentStores { _, error in
    guard error == nil else {
        fatalError("Could not load persistent stores. \(error!)")
    }
}

return container
{% endhighlight %}

Finally, I instruct the container to try and load the database when the DataStore object is first initialized (which occurs in the very first line of the DataStore struct block). It prints out an error message if needed.

That's pretty much it! There were some other bits of code I had to write to do things like save data to the database or wipe the database (I made a lot of mistakes when I was first trying to save the puzzle data!). You can see the full file [here](https://github.com/jonstrutz11/KameKurosu/blob/fd2a071845f3a47f803fd470b94980d55e63de70/KameKurosu/Controllers/DataStore.swift) if interested.

### Puzzle Data Model
So even though I created a data model already in the app, I still need to specify exactly what data the app should expect when it reads the JSON file with all our puzzle data. To do this, I created a Swift "model" file called KameKurosuPuzzleData.swift:

{% highlight swift %}
//
//  CrosswordData.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/27/21.
//

import Foundation


struct KameKurosuPuzzleData: Decodable {
    
    let levelData: [LevelData]
    let version: String
    
}

struct LevelData: Decodable {
    
    let id: Int
    let name: String
    let nrows: Int
    let ncols: Int
    let puzzles: [PuzzleData]
    
}

struct PuzzleData: Decodable {
    
    let number: Int
    let id: String
    let level: Int
    let words: [WordData]
    
}

struct WordData: Decodable {
    
    let across: Bool
    let clueNumber: Int
    let row: Int
    let col: Int
    let kanjiForm: String
    let reading: String

}
{% endhighlight %}

Note that this "model" for our JSON file is technically different than our "data model" I alluded to earlier. However, you can see it is very similar - and this is because we intentionally made our JSON file look similar to our "data model" to make it as easy as possible to import all the data!

Specifically, to read a JSON file, Swift requires you to create a hierarchy of structs following the "Decodable" protocol. This ensures that as Swift reads the JSON file, it knows exactly what to expect in terms of how the data is organized.

At this point, we can do something like the following to read all this data into the app (but not yet into the database!):

{% highlight swift %}
let decoder = JSONDecoder()

let url = Bundle.main.url(forResource: "KameKurosuPuzzleData", withExtension: "json")!
do {
    let data = try Data(contentsOf: url)
    let jsonData: KameKurosuPuzzleData = try! decoder.decode(KameKurosuPuzzleData.self, from: data)
} catch {
    fatalError("Could not find crossword data flat file when attempting to read puzzle data.")
}
{% endhighlight %}

The first line initializes the decoder we can use to read the JSON file. Note that in the next line we feed it the filepath to our JSON data file. We then read the data from the file, printing out a useful error message in case we run into any errors or bugs. The KameKurosuPuzzleData.self argument to decoder.decode() points the decoder to our model file which we created above.

Now that we have all this data loaded into the jsonData variable, above, we can figure out how to insert it all into the database!

### Database Initialization

The next thing I did was create a function called loadJsonDataIntoCoreData to take our jsonData and insert it into our database. It's quite long, but here is the finished result:

{% highlight swift %}
/// Load all JSON puzzle data into core data. This includes levels -> puzzles -> words -> cells.
private func loadJsonDataIntoCoreData(data: KameKurosuPuzzleData) {
    
    func loadCellData(level: Level, puzzle: Puzzle) {
        ...
    }
    
    func getWordCoords(word: Word) -> [(Int16, Int16)] {
        ...
    }
    
    data.levelData.forEach { (levelData) in
        let newLevel = Level(context: context)
        newLevel.id = Int16(levelData.id)
        newLevel.currentPuzzleID = "\(levelData.id)-0001"  // Start at puzzle 1 for this level
        newLevel.name = levelData.name
        newLevel.nrows = Int16(levelData.nrows)
        newLevel.ncols = Int16(levelData.ncols)
        newLevel.npuzzles = Int16(levelData.puzzles.count)
        newLevel.npuzzlesUnlocked = 20  // TODO: update this for IAP
        
        levelData.puzzles.forEach { (puzzleData) in
            let newPuzzle = Puzzle(context: context)
            newPuzzle.id = puzzleData.id
            newPuzzle.completed = false  // TODO: update this to sync with cloudkit
            newPuzzle.level = newLevel
            newPuzzle.number = Int16(puzzleData.number)
            
            puzzleData.words.forEach { (wordData) in
                let newWord = Word(context: context)
                newWord.across = wordData.across
                newWord.puzzle = newPuzzle
                newWord.clueNumber = Int16(wordData.clueNumber)
                newWord.row = Int16(wordData.row)
                newWord.col = Int16(wordData.col)
                newWord.kanjiForm = wordData.kanjiForm
                newWord.reading = wordData.reading
                
                newPuzzle.addToWords(newWord)
            }
            
            loadCellData(level: newLevel, puzzle: newPuzzle)
            
            newLevel.addToPuzzles(newPuzzle)
        }

    }
    
    do {
        try context.save()
    } catch {
        fatalError("Error while saving local core data context during data initialization, \(error)")
    }
    return
}
{% endhighlight %}

I've omitted the first two helper functions for the sake of brevity. At a high level, the function basically:
1. Loops through each level type (Beginner I, Beginner II, Intermediate I, etc.) and creates a Level object with the associated data
2. Within each level type, loops through each puzzle and creates a Puzzle object with the associated data
3. Within each puzzle, loops through each word and creates a Word object with the associated data
4. After looping through each word in a given puzzle, creates a bunch of Cell objects with their associated data (loadCellData)
5. Saves everything to the database with a context.save() command

Note that the Level, Puzzle, Word, and Cell objects relate directly back to our data model at the beginning of the blog post. These structs are created automatically for us to use by XCode.

I wrote a very similar (but much shorter) function as well for the cloud-based data (creating and saving UserInfo and CompletedPuzzle objects to the cloud store).

And thats it! We can now use these functions to take all the data in our JSON file and write it into our core data database!

### Loading Data only Once

The final step is to make sure that all of this JSON data is only loaded the *very* first time the user loads up the app. Every subsequent time, we will already have the data in the database so don't need to transfer it!

I started by updating my main file that starts up the app. I added some code to let me know in the debugger (text output only the developer can see) whether the app is being entered, exited, etc. I also wrote a bit here to make sure to save any data that has recently changed to the database if the user leaves the app.

I also wrote some code near the top of this file to grab our DataStore's context (so we can interact with it from this file) and load everything if the DataStore is empty.

{% highlight swift %}
//
//  KameKurosuApp.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/18/21.
//

import SwiftUI

@main
struct KameKurosuApp: App {
    @Environment(\.scenePhase) var scenePhase

    let context = DataStore.shared.persistentContainer.viewContext
    
    init() {
        let coreDataInitializer = CoreDataInitializer(context: context)
        coreDataInitializer.initializeCoreDataIfEmpty()
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView().environment(\.managedObjectContext, context)
        }
        .onChange(of: scenePhase) { (newScenePhase) in
            switch newScenePhase {
            case .active:
                print("App is active")
            case .inactive:
                print("App is inactive")
            case .background:
                print("App is in background")
                do {
                    try context.save()
                    print("Context saved to core data store upon leaving app.")
                } catch {
                    fatalError("\(error)")
                }
            default:
                print("Unexpected scene phase: \(newScenePhase)")
            }
        }
    }
    
}
{% endhighlight %}

Specifically, I wrote a method called CoreDataInitializer.initializeCoreDataIfEmpty() which is shown below:

{% highlight swift %}
//
//  CoreDataInitializer.swift
//  KameKurosu
//
//  Created by Jon Strutz on 1/24/21.
//

import CoreData

struct CoreDataInitializer {
    
    let context: NSManagedObjectContext
    
    /// If there is no data in the core data store yet (e.g. the first time the app is run), then we load all data from flat files distributed with the app.
    public func initializeCoreDataIfEmpty() {
        coreDataLocalIsEmpty() ? initializeLocalCoreData() : print("Local Core Data already loaded. Skipping initialization.")
        coreDataCloudIsEmpty() ? initializeCloudCoreData() : print("Cloud Core Data already loaded. Skipping initialization.")
    }
    
    /// Returns true if there are no Level entities present in the core data store. False otherwise.
    private func coreDataLocalIsEmpty() -> Bool {
        return !entityExists(for: "Level")
    }
    
    /// Returns true if there are no CompletedPuzzle entities in the core data store. False otherwise.
    private func coreDataCloudIsEmpty() -> Bool {
        return !entityExists(for: "CompletedPuzzle")
    }
    
    /// Returns true if at least one instance of given entity exists. False, otherwise.
    private func entityExists(for entityName: String) -> Bool {
        do {
            let request = NSFetchRequest<NSFetchRequestResult>(entityName: entityName)
            let count = try context.count(for: request)
            return count > 0
        } catch {
            return false
        }
    }

    ...
}
{% endhighlight %}

I also moved all the functions I wrote previously to transfer the data into the database from the JSON file into this struct.

I basically just check to see if there are any Level entries in the local store and any CompletedPuzzle entries in the cloud store. If not, I assume it is empty and use the functions written previously to load the data in.

### Summary

We first had to choose a good method to persist our data inside our app. Because I wanted something that was *relatively* easy to set up, I picked **Core Data**, which Apple themselves use for many of their built-in apps. I also decided to use **CloudKit** to allow for syncing of some data (like which levels have been completed) between devices.

I then created a barebones app that basically does nothing except show some text on the screen.

Once I had this minimal app up and running, I wrote (and rewrote!) a bunch of code to set up the database and transfer all of our puzzle data into it the first time the user opens up the app.

And that's it! As you can see, this was quite a bit of work and took me a *very* long time. But I'm happy this step is finally done! Now that all our data is present in our app, we can work on the UI and logic of how our app behaves!
