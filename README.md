App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool

## Project Information:

My appspot app url is:

https://hmbmk-conference.appspot.com/

## Tasks

1. Add Sessions to a Conference


    Defined classes for Session, SessionForm and SessionForms in models.py. Session contains:
        - name (required)
        - highlights
        - speaker
        - duration (in minutes)
        - typeOfSession
        - date
        - startTime
    
    The speaker is stored as a string. Duration is meant to represent minutes.

    The following endpoints and methods were created:
        createSession: 
            Creates a session object by calling `_createSessionObject` and passing SessionForm and webSafeConferenceKey. 
        _createSessionObject: 
            Takes SessionForm and webSafeConferenceKey. creates a session object with the conference matching the websafe key as the parent. Also creates a taskqueue task to check the speaker (see task 4).
        _copySessionToForm:
            Copies all available session properties to a SessionForm
        getConferenceSessions:
            Takes a websafeConferenceKey and returns all sessions descending from the matching conference
        getConferenceSessionsByType:
            Takes a websafeConferenceKey and typeOfSession, querying and returning all sessions of the matching conference key and type
        getSessionsBySpeaker:
            Takes a speaker and returns all sessions with matching speaker values, across all conferences

    
    I chose to implement sessions in a very similar way that conferences are implemented. The logic for creating sessions is based on the conference logic. 

    Speakers are represented as just a string rather than a dedicated entity. I didn't see any compelling reason to add a separate entity just for speakers given the app's function. Especially considering the speaker name is an optional field at this point.

2. Add Sessions to User Wishlist

    User wishlists are represented and stored as a repeated property, wishlistSessionKeys, in the Profile entity. This property stores sessionKeys

    The following endpoints were created to handle Wishlists:
        addSessionToWishlist:
            Takes a sessionKey for the desired session and appends the key to the profile entities wishlistSessionKeys property
        getSessionsInWishlist:
            Queries and returns a SessionForm with the values of all sessions in the current user's wishlist
        deleteSessionInWishlist:
            Takes a sessionKey and removes the matching wishlistSessionKey value, if it exists

3. Work on Indexes and Queries

    Added indexes for necessary queries in index.yaml

    _Come up with 2 additional queries_

        Created the following two queries as endpoints methods:
            getSummerConferences:
                Queries for conferences whose month property is greater than or equal to 6 and less than or equal to 8 (between June and August)
            getLongSessions:
                Takes a webSafeConferenceKey and queries within that conferences sessions, returning any with a duration over 180 (representing minutes i.e. greater than 3 hours)

    _Solve the query related problem_

        The problem with creating a query to select sessions that are neither a workshop nor start after 7pm is that datastore does not allow for an inequality query on two different properties at once. One solution I thought of for this would be querying for all sessions that are not workshops, saving the session keys for those in an array, and then removing any sessions starting after 7 from the array. The array would then contain only the keys for the desired sessions.

4. Add a Task

    Within the _createSessionObject method a taskqueue task is added, passed the value of the speaker of the newly created session along with the websafeConferenceKey of the conference the session is a part of.

    The task is handled with the setFeaturedSpeakerHandler, defined in main.py. This handler queries for sessions with the same speaker and of the same conference as the last added session added. If there is more than one session with the specified speaker, the _cacheFeaturedSpeaker method is called, and passed the speaker and sessions. _cacheFeaturedSpeaker then creates a string containing the speaker name and list of all sessions as the current conference that speaker is the speaker for and adds that to memcache.

    getFeaturedSpeaker:
        Returns the value of memcache, or "" if there is no value