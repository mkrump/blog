---
mathjax: true
title:  A Step-By-Step Guide to Authenticating Netlify Go Functions Using Netlify Identity
date:   2020-07-13 08:30:00 -0500
authors: ["mkrump"]
draft: true
categories: ["go", "golang", "netlify", "authentication"]
---
I’ve hosted this blog on Netlify for a couple years now, and found it to be a really fantastic service. It's especially ideal for something like a blog, since it’s likely to be [free](https://www.netlify.com/pricing/), doesn’t require managing any infrastructure, and deploys are done via a simple git push. But since I first started using Netlify, they’ve dramatically expanded their offerings so that it’s now possible to build a lot more than just static sites.  

I recently had an idea for a [small language learning app](https://contemos.app/), and I was curious to see how difficult it would be to get something slightly more complicated than a blog deployed on Netlify. My plan was to write the backend in Go, which along with JavaScript, is one of the backend languages supported by [Netlify Functions](https://www.netlify.com/products/functions/). These are essentially AWS Lambda functions, but with all of the setup and deployment handled by Netlify. Also, I decided to add a login page, since I saw that Netlify has its own identity management service as well (via Netlify Identity, which also has a free tier).

Overall, everything worked very smoothly, there were only a few places that I was a little unsure how to proceed after reading the relevant docs. I think the issues that I ran into, were probably specific to the Go side of things, since at the moment it seems slightly less well supported than JavaScript as a backend. For example, [Netlify Dev](https://www.netlify.com/products/dev/) is a tool that mimics the Netlify environment in local development, but it appears that it [doesn’t support Go at the moment](https://community.netlify.com/t/working-with-go-functions-locally-and-in-deployment/3530/7). 

Since I had some questions along the way, to prove to myself how everything ties together, I put together an example site with a Go API that uses Netlify’s Identity service for authentication. The full repo can be found at [mkrump/go-netlify-login-example](https://github.com/mkrump/go-netlify-login-example) and the live site [here](https://go-netlify-login-example.netlify.app/). Below is a step-by-step guide for deploying this repo on Netlify.

## Initial setup
Sign up for a Netlify account if you don’t have one already and install the cli tool `npm install netlify-cli -g`.

```bash
# clone and fork the repo
hub clone mkrump/go-netlify-login-example
cd go-netlify-login-example
hub fork 

# builds netlify site after answering questions
netlify init
# Answers:
# What would you like to do? -> Create & configure a new site
# Site name (optional): -> example-site-123456
# Your build command (hugo build/yarn run build/etc): -> accept default
# Directory to deploy (blank for current dir): -> accept default

# logs into to admin page netlify site
netlify open
```

The front-end is based on the example found at [`example/react`](https://github.com/netlify/netlify-identity-widget/tree/master/example/react), which uses the [Netlify identity widget](https://github.com/netlify/netlify-identity-widget). Out of the box, the identity widget handles: signups, logins, and password recovery. However, if more customization is needed, there is a list of alternatives on the netlify-identity-widget GitHub page. 

If we startup the dev server with `npm install && npm start` we’ll see a link for a "Protected Page". If we try to visit the "Protected Page" we'll see a message “Must be logged in to access this route"

And when we try to login we’ll see the following message:

> Looks like you’re running a local server. Please let us know the URL of your Netlify site.

First, we’ll need to enable Identity for our Netlify site. `netlify open` (or open up the Netlify UI) -> click the "Identity" tab -> "Enable Identity". Now copy the url of our site, which can be found on the "Overview" tab, and paste it into the dialog. 

We should now be able to sign up, login and once logged in we'll now be able to visit the "Protected Page". 

## Authenticate Go Backend Using Netlify Identity 
By default Netlify assumes that the `functions` directory contains the various Netlify functions that will be used in an app.  So on each deploy, this directory will be referenced, and each supported code file will be zipped and deployed as a Lambda function on AWS. 

In our example repo, I’ve used the default `functions` directory, but this directory can be changed either via the Netlify web interface or the `netlify.toml` file. Also for Go functions, you can provide the compiled binaries if you don't want to have Netlify do it for you. In the example repo, there is a small `Makefile` that handles this step, and the `netlify.toml` has been updated with our build command. 

If we open the Netlify console and click the "Functions" tab, we should see the deployed function, and if we click the function itself (`example.js` in this case), we’ll see the logs along with the url for our endpoint. By default this endpoint will be public, so say instead we only want to allow authenticated users to access our endpoint. We can do this pretty easily by leveraging Netlify Identity. The [docs](https://docs.netlify.com/functions/functions-and-identity/#access-identity-info-via-clientcontext) explain:

> The user object is present if the function request has an Authorization: Bearer <token> header with a valid JWT from the Identity instance. In this case the object will contain the decoded claims.

The associated [Go docs](https://docs.netlify.com/functions/build-with-go/#access-the-clientcontext) show the example code below:

```golang
func handler(ctx context.Context, request events.APIGatewayProxyRequest) (*events.APIGatewayProxyResponse, error) {
  lc, ok := lambdacontext.FromContext(ctx)
  if !ok {
    return &events.APIGatewayProxyResponse{
      StatusCode: 503,
      Body:       "Something went wrong :(",
    }, nil
  }

  cc := lc.ClientContext

  return &events.APIGatewayProxyResponse{
    StatusCode: 200,
    Body:       "Hello, " + cc.Client.AppTitle,
  }, nil
}
```

From the example code, it wasn’t clear to me how we should be checking that a user’s claim is valid. When I logged the data associated with requests a valid `lambdacontext` was always present regardless of the validity of the user's claim. But if we log the context object returned from `lambdacontext.FromContext` we see a `Custom` context map and this is where the response from the identity service is attached. Specifically, when a claim is valid the decoded "netlify" entry, will contain a `User` object, while the `User` object will not be present when the claim is not valid. Also, if we check the logs after making a request to our deployed site, we'll see the `Custom` context map in the log output.

## Verifying claims

So the process ends up of being: frontend attaches jwt to request -> Netlify's identity service verifies the jwt -> backend checks that user object is present. If we create a new user and login to the [example app](https://go-netlify-login-example.netlify.app/) we can verify the entire process. 

Once logged in, visit the "Protected Page" and we'll see two claims, each with its own submit button. The first is the current logged in user's jwt, which will be submitted along with all requests to our backend. The second claim is a fake jwt, which can be edited. If we open the network tab in our dev tools and submit a request, we'll see each jwt being sent in the request header. In the case of the valid jwt, we'll see the decoded Netlify Identity response containing the user, however when we submit the fake jwt, the response will not have a user object attached. We can experiment a little further, by copying our actual jwt into [jwt.io](https://jwt.io/) and then slightly altering it (i.e. change the email) and then trying to submit the altered token. Again, the user object won't be attached, so we can use this fact to gate our API.

## Yay Netlify!

All in all, it's pretty amazing how quickly you can get a reasonably complicated project up and running on Netlify without needing to manage any infrastructure. 

The deployed version of the example app can be found [here](https://go-netlify-login-example.netlify.app/) and its associated GitHub repo [here](https://github.com/mkrump/go-netlify-login-example).
