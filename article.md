# How to integrate OAuth2 into your Django/DRF backend without going insane

We've all been there: you've been working on the API backend, and you're happy with how it's going. You've recently completed the minimal viable product, the tests are all passing, and you're looking forward to implementing some new features. Then the boss sends you an email: "By the way, we need to let people log in via Facebook and Google; they shouldn't have to create an account just for a little site like ours."

```shell
$ pip search oauth | wc -l
278
$ pip search oauth | grep -i django | wc -l
53
```

The bad news is that `pip` knows about 278 packages which deal with oauth, 53 of which specifically mention django. It's a week's work just to research the options in any depth, let alone start writing code. It might happen that you're not familiar with OAuth2 at all; I wasn't, when this situation first happened to me. So what are you supposed to do?

The good news is that OAuth2 has emerged as the industry standard for social and third-party authentication, so you can focus on understanding and implementing that standard. Let's get going, then.

## A quick overview of the OAuth2 flow

OAuth2 was designed from the beginning as a web authentication protocol. This is not quite the same as if it had been designed as a _net_ authentication protocol; it assumes that tools like HTML rendering and browser redirects are available to you. This is obviously something of a hindrance for a JSON-based API, but we can work around that. First, let's go through the process as if we were writing a traditional, server-side website.

### The server-side OAuth2 flow

The first step in the process happens outside the application flow entirely: the project owner has to register your application with each OAuth2 provider with whom logins should be supported. During this registration, they provide the OAuth2 provider with a **callback URI**, at which your application will be available to receive requests. In exchange, they receive a **client key** and **client secret**. These tokens are exchanged during the authentication process to validate the login requests.

> The below image currently hotlinks to an image I found on imgur, but for the article proper we should probably create our own.

![](https://i.stack.imgur.com/SCJZO.png)

The flow begins when your application generates a page which includes a button like "Log in with Facebook" or "Sign in with Google+". Fundamentally, these are nothing but simple links, which points to a URL like `https://oauth2provider.com/auth?response_type=code&client_id=CLIENT_KEY&redirect_uri=CALLBACK_URI&scope=profile&scope=email`.

Let's unpack that a little. You've provided your client key and redirect URI, but no secrets. In exchange, you've told the server that you'd like an authentication code in response, and access to both the 'profile' and 'email' scopes. These scopes define the permissions you request from the user, and limit the extent of the provider's API for which the access token you will receive will be valid.

Upon receipt, the OAuth2 provider verifies that the callback URI and client key match each other before proceeding. If they do, the flow diverges briefly depending on the user's session tokens; if the user isn't currently logged in to that service, they'll be prompted to do so first. Once they're logged in, the user is presented with a dialog requesting permission to allow your application to log in.

> We definitely need to re-create this image, but it's pretty much exactly what we want. To match our example here, it should say "The app **Example** by Toptal Blog would like the ability to access your basic profile information and email address."

![](https://aaronparecki.com/2012/07/29/2/oauth-authorization-prompt.png)

Assuming the user approves, the OAUth2 server then redirects them back to the callback URI that you provided, including **authorization code** in the query parameters: `GET https://api.yourapp.com/oauth2/callback/?code=AUTH_CODE`. This is a fast-expiring, single-use token; immediately upon its receipt, your server should turn around and make another request to the OAuth2 provider, including both the auth code and your client secret: `POST https://oauth2provider.com/token/?grant_type=authorization_code&code=AUTH_CODE&redirect_uri=CALLBACK_URI&client_id=CLIENT_KEY&client_secret=CLIENT_SECRET`.

This request provides both your client key and your client secret, alongside the authorization code, and your callback URI. If everything matches up, the server returns an **access token**, with which you can make calls to that provider while authenticated as the user. As the request is generated directly from your server to the OAuth2 server over a TLS-secured connection, your client secret is never revealed to the client or to any attacker.

Once you've received the access token from the server, your server then redirects the user's browser once more to the landing page for just-logged-in users. It's common to retain the access token in the user's session, so that the server can make calls to the given social provider whenever necessary.

There are more details which we could go into: for example, Google will include a **refresh token** with which you can extend the life of your access token, while Facebook simply provides an endpoint at which you can exchange the short-lived access token they give by default for something longer-lived; these details don't matter to us, though, because we're not going to use this flow.

This flow is cumbersome for a REST API: while you could have the frontend client generate the initial login page, have the backend provide a callback URL, you'll eventually run into a problem: you want to redirect the user to the frontend's landing page once you've received the access token, and there's no clear, RESTful way to do so. Luckily, there's another OAuth2 flow available, which works much better in this case.

### The client-side OAuth2 flow

## Here's a recipe

## That looks like magic. How does it work?

## Why not just roll my own? OAuth2 isn't _that_ hard.