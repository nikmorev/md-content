# Flows of user authentication and authorization

## Authorization flow using JWT

### On signup/login request

1. Create user (and save to db) using credentials (email/phone/username and password) or find user trying to login;
2. If user is saved/found - generate JWT token (in payload save user's ID), save it in db for that user (probably redis, and consider that multiple token can be available for one user), send JWT token in a response;
3. Save token on the client side in local storage, or save token in cookie

### During the session
1. On every response client should send JWT token via header (Authorization: Bearer token) or via cookie
2. On every request server validates JWT token (decodes token, extracts ID of user, find user, and checks if user has that token and token is valid)
3. If token is valid - server should give a response to the request, otherwise it should send 401 code status or redirect to login page.



