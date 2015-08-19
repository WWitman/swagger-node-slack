# Integrate your Slack with Swagger!

[Slack](https://slack.com/) is a messaging app for team communication. A nice thing about Slack is that you can easily [integrate external services](https://slack.com/integrations) to provide extra features. For example, out-of-the-box integrations are available for services like GitHub, Google Drive, Heroku, Jira, and many others.

The [swagger](https://www.npmjs.com/package/swagger) NPM module provides tools for designing and building Swagger-compliant APIs entirely in Node.js. You can build and test `swagger` projects locally and deploy them to any Cloud platform that supports Node.js.

![alt text](./images/swagger-flow-2.png)

In this blog, we'll show how easy it is to integrate a `swagger` API into Slack. 

## About the `swagger` API

Our `swagger` API fetches a stock quote and posts it directly into a Slack team conversation. Yes, it's amazing!

We'll create what Slack calls an "Incoming WebHook" integration. This type of Slack integration lets you post data from an external source/service into Slack. 

We'll call the back-end API using cURL, like this...

`curl -X POST -H "Content-Type: application/x-www-form-urlencoded" http://localhost:10010/ticker -d "text=AAPL`

...and get back a nicely formatted response in Slack, like this:

![alt text](./images/stockbot.png)

We'll run and test the API locally; however, it can be deployed to any Cloud platform that supports Node.js, such as Apigee, AWS, or Heroku. More on deployment later.

## Before you begin

If you're going to try to do the steps outlined below, you must be a member of a Slack team or create a new one. Go to [slack.com](http://slack.com/) for details. In either case, you need to have permission to create integrations.

## Get the sample swagger-node-slack app from GitHub

To make things extra-simple, we've written the back-end API ahead of time.

1. Download or clone the [swagger-node-slack](https://github.com/apigee-127/swagger-node-slack) project on GitHub. 

2. cd to the root project directory `swagger-node-slack`. 

3. Execute this command to get the Node.js dependencies:

    `npm install`

That's it. We're ready to integrate!

## Building the Ticker-bot integration

Let's walk through the steps for integrating the `/ticker` API with Slack. We're not going to go overboard to explain how to set things up in Slack, but we'll give pointers to keep you on track. It's remarkably easy. 

### But first, a quick peek under the hood

Let's take a quick look at the `swagger-node-slack` project. 

>If you're not familiar with the `swagger` NPM module project structure, you can check out [the docs](https://github.com/swagger-api/swagger-node/blob/master/docs/introduction.md), and try the quick-start tutorial if you like. 

The key to understanding how the `swagger-node-slack` API works is to look at these two files:

* `./swagger-node-slack/api/swagger/swagger.yaml` -- This is the Swagger definition for the API. Note that it defines the paths, operations, and parameters for the API. These entities tie directly to corresponding controller files, described next.

    ```
    ...
    paths: 
        /ticker:
        # binds app logic to a route
        x-swagger-router-controller: ticker
        post:
          description: look up a stock price
          # used as the method name of the controller
          operationId: ticker
          consumes:
            + application/x-www-form-urlencoded
          parameters:
            + $ref: "#/parameters/text"
            + $ref: "#/parameters/user_name"
            + $ref: "#/parameters/icon_url"
            + $ref: "#/parameters/icon_emoji"
            + $ref: "#/parameters/channel"
          responses:
            "200":
              description: Success
            # responses may fall through to errors
            default:
              description: Error
              schema:
                $ref: "#/definitions/ErrorResponse"
    ...
    ```

* `./swagger-node-slack/api/controllers/ticker.js` -- This is a controller file. It implements the logic that is executed for a specific API path (or route). In the `swagger.yaml` file, the `x-swagger-router-controller` attribute specifies the name of the controller file (the `.js` is not needed). The `operationId` specifies the name of the function to call when the `/ticker` path is requested. So, for this API, when you call the `/ticker` API, it executes a function called `ticker()` in a controller file called `ticker.js`.

Here's the controller code. The value of the `URL` variable comes from Slack. We'll show you how to get it next.  

   ```
   var util = require('util');
   var request = require('request');
   var googleStocks = require('google-stocks');

   module.exports = {
      ticker: ticker
};
   //var URL = "https://hooks.slack.com/services/GET/FROM/SLACK";

   function ticker(req, res) {
        var symbol = req.swagger.params.text.value.toUpperCase();
        googleStocks.get([symbol], function(error, data) {
            console.log(symbol+": "+data[0].l);
            var text = symbol+" is now $"+data[0].l+" per share.\nThanks for asking, @"+req.swagger.params.user_name.value+"!";
            request.post({ url:URL, body:{"text":text}, json:true});
            res.status(200).type('application/json').end();
        });
   }
   ```


### Create the Slack integration

Let's go over to the Slack side now.

1. Log in to your Slack account. 

1. From your Slack team menu, choose **Configure Integrations**.

![alt text](./images/slack-menu.png)

2. Scroll down to **DYI Integrations & Customizations** and click **Incoming WebHooks**. 

3. In **Post to Channel**, select the channel to post the API response to. In other words, whenever someone calls the Ticker-bot, a stock price is posted to this channel for everyone to see. 

4. Click **Add Incoming WebHooks Integration**.

4. Review the setup instructions. 

5. Copy the WebHook URL. **Hint: This is the URL you need to add to the controller file.**

6. In the name field, change the default name to "Ticker-bot".

### Edit the controller

Now, we'll add the WebHook URL to the `swagger` controller.

1. Open the file `swagger-node-slack/api/controllers/ticker.js` in a text editor.

2. Locate this variable and uncomment it:

    `var URL = "https://hooks.slack.com/services/https://hooks.slack.com/services/GET/SLACK URL";`

3. Replace the value of the URL variable with the Webhook URL. For example:

    `var URL = "https://hooks.slack.com/services/https://hooks.slack.com/services/X012434/BT3899/PSbPEfQybmoyqXM10ckdQoa";`

4. Save the file.


### Try it!

A nice thing about `swagger` projects is you can build and test them locally. Let's try out our Ticker-bot!

Remember, with an Incoming WebHooks integration, the idea is to send a message FROM another service INTO a slack channel. 

1. cd to the `swagger-node-slack` directory on your system.
2. If you haven't done so previously, execute this command to update the Node.js dependencies: 

    `npm install`

3. Start the project:

    `swagger project start`

4. Call the API, like this...

`curl -X POST -H "Content-Type: application/x-www-form-urlencoded" http://localhost:10010/ticker -d "text=AAPL&user_name=marsh"`

...and you get back a nicely formatted response in your Slack session, like this:

![alt text](./images/stockbot.png)

### What happened?

The API posted data to Slack via a Slack WebHook Integration. Slack retrieved the response and printed it to the chat window. 

### What next?

We've seen how easy it is to create a Slack "WebHooks Integration command" integration with a `swagger` back-end API. 

Another cool Slack integration is the "Slash Command". If you like, jump over to the [swagger-node-slack](https://github.com/apigee-127/swagger-node-slack) project on GitHub to see how to make a Slash Command integration that reverses whatever text you enter.

So, you can do this in Slack...

`/reverse The quick brown fox jumps over the lazy dog`

![alt text](./images/quickfox-1.png)

... and Slack returns the letters in reverse:

![alt text](./images/quickfox-2.png)

