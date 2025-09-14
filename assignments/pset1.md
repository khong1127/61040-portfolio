# PSet 1: Concept Design

## Exercise 1: Reading a Concept

1. Each purchase must correspond with a request -- people cannot just buy nonrequested / already-fulfilled items for someone. Additionally, counts of items in a registry must be non-negative, and the aggregated count of a purchased item cannot exceed the original requested count. The count/aggregation invariant would be more important as a negative count would make no logical sense. The "purchase" action would be most affected by this as this invariant adds a precondition of the registry having "a request for this item with at least count" so that the count decrement effect of the action does not make the count of the item become negative.

2. The removeItem action could create issues in that removing a request doesn't change the fact that purchases for that request were made. We would lose information about the purchases and their aggregate of the request removed, which can be unideal. One way to fix this could be to add an extra field to Requests that mark their status (Existing, Removed). If marked as Removed, then the item would not be listed within the gift recipient's list of desired items to gift givers. By doing this instead of completely deleting the request data, we can continue to keep track of request-purchase relationships.

3. Although the principle frames the registry as a one-time use, the specs of the actions imply that a registry can be repeatedly opened and closed. Allowing this may be beneficial for gift recipients who may want to reuse their registries (e.g open it up for their birthday and for Christmas later on).

4. I don't believe this would be too important in practice in that privacy can be protected simply by closing the registry. Deletion could potentially be helpful for cleaning up data storage from the perspective of the application admins, but there are now many lower-cost databases that make this less of an issue.

5. The registry owner may reasonably want to see a list of all purchases made for them. Certainly, closing their registry can do the trick, but it can be inconvenient to have to open/close your registry repeatedly simply to check purchase lists. On the other hand, gift givers would reasonably want to know which items on the recipient's list are still up for purchasing.

6. A surprise flag can be added at the registry level such that when turned on, the registry owner will be unable to query for the list of purchases made for them. Registry owners can then choose to reveal the surprise either by turning off the surprise flag or by closing their registry.

7. Item names can overlap with one another and cause confusion. Item descriptions and prices can change as the item / market evolves. Similar arguments can likely be made for other aspects of the item. Sticking with generic parameters like SKU codes would save this application from having to stay on track with the current state of items since SKU codes are unchanging.

## Exercise 2: Extending a Familiar Concept

1 and 2.

* **concept** PasswordAuthentication 
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

3. No two users can have the same username. This is preserved by the register action that has a precondition checking whether a username is available or not before creating a new user with that username.

4. 

* **concept** PasswordAuthentication 
* **purpose** limit access to known users 
* **principle** after a user registers with a username and a password, they must confirm their registration with a token. Afterwards, they can authenticate with that same username and password and be treated each time as the same user 
* **state** 
    * a set of Users with 
        * a username String 
        * a password String 
        * a verification flag Boolean 
    * a set of UnverifiedRegistrations with 
        * a username String 
        * a token String 
* **actions** 
    * register (username: String, password: String): (user: User, token: String) 
        * *requires* username to not already exist in the set of Users 
        * *effects* creates a new user of that username and password with unverified flag (false), adds the username + password to the set of Users, generates a secret token, adds the username + secret token to the set of UnverifiedRegistrations, and returns the new user and secret token 
    * authenticate (username: String, password: String): (user: User) 
        * *requires* user of the argument username and password must exist in the set of Users, user must be confirmed/verified
        * *effects* authenticates and returns the corresponding User 
    * confirm (username: String, token: String) 
        * *requires* username corresponding with token must exist within the set of UnverifiedRegistrations 
        * *effects* marks User of that username as verified (set verification flag to true), remove username/token from the set of UnverifiedRegistrations, 

## Exercise 3: Comparing Concepts

* **concept** PersonalAccessToken 
* **purpose** provide an alternative to using passwords with additional expiry/deletion features  
* **principle** a verified user can generate a personal access token. This token can then be used in conjunction with their username to authenticate themselves. Personal access tokens can be automatically deleted once their expiry date is reached, or they can be manually deleted.
* **state** 
    * a set of Users with 
        * a username String 
        * a verification flag Boolean 
        * a set of Tokens 
    * a set of Tokens with 
        * a token id String 
        * an expiry date Time 
* **actions** 
    * register (username: String): (user: User) 
        * *requires* username to not already exist in the set of Users 
        * *effects* creates a new user of that username with unconfirmed flag (flag = false), adds that user to the set of Users, and sends a verification link 
    * confirm (username: String) 
        * *requires* username to exist 
        * *effects* marks user of that username as verified (flag = true) 
    * createToken (username: String, expiryDate: Time) : (token: Token) 
        * *requires* user identified by username must exist, expiry date must be some time in the future 
        * *effects* creates a token with expiry date specified by Time parameter, adds token to user's set of tokens, returns the token
    * deleteToken (username: String, token: Token) 
        * *requires* user identified by username must exist, user must have token within their set of tokens
        * *effects* token is removed from the user's set of tokens
    * authenticate (username: String, tokenID: String): (user: User) 
        * *requires* User identified by username must exist, user must have a token identified by tokenID
        * *effects* authenticates and returns the corresponding User 

The two concepts differ in user verification and authentication management. PasswordAuthentication verifies users by sending a secret token to their emails whereas PersonalAccessToken verifies users simply by sending a confirmation link. The more important difference is that PasswordAuthentication passwords are created once and last forever. On the other hand, PersonalAccessToken tokens can be created and deleted easily (the latter can be done either manually or automatically through expiry dates that are set during token creation).

The Github page can certainly be improved by making this clarification between passwords and personal access tokens. The article as is explains that personal access tokens are a password alternative and does well to explain how to use them, but a more explicit explanation on the differences between personal access tokens and passwords would help explain why this alternative exists to begin with.

## Exercise 4: Defining Familiar Concepts

### URL Shortener

* **concept** URLShortener 
* **purpose** create shorter, easier links  
* **principle** a user enters a link that they want shortened and have the option to customize the link with their own suffix. The application then returns a short link (using the user-defined suffix if provided) that when used will redirect the user to the webpage that the original long link pointed to.
* **state** 
    * a set of ShortURLS with 
        * a originalURL String
        * a suffix String
* **actions**
    * shortenURLAuto (originalURL: String): (shortenedURL: ShortURL) 
        * *requires* originalURL must be valid 
        * *effects* creates a short and unique suffix, creates and returns a new ShortURL with that suffix that links to originalURL 
    * shortenURLUserDefined(originalURL: String, suffix: String): (shortenedURL: ShortURL) 
        * *requires* originalURL must be valid, no existing ShortURL uses the suffix already 
        * *effects* creates and returns a new ShortURL with the specified suffix that links to originalURL 
    * redirectURL (suffix: String): (originalURL: String) 
        * *requires* ShortURL of that suffix to exist 
        * *effects* returns the original long URL corresponding to the ShortURL of that suffix 

This is based off of the most basic functionality of tinyurl, which does not include a way to delete shortened links that have been created. 

### Billable Hours Tracking

* **concept** BillableHourTracking 
* **purpose** Track work sessions by employees 
* **principle** An employee starts a work session by selecting a project and describing the work to be done. The session is tracked until ended either by the employee themselves or automatically upon reaching a duration cutoff in the case that the employee forgets to end their work session. 
* **state** 
    * a set of Sessions with 
        * an employee ID 
        * a project String 
        * a work description String 
        * a start time Time 
        * an end time Time 
* **actions** 
    * startSession (employeeID: int, project: String, workDescription: String, startTime: Time): (session: Session) 
        * *requires* employee id and project must exist
        * *effects* creates a new work session for the employee defined by employeeID for the specified project with the given work description, records the start time of the session using startTime, marks end time of the session as null (for now, equates to the session currently being active), returns the work session 
    * endSession (session: Session, endTime: Time) 
        * *requires* session to exist and be active (indicated by a null end time) 
        * *effects* records endTime as the end time of the session 

To handle the case that an employee forgets to end their work session, we can set up a maximum session time (perhaps 8 hours to mimic the standard work day duration) that, if reached, will trigger a call to the endSession action that passes in the session and the current time as the endTime. This would be done automatically rather than by an employee.

### Conference Room Booking

* **concept** RoomBooking
* **purpose** create and manage room bookings without scheduling conflicts 
* **principle** a user selects a room and a timeframe to book that room for. Once the booking is made, no other user can book that room within that timeframe. Users can delete bookings that they have made.
* **state** 
    * a set of Rooms with 
        * a name String 
        * a set of Bookings 
    * a set of Bookings with 
        * a bookee User 
        * a start time Time 
        * an end time Time 
* **actions** 
    * createBooking (username: String, roomname: String, starttime: Time, endtime: Time): (booking: Booking) 
        * *requires* User identified by username and Room identified by roomname to exist, corresponding room to be available from starttime to endtime 
        * *effects* creates a new booking for the specified room and timeframe under username, adds the booking to the room's set of bookings, returns the booking 
    * deleteBooking (username: String, room: Room, booking: Booking) 
        * *requires* username + booking + room must exist, booking must exist within the room's set of bookings, booking must belong to the user identified by username
        * *effects* removes the booking from the room's set of bookings 

A user would reasonably want to query a room's set of bookings, but as per Piazza (https://piazza.com/class/melw05erqym3qt/post/39), I have left this out as an action.