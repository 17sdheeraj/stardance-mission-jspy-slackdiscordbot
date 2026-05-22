# Make your own Discord bot by using JS!

## Introduction

Ever wanted to have your own Discord bot which responds to your commands, automates your tasks, and does other useful stuff in your server?
In this guide, you will learn how to:

* Create a Discord application
* Build a Discord bot using JavaScript
* Add slash commands such as `/ping`
* Deploy your bot in [Hack Club Nest](https://nest.hackclub.com) for free
* Keep your bot online 24/7

This guide assumes no JavaScript experience. If you want a short intro to JavaScript first, read MDN's friendly overview: [MDN JavaScript Introduction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction)

This guide is very beginner friendly and uses the following:

* JavaScript (no prior coding required)
* Node.js
* Discord.js
* Discord slash commands
* Nest @ Hack Club

At the end of this tutorial you will have your own fully hosted Discord bot.

---

## What You Need

Before you start, gather these things:

* A Discord account
* A Discord server where you can test your bot
* Node.js and npm installed locally
* VS Code
* A GitHub account (for publishing your code)
* A Hack Club Nest account (used for hosting) - see notes below about applying

### Notes for beginners

* If you do not know JavaScript, the MDN intro above is a good quick primer.
* Discord bots are easier to build when you test them in a private server first.
* Hack Club Nest is a free place Hack Club provides to host projects. Search for "Hack Club Nest" or visit [Hack Club Dashboard](https://dashboard.hackclub.app) and follow the signup flow.
* Nest sometimes requires an application step, so do not let the wait discourage you. If you already have HCA (Hack Club Auth account) set up, the process is usually quick :)

---

## What We Will Build Today

In this tutorial, we will be creating a simple Discord bot using JS that responds to slash commands.
A slash command is a command you type starting with a slash.

### Examples you'll try

```txt
/ping
/hello
/userinfo
/serverinfo
/catfact
/joke
```

When a user runs one of these commands, the bot can do multiple stuff like show information, send a message, run automations, interact with APIs, run workflows, and more.

For example, if you make a command such as `/joke` or something else, you can make the bot fetch a joke from an API and send it in the channel.

---

## Why Slash Commands Instead Of Prefix Commands?

Discord has a lot of bots installed in servers, and slash commands are the modern way to build bots.
They are easier for beginners, easier for users to discover, and they do not need your bot to read every message in the server.

That is a big reason this tutorial uses slash commands instead of old prefix commands like `!ping`.

Slash commands are also the current recommended style for Discord bots, so that is what we will use here.

---

## Creating The Bot Application

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) and click **New Application**.
2. Give your app a name and click **Create**.
![Image](https://cdn.hackclub.com/019e4eb4-6c33-7f87-87b6-c51b146ee5e5/image.png)

### Resetting and copying the token

1. On the **Bot** page, find the **Token** section.
2. Click **Reset Token** if you need a new one, then copy it immediately.
3. Keep this token private. It is like a password for your bot.
![Image](https://cdn.hackclub.com/019e4ec6-d7ca-7bca-9812-86638181b99f/image.png)

### Important

Discord separates a few different things:

* Bot Token: used by your code to log in as the bot
* Client ID: used when registering slash commands and inviting the bot
* Client Secret: used for OAuth flows, not for this tutorial

Do not confuse the bot token with the client secret.

### Privileged Gateway Intents

On the **Bot** page you will see a section called **Privileged Gateway Intents**.

These are special toggles for advanced features:

* **Presence Intent** - lets your bot see online, idle, and do not disturb status changes
* **Server Members Intent** - lets your bot receive member join and leave information, and some member data
* **Message Content Intent** - lets your bot read message text in servers

For this tutorial, slash commands do not need Message Content Intent because the bot is not reading normal chat messages.
You can leave these off for now unless you add a feature that truly needs them.

### Why we do this

Keeping intents off unless you need them is cleaner and safer.
It also makes it easier to understand exactly what your bot is allowed to see.

### OAuth2 URL Generator

1. In the left sidebar, click **OAuth2** and then **URL Generator**.
2. Under **Scopes**, select:

```txt
bot
applications.commands
```

3. Under **Bot Permissions**, select the permissions your bot needs.

For this tutorial, choose:

```txt
Send Messages
Embed Links
Read Message History
```

You can add more later if your bot grows.

### What these permissions do

* **Send Messages** lets the bot reply in channels
* **Embed Links** lets the bot send nice embed cards for user info and server info
* **Read Message History** is useful for bots that need to look at earlier messages later

You do not need Administrator for this tutorial. Keeping permissions small is better.

### Invite the bot to your server

1. Copy the generated invite link from the bottom of the URL Generator page.
2. Open it in your browser.
3. Choose your test server.
4. Authorize the bot.
![Image](https://cdn.hackclub.com/019e4ec1-e830-7bc8-84e7-18f32bfe1d09/image.png)
![Image](https://cdn.hackclub.com/019e4ec2-c1ba-7ae9-ab1f-21d53d7fd95f/image.png)
![Image](https://cdn.hackclub.com/019e4ec4-654d-7502-8ab5-60e3ead44760/image.png)
### Important

The bot must be invited to a server before slash commands can appear there.
Also, slash commands do not show up just because you invited the bot. You still need to register the commands in code, which we will do soon.

---

## Project Setup

1. Create a new project folder and open it in your code editor.
2. VS Code is recommended because it makes the terminal and files easy to manage.

### Important

Open the folder in VS Code before running commands.
In VS Code use **Terminal → New Terminal** so commands run in the correct folder.

### Initialize the project and install dependencies

```bash
npm init -y
npm install discord.js dotenv axios
```

### What this does

* `npm init -y` creates a `package.json` file for your project
* `discord.js` gives you the Discord bot library
* `dotenv` loads secret values from `.env`
* `axios` lets the bot fetch data from APIs

### Why we do this

These packages give you the core pieces you need for a modern Discord bot.
`discord.js` already includes the tools we need for `Client`, `Events`, `REST`, `Routes`, and slash command building.

---

## File Structure

Create these files in your project folder:

* `index.js` - runs the bot and listens for slash commands
* `deploy-commands.js` - registers slash commands with Discord
* `.env` - stores private tokens and IDs
* `.gitignore` - keeps secrets and installed packages out of GitHub

### Why we split the files

Keeping command registration in `deploy-commands.js` and bot logic in `index.js` makes the project easier to understand.
One file tells Discord what commands exist.
The other file makes those commands actually do something.

---

## Basic Bot Setup

Create an `index.js` file in the project folder.

This is the base file. We will add the commands in parts so the project is easier to build.

```js
require("dotenv").config();

const { Client, Events, GatewayIntentBits } = require("discord.js");
const client = new Client({
  intents: [GatewayIntentBits.Guilds]
});

client.once(Events.ClientReady, (readyClient) => {
  console.log(`Logged in as ${readyClient.user.username}!`);
});

client.on(Events.InteractionCreate, async (interaction) => {
  if (!interaction.isChatInputCommand()) return;

  try {
    await interaction.reply("I do not know this command yet.");
  } catch (error) {
    console.error(error);

    if (!interaction.replied) {
      await interaction.reply({
        content: "Something went wrong while running that command.",
        ephemeral: true
      });
    }
  }
});

client.login(process.env.DISCORD_TOKEN);
```

### What this does

This is the starting point for the bot.
It connects to Discord, waits for slash commands, and gives us a place to add each command one at a time.

### Why we do this

`index.js` is the main file that stays running while your bot is online.
It is the place where the bot reacts to people using commands in Discord.

### Line by line explanation

* `require("dotenv").config();` loads values from your `.env` file
* `Client`, `Events`, and `GatewayIntentBits` come from `discord.js`
* `new Client({ intents: [GatewayIntentBits.Guilds] })` creates the bot client and tells Discord we only need guild command events for this tutorial
* `client.once(Events.ClientReady, ...)` runs one time when the bot finishes logging in
* `interactionCreate` runs every time someone uses a slash command
* `interaction.isChatInputCommand()` makes sure we only handle slash commands, not buttons or other interaction types
* `await interaction.reply("I do not know this command yet.")` is the base response before we add real commands
* `client.login(process.env.DISCORD_TOKEN)` logs the bot in with your secret token

### Build the commands in parts

Now we will replace the base reply with real commands one by one.

### Add the `/ping` command

```js
if (interaction.commandName === "ping") {
  const start = Date.now();
  await interaction.deferReply();
  const replyTime = Date.now() - start;
  const websocketLatency = Math.round(client.ws.ping);

  await interaction.editReply(
    `Pong!\nResponse time: ${replyTime}ms\nWebSocket ping: ${websocketLatency}ms`
  );
  return;
}
```

### What this does

This command checks how quickly the bot can answer.
It also shows the WebSocket ping so you can see how the bot is connected to Discord.

### Add the `/hello` command

```js
if (interaction.commandName === "hello") {
  await interaction.reply(`Hello, ${interaction.user.username}!`);
  return;
}
```

### What this does

This command sends a simple friendly greeting back to the user.
It is the easiest command to understand when you are just starting out.

### Add the `/userinfo` command

```js
if (interaction.commandName === "userinfo") {
  const user = interaction.user;

  await interaction.reply({
    embeds: [
      {
        title: "User Info",
        color: 0x5865f2,
        thumbnail: {
          url: user.displayAvatarURL({ size: 256 })
        },
        fields: [
          { name: "Username", value: user.username, inline: true },
          { name: "User ID", value: user.id, inline: true },
          { name: "Bot", value: user.bot ? "Yes" : "No", inline: true },
          {
            name: "Account Created",
            value: `<t:${Math.floor(user.createdTimestamp / 1000)}:F>`,
            inline: true
          }
        ]
      }
    ]
  });
  return;
}
```

### What this does

This command shows information about the person who used the command.
It is a good example of a command that uses Discord user data and an embed.

### Add the `/serverinfo` command

```js
if (interaction.commandName === "serverinfo") {
  const guild = interaction.guild;

  if (!guild) {
    await interaction.reply("This command can only be used in a server.");
    return;
  }

  const guildIcon = guild.iconURL({ size: 256 });

  await interaction.reply({
    embeds: [
      {
        title: "Server Info",
        color: 0x57f287,
        thumbnail: guildIcon ? { url: guildIcon } : undefined,
        fields: [
          { name: "Server Name", value: guild.name, inline: true },
          { name: "Server ID", value: guild.id, inline: true },
          { name: "Members", value: `${guild.memberCount}`, inline: true },
          {
            name: "Created",
            value: `<t:${Math.floor(guild.createdTimestamp / 1000)}:F>`,
            inline: true
          },
          {
            name: "Owner ID",
            value: guild.ownerId ?? "Unknown",
            inline: true
          }
        ]
      }
    ]
  });
  return;
}
```

### What this does

This command shows information about the server where the bot was used.
It is a good example of using `interaction.guild`.

### Add `axios` for the API commands

Before you add `/catfact` and `/joke`, add this near the top of `index.js`:

```js
const axios = require("axios");
```

### What this does

This gives your bot the tool it needs to ask other websites for data.
We only need it once, and then both API commands can use it.

### Add the `/catfact` command

```js
if (interaction.commandName === "catfact") {
  await interaction.deferReply();

  try {
    const response = await axios.get("https://catfact.ninja/fact");
    await interaction.editReply(`Cat Fact:\n${response.data.fact}`);
  } catch (error) {
    await interaction.editReply("Failed to fetch a cat fact.");
  }

  return;
}
```

### What this does

This command uses `axios` to ask an API for a random cat fact.
Then it sends the fact back into Discord.

### Add the `/joke` command

```js
if (interaction.commandName === "joke") {
  await interaction.deferReply();

  try {
    const response = await axios.get(
      "https://official-joke-api.appspot.com/random_joke"
    );

    await interaction.editReply(
      `${response.data.setup}\n\n${response.data.punchline}`
    );
  } catch (error) {
    await interaction.editReply("Failed to fetch a joke.");
  }

  return;
}
```

### What this does

This command uses `axios` to ask an API for a random joke.
Then it sends the joke back into Discord.

### Important

The bot only needs the `Guilds` intent for this tutorial because slash commands do not require message content.
That keeps the bot simple and follows modern Discord bot design.

---

## Slash Commands

Slash commands are the commands users type with a slash, like `/ping` or `/hello`.
Discord needs to know about your commands before they can appear in the server menu.

That is why we use a separate file called `deploy-commands.js`.

### Why `deploy-commands.js` exists

The bot code in `index.js` listens for commands.
The deploy file tells Discord which commands exist.

If you change a command name or add a new command, you run the deploy file again so Discord gets the updated list.

### Why we deploy to one server first

For beginners, guild command deployment is easier because the commands usually appear very quickly.
Global commands can take longer to update, so we start with one server while testing.

---

## Deploying Slash Commands

Create a `deploy-commands.js` file.

This file also starts as a base, then we add the command list in parts.

```js
require("dotenv").config();

const { REST, Routes, SlashCommandBuilder } = require("discord.js");

const commands = [
  new SlashCommandBuilder()
    .setName("ping")
    .setDescription("Check if the bot is alive."),
  new SlashCommandBuilder()
    .setName("hello")
    .setDescription("Say hello to the bot."),
  new SlashCommandBuilder()
    .setName("userinfo")
    .setDescription("Show information about you."),
  new SlashCommandBuilder()
    .setName("serverinfo")
    .setDescription("Show information about this server."),
  new SlashCommandBuilder()
    .setName("catfact")
    .setDescription("Get a random cat fact."),
  new SlashCommandBuilder()
    .setName("joke")
    .setDescription("Get a random joke.")
].map((command) => command.toJSON());

const rest = new REST({ version: "10" }).setToken(process.env.DISCORD_TOKEN);

(async () => {
  try {
    console.log(`Refreshing ${commands.length} slash commands...`);

    await rest.put(
      Routes.applicationGuildCommands(
        process.env.DISCORD_CLIENT_ID,
        process.env.DISCORD_GUILD_ID
      ),
      { body: commands }
    );

    console.log("Slash commands were registered successfully.");
  } catch (error) {
    console.error(error);
  }
})();
```

### Build the command list one by one

If you want to see the parts more clearly, here is how the command list is built:

```js
new SlashCommandBuilder().setName("ping").setDescription("Check if the bot is alive.")
```

```js
new SlashCommandBuilder().setName("hello").setDescription("Say hello to the bot.")
```

```js
new SlashCommandBuilder().setName("userinfo").setDescription("Show information about you.")
```

```js
new SlashCommandBuilder().setName("serverinfo").setDescription("Show information about this server.")
```

```js
new SlashCommandBuilder().setName("catfact").setDescription("Get a random cat fact.")
```

```js
new SlashCommandBuilder().setName("joke").setDescription("Get a random joke.")
```

### What this does

This file creates the list of slash commands and sends them to Discord.
After you run it, Discord learns about `/ping`, `/hello`, `/userinfo`, `/serverinfo`, `/catfact`, and `/joke`.

### Why we do this

Discord does not guess your commands.
You must register them first so they appear in the command picker.

![Image](https://cdn.hackclub.com/019e4efa-1324-7313-98c6-d4c68482ca77/image.png)

### Line by line explanation

* `require("dotenv").config();` loads your secret values
* `REST` is the Discord.js helper for making REST API requests
* `Routes` gives you the correct API route for command registration
* `SlashCommandBuilder` builds each slash command in a safe, modern way
* `.setName()` gives the command its public name
* `.setDescription()` tells users what the command does
* `.map((command) => command.toJSON())` turns the command builders into plain JSON Discord can accept
* `new REST({ version: "10" })` creates a REST client for Discord API v10
* `Routes.applicationGuildCommands(...)` registers commands in one server
* `process.env.DISCORD_CLIENT_ID` is your application's client ID
* `process.env.DISCORD_GUILD_ID` is the server ID where you want the commands to appear fast

### Important

If you change a command later, run `node deploy-commands.js` again.
If you add a new command and forget to redeploy, Discord will not know the command exists yet.

### How to get your server ID

1. Open Discord settings.
2. Turn on **Developer Mode**.
3. Right-click your test server.
4. Click **Copy Server ID**.

### How to get your client ID

1. Open your application in the Discord Developer Portal.
2. Go to **OAuth2** or **General Information**.
3. Copy the **Application ID**.

---

## Environment Variables

Create a `.env` file in the same folder and add your values.

```env
DISCORD_TOKEN=your_bot_token_here
DISCORD_CLIENT_ID=your_application_id_here
DISCORD_GUILD_ID=your_test_server_id_here
```

### What this does

This file stores your private token and IDs in one place.
Your code can read them without you typing secrets directly into the JavaScript files.

### Why we do this

Secrets should not go into GitHub.
Keeping them in `.env` helps prevent accidental leaks.

### Important

`DISCORD_TOKEN` is your bot token, not your client secret.
If you ever think a token was leaked, go back to the Developer Portal and reset it.

---

## Git Ignore File

Create a `.gitignore` file with this content:

```gitignore
node_modules
.env
```

### What this does

* `node_modules` is the big folder that npm installs
* `.env` is your secret file

### Why we do this

You do not want to upload installed packages or secrets to GitHub.

---

## Understanding How Commands Work

### Basic command structure

```js
new SlashCommandBuilder()
  .setName("ping")
  .setDescription("Check if the bot is alive.");
```

### Explanation

| Part | What it does |
| --- | --- |
| `new SlashCommandBuilder()` | Creates a new slash command |
| `.setName("ping")` | Gives the command its name |
| `.setDescription(...)` | Tells people what the command does |

:::callout type="warning"
Slash command names must be lowercase and cannot contain spaces.
If you try to use a bad name, Discord will reject it.
:::

### How the bot receives a command

1. A user types `/ping`
2. Discord sends the interaction to your bot
3. Your bot checks `interaction.commandName`
4. Your code runs the right command block
5. The bot replies in Discord

### What `deferReply()` means

Some commands need a little more time.
`deferReply()` is the bot saying, "I saw your command, give me a second."

That is why the API commands use `deferReply()` before fetching data.

---

## Expanding Your Bot And Adding More Commands

Now that you got your bot up and running you can add more commands.

### Example - help command

```js
new SlashCommandBuilder()
  .setName("help")
  .setDescription("Show the available commands.");
```

Example of a help command reply:

```js
await interaction.reply(
  "Available Commands:\n/ping - Check bot latency\n/hello - Say hello\n/userinfo - Show your info\n/serverinfo - Show server info\n/catfact - Get a cat fact\n/joke - Get a joke"
);
```

### API commands

You can fetch stuff via APIs such as cat facts, jokes, weather information, and more.

You can check out [Free APIs List](https://free-apis.github.io/#/browse) to find many APIs which are free and easy to test.

### Why API commands are fun

API commands make your bot feel alive.
Instead of always giving the same reply, the bot can fetch new information every time.

### Important

APIs can fail sometimes.
That is why the tutorial uses `try/catch` around the API requests.
If the API is down, the bot should say so nicely instead of crashing.

---

## Example Commands

The main code above already includes these commands:

* `/ping`
* `/hello`
* `/userinfo`
* `/serverinfo`
* `/catfact`
* `/joke`

### `/ping`

This command checks how quickly the bot can answer.
It also shows the WebSocket ping so you can see how the bot is connected to Discord.

### `/hello`

This command sends a simple friendly greeting back to the user.
It is the easiest command to understand when you are just starting out.

### `/userinfo`

This command shows information about the person who used the command.
It is a good example of a command that uses Discord user data and an embed.

### `/serverinfo`

This command shows information about the server where the bot was used.
It is a good example of using `interaction.guild`.

### `/catfact`

This command uses `axios` to ask an API for a random cat fact.
Then it sends the fact back into Discord.

### `/joke`

This command uses `axios` to ask an API for a random joke.
Then it sends the joke back into Discord.

---

## Running The Bot Locally

Open a terminal in VS Code with your project folder as the current directory. Then run:

```bash
node deploy-commands.js
node index.js
```

### What this does

* `node deploy-commands.js` registers the slash commands with Discord
* `node index.js` starts the bot itself

### Why we do it in this order

The bot cannot use commands that Discord does not know about yet.
So first we register the commands, then we start the bot.

### If successful

You should see something like:

```txt
Slash commands were registered successfully.
Logged in as YourBotName!
```
![Image](https://cdn.hackclub.com/019e4efb-33f0-7292-a0f4-4c47c3f29319/image.png)

Then test your slash command in Discord, for example `/ping`.

### Common local mistakes

* Running the command in a terminal that is not in your project folder - make sure the terminal's current directory contains `index.js`
* Forgetting to run `node deploy-commands.js` before testing
* Copying the wrong token
* Using the wrong server ID in `.env`

---

## Common Beginner Mistakes

### Invalid token

If the bot says the token is invalid, check that you copied the bot token from the Bot page and not the client secret.

### Missing intents

For this tutorial, you only need the `Guilds` intent in code.
If you later add message-reading features, you may need other intents too.

### Bot offline

If the bot is offline, make sure `node index.js` is running and that the token is correct.

### Commands not appearing

If slash commands do not show up, run `node deploy-commands.js` again and make sure the server ID and client ID are correct.

### Wrong permissions

If the bot cannot send replies or embeds, go back to the OAuth2 URL Generator and make sure the right permissions were selected.

### Forgot to run deploy-commands.js

This is one of the most common mistakes.
Your bot can be online and still not have any commands if you forgot to deploy them.

### Missing .env

If the bot cannot find your token, make sure the `.env` file exists and is in the same folder as `index.js`.

### Wrong Node.js version

Use a modern Node.js version.
For 2026, Node.js 20 or newer is a safe choice for a project like this.

---

## Push Your Code To Github

Run the following code to publish the code to GitHub.

```bash
git init
git add .
git commit -m "Initial commit"
```

### What this does

* `git init` starts a Git repository
* `git add .` stages your files
* `git commit -m "Initial commit"` saves your first snapshot

### Why we do this

GitHub makes it easier to save, back up, and deploy your project later.

### Optional next steps

If you want to push it to GitHub after this, add your remote and push:

```bash
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

---

## Bot Setup Via Nest

Now once you are done with adding all the commands to your bot, you remember that you cannot keep your bot running 24/7 on your local computer.

Now comes [Nest](http://nest.hackclub.com), a free Debian server provided to students by Hack Club.

Now you will set this up so you can have your bot running 24/7.

Make sure to have a Nest account for this.

---

## Basic Setup

Now SSH into the Nest server with your credentials.
If you do not know how to do this, do not worry. Refer to this [Quickstart Guide](https://guides.hackclub.app/index.php/Quickstart).

Once you are logged in, run the following one by one:

```bash
git clone https://github.com/<yourgithubusername>/<yourreponame>
cd <yourreponame>
npm install
```

### What this does

* `git clone` downloads your project onto the Nest server
* `cd` moves into the project folder
* `npm install` installs the dependencies from `package.json`

### Git not found, or missing?

If Git is not installed, run these commands one by one:

```bash
apt update && apt upgrade -y
apt install sudo -y
sudo apt install git -y
```

### What this does

These commands update the server and install Git so you can clone your repository.

### Create your .env file

Once that is done, create the `.env` file here by running:

```bash
nano .env
```

Add the same things here as the `.env` file in local.

```env
DISCORD_TOKEN=your_bot_token_here
DISCORD_CLIENT_ID=your_application_id_here
DISCORD_GUILD_ID=your_test_server_id_here
```

### To save and exit

* Press `Ctrl + O` to save
* Press Enter
* Press `Ctrl + X` to exit

### Run the bot

Again run:

```bash
node deploy-commands.js
node index.js
```

and test out the bot.

If that is working, continue with the guide to make it 24/7.

---

## Make Your Bot Run 24/7

### Creating a systemd service

Without systemd, your bot stops when:

* You disconnect SSH
* Nest restarts
* The process crashes

We will use systemd to keep it alive.

Run:

```bash
cd ~/.config/systemd/user
```

to change your directory.

And now create a new service by running:

```bash
nano discordbot.service
```

In that file add the following. Edit the working directory to match your Nest path.

```ini
[Unit]
Description=Discord Bot
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

### What this does

This tells Nest to keep your bot running in the background.
If the bot crashes, systemd restarts it automatically.

### Why we do this

This is what keeps the bot online even when you close SSH.

### Save and reload systemd

Save and exit as before.

Now run it:

```bash
systemctl --user daemon-reload
systemctl --user enable --now discordbot.service
```

Congratulations. Your bot is now running 24/7.

---

## Something Broken?

Run this to check what the issue is:

```bash
journalctl --user -u discordbot.service
```

### Common issues

* Wrong token
* Missing `.env`
* Incorrect working directory
* Missing dependencies
* Forgot to run `node deploy-commands.js`

### What to look for

The log usually tells you exactly what is wrong.
If the token is bad, the log will usually mention login problems.
If the file path is wrong, it will usually say it cannot find `index.js`.

---

## Useful Commands

### Restart the bot

```bash
systemctl --user restart discordbot.service
```

### Stop the bot

```bash
systemctl --user stop discordbot.service
```

### Start the bot

```bash
systemctl --user start discordbot.service
```

### View logs

```bash
journalctl --user -u discordbot.service
```

---

## Helpful Documentations And Links

* [Discord Developer Portal](https://discord.com/developers/applications)
* [Discord.js Guide](https://discordjs.guide/)
* [Discord.js Docs](https://discord.js.org/)
* [Node.js Docs](https://nodejs.org/)
* [Nest Quickstart Guide](https://guides.hackclub.app/index.php/Quickstart)

### Inspiring bot ideas to try

* Daily standup reporter that posts a summary at 9am
* Fun facts bot (`/fact`)
* Moderation bot that auto-flags messages with banned words
* Games bot with trivia and score tracking
* Integrations bot that posts GitHub PR updates


## Final result after working on this:
![Image](https://cdn.hackclub.com/019e4efd-2fe9-7d10-b089-7d1257d626ec/image.png)