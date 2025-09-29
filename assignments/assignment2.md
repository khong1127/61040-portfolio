# Assignment 2: Functional Design

## Problem Statement

### Problem Domain

**Chosen Domain: Birding**

I started researching birds in my sophomore year of high school as part of Science Olympiad, but it quickly grew into a more personal interest/hobby that I have carried with me to this day. I like to keep track of which birds I've spotted, and I also like to share my birding experiences with others. While I don't have as much time to birdwatch anymore, I still have no problem talking/teaching about birds for hours. I believe it would be satisfying to help popularize the hobby considering that it is more popular among older generations than younger.

### Problem

* Collaborating in birding: making birding more personable

I find birding more fun with others especially considering how slow-paced the activity usually is. Having in-person birding trips with others is able to fulfill that, but I find that there isn't a very good remote equivalent. Existing apps offer ways to keep track of which birds you and others have seen, but the latter is usually at the global scope in the form of a research-oriented database (generally linked to the Cornell Lab of Ornithology). I think it would be nice to have an app that can make collaboration more focused, allowing people to have more control over whose bird activity they can follow.

### Stakeholders

* Birders: people who engage in the hobby of birding
* Birding societies: communities dedicated to raising awareness of birds and birding
* Family and friends of birders: family and friends, may accompany birders on their birding trips / have an interest in being involved in something the birder enjoys
* Environmentalists: those with a vested interest in the health of the environment

Birders will have a more official platform to share their findings with those they trust and love. Members within a birding society can remain connected with each other by having an app that allows you to see each other's achievements compared to just a global database of people's spottings. Family and friends may feel connected to the birder by being able to participate casually with them. Environmentalists could benefit as this can promote greater appreciation for birds and potentially consequently for nature/the environment.

### Evidence and Comparables

* [eBird News](https://ebird.org/news/2024-year-in-review) In 2024, 11.8 million people used Merlin Bird ID to help identify birds. This was a 40% increase from the previous year, thus showing an increased interest in birding and in the use of birding apps. This suggests a decent potential audience should a collaborative birding app be made.
* [The Guardian](https://www.theguardian.com/australia-news/2024/jan/07/birdwatching-changes-the-way-you-look-at-the-world-it-truly-is-the-gateway-drug-to-environmental-awareness) Increasing bird awareness/knowledge can promote people's roles in the environment from observers to advocates.
* [eBird](https://ebird.org/home) eBird is a widely popular app for searching up bird species and submitting bird spottings. It's incredibly effective as a database and for citizen data science collection but doesn't seem to have a feature where you can narrow your collaborative birding scope from global.
* [Merlin Bird ID](https://merlin.allaboutbirds.org/) Merlin is very good for birders who may need some help identifying a bird that they've seen or heard outside. However, like eBird, it doesn't seem to have an option to create focused birding groups.
* [iNaturalist](https://www.inaturalist.org/) iNaturalist allows users to track species that they've spotted and share that with specific people. However, it is not limited to birds.
* [Audubon](https://www.audubon.org/bird-guide) Audubon is a highly trusted field guide for birds. However, its usage is more exploratory and better for research than for birders that are more interested in identification and tracking.

## Application Pitch

### Name

**Hatch**

Meant to be short and memorable. The name comes from the idea of creating/hatching a new birding community to grow together with.

### Motivation

Birding is more fun and meaningful when shared, but there lacks a simple way for birders to share spottings with chosen groups of people compared to a large, global database (the standard for existing birding apps today) as existing options either focus on global scientific research or general-purpose social media.

### Features
* Friending
    * Explanation: Users add each other as friends by searching up their username. Users' information and sightings will only be visible to other users who they are friends with.
    * How it mitigates the problem: Friending decreases the birding scope from global anonymity to more personal while remaining controlled for privacy/security reasons.
    * How it impacts stakeholders: Birding societies can easily friend each other to share accomplishments, as can family/friends. In making birding more personable, the hobby as a whole may become more attractive/popular and inspire more people to become environmentalists.

* Bird Logging as Posts
    * Explanation: Upon starting a birdwatching session, users can use the app to take pictures of birds they find. When the session is ended, the pictures will be amalgamated into a Instagram-style post for which users can write a caption for the overall session and then publish that post for their friends to see. Others can then comment on these posts as they wish.
    * How it mitigates the problem: Posts and comments are the primary way users communicate with others in their groups about bird sightings, milestones, information, etc. This is a step up from an anonymized database with which users cannot truly interact. This feature highlights engagement and storytellers.
    * How it impacts stakeholders: Birding societies, birders, family, and friends using the app can share their accomplishments and insights with other passionate individuals to foster a greater sense of community centered around birding. In making birding more personable, the hobby as a whole may become more attractive/popular and inspire more people to become environmentalists.

## Concept Design

### Concepts

* **concept** PasswordAuthentication (User)
* **purpose** limit access to known users 
* **principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user 
* **state** 
    * a set of Users with 
        * a username String
        * a password String
* **actions** 
    * register (username: String, password: String): (user: User) 
        * *requires* username to not already exist in the set of Users 
        * *effects* creates a new user of that username and password, adds that user to the set of users, and returns the new user
    * authenticate (username: String, password: String): (user: User) 
        * *requires* user of the argument username and password to exist in the set of Users 
        * *effects* returns the corresponding User 

--------------------

* **concept** Posting (User, Image)
* **purpose** allow users to publish and share content for others to see
* **principle** users can create/publish posts that consist of a caption and at least one image. These posts can be edited and deleted by their owners.
* **state**
    * a set of Posts with
        * a caption String
        * a set of Images
        * an author User
* **actions** 
    * create (user: User, images: Set<Image>, caption: String): (post: Post) 
        * *requires* user to exist, images cannot be empty
        * *effects* creates a new post authored by the user with its content being the caption and images given
    * delete (post: Post):
        * *requires* post to exist
        * *effects* deletes the post for the author and their friends
    * edit (post: Post, new_caption: String): (post: Post) 
        * *requires* post to exist
        * *effects* edits the caption of the post to be that of the new one

--------------------

* **concept** Commenting (User, Post)
* **purpose** enable discussion around shared posts
* **principle** users can comment on posts that are visible to them. Comments can be added, deleted, and edited.
* **state** 
    * a set of Comments with
        * an author User
        * a content String
        * an associated Post
* **actions** 
    * addComment (author: User, content: String, post: Post): (comment: Comment) 
        * *requires* author and post must exist, author must have visibility of the post, content cannot be empty
        * *effects* creates a comment authored by the user under the post with the text content
    * deleteComment (user: User, comment: Comment)
        * *requires* comment must exist, comment must belong to the user
        * *effects* deletes the comment
    * editComment (user: User, comment: Comment, new_content: String) 
        * *requires* comment must exist, comment must belong to the user, new_content cannot be empty
        * *effects* edits the comment content to be that of new_content

--------------------

* **concept** Friending (User)
* **purpose** allow users to add each other as friends to share information with
* **principle** users may add and remove each other as friends. These friendships define boundaries regarding who has access to see whose information.
* **state** 
    * set of Users with 
        * a set of friends User
        * a set of sentRequests Users
        * a set of receivedRequests Users
* **actions** 
    * sendRequest (sender: User, receiver: User)
        * *requires* sender and receiver to exist, sender is not the receiver, receiver not already in sender's set of friends or sent requests
        * *effects* adds sender to receiver's set of received requests, adds receiver to sender's set of sent requests
    * acceptRequest (sender: User, receiver: User)
        * *requires* request from sender exists in receiver's set of received requests
        * *effects* removes sender from receiver's set of received requests, removes receiver from sender's set of sent requests, adds sender to receiver's set of friends and vice versa
    * removeFriend (user: User, to_be_removed_friend: User)
        * *requires* to_be_removed_friend to be in user's set of friends
        * *effects* removes to_be_removed_friend from user's set of friends and vice versa

--------------------

* **concept** SessionLogging (User, Image, Flag)
* **purpose** capture photo records of a user's activity during a trip session
* **principle** users can start sessions during which they can log image entries. Entries for a session cannot be recorded once the session is ended. Recorded entries will remain associated with the session even after it is ended.
* **state** 
    * a set of Sessions with
        * an owner User
        * a set of Images
        * an active Flag
* **actions** 
    * startSession (user: User): (session: Session) 
        * *requires* user to exist
        * *effects* creates a new session (active = true) under the user
    * addEntry (session: Session, image: Image)
        * *requires* session and image must exist, session must be active
        * *effects* adds image to the set of images associated with the session
    * endSession (session: Session)
        * *requires* session must exist
        * *effects* ends the session (active = false)

### Syncs

**sync: sessionToPost**
* **when** SessionLogging.endSession(session)
* **then** Posting.create(user, images, caption)
* **where**
    * user = session.owner
    * images = session.images
    * caption = user-provided caption

**sync: comments**
* **when** Commenting.addComment(author: User, content: String, post)
* **then** Allow action execution if author is post.author or post.author is in author's list of friends (query of Friending concept). Otherwise, reject the action.

### Brief Note

PasswordAuthentication handles the registration and basic security aspects of the application. After registering, users can manage their friend lists using the Friending concept. The SessionLogging and Posting concepts are aligned to follow these general steps:

1. User starts a birding session (SessionLogging.startSession)
2. During the trip, the user can take any number of images. (Certainly, there is no barrier from taking non-ornithological images, but we assume user genuinity for the scope of this half-semester project.) (SessionLogging.addEntry)
3. User ends a birding session (SessionLogging.endSession)
4. The images taken during the session will be bundled together to create a draft post (via sync from SessionLogging.endSession)
5. The user can edit the post caption before publishing (Posting.create)
6. After publishing, the user can still edit or delete the post as they wish. (Posting.edit or Posting.delete)

I chose to design the app as such for the following reasons:
* My original thought was to have one post per bird, but I figure this can create quite a bit of clutter and is in general more disorganized.
* I liked the idea of being able to group bird images together into one post, but I wanted to have some more creative difference from Instagram.
* I was inspired by Strava's system for sharing runs for making bird picture grouping more efficient and tied to birding sessions.

From there, users can comment on their own posts or on others whose authors they are friends with.

(This application would need to be a mobile feature due to the mobility that the hobby requires.)

## UI Sketches

![Sketches 1](assets/ui_1.JPG)

![Sketches 2](assets/ui_2.JPG)

## User Journey

Alex is an avid birdwatcher who loves sharing his findings with a small circle of friends. Recently, though, Alex has felt frustrated: larger platforms like Instagram feel too public, while eBird feels too formal. Regular messaging also lacks the amount of organization and filtering that he wants. What Alex really wants is a better way to casually share bird photos and notes with just his close birding friends. 

So, Alex decides to install the Hatch app with his other friends. They all register themselves as a user and add each other as friends. That way, as they start to publish their sightings, they will be able to see each other's posts.

One Saturday morning, Alex heads to the local wetlands for a birding trip. On the app’s home screen, Alex taps “Start Trip" to begin a new session. As he walks along the trail, he spots a Great Blue Heron standing in the shallow waters. Excited, Alex taps the “Log a Bird” button to snap a picture. The photo is then automatically logged to his active trip session.

Over the next hour, Alex adds several more sightings — a Red-winged Blackbird, a Belted Kingfisher, and even a family of Wood Ducks. Each picture is saved to the same session, creating a neat record of his morning findings.

When Alex finishes, he taps “End Trip”, and the app prompts him to write a caption. Alex types: “A peaceful morning at the wetlands :D". The session is then automatically turned into a post that appears in the feed of Alex’s birding friends and in Alex's profile. The post includes the bird pictures that Alex took during his trip at the wetlands as well as his caption.

Later that afternoon, one of Alex’s friends, Maya, opens the app and sees Alex’s trip post on her home feed. She leaves a comment: “Beautiful shot of the heron!” Another friend adds, “I'm so jealous, I haven't spotted a kingfisher in person myself yet!” Since posting and commenting are limited to friends, Alex feels safe sharing his sightings.

By the end of the day, Alex feels both satisfied and connected. The app has given him an easy way to document his birding, share it privately, and spark friendly conversations around each sighting. Instead of feeling scattered across public platforms or isolated with private notes, Alex enjoys a focused space that balances birdwatching, privacy, and community.


