# Make your own Slack bot by using JS!

## Introduction:

Ever wanted to have your own Slack bot which responds to your commands, automates your tasks and other stuff in Hack Club?  
In this guide, you will learn how to:

- Create a slack app
- Build a slack bot using javascript
- Adding slash commands such as `/ping`
- Deploy your bot in [Hack Club Nest](https://nest.hackclub.com) for free
- Keeping your bot online 24/7

This guide is very beginner friendly and uses the following:

- Javascript (You know)
- Node.js
- Slack Bolt
- Socket Mode
- Nest @ Hack Club

At the end of this tutorial you will have your own fully hosted Slack bot.

---

## What you will need:

Before starting make sure to have the following:

- Make sure your account is in hack club slack workspace
- A github account
- A hack club nest account ([apply/login here](https://dashboard.hackclub.app))
- Basic knowledge of Javascript & Node.js

---

## What we will build today:

In this tutorial, we will be creating a simple slack bot using js that responds to slash commands. A slash command is a command you type starting with a slash.

Examples:

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

## Creating the bot:

First head over to the [slack apps dashboard](https://api.slack.com/apps/) and create a new bot “from scratch”, put a name, select Hack Club as workspace and create the bot.

Then navigate to socket mode and enable socket mode. Put in a name and make sure you have the below scope enabled:

```txt
connections:write
```

Socket mode allows the bot to communicate with slack. Slack will generate a token, save this token somewhere, do not share it with anyone!

Now, go to OAuth and permissions tab and under the bot token scopes add:

```txt
chat:write
commands
app_mentions:read
channels:history
```

You can add more scopes later according to what things you add to the bot.

These permissions allow your bot to:

- Send messages
- Use slash commands
- Read mentions
- Access channel messages

Once that's done, go to install app tab and install it to Hack Club, click on allow to get the bot added to the workspace.

Once you add it, you will get a Bot User OAuth Token, save this token somewhere and do not share it with anyone as that will enable other people to do bad stuff with your bot.

---

## Slash commands:

Go to the slash commands tab and create a new slash command such as `/dsb-ping` or other commands you would like to make/have, make sure to read the “Why is there /dsb-... instead of plain /ping, etc?” section at the start!

---

## Setting up the bot:

Make a folder and open up the terminal and run the following commands (make sure to have npm/nodejs installed)

```bash
npm init -y
```

This is to initialize our project (`package.json`)

These are the packages we are using for the basic ping command:

```bash
npm install @slack/bolt dotenv
```

Now make a file named `index.js` and use the following as the base code (this includes the basic slash command we made before, make sure to change the command before running it):

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

  await respond({
    text: `Pong!\nLatency: ${latency}ms`
  });
});

(async () => {
  await app.start();

  console.log("bot is running!");
})();
```

In the same folder, make a file named `.env`
And add the tokens which we got from the dashboard before like this:

```env
SLACK_BOT_TOKEN=xoxb-.....
SLACK_APP_TOKEN=xapp-....
```

We should always store tokens,etc in an `.env` file and never hardcode it in a js file. This keeps us safe from accidentally leaking the tokens.

Now make a new file named `.gitignore` in the same folder and add the following in it:

```gitignore
node_modules
.env
```

This prevents the tokens and dependencies from being added/pushed into the repository.

---

## Running the bot (locally):

Now start the bot by using:

```bash
node index.js
```

You should see:

```txt
bot is running!
```

Now go over to slack and try to test out the slash command you made.

Congratulations! You got your bot up and running :))

---

## Understanding How Commands Work

### Basic command structure:

```js
app.command("/command-name", async ({ ack, respond }) => {

});
```

### Explanation:

| Part | What it does |
|---|---|
| `app.command()` | Registers a slash command |
| `"/command-name"` | The command Slack listens for |
| `async` | Allows asynchronous operations like APIs |
| `ack()` | Acknowledges the command to Slack |
| `respond()` | Sends a message back to Slack |

:::callout type="warning"
`ack()` is required. If you do not acknowledge the command quickly enough, Slack will think the command failed.
:::

---

## Expanding your bot/adding more commands:

Now that you got your bot up and running you can add more commands such as:

```txt
/dsb-help
/dsb-catfact
/dsb-joke
```

and many more!

### Example help command:

```js
app.command("/dbt-help", async ({ ack, respond }) => {
  await ack();

  await respond({
    text:
`Available Commands:

/dbt-ping - Check bot latency
/dbt-catfact - Get a random cat fun fact
/dbt-help - Shows this help menu`
  });
});
```

### How the Help Command Works

#### Step 1: User Runs the Command

Inside Slack:

```txt
/dbt-help
```

Slack sends this interaction to your bot.

#### Step 2: Bot Detects the Command

This line:

```js
app.command("/dbt-help"
```

Tells Slack Bolt:

> “Whenever someone runs /dbt-help, execute this block of code.”

#### Step 3: Acknowledge the Command

```js
await ack();
```

This tells Slack:

> “The bot received the command successfully.”

Without this, Slack may show:

- “Something went wrong”
- “The app did not respond”

#### Step 4: Respond Back to Slack

```js
await respond({
 text: "..."
});
```

This sends a message back into Slack.

In this case, it sends a list of commands.

---

## API commands (catfact,etc):

You can fetch stuff via API’s such as cat fun facts, jokes, weather information,etc.

You can check out [this list](https://free-apis.github.io/#/browse) to find many API’s which are free,etc and use them in your bot.

### Example (catfact):

To make API requests easier, install axios:

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

---

### Another example (joke command):

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

The basic flow is:

1. User runs a slash command
2. The bot receives the command
3. The bot sends a request to an API
4. The API returns data
5. The bot sends that data back into Slack

Both commands use:

- `axios` → to make API requests
- `try/catch` → to prevent crashes if the API fails
- `respond()` → to send messages back to Slack

Like this you can make how many commands as you want, make sure to add the commands via the dashboard!

:::callout type="info"
While using API’s make sure to remember APIs have rate limits and always use try/catch when making requests so your bot doesn’t crash if the API fails.

Also some APIs require authentication so put the apikeys in your `.env` file!
:::

---

## Push your code to Github:

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

## Bot setup via Nest:

Now once you are done with adding all the commands to your bot, you remember that you cannot keep your bot running 24/7.

Now comes [Nest](http://nest.hackclub.com), a free debian server provided to students by Hack Club.

Now you will set this up so you can have your bot running 24/7!

Make sure to have a nest account for this!

---

## Basic setup

Now SSH into the nest server with your credentials, if you do not know how to do this don’t worry; Refer to this [guide](https://guides.hackclub.app/index.php/Quickstart).

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

To save & exit:

- ::kbd[Ctrl+O]
- Press Enter
- ::kbd[Ctrl+X]

Again run:

```bash
node index.js
```

and test out the bot.

If that is working, continue with the guide to make it 24/7.

---

## Make your bot run 24/7:

### Creating a systemd service

Without systemd, your bot stops when:

- You disconnect SSH
- Nest restarts
- The process crashes

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

### Common issues:

- Wrong token
- Missing `.env`
- Incorrect working directory
- Missing dependencies

---

## Useful commands

### Restart the bot:

```bash
systemctl --user restart slackbot.service
```

### Stop the bot:

```bash
systemctl --user stop slackbot.service
```

### Start the bot:

```bash
systemctl --user start slackbot.service
```

---

## Helpful documentations/links:

- [Slack Docs](https://docs.slack.dev/)
- [Slack Bolt JS](https://slack.dev/bolt-js/tutorial/getting-started)
- [Node.js docs](https://nodejs.org/docs/latest/api/documentation.html)
- [Nest docs](http://guides.hackclub.app)

