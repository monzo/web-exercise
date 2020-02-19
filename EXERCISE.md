# Build a developer portal!

Your task is to build a browser-based developer portal in JavaScript against a dummy API that we've created! 🎉

Your portal must meet the following requirements:

- The user can log into the portal using email and password
- If the user's access token (obtained during login) expires at any point during a session, the user should be asked to re-authenticate via email and password
- The user can list and update their apps
- For each app, the user can page through a sub-list of the app's users

Other than that, you can build the app **however you want** :) Feel free to host your submission on GitHub or send it in zipped up via email.

The data model is designed to resemble the kinds of data many of our internal tools as well as our [actual developer tools](https://monzo.com/blog/2016/02/03/mondo-api/) use.

You shouldn't spend more than 4 hours on this and not everything has to work. The main objective is to see how you approach things and have something to talk about in our interview :)

Please note down any changes that you would make if you would have more time.

# API

We've put up a mock API server at https://guarded-thicket-22918.herokuapp.com/.

### Authentication

All API requests (except for logins) are authenticated by passing an access token in via the `"Authorization"` HTTP header. If the token is missing, invalid or expired the API returns `401 Unauthorized`.

Access tokens expire after a certain amount of time (defaults is 30m) but you can override this during login.

You can check whether or not an access token is valid and not expired by hitting `GET /`.

### Obtaining an access token

You obtain an access token by making JSON-encoded POST request to `/login` with the following properties:
- `email`: You can put whatever you want here. Each `email` gets its own little database of mock data.
- `password`: If the password is "hunter2", the login will succeed. Otherwise it will fail with status 401.
- `expiry`: Optionally, you can pass in an expiration timespan in [rauchg/ms](https://github.com/rauchg/ms.js) format (e.g. `"60s"`, `"10h"`, etc). This is useful for testing your re-authentication code.

For example:
```bash
# Obtain an access token
curl -H "Content-Type: application/json" -X POST -d '{"email":"mondo@example.com","password":"hunter2"}' https://guarded-thicket-22918.herokuapp.com/login
# Status: 200
# {
#     "accessToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Im1vbmRvQGV4YW1wbGUuY29tIiwiaWF0IjoxNDU0NTMzMDc4LCJleH# AiOjE0NTQ1MzQ4Nzh9.9nnNyJaR-oZeOjlGFUrimSuLzRUJ3kfzuxbQwTuODBg"
# }

# Test your access token
curl -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Im1vbmRvQGV4YW1wbGUuY29tIiwiaWF0IjoxNDU0NTMzMDc4LCJleHAiOjE0NTQ1MzQ4Nzh9.9nnNyJaR-oZeOjlGFUrimSuLzRUJ3kfzuxbQwTuODBg" https://guarded-thicket-22918.herokuapp.com/
# Status: 401
# {
#     "message": "The API is alive and your access token is valid :)",
#     "token": {
#         "email": "mondo@example.com",
#         "iat": 1454533078,
#         "exp": 1454534878
#     }
# }

# Login failure
$ curl -H "Content-Type: application/json" -X POST -d '{"email":"mondo@example.com","password":"not hunter2"}' https://guarded-thicket-22918.herokuapp.com/login
# Status: 200
# {
#     "error": "Cannot log in with the given email and password."
# }

# Get a short-lived access token to test re-authentication
curl -H "Content-Type: application/json" -X POST -d '{"email":"mondo@example.com","password":"hunter2","expiry":"10s"}' https://guarded-thicket-22918.herokuapp.com/login
```

### Apps and their users!

The developer portal deals with two domain models: `apps` and their `users`. The API has three endpoints:

`GET /apps` returns a list of all apps. This list doesn't include the apps' users.

`PUT /apps/$appId` updates an app and returns the updated app. You can change the `name` and `logo` properties.

`GET /apps/$appId/users?limit=25&offset=0` returns a list of users for a given app. This list can be quite long so you'll need to implement paging using the limit and offset parameters. The API doesn't support returning more than 25 results at once.

Example calls and data model:
```bash
# List all apps (after obtaining an access token as described above)
curl -H "Authorization: $token" https://guarded-thicket-22918.herokuapp.com/apps
# {
#     "apps": [
#         {
#             "id": "ebdb9723-39ba-4157-9d36-aa483581aa13",
#             "name": "Intelligent Steel Car",
#             "created": "2016-01-25T03:57:53.873Z",
#             "logo": "http://lorempixel.com/400/400/animals"
#         },
#         // and so on...
#     ]
# }

# Update an app
curl -H "Content-Type: application/json" -H "Authorization: $token" -X PUT -d '{"name":"New Name"}' https://guarded-thicket-22918.herokuapp.com/apps/ebdb9723-39ba-4157-9d36-aa483581aa13
# {
#     "app": {
#         "id": "ebdb9723-39ba-4157-9d36-aa483581aa13",
#         "name": "New Name",
#         "created": "2016-01-25T03:57:53.873Z",
#         "logo": "http://lorempixel.com/400/400/animals"
#     }
# }

# List users of an app (first page of 25 users)
curl -H "Authorization: $token" https://guarded-thicket-22918.herokuapp.com/apps/ebdb9723-39ba-4157-9d36-aa483581aa13/users
# {
#     "users": [
#         {
#             "id": "6b09a204-0653-4303-9370-222b06c478a8",
#             "name": "Madeline Runte",
#             "email": "Viviane.Beatty58@yahoo.com",
#             "avatar": "https://s3.amazonaws.com/uifaces/faces/twitter/chrisstumph/128.jpg"
#         },
#         {
#             "id": "f73b5837-6035-40d8-8008-c0e71605670b",
#             "name": "Marlin Goodwin",
#             "email": "Zechariah.Fisher@yahoo.com",
#             "avatar": "https://s3.amazonaws.com/uifaces/faces/twitter/HenryHoffman/128.jpg"
#         },
#         // and so on...
#     ]
# }

# List users of an app (second page of 25 users)
curl -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Im1vbmRvQGV4YW1wbGUuY29tIiwiaWF0IjoxNDU0NTM1MDg4LCJleHAiOjE0NTQ1MzY4ODh9.7ehzJgS_OojT37j076I05l1ZNKc62AKOpL-aeqR0GkM" https://guarded-thicket-22918.herokuapp.com/apps/ebdb9723-39ba-4157-9d36-aa483581aa13/users?offset=25
```

# What we look for

We picked this code test because it allows us to understand a number of different things:

- How do you structure your code?
- Are you comfortable implementing more technical bits such as authentication and pagination?
- Does that app look all right visually? We're not looking for any fancy designs but we're a product-driven company and like it when things look nice.

Lastly, we'll use the code-test as a basis to discuss some aspects of web frontend development in more detail later in the interview process :)

Happy hacking!