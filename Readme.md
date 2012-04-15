# Octo.js

Octo.js is a simple, flexible, funtional JavaScript library for interaction with the GitHub API v3.  It supports Basic Auth, OAuth 2, and paging.

**Currently requires jQuery or zepto**

## Quick Example

``` coffeescript
api = octo.api()
api.get('/events').on('success', (data) ->
  console.log data
)()
```

`api.get` sets up a closure, so you'll need to invoke it before the request is sent.

``` coffeescript
events = api.get('/events').perpage(50)
  .on 'success', (data) ->
    console.log api.limit()
    console.log events.page() #1

events()
```

## Paging
One goal of octo.js was to make paging very simple.  Paging is built right into the library.

``` coffeescript
events = api.get('/events').on('success', (data) ->
  # the current page
  events.page()

  # go to the next page
  events.next()

  # go to the previous page
  events.prev()
)
events()
```

What if you want to start on a different page and limit the number of results per page?

```coffeescript
# Start on page 5 only returning 10 results per page
events = api.get('/events').page(5).perpage(10)
```

## Callbacks
Octo.js supports two callbacks through `on`: `"success"` and `"error"`.  These callbacks are registered per pager.  This makes it easy to use the same callbacks for each page you request.

```coffeescript
events = api.get('/events')
  .on('success', (data) -> console.log(data))
  .on('error', (name, msg, xhr) -> console.log(xhr.status, name, message))()
```
You can get access to the raw `xhr` object as the last arg of each callback.

## Basic Auth
``` coffeescript
api = octo.api().username('foo').password('bar')
api.get('/user').on('success', (u) -> console.log(u))()
```

## OAuth2
If you've [registered your script or app](https://github.com/settings/applications/new) as an OAuth app, you can use your token to authenticate with the api.

```coffeescript
api = octo.api().token('MY APP TOKEN')
api.get('/user').on('success', (u) -> console.log(u))()
```

This will work with any registered OAuth application, but will return *unauthorized* if you've not registered your application with GitHub.

### Getting an OAuth 2 token from the API
GitHub APIv3 allows you to programmatically fetch a token for use in scripts that might not be websites.  Grabbing an OAuth token **requires a username and password**.  Once you have a token, you can use it without a need for your username and password.

```coffeescript
api = octo.api().username('foo').password('bar')
api.post('/authorizations', {note: 'my script', scopes: ['public_repo']})
   .on('success', (data) -> console.log(data))()
```

## Checking Rate limits
The GitHub API has a rate limit that's returned with the headers of every request.  You can easily access this info to see your limit and how many requests you have left

```coffeescript
api.get('/users/caged/repos').on('success', ->
  # Your limit per hour
  console.log api.limit()

  # Amount you have remaining in that hour
  console.log api.remaining()
)()
```