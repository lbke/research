# Authenticating with Next.js - The definitive guide

## Goal

Provide a comprehensive description of the most common authentication patterns for human users in full-stack web application. 
Next.js is taken as an example as it provides all relevant features (API routes, web pages, and middlewares acting as proxy) for authentication.

This guide is targeted at Next.js users, therefore it should be understandable by frontend developers with basic backend knowledge.

## Use cases

Users can authenticate themselves using password and email. A user access a "/private" page. 
We want to cover 2 scenarios:

1) Check if they are logged in and redirect them, server-side, before the page is even loaded.

Scenario 1 is mandatory when the content of the page is sensitive, eg statically rendering private content.
It supposes using Next.js server-side feature (mainly middlewares). It is appliable to other full-stack frameworks such as Meteor.

2) Check client-side, for non-sensitive client.

Scenario 2 is appropriate when the content is not sensitive. It is not using Next.js server-side feature, so it should work for other client-only frameworks such as Gatsby, Create-React-App.

### Additional features to support

- "Remember me mode": user stays authenticated for at least one month. If not ticked, user is logged out at the end of the session.
- Token revokation: admin can log out an user (eg after an usurpation is identified)

## Non-goals
We don't want to limit ourselves to setup server-to-server communication, or client-to-api communication.
**We also want to secure web pages**. This is important, as many existing documentation are only explaining how to secure an API, and not how to secure web pages.
We don't focus on patterns using a 3rd party (Oauth) yet.
We don't introduce roles or tenants.

## About JWT

This is a summary only for the purpose of this guide. JWT are a vast subject. More information can be found here: https://jwt.io/

A JSON Web Token is a token that encode informations about the user. Typically, you would store a few non-sensitive meta-information, like the user email, name, organization, unique id.

You can do 2 operations on an existing token:

- Decoding: it means getting the information on the token.
**Decoding a token DOESN'T mean it's valid or that the user is authenticated.** Because anyone can craft a token that can be decoded! Think of it as a convenience to get basic user info from the token
- Verifying: it means checking the token validity
**Verifying the token DOES MEAN it's valid and the user is authenticated**. That's what you want to do to use to authenticate someone.

## Types of authentication

Asymmetrical auth token with public/private RSA256
Symmetrical auth tokens with secret => this is used in 

Token stored in the database
Access + refresh token

TODO: find the right naming for those patterns

## Authentication at the Edge (middlewares, Edge API routes)
In order to verify a token, you need a key. 
In a symmetrical approach, there is one **secret** key, used to created and verify token. This mean this operation can only be done server-side, by the authentication server.

In an assymetrical approach, the website owner, you, has a **secret** key used to create tokens. And the client application have a public key, used to verify the token.
It means an assymetrical approach let's anyone check the token validity. That's nice if you are using a 3rd party authentication service like FusionAuth. It means you don't even need to call it to check the token. Technically, even a browser can do it.


(copy pasted from stack overflow)
This question might need an updated answer, now middlewares are there in Next.js 12 (october 2021): https://nextjs.org/docs/middleware

### Verifying the token after auth and redirecting accordingly

Most of the answer from @Yilmaz from april 2020 is still relevant. However, previously, we had to use `getInitialProps` in `_app` to process the request OR a custom server.

**This is no longer the case.**. Using a middleware let's you achieve a similar purpose, with cleaner code. Because middleware are specifically designed for such use cases.

Here, I suppose you get a JWT access token using an asymetrical algorithm like RS256, exactly like in this previous answer.


Here is a possible implementation:


```js
import { NextFetchEvent, NextRequest, NextResponse } from "next/server";

const removeCookie = (res: NextResponse, cookieName: string) => {
  res.headers.append("Set-Cookie", `${cookieName}=; Max-Age=-1; Path=/`);
  return res;
};

export default async function middleware(
  req: NextRequest,
  ev: NextFetchEvent
) {
  const { pathname } = req.nextUrl;
  const isPublic = isPublicRoute(pathname);

  if (isPublic) {
    return NextResponse.next();
  }

  const accessToken = req.cookies[TOKEN_PATH];
  if (!accessToken) {
    return NextResponse.redirect(LOGIN_HREF);
  }

  const isValidToken = await checkAccessToken(accessToken);

  if (!isValidToken) {
    let res = NextResponse.redirect(LOGIN_HREF);
    res = removeCookie(res, TOKEN_PATH);
    return res;
  }

  return NextResponse.next();
}
```
### How to verify the token

In my example, the `checkAccessToken` should verify the token (**not decode**, verify the signature).

This is where things are the most complicated imo.

When using the RSA256 algorithme you also get a PUBLIC certificate (in addition to the SECRET key that must be... kept secret). Eventhough you do the check in the `middleware`, which is private and server-only code, that's good news because it means you could even use it in the browser, in theory.

So, you can either `fetch` the token validation endpoint provided by your auth server, or verify the token yourself. 
**Fetching is not the recommended option** because it might break Vercel/Next edge capabilities and add latency, [according to the documentation](https://nextjs.org/docs/api-reference/edge-runtime#fetch).

I must admit that I did not succeed to verify the token yet using Next.js :) I'll update this answer if I manage to have a code sample that works.



### Login/logout: how to store the token and drop it

This doesn't change with middlewares. You store your access token as an `httpOnly` cookie. When logging out, you unset this cookie.

Managing those Set-Cookies headers are the responsibility of your auth server.


This is a basic workflow but it should work. You can then add a refresh token in the mix with a similar approach.

#### Another approach without httpOnly

Using httpOnly cookie is meant to prevent XSS attack, which happens when people force running some JavaScript code in your browser,
for instance through an unsanitized script or image url.

It should be possible to handle the token storage client-side, as long as you fingerprint the browser as well. This way,
the token only works for a specific browser, and the user have to login again if they swap browsers.
You can store the print in the server database, or within the token itself, and verify it in the request.

https://fingerprintjs.com/


### About token revokation

- If you verify the token in your middleware, **there is no immediate revokation mechanism for the access token**. Because there is no call to a database. 

Therefore, in this scenario, you'd want to opt-in for short lived access token (eg 5 minutes) coupled with a refresh token. You can revoke the refresh token, so basically revoking works but takes a few minutes.

- If a 3rd party server verifies the token: then it could check for blacklisted tokens.


### Caveats

Also, some piece of advice: most articles, tutorials etc. online are focused on server-to-server communication. Or client-to-API. They completely suck when it comes to check authentication before accessing web pages.

For instance, setting the `Authorization` header is not possible in the browser. It works only when communicating with an API. Cookies are mandatory for web pages.

Even then, if this API is meant to be called from a browser, it should preferably accept a cookie.

When discussing with experts on the field, you need to always clarify the Next.js use case.

### Open questions: about session-based authentication

Some frameworks seem to prefer relying on the database. They store a hashed token in the db, which acts as a session. If you want to check auth, you need a server that will check the user's token against the stored token (= checking that there is an active session with this token).

I am thinking of Meteor for instance.

I couldn't find the name of this mechanism and its actual relation to JWT however. Are they simply variations of the JWT approach?

[Next.js official authentication doc](https://nextjs.org/docs/authentication) is not showing middlewares at the time of writing, but instead use `getServerSideProps`. I really don't like this pattern. 
It uses a kind of session system but I am not clear about the internals of it, I am not even sure of the name (is that session-based auth?).

[Vercel edge handles examples](https://github1s.com/vercel/examples/tree/main/edge-functions/jwt-authentication) shows how to secure an API route, but not a page (at the time of writing)

## Client-side redirection

We talk about redirection more than authentication.

When using a JWT token with a public certificate, technically, you can authenticate client-side, but use cases are limited.
Here, the goal is only to redirect non-authenticated users as soon as possible using frontend code.

See this ticket for some advanced pattern for Next.js inspired by Vercel dashboard: https://github.com/VulcanJS/vulcan-next/issues/71

## About the official Next.js authentication documenation
https://nextjs.org/docs/authentication
TODO: describe the pattern selected by Next.js official doc more thoroughly.

### With Passport

The "With Passport" example is using a symmetrical token.

## About next-auth
https://next-auth.js.org/
TODO: describe the pattern selected by Next auth

## About various implementation of auth in modern frameworks
 Remix, Blitz, Redwood... 
 
 ### Remix
 
The "Jokes" tutorial includes an authentication example with detailed explanations of the auth flow.

 https://remix.run/docs/en/v1/tutorials/jokes#authentication

Sessions are documented here: https://remix.run/docs/en/v1.0.3/api/remix#sessions

- The token is hashed using a secret variable. It contains the user id.
- The server checks the user id against the database to confirm existence.
- It does describe an API authentication in this example (securing the creation of a new joke), however, since it relies correctly on cookies and Set-Header it will work ok for pages as well.

## About session-based authentication

Session-based authentication stores the session information in a database, instead of storing them in a JSON web token. Then the sessionId is stored in a cookie, similarly to how we store user information in the JWT.

## Resources

https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.md

http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/

https://evertpot.com/jwt-is-a-bad-default/

https://github.com/vercel/next.js/blob/canary/examples/with-passport

https://nextjs.org/docs/authentication

https://next-auth.js.org/

https://stackoverflow.com/questions/49920234/how-to-implement-authentication-in-next-js

https://jwt.io/

https://supertokens.io/: creator of Super Tokens contributed to Blitz.js auth implementation

https://www.npmjs.com/package/iron-session


<!--stackedit_data:
eyJoaXN0b3J5IjpbNDY4MTU2ODI1LDEzNzAzMDM4OTgsMjA2NT
YyMTk1OCwtOTM1MzY5NjY3LC00MzYwNzY4NjYsLTE2NzExMTQ3
NjYsOTA4ODU3MjY0LDkyMTYxMTYxNiw4NTcwNDA5MjksMTc3Nj
k5NjU4MSwtMzMyNDU1MzYzXX0=
-->
