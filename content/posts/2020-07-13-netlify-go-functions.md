---
mathjax: true
title:  A Step-By-Step Guide to Authenticating Netlify Go Functions Using Netlify Identity
date:   2020-07-13 08:30:00 -0500
authors: ["mkrump"]
categories: ["go", "golang", "netlify", "authentication"]
---
I’ve hosted this blog on Netlify for a couple years now, and have found it to be a really fantastic service. It's especially ideal for something like a blog, since it’s likely to be [free](https://www.netlify.com/pricing/), doesn’t require managing any infrastructure, and deploys are done via a simple git push. But since I first started using Netlify, they’ve dramatically expanded their offerings so that it’s now possible to build a lot more than just static sites.  

I recently had an idea for a [small language learning app](https://contemos.app/), and I was curious to see how difficult it would be to get something slightly more complicated than a blog deployed on Netlify. My plan was to write the backend in Go, since it along with JavaScript, is one of the two languages supported by [Netlify Functions](https://www.netlify.com/products/functions/), which are essentially AWS Lambda functions, but with all of the setup and deployment handled by Netlify. Also, I decided to add a login page, because why not?, and conveniently Netlify has its own [identity management service](https://docs.netlify.com/visitor-access/identity/).

Overall, it was a very smooth process, and I was able to get my app deployed with minimal hassle. The various services all worked well together, and Netlify's docs are really good, however there were a couple spots where I did get a little stuck even after reading the relevant docs. At least some of the issues that I ran into, are probably specific to the Go side of things, since at the moment it seems slightly less well supported than JavaScript as a backend. As an example, if you want to test your deployed setup locally you can use [Netlify Dev](https://www.netlify.com/products/dev/) which looks great, however at the moment it appears that it [doesn’t support Go](https://community.netlify.com/t/working-with-go-functions-locally-and-in-deployment/3530/7). 

Since I had some questions along the way, to fill in gaps in my own understanding, and serve as documentation to my future self, I put together an  example site, demonstrating how to secure a Go Netlify Function using Netlify's Identity service. The repo can be found at [mkrump/go-netlify-login-example](https://github.com/mkrump/go-netlify-login-example) and the live site [here](https://go-netlify-login-example.netlify.app/). Below is a step-by-step guide for deploying this repo with Netlify.

## Initial setup
Sign up for a Netlify account if you don’t have one already and install the [cli tool](https://docs.netlify.com/cli/get-started/) with `npm install netlify-cli -g`.

Then open up a terminal and run:

```bash
# clone and fork the repo
hub clone mkrump/go-netlify-login-example
cd go-netlify-login-example
hub fork 

# netilfy init creates and deploys netlify site after answering questions
netlify init
# Answers:
# What would you like to do? -> Create & configure a new site
# Site name (optional): -> example-site-123456
# Your build command (hugo build/yarn run build/etc): -> PRESS ENTER
# Directory to deploy (blank for current dir): -> PRESS ENTER

# netlify open logs into to admin page for our new netlify site
netlify open
```

We aren't going to focus too much on the front-end, since I found plenty of examples and lots of good documentation here. The example app is based on,  [`example/react`](https://github.com/netlify/netlify-identity-widget/tree/master/example/react), which uses the [Netlify identity widget](https://github.com/netlify/netlify-identity-widget). The identity widget is a nice place to start because out of the box it handles: signups, logins, and password recovery. However, if more customization is needed, there is a list of alternatives at the bottom of the netlify-identity-widget GitHub page. 

If we startup the dev server with `npm install && npm start` we’ll see a link for a "Protected Page". However, if we try to visit the "Protected Page" we'll see a message “Must be logged in to access this route". And when we try to login we’ll see the following message: "Looks like you’re running a local server. Please let us know the URL of your Netlify site."

Before we can login in to either local dev or the live site, we’ll need to enable Identity for our site. To do this run `netlify open` (or open up the Netlify UI) then click the "Identity" tab followed by "Enable Identity". Now that we have our identity service enabled, head back to the "Overview" tab and copy the full url of our site and paste it into the dialog that we were seeing on local dev. 

Now that our identity service is setup, we should be able to sign up and login to our site and once logged in we'll now be able to visit the "Protected Page". 

## Deploying the Go Function 
Our Go function will have automatically been deployed with our initial site deploy and we should be able to see it when we visit the "Functions" tab in the Netlify UI.

Netlify assumes our functions are contained in a top-level `functions` directory; however, this can be overridden via the UI or in the `netlify.toml` file. For each deploy, the designated directory will be referenced, and each supported code file will be zipped and deployed as a Lambda function on AWS. 

By default Netlify will compile the Go binaries for us; however, you can also provide the compiled binaries if you don't want to have Netlify do the compilation step for you. In the example repo, there is a small `Makefile` that handles the compilation step. Also, the `netlify.toml` file has been updated to let Netlify know to use the `Makefile` in the build step. 

## Authenticate Go Backend Using Netlify Identity 
If we open the Netlify console and click the "Functions" tab, we should see our deployed function, and if we click the function itself (`example.js` in this case), we’ll see the logs along with the url for our endpoint. By default this endpoint will be public and anyone can access it, so say instead we only want to allow authenticated users to access our endpoint. We can do this pretty easily by leveraging Netlify Identity. The [relevant section](https://docs.netlify.com/functions/functions-and-identity/#access-identity-info-via-clientcontext) of the docs summarize how this process works:

> The user object is present if the function request has an Authorization: Bearer <token> header with a valid JWT from the Identity instance. In this case the object will contain the decoded claims.

Our frontend requests already attach the jwt on each request:

```javascript
export function generateHeaders() {
  const headers = { "Content-Type": "application/json" };
  if (netlifyIdentity.currentUser()) {
    return netlifyIdentity
      .currentUser()
      .jwt()
      .then((token) => {
        return { ...headers, Authorization: `Bearer ${token}` };
      });
  }
  return Promise.resolve(headers);
}
```

So the question is how do we verify if a request is authorized on the backend. The relevant [Go docs](https://docs.netlify.com/functions/build-with-go/#access-the-clientcontext) show the example code below:

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

However, based on the example code, it wasn’t clear to me how we should be checking that a user’s claim is valid. When I logged the `LambdaContext` associated with every request, a valid `LambdaContext` was always present regardless of the validity of the user's claim. However, if we look at the `LambdaContext` object returned from `lambdacontext.FromContext` we can see it has a `Custom` map attached to `ClientContext` and if we log this map we can see that this is where the response from the identity service gets attached. Specifically, when a claim is valid the decoded "netlify" entry in the `Custom` map will contain a `User` object, while the `User` object will not be present when the claim is not valid. We can verify this by checking our logs after making a request to our deployed site. After each request, in the logs we'll see a `Custom` map with a "netlify" key.  

So our handler ends up looking like below ([full code](https://github.com/mkrump/go-netlify-login-example/blob/master/api/main.go#L45)):  

```golang
func handler(ctx context.Context, request events.APIGatewayProxyRequest) (*events.APIGatewayProxyResponse, error) {
  lc, ok := lambdacontext.FromContext(ctx)
  if !ok {
    return &events.APIGatewayProxyResponse{
      StatusCode: 500,
      Body:       "server error",
    }, nil
  }
  log.Printf("lc.ClientContext.Custom: %+v\n", lc.ClientContext.Custom)
  identityResponse := lc.ClientContext.Custom["netlify"]
  raw, _ := base64.StdEncoding.DecodeString(identityResponse)
  data := IdentityResponse{}
  _ = json.Unmarshal(raw, &data)
  if data.User == nil {
    r := &Response{
      Msg: fmt.Sprintf("Your claim isn't valid. Try logging in and resubmitting your request"),
      IdentityResponse: identityResponse,
    }
    resp, _ := json.Marshal(r)
    return &events.APIGatewayProxyResponse{
      StatusCode: 403,
      Body:       string(resp),
    }, nil
  }
  r := &Response{
    Msg:              fmt.Sprintf("Hi %s your is claim is valid", data.User.UserMetadata.FullName),
    IdentityResponse: identityResponse,
  }
  resp, _ := json.Marshal(r)
  return &events.APIGatewayProxyResponse{
    StatusCode: 200,
    Body:       string(resp),
  }, nil
}
```

The process of checking these claims could pretty easily be extracted into a middleware.

## Verifying claims

So the full process ends up being: frontend attaches jwt to request -> Netlify's identity service verifies the jwt -> backend checks that the `User` object is present. We can verify this process by creating a new user and logging into the [example app](https://go-netlify-login-example.netlify.app/).

Once logged in, visit the "Protected Page" and we'll see two claims, each with its own submit button. The first is the current logged in user's jwt, which would be submitted along with all real requests. The second claim is a fake jwt, which can be edited. If we open the network tab in our dev tools and submit a request, we'll see the jwt being sent in the request header. In the case of the valid jwt, our app will render the decoded Netlify Identity response containing the logged in user. However, when we submit the fake jwt, the rendered response will not have a user object attached. We can experiment a little further, by copying our actual jwt into [jwt.io](https://jwt.io/) and then slightly altering it (i.e. change the email) and then submitting the altered token. Again, the user object won't be attached, and the associated request would be rejected.

## Yay Netlify!

All in all, it's pretty amazing how quickly you can get a reasonably complicated project up and running on Netlify all without needing to manage any infrastructure. 

The deployed version of the example app can be found [here](https://go-netlify-login-example.netlify.app/) and its associated GitHub repo [here](https://github.com/mkrump/go-netlify-login-example).
