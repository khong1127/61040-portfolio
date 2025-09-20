# PSet 1: Composing Concepts

## Concept Questions

1. In the case of URL shortening, nonce generation is useful in that when a user wants to shorten a link, a unique nonce can be generated to be used as the URL suffix. The main note is that it is okay for two URLs to use the same suffix if they use different bases. This is precisely why the idea of "contexts" exists, with a "context" being a URL base (e.g tinyurl, shorturl).

2. Storing a set of used strings would help ensure that the same nonce is not returned for a context. Alternatively, we can use a counter behind the scenes that is then converted into a string. (ASCII's range is limited, but we can make use of base notation. Hex has 0-9A-F; we can certainly use a larger base to include more letters of the alphabet.) So, the abstraction function would look vaguely like int -> string. This way, whenever the generate action is called, we return the string version of the counter, increment the counter, and then return the string. This will guarantee uniqueness.

3. As said in the question, using common dictionary words would make the URL easier to remember. However, a disadvantage of this is that there are only so many common dictionary words that exist, which may lead to using more difficult words. This could run into a situation where a user trying to shorten their URL keeps on trying to regenerate a link in the hopes that they get an easier, more memorable nonce / URL suffix when behind the scenes, it doesn't exist. Nevertheless, if we wanted to modify our concept to follow this idea, we would need a set of the most common dictionary words to extract nonces from. This combined with a set of Contexts (each with a set of used strings) would do the trick; each Context can extract nonces from the same dictionary set (storing one dictionary set per Context would be pretty wasteful from an implementation perspective).

## Synchronization Questions

1. It's likely just that in the generate sync, the target URL simply isn't relevant, so it is omitted to avoid confusion. Vice versa for the register sync (for which it is necessary to know the target URL). Perhaps a somewhat similar idea to this is the common practice of not committing/pushing commented code as it detracts from the main code that you're committing. 

2. Omission is only practical when names and parameters are the same since otherwise you would lose information (parameter name is important for giving context of the function, argument name is important for knowing what you're using). While nothing stops us from having all variables with the same names as parameters, it can be helpful to use other names for variables that are a bit more customized (e.g if I had an Animal class and a function taking in animal: Animal, it would be pretty unhelpful to just have all Animal objects be called animal1, animal2, etc. Something like dog, cat, etc. would be more useful). 

3. The expiry setting doesn't directly require any information from the request action. Rather, it just depends on an already created short URL. Certainly, we could include the request, generate, and register actions within the "when," but what is currently written is convenient shorthand that doesn't detract from the understanding of the concept/sync.

4.

* sync generate
    * when Request.shortenUrl ()
    * then NonceGeneration.generate (context: "bit.ly")

* sync register
    * when
        * Request.shortenUrl (targetUrl)
        * NonceGeneration.generate (): (nonce)
    * then UrlShortening.register (shortUrlSuffix: nonce, "bit.ly", targetUrl)

* sync setExpiry
    * when UrlShortening.register (): (shortUrl)
    * then ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)

Notably, the setExpiry sync remains unchanged. Of course, if a different fixed domain is used, then all instances of "bit.ly" above can just be replaced with that.

5.

* sync delete
    * when ExpiringResource.expireResource (): (resource)
    * then UrlShortening.delete(shortUrl: resource)

## Extending a Design

1.

**concept** IncrementCounter
* **purpose** increment the count of an item
* **principle** when an item is created, its counter starts at 0. Afterwards, users can incrementally increase the count of the item by 1.
* **state** a set of Items with
    * an identifier String
    * a number Counter
* **actions**
    * createItem (id: String)
        * *requires* item to not already exist
        * *effects* creates an item with the parameter id and counter 0, puts that item into the set of Items
    * deleteItem (item: Item) : ()
        * *requires* item to exist
        * *effects* deletes the item from the set of Items
    * increment (item: Item) : ()
        * *requires* item to exist
        * *effects* increments the counter of the item by 1
    * analyze (item: Item) : (count: Counter)
        * *requires* item to exist
        * *effects* returns the current counter of the item

**concept** GrantAccess
* **purpose** limit access of items to their owners
* **principle** when an owner-item relation is defined, then authorization to that item will only be granted for the owner
* **state** a set of OwnerRelations with
    * an owner User
    * an item Item
* **actions**
    * defineOwnership (user: User, item: Item) : (ownerRelations: OwnerRelations)
        * *requires* user and item to exist
        * *effects* creates and returns an owner relation between the user and item
    * deleteOwnership (ownerRelations: OwnerRelations) : ()
        * *requires* ownerRelations to exist
        * *effects* deletes the owner relation from the set of OwnerRelations
    * authorize (user: User, item: Item)
        * *requires* ownership relation exists between user and item
        * *effects* successfully authenticates the user for the item

2.

Omissions similar to what was done in the Synchronization section have been made for simplicity and readability.

sync createItem
* when
    * Request.shortenUrl(user)
    * UrlShortening.register(): (shortUrl)
* then
    * IncrementCounter.createItem(item: shortUrl)
    * GrantAccess.defineOwnership(user, item: shortUrl)

sync increment
* when UrlShortening.lookup(shortUrl)
* then IncrementCounter.increment(shortUrl)

sync analyze
* when GrantAccess.authorize(user, item: shortUrl)
* then IncrementCounter.analyze(item: shortUrl)

3.

a. Allowing users to choose their own short URLs

The Request concept is referenced but not shown in the pset, so I do not have a full grasp of what the concept may need for modification. However, it should be checked if there's an action to shorten a URL that takes a target URL, short URL base, and a short URL suffix. We could then add a sync that would look something like:

* sync registerManualSuffix
    * when Request.shortenUrl (targetUrl, shortUrlBase, shortUrlSuffix)
    * then UrlShortening.register (shortUrlSuffix, shortUrlBase, targetUrl)

b. Using the “word as nonce” strategy to generate more memorable short URLs

I believe "Concept Questions" #3 already addresses this, but to repeat: if we wanted to modify our concept (NonceGeneration) to follow this idea, we would need a set of the most common dictionary words to extract nonces from. This combined with a set of Contexts (each with a set of used strings) would do the trick; each Context can extract nonces from the same dictionary set (storing one dictionary set per Context would be pretty wasteful from an implementation perspective).

c. Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL

One way to realize this would be to create a new concept to link target URLs to their set of short URLs. This concept would have actions to (de)associate a short URL to a target URL and to get analytics on the target URL. There can then be a sync where when the analyze action is called for the target URL, it calls the analyze action for all of the short URLs associated with the target URL. 

Presumably, those who get the analytics of the target URL cannot see the individual analytics of each short URL since that would violate the user-only access concept for the short URLs. Rather, I assume that the analytics of the target URL is instead presented anonymously as an aggregate count.

d. Generate short URLs that are not easily guessed

This generally would not be too different from what is written for the NonceGeneration concept since the original concept is already random, but a modification could be made so that the generated nonce is more random (e.g crytographic).

e. Supporting reporting of analytics to creators of short URLs who have not registered as user

This goes against our notion of ownership with analytics. If someone does not register themselves as a user, then from an application standpoint, there wouldn't really be a way to connect someone to their created short URLs.