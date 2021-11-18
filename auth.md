# Authenticating with Next.js - The definitive guide

## Goal

Provide a comprehensive description of the most common authentication patterns for human users in full-stack web application. 
Next.js is taken as an example as it provides all relevant features (API routes, web pages, and middlewares acting as proxy) for authentication.

## Use case

A user access a "/private" page. 
We want to cover 2 scenarios:

1) Check if they are logged in and redirect them, server-side, before the page is even loaded.

Scenario 1 is mandatory when the content of the page is sensitive, eg statically rendering private content.
It supposes using Next.js server-side feature (mainly middlewares). It is appliable to other full-stack frameworks such as Meteor.

2) Check client-side, for non-sensitive client.

Scenario 2 is appropriate when the content is not sensitive. It is not using Next.js server-side feature, so it should work for other client-only frameworks such as Gatsby, Create-React-App.

Server-side checks and client-side
This check may happen:
- client-side

## Non-goals
We don't want to limit ourselves to setup server-to-server communication, or client-to-api communication. 
We also want to secure web pages.

## About JWT


## Types of authentication


## Client-side authentication

## Middleware authentication

## About the official Next.js authentication documenation

## About next-auth

## Resources
https://next-auth.js.org/
https://nextjs.org/docs/authentication
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NjUwODE0LDE3NzY5OTY1ODEsLTMzMj
Q1NTM2M119
-->