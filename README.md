# REST API Best Practices

I've had the good fortune of being able to create quite a few REST APIs for a number of different projects and I've found that I've fallen into patterns with the design that seem to work very well.

## Context

To help explain my ideas, I'm going to talk about them in the context of the theoretical Pets API which lets consumers interact with information regarding dogs and cats.

## Route Vocabulary

A route is where the rubber meets the road for a REST API.  They're what your consumers use to actually interat with your API so it's critical that they be designed in a standard, predictable, and intuitive way.  A route may look like this one: `/pets/v1/dogs/1`

The best pattern that I've found for doing this is most simply described below:

### `/name/...`
`
- `name`: This is the name of your API, `pets` in our example.  It should be something short, simple, and user-friendly.  It gives you a namesapce on the server to attach your API.

### `/name/version/...`

- `version`: I've seen a lot of different ways to handle the versioning of an API and most approaches work, I've come to appreciate using the Major version number as this part of the route.  Prefixing the major version number with a `v` helps to identify the function of this part of the route.  This is the `v1` part of our Pets API.

### `/name/version/nouns/{id}/...`

- `nouns`: Nouns refer to the collection of top-level objects that your API provides, `dogs` for example.  __Nouns should be plural!__  I recommend against allowing the nouns routes to be called without an ID specified.  Making that possible introduces the eventuality that making that call will return a huge response and crash either the client or the server.  If you need to return a list of nouns, consider using a `search`, `find`, or `list` action to do so.

### `/name/version/nouns/{id}/{action}`

- `action`: This refers to an action that you may want to take either on your API or on the nouns that your API is serving.  For example, if you wanted to a list of dogs with brown fur: `/pets/v1/dogs/search?fur=brown`
  - `!!!CAUTION!!!` If you use actions in your routes, you should give serious consideration to the synchronicity of the action.  If your action initiates a long-running or asynchronous action, you should return a 'got-it' message and have some way to track progress of the action on the noun.  If you hold the response until the action is complete, you __must__ ensure that you have a timeout set.  Bad stuff happens if you don't do this!

### `/name/version/nouns/{id}/subnouns/{id}`  

- `subnouns/{id}`: Sometimes, it makes sense to organize nouns in a parent-child, local-key, or foreign-key relationship via the route and I've found that subnouns make that really easy.  For example, if you wanted to get information about dog 1's third tooth, it makes sense to ask for them with a route like `/pets/v1/dogs/1/teeth/3`.  How a subnoun route is actually implemented is up to the preference of the developer, however, by exposing this route, you're making it possible for a client receive subnoun data with a single request instead of multiple.
  - Client operation without subnouns
    1. `GET /pets/v1/dogs/1`
    2. Find `dog.teeth[3].id`
    3. `GET /{some other call with tooth id}`
  - Client operation with subnouns
    1. `GET /pets/v1/dogs/1/teeth/3`
It's still possible to have teeth and dogs both be top-level nouns and track relationships with local keys OR you can store teeth on each dog object.  This technique doesn't remove overall complexity, but it does push it into the API code which makes your API easier to consume and probably more responsive.  For some things (teeth!), it is reasonable to propose that there are only ever going to be a set limit of children that should simply be stored on the parent object with an ID unique to the parent.  However, sometimes it makes sense to promote the subnoun to a top-level noun and keep track of relationships between nouns with a secondary key.  If the dog has fleas, storing them on the dog object may cause problems if the number of fleas continues to grow.

## CRUD-A

The term "CRUD" is fairly well-known in the REST API world to refer to "create, read, update, and delete" which are the basic functions of a REST API and typically map to HTTP verbs like this:

- `POST` - create
- `GET` - read
- `PUT` - update
- `DELETE` - delete

Actions should be used either to initiate a server-side process or to reduce/filter the response from the server.

Actions must __not__ be used to do CRUD operations!  HTTP verbs do that!
