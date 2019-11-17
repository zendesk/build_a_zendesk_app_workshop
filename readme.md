# How to build a Zendesk Support App

Hello! Well done on taking your first step to building your first Zendesk Suppport App. This exercise will take you through the steps to build your own version of Zendesk's Bookmark App.


## Getting Started

1. Install Gem [Zendesk Apps Tool](https://github.com/zendesk/zendesk_apps_tools#install-and-use-zat). This tool is used by external Zendesk developers to Create, Update, Validate and Serve their app in their local environment. For the purpose of this exercise, we will only be using ZAT to validate and serve our app in our Zendesk Support instance.

2. Let's take a moment to understand the current directory structure and purpose of each files.

    * **./requirements.json** - used to specify the external resource you need in Zendesk Support or any Zendesk Product that can host an app. For instance, you can create custom fields in a ticket sidebar and user sidebar. Resource: [App Requirements](https://developer.zendesk.com/apps/docs/developer-guide/apps_requirements#specifying-apps-requirements)

    * **./manifest.json** - contains an app's metadata. This is also where we declare App Locations and the source files that they point to.

      ![Support App Locations](https://camo.githubusercontent.com/624fd3658b0f6ad418620b74c50fc3bc16444939/68747470733a2f2f7a656e2d6d61726b6574696e672d646f63756d656e746174696f6e2e73332e616d617a6f6e6177732e636f6d2f646f63732f656e2f6170705f6c6f636174696f6e732e706e67)

      In the example below, support locations ticket sidebar's app and top_bar app point to source files `assets/sidebar.html` and `assets/topbar.html`, respectively. Resource: [Manifest Reference](https://developer.zendesk.com/apps/docs/developer-guide/manifest)

      ```
      (manifest.json)
        ...
        ...
        "location": {
          "support": {
            "ticket_sidebar": "assets/sidebar.html",
            "top_bar": "assets/topbar.html"
          }
        },
        ...
      ```

    * **./translations** - Irrelevant for us now, but this is where you put your translated strings when you want to have your app work in different languages.

    * **assets** -
      - `assets/logo-small.png`, `assets/logo.png`, `assets/logo.svg` - image assets to display your app in Zendesk Support.
      - `assets/sidebar.html` - app source for ticket_sidebar app
      - `assets/topbar.html` - app source for top_bar app

## Scoping the work

**So what exactly does your bookmark app do? Your bookmark app will allow agents to pin a ticket that they are current working on or would like to prioritise to a dashboard at a click of a button.**

This button will be located in a ticket sidebar (accessible in a ticket's view), where as the dashboard will be located in the top bar (accessible in any navigation view).

The work involved could just be as simple as moving data (pinned ticket) from one app to another. However, bear in mind that these apps are server side, therefore refreshing the page or a user's session can easily wipe out appended data.

Our solution to this would be the use of Custom User Field. We create a custom user field using **requirements.json**. And then, write our data into this field when the button is clicked. This provides us the ability to persist our data into a Zendesk Database infrastructure. We can then make our top bar app read from this field. Clicking the button a second time would remove current ticket from tickets collection, updating the dashboard list in top bar as well.

## Instructions


### Part One: Setting up Custom User Field and App Locations

1. Open **Manifest.json** and replace *name*, *author's name* and *author's email* with your information. Take note of the 2 app locations and their app sources.

2. Open **requirements.json** and take note of line 3 and 5 where it says `bookmark_app_custom_field`. If you are sharing a support instance with someone else who is doing the same exercise, make sure to use a unique identifier as each key has to be unique to one another.

3. Log into your Zendesk Support instance and upload this file as a private app. The "Upload Private App" button can be found at the top right corner of `https://<your-subdomain>.zendesk.com/agent/admin/apps/manage`. Compress the files (2 directories and 3 files) at the root directory and move the zip somewhere accessible for upload. Click Install and hard refresh.

4. A few additional steps for sanity check.
  * Checkout your user page `https://<your-subdomain>.zendesk.com/agent/users/<user_id>/assigned_tickets`, you should now have a custom user field at the bottom left corner. If so, that means that this field is also readable in the API endpoint `https://<your-subdomain>.zendesk.com/api/v2/users/<user_id>.json` _(psst.. useful hint for our top bar app..)_.
  * Next, checkout the top bar and a ticket sidebar UI. You should now notice a default app Icon appearing on the top bar. The app contains nothing at the moment as the source files are empty skeletons.

5. For here on out, you will only need to upload app with a newly compressed zip file when,
   - You start a new project with new manifest reference and app requirements,
   - You need to add or remove app locations, or,
   - You wish to upload your finalised work forever!

6. [Serve your app locally using ZAT](https://developer.zendesk.com/apps/docs/developer-guide/using_sdk#testing-an-app-locally). You will need to append `?zat=true` in the URL and load unsafe script.


### Part Two: Ticket Sidebar App - a tale of writing-into and reading-from Custom User Field

Part two is the trickiest and hardest of this exercise. If you can overcome this, you can overcome anything. This is because you will have to learn the methods that were made available to us by App Framework. And at the same time, understand how Javascript Promises work. This makes asynchronous code a whole lot more interesting. Make sure that Olaf is not around when you say _"Async awaaaaait?"_ unless you want to hear about how much he loves promises. He taught me everything to know about Promises. Note that Promises can be challenging to understand, especially when figuring out the sequence of execution. Do not hesitate to reach out and call for help at any time during this exercise.

Diving into the existing code in `sidebar.html`, you would see that we are using [Zendesk Garden CSS Components](https://garden.zendesk.com/css-components/) served to us via a [jsdeliver](https://www.jsdelivr.com/) CDN link.
```
https://cdn.jsdelivr.net/combine/npm/@zendeskgarden/css-bedrock,npm/@zendeskgarden/css-buttons
```

If you would like to exercise your creativity on the UI, look up for a CSS/HTML element you would like to add into ticket sidebar, find the name of the NPM and add to the back of the link. For example, if the NPM is called [`@zendeskgarden/css-forms`](https://www.jsdelivr.com/package/npm/@zendeskgarden/css-forms). You would add `,npm/@zendeskgarden/css-forms` to the back of the CDN link above. Then you can customise your HTML components to use CSS classes according to [this page](https://garden.zendesk.com/css-components/forms/text/).

ZAF_SDK defined on line 13 enables our app to interact with Zendesk App Framework. As a result, we can instantiate [ZAF Client API](https://developer.zendesk.com/apps/docs/core-api/client_api#zafclient-object) on line 15 to use [ZAF Client Object methods](https://developer.zendesk.com/apps/docs/core-api/client_api#client-object). A few of these components can also be found in `topbar.html`.

We have also created a button with class`.bookmark-btn` in `sidebar.html` and a dashboard with class `.ticket-list-dashboard` in `topbar.html`. These are DOM elements tied to javascript constants for your scripts to interact with.

1. **Get** the ticket object from the ticket sidebar and store it an object with ticket id as key and ticket subject as value. it should look something like this,

    ```
      currentTicket = {
        ticketId: ticketSubject
      }
    ```

2. The next big piece is discovering how to read existing Custom User Field value. Steps include, needing to **Request** for a specific API end point for a specific User. Checkout Part 1.4. When you have achieved this, you will need to convert JSON strings to a Javascript Object as below.

    ```
    Converting this
      "{"22":"Test Subject", "75":"OLAF OLAF OLAF"}"

    To this
      {
        '22': 'Test Subject',
        '75':'OLAF OLAF OLAF'
      }
    ```

3. Now let's utilise the button and tie onclick function to it. On button click, add `currentTicket` to the stack and convert this back to Strings before posting the new tickets_collection to the same endpoint. Checkout the endpoint url with your browser to make sure that the details have been updated. This change should also reflect on the front end UI.

4. **Optional:** Switch the on click button to a toggle between adding and removing current_ticket, to and from tickets_collection. You can come back to this when you have additional time to spare.

5. Give yourselves a pat on the shoulder when you have reached this point because the hardest part is over.


### Part Three: Top Bar App - a tale of triggered COPY CATS!

Apps in different locations are designed to behave differently. Some app locations are initialised as soon as you log in, such as ticket_sidebar apps and background apps, where as some requires nudging, such as topbar app. To nudge our topbar app, we will need to [preload pane](https://developer.zendesk.com/apps/docs/support-api/nav_bar#preloadpane) from ticket sidebar. We are not quite done with `sidebar.html` yet.

1. Ideally we want to preload pane as soon as ticket_sidebar becomes available. **Preloading Pane** involves a 2 steps process. First, obtain the top_bar instance, and then call `.invoke('preloadPane')` on the topbar instance. This step can be challenging, see if you can draw inspiration from the example [here](https://developer.zendesk.com/apps/docs/developer-guide/using_sdk#messaging-between-locations). As usual, please do not hesitate to ask for help.

2. Next, try to [**trigger** a method](https://developer.zendesk.com/apps/docs/core-api/client_api#client.triggername-data) from `sidebar.html`'s button. Call methods `.trigger` on top_bar's instance in `sidebar.html`. A `.on` listener should present in `topbar.html`. Start off small with a console log to confirm that this is working, before thinking about what the listener method should do.

3. What exactly should this method do? Read the same Custom User Field from the API endpoint. This is where you copy existing functionalitiessss from `sidebar.html`. Append tickets_collection to dashboard using `.innerHTML = tickets_collection` on Button Click. The dashboard should only display the ticket subjects from tickets_collection.

4. It is important to think about when else besides on button click should we read from Custom User Field and implement that.

5. Last but not least, make each ticket subject [**route to**](https://developer.zendesk.com/apps/docs/support-api/all_locations#routeto) the ticket directly.


## Closing thoughts

Congratulation! You have built your first Zendesk Support App. Hopefully you were able to learn alot from this exercise and perhaps think about what app to build that would be helpful to your current team.

It is important to think about some of the limitations that are present in our app and what we can do to improve them. For instance, we can have split our files into 3 components, namely templates, css style sheets and javascript. Or even, writing modern and dry code using Webpack. How about building this in React? What are some existing tools that are available out there to make the lives of our external developers easier? Ask Fabio.

Please reach out to the apps team if you have more questions. Take care and good bye now! :)
