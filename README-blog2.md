# Integrating Slack with the swagger NPM module

[Slack](https://slack.com/) is a messaging app for team communication. A nice thing about Slack is that you can easily [integrate external services](https://slack.com/integrations) to provide extra features. 

The [swagger](https://www.npmjs.com/package/swagger) NPM module provides tools for designing and building Swagger-compliant APIs entirely in Node.js. 

In this blog, we'll show how easy it is to integrate a `swagger` API with Slack. 

## About the `swagger` API

For this blog, we built a `swagger` API implementation that provides the back-end feature that we'll integrate into Slack. The API fetches a stock quote and posts it directly into a Slack team conversation. Yes, it's amazing!

To use this API in Slack, we create what Slack calls an "Incoming WebHook" integration. This type of Slack integration lets you post data from an external source/service into Slack. 

We'll call the API it like this...

`curl -X POST -H "Content-Type: application/x-www-form-urlencoded" http://localhost:10010/ticker -d "text=AAPL`

...and get back a nicely formatted response in Slack, like this:

![alt text](./images/stockbot.png)

### Before you begin

If you're going to try to do this tutorial, you either must be a member of or create a new Slack team. Go to [slack.com](slack.com) for details. In either case, you need to have permission to create integrations.

## Get the sample swagger-node app from GitHub

To make things extra-simple, we've written the backend API using the `swagger` NPM module.

1. Download or clone the [swagger-node-slack](https://github.com/apigee-127/swagger-node-slack) project on GitHub. 
2. cd to the root project directory `swagger-node-slack`. 
3. Execute this command to get the Node.js dependencies:

    `npm install`





## Building the Ticker-bot

The Ticker-bot is an API implemented in `swagger-node` and added to Slack as an Incoming WebHook integration. This type of integration is designed to fetch some external data and display it in Slack. In this case, we implemented a `swagger-node` API that takes a stock symbol and sends the stock price to Slack. 

Let's walk through the steps for integrating the "text reverser" API with Slack. We're not going to go overboard to explain how to set things up in Slack, but we'll give pointers to keep you on track. It's remarkably easy. 

### Quick peek under the hood

Take a look at the `swagger-node-slack` project. If you're not familiar with `swagger-node`, you can check out [the docs](https://github.com/swagger-api/swagger-node/blob/master/docs/introduction.md), and try the quick-start tutorial if you like. 

The key to understanding how the `swagger-node-slack` API works is to look at these two files:

* `./swagger-node-slack/api/swagger/swagger.yaml` -- This is the Swagger definition for the API. Note that it defines the paths, operations, and parameters for the API. Note that the `/reverse` path is associated with a controller file called `reverse` and an operation (a function in the controller file), also called `reverse`. 

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
    ...
    ```

* `./swagger-node-slack/api/controllers/ticker.js` -- This is a controller file. It implements the logic that is executed for a specific API path (or route). In the `swagger.yaml` file, the `x-swagger-router-controller` attribute specifies the name of the controller file (the `.js` is not needed). The `operationId` specifies the name of the function to call when the `/ticker` path is requested.

Here's the controller code. The value of the URL variable comes from Slack, and we'll show you how to get it next.  

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

2. Scroll down to **DYI Integrations & Customizations** and click **Incoming WebHooks**. 

3. In **Post to Channel**, select the channel to post the API response to. In other words, whenever someone calls the Ticker-bot, a stock price is posted to this channel for everyone to see. 

4. Click **Add Incoming WebHooks Integration**.

4. Review the setup instructions. 

5. Copy the Webhook URL. **Hint: This is the URL you need to add to the controller file.**

6. In the name field, change the default name to "Ticker-bot".

### Edit the controller

Now, we'll add the WebHook URL to the `swagger` controller.

1. Open the file `swagger-node-slack/api/controllers/ticker.js` in a text editor.

2. Locate this variable and uncomment it:

    `var URL = "https://hooks.slack.com/services/https://hooks.slack.com/services/GET/SLACK URL";`

3. Replace the value of the URL variable with the Webhook URL. For example:

    `var URL = "https://hooks.slack.com/services/https://hooks.slack.com/services/X012434/BT3899/PSbPEfQybmoyqXM10ckdQoa";`

9. Save the file.


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

...and you get back a nicely formatted response your Slack, like this:

![alt text](./images/stockbot.png)

### What happened?

The Slack Slash Command Integration called the `swagger-node-slack` API, which posted a response directly to Slack. Slack retrieved the response and printed it to the chat window. 

### What next?

We've seen how easy it is to create a Slack "WebHooks Integration command" integration with a `swagger` backend API. 

Another cool Slack integration is the "Slash Command". If you like, jump over to the [swagger-node-slack](https://github.com/apigee-127/swagger-node-slack) project on GitHub to see how to make a Slash Command integration that reverses whatever text you enter.

So, you can do this in Slack...

`/reverse The quick brown fox jumps over the lazy dog`

![alt text](./images/quickfox-1.png)

... and Slack returns the letters in reverse:

![alt text](./images/quickfox-2.png)









## Extra credit

For extra credit, try creating a Slack Slash command `/ticker` that calls the Ticker-bot API. The command will let users enter something like this in Slack to post a stock price:

`/ticker AAPL`

Be sure to add token validation to the API, as implemented in the `/reverse` example.













