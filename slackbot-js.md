# Make your own Slack bot by using JS! 

## Introduction

Ever wanted to have your own Slack bot which responds to your commands, automates your tasks and other stuff in Hack Club?
In this guide, you will learn how to:

* Create a slack app
* Build a slack bot using javascript
* Adding slash commands such as `/ping`
* Deploy your bot in [Hack Club Nest](https://nest.hackclub.com) for free
* Keeping your bot online 24/7

This guide assumes no JavaScript experience. If you want a short intro to JavaScript first, read MDN's friendly overview: [MDN JavaScript Introduction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction)

This guide is very beginner friendly and uses the following:

* JavaScript (no prior coding required)
* Node.js
* Slack Bolt
* Socket Mode
* Nest @ Hack Club

At the end of this tutorial you will have your own fully hosted Slack bot.

---

## What you will need

Before you start, gather these things:

* A Hack Club Slack account
* A GitHub account (for publishing your code)
* A Hack Club Nest account (used for hosting) - see notes below about applying
* Node.js and npm installed locally

### Notes for beginners

* If you don't know JavaScript, the MDN intro above is a good quick primer.
* "Hack Club Nest" is a free place Hack Club provides to host projects. Search for "Hack Club Nest" or visit [Hack Club Dashboard](https://dashboard.hackclub.app) and follow the signup flow.
* Nest sometimes requires an application step, don't let the wait discourage you. If you already have HCA (Hack Club Auth account) set up, the process is usually quick :)

---

## What we will build today

In this tutorial, we will be creating a simple slack bot using js that responds to slash commands. A slash command is a command you type starting with a slash.

### Examples you'll try

```txt
/dsb-ping
/dsb-hello
/dsb-status
```

When a user runs one of these commands, the bot can do multiple stuff like show information, send a message, run automations, interact with API’s, run workflows and more.

For example: if you make a command such as `/dsb-joke` or other, you can make the bot fetch a joke from an api and send it in the channel.

---

## Why is there `/dsb-...` instead of plain `/ping`, etc?

Hack club slack workspace has many bots installed and if you end up keeping your bot’s commands generic like `/ping` it usually overwrites other bots commands, therefore you are required to use a random letters or shortname of your bot like `dsb`, the commands will be starting with `/dsb-` and then the actual command comes in.

---

## Creating the bot

1. Go to the Slack Apps dashboard: [Slack Apps Dashboard](https://api.slack.com/apps) and click **Create New App → From scratch**.
2. Give it a name and pick the Hack Club workspace and create the app.

### Enable Socket Mode (step-by-step)

* Open the **Socket Mode** page in your app left sidebar and toggle **Enable Socket Mode**.
* Socket Mode needs an *App-Level Token* with the `connections:write` scope. To create that token:

1. Open **Basic Information** in the left sidebar.
2. Scroll to **App-Level Tokens** and click **Generate Token and Scopes**.
3. Give the token a name (example: `my-bot-socket`) and add the `connections:write` scope.
4. Click **Generate** and copy the token immediately. App-level tokens start with `xapp-`.

### Important

Slack separates two token types:

* Bot User OAuth Token (starts with `xoxb-`): used by your bot to perform actions like sending messages.
* App-Level Token (starts with `xapp-`): used for Socket Mode and other app-level functionality.

### Set Bot scopes (OAuth & Permissions)

1. Open **OAuth & Permissions** in the left sidebar.
2. Under **Bot Token Scopes** add:

```txt
chat:write
commands
app_mentions:read
channels:history
```

You can add more scopes later according to what things you add to the bot.

These permissions allow your bot to:

* Send messages
* Use slash commands
* Read mentions
* Access channel messages

3. After adding scopes, go to **Install App** (left sidebar) and click **Install to Workspace**. Grant permissions.
4. On the same **OAuth & Permissions** page you'll see the **Bot User OAuth Token** (starts with `xoxb-`) - copy and save it. If you don't see it, revisit **Install App**.

### Where to find tokens if you miss them

* App-Level Token (xapp-): **Basic Information** → **App-Level Tokens** → click the token name to view or generate.
* Bot User OAuth Token (xoxb-): **OAuth & Permissions** → look for **Bot User OAuth Token** after installing the app.

**Treat tokens like passwords: do not share them or commit them to GitHub.**

---

## Slash commands

1. Open **Slash Commands** in the left sidebar and click **Create New Command**.
2. Enter a command name like `/dsb-ping` and provide a short description and usage hint.
3. Save the command. You will use this command name in your code.

### Tip

Keep a short note of the exact command name so you can update your code later (it's case-sensitive).

---

## Setting up the bot (project files and editor)

1. Create a new project folder and open it in your code editor (VS Code is recommended).

### Important

Open the folder in VS Code before running commands. In VS Code use **Terminal → New Terminal** so commands run in the correct folder.

### Initialize the project and install dependencies

```bash
npm init -y
npm install @slack/bolt dotenv
```

Create an `index.js` file in the project folder and paste this base code. Change the command name if you used a different slash command above.

### Note

Update the command name in this block to what slash command you set in the slack dashboard.

```js
require("dotenv").config();

const { App } = require("@slack/bolt");

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  appToken: process.env.SLACK_APP_TOKEN,
  socketMode: true
});

app.command("/dsb-ping", async ({ command, ack, respond }) => {
  const start = Date.now();
  await ack();
  const latency = Date.now() - start;
  await respond({ text: `Pong!\nLatency: ${latency}ms` });
});

(async () => {
  await app.start();
  console.log("bot is running!");
})();
```

Create a `.env` file in the same folder and add the tokens you copied from Slack:

```env
SLACK_BOT_TOKEN=xoxb-...   # Bot User OAuth Token (from OAuth & Permissions)
SLACK_APP_TOKEN=xapp-...   # App-Level Token (from Basic Information → App-Level Tokens)
```

Create a `.gitignore` with:

```gitignore
node_modules
.env
```

### Why put tokens in `.env`

Prevents accidental commits of secrets to GitHub. Keep `.env` private.

---

## Running the bot (locally)

Open a terminal in VS Code (Terminal → New Terminal) with your project folder as the current directory. Then run:

```bash
node index.js
```

If successful you'll see:

```txt
bot is running!
```

Then test your slash command in Slack (for example `/dsb-ping`). If nothing happens, check the app's logs and tokens.

![image](https://cdn.hackclub.com/019e4e05-aef2-7b7c-8064-8599e2b8b368/image.png)


### Common local mistakes

* Running the command in a terminal that isn't in your project folder - make sure the terminal's current directory contains `index.js`.
* Copying the wrong token (see token prefixes below).

---

## Understanding How Commands Work

### Basic command structure

```js
app.command("/command-name", async ({ ack, respond }) => {

});
```

### Explanation

| Part              | What it does                             |
| ----------------- | ---------------------------------------- |
| `app.command()`   | Registers a slash command                |
| `"/command-name"` | The command Slack listens for            |
| `async`           | Allows asynchronous operations like APIs |
| `ack()`           | Acknowledges the command to Slack        |
| `respond()`       | Sends a message back to Slack            |

:::callout type="warning"
`ack()` is required. If you do not acknowledge the command quickly enough, Slack will think the command failed.
:::

---

## Expanding your bot/adding more commands

Now that you got your bot up and running you can add more commands such as:

### Example - help command

```js
app.command("/dbt-help", async ({ ack, respond }) => {
  await ack();
  await respond({ text: `Available Commands:\n/dbt-ping - Check bot latency\n/dbt-catfact - Get a cat fact` });
});
```
Example of a help command:
![Example](https://cdn.hackclub.com/019e4e08-044a-7577-995f-e135b03ca9a3/image.png)

### API commands (cat facts, jokes)

You can fetch stuff via API’s such as cat fun facts, jokes, weather information, etc.

You can check out [Free APIs List](https://free-apis.github.io/#/browse) to find many API’s which are free, etc and use them in your bot.

### Install `axios`

```bash
npm install axios
```

Put the following at the top of the `index.js` file:

```js
const axios = require("axios");
```

Now put this in your `index.js` file to add this command:

```js
app.command("/dbt-catfact", async ({ ack, respond }) => {
  await ack();

  try {
    const response = await axios.get("https://catfact.ninja/fact");

    await respond({
      text: `Cat Fact:\n${response.data.fact}`
    });
  } catch (err) {
    await respond({
      text: "Failed to fetch a cat fact."
    });
  }
});
```
![Example](https://cdn.hackclub.com/019e4e0a-7a33-7c16-9c38-53e620ff89af/image.png)

---

### Another example (joke command)

```js
app.command("/dbt-joke", async ({ ack, respond }) => {
  await ack();

  try {
    const response = await axios.get(
      "https://official-joke-api.appspot.com/random_joke"
    );

    await respond({
      text:
`${response.data.setup}

${response.data.punchline}`
    });
  } catch (err) {
    await respond({
      text: "Failed to fetch a joke."
    });
  }
});
```

---

## How the Cat Fact & Joke Commands Work

Both the cat fact and joke commands work almost the exact same way.

### The basic flow

1. User runs a slash command
2. The bot receives the command
3. The bot sends a request to an API
4. The API returns data
5. The bot sends that data back into Slack

Both commands use:

* `axios` → to make API requests
* `try/catch` → to prevent crashes if the API fails
* `respond()` → to send messages back to Slack

Like this you can make how many commands as you want, make sure to add the commands via the dashboard!

:::callout type="info"
While using API’s make sure to remember APIs have rate limits and always use try/catch when making requests so your bot doesn’t crash if the API fails.

Also some APIs require authentication so put the apikeys in your `.env` file!
:::

---

## Push your code to Github

Run the following code (edit and run) to publish the code to github.

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

---

## Bot setup via Nest

Now once you are done with adding all the commands to your bot, you remember that you cannot keep your bot running 24/7.

Now comes [Nest](http://nest.hackclub.com), a free debian server provided to students by Hack Club.

Now you will set this up so you can have your bot running 24/7!

Make sure to have a nest account for this!

---

## Basic setup

Now SSH into the nest server with your credentials, if you do not know how to do this don’t worry; Refer to this [Quickstart Guide](https://guides.hackclub.app/index.php/Quickstart).

Once you are logged in, run the following one by one:

```bash
git clone https://github.com/<yourgithubusername>/<yourreponame>
cd <yourreponame>
npm install
```

Once that is done, create the `.env` file here by running:

```bash
nano .env
```

Add the same things here as the `.env` file in local.

### To save & exit

*
* Press Enter
*

Again run:

```bash
node index.js
```

and test out the bot.

If that is working, continue with the guide to make it 24/7.

---

## Make your bot run 24/7

### Creating a systemd service

Without systemd, your bot stops when:

* You disconnect SSH
* Nest restarts
* The process crashes

We’ll use systemd to keep it alive.

Run:

```bash
cd ~/.config/systemd/user
```

to change your directory.

And now create a new service by running:

```bash
nano slackbot.service
```

In that file add the following (edit the working directory):

```ini
[Unit]
Description=Slack Bot
DefaultDependencies=no
After=network-online.target

[Service]
Type=simple
Restart=always
WorkingDirectory=/home/<YOUR-NEST-USERNAME>/<YOUR-REPO-NAME>
ExecStart=/usr/bin/node index.js
TimeoutStartSec=0

[Install]
WantedBy=default.target
```

Save and exit as before.

Now run it:

```bash
systemctl --user daemon-reload
systemctl --user enable --now slackbot.service
```

Congratulations! Your bot is now running 24/7.

---

## Something broken?

Run this to check what the issue is:

```bash
journalctl --user -u slackbot.service
```

### Common issues

* Wrong token
* Missing `.env`
* Incorrect working directory
* Missing dependencies

---

## Useful commands

### Restart the bot

```bash
systemctl --user restart slackbot.service
```

### Stop the bot

```bash
systemctl --user stop slackbot.service
```

### Start the bot

```bash
systemctl --user start slackbot.service
```

---

## Helpful documentations/links

* [Slack Docs](https://docs.slack.dev)
* [Slack Bolt JS Tutorial](https://slack.dev/bolt-js/tutorial/getting-started)
* [Node.js Docs](https://nodejs.org/)
* [Nest Quickstart Guide](https://guides.hackclub.app/index.php/Quickstart)

### Inspiring bot ideas to try

* Daily standup reporter (posts a summary at 9am)
* Fun facts bot (`/dsb-fact`)
* Moderation: auto-flag messages with banned words
* Games: trivia bot with score tracking
* Integrations: post GitHub PR updates

