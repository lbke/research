# Authenticating with Next.js - The definitive guide

## Goal

Provide a comprehensive description of the most common authentication patterns for human users in full-stack web application. 
Next.js is taken as an example as it provides all relevant features (API routes, web pages, and middlewares acting as proxy) for authentication.

## Use cases

A user access a "/private" page. 
We want to cover 2 scenarios:

1) Check if they are logged in and redirect them, server-side, before the page is even loaded.

Scenario 1 is mandatory when the content of the page is sensitive, eg statically rendering private content.
It supposes using Next.js server-side feature (mainly middlewares). It is appliable to other full-stack frameworks such as Meteor.

2) Check client-side, for non-sensitive client.

Scenario 2 is appropriate when the content is not sensitive. It is not using Next.js server-side feature, so it should work for other client-only frameworks such as Gatsby, Create-React-App.

### Additional features
- "Remember me mode": user stays authenticated for at least one month. If not ticked, user is logged out at the end of the session.
- Token revokation: admin can log out an user (eg after an attack)

## Non-goals
We don't want to limit ourselves to setup server-to-server communication, or client-to-api communication. 
We also want to secure web pages.

## About JWT


## Types of authentication
Asymmetrical auth token with RSA256
Tokens with secret
Token stored in the database


## Client-side authentication

## Middleware authentication

## About the official Next.js authentication documenation
https://nextjs.org/docs/authentication

## About next-auth
https://next-auth.js.org/

## Resources
https://nextjs.org/docs/authentication
https://next-auth.js.org/
https://stackoverflow.com/questions/49920234/how-to-implement-authentication-in-next-js
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNTU4NjAyNDEsOTIxNjExNjE2LDg1Nz
A0MDkyOSwxNzc2OTk2NTgxLC0zMzI0NTUzNjNdfQ==
-->