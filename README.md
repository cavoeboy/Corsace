Please use this repository when you are changing any of the code for Corsace projects.

To get started:
```
git clone https://github.com/Corsace/Corsace
```

Install node-modules:
```
npm i
```

## Getting Started

Duplicate `config/default.json` to `config/user/$USER.json`, `$USER` being your system username. Edit all parts as necessary. 
The values in your personal `config/user/$USER.json` config file will be referred to as `config` from now on.

### osu! API

`config.osu.v1.apiKey`

You can obtain your osu! API V1 key at https://osu.ppy.sh/p/api/]

`config.osu.v2.clientId`
`config.osu.v2.clientSecret`

You will need to create a "New OAuth Application" at the bottom of https://osu.ppy.sh/home/account/edit.

The callback URL should be set to:
```
config.corsace.publicUrl + /api/login/osu/callback
```

### Database

#### Setup

`config.database`

##### Manual MariaDB Setup

You will need to install [MariaDB](https://mariadb.org/) and create an empty database, named whatever you like. 

It can be as simple as running:
```
mysql -u root -p
MySQL> create database <new_db_name>; 
```

Make sure to update `config.database` to reflect your choice of database name and credentials.

##### Via Docker

We are shipping a production-like `docker-compose.yml` file. You can start only the database service using: `docker-compose up -d database`

The database will listen on `127.0.0.1:3306`, with `corsace` being the database name, username and password.

#### Seeding the database

Create and seed the whole Corsace database using: `npm run typeorm migration:run`

### Discord

#### Setup

`config.discord`

This is the most time-consuming part of the setup. 
You will need the following:

##### Enable Developer Mode
Check the option at 
```
User Settings > Appearance > Advanced > Developer Mode
```

This will allow you to right click users, roles, channels, etc to copy their IDs.

##### A Discord Server
Create a new Discord Server if you don't have one already. All it needs to have is a single channel.
Create a "staff" role and give it to yourself.

Right-click your server name and "Copy ID". Paste this into `config.discord.guild`.

Right-click your staff role and "Copy ID". You can either create a role for each corresponding role in the config, OR 
paste that role ID into the following config values to give yourself god-tier permissions.
```
config.discord.roles.corsace.corsace
config.discord.roles.corsace.headStaff
config.discord.roles.corsace.staff
```
and then into every other "staff" role in the config.

##### Discord Application
Go to https://discord.com/developers/applications and create a "New Application".

###### Client
You will need to add the "Client ID" and "Client Secret" to the config as follows:
```
discord: {
    ...,
    clientId: "<Client ID>",
    clientSecret: "<Client Secret>",
}
```

###### OAuth2
Head to the OAuth2 section of the bot and add the following redirect URLs:
```
http://localhost:3000/api/login/discord/callback
http://localhost:4000/api/login/discord/callback
http://localhost:5000/api/login/discord/callback
http://localhost:7000/api/auth/discord/callback
http://localhost:7000/api/login/discord/callback
http://localhost:8000/api/login/discord/callback
```

Also add a redirect URL with your bot's specific Client ID that looks like:
```
https://discordapp.com/oauth2/authorize?&client_id=<CLIENT ID>&scope=bot&permissions=8
```
Follow this link to add your bot to your server.

Below, in the OAuth2 URL Generator section, set your redirect URL to
```
http://localhost:3000/api/login/discord/callback
```
This might also change based on which app you are working on.

###### Bot Token
Head to the Bot section of the bot and copy your bot token. 
Paste it into `config.discord.token`

#### Development

Run `npm run dev`, if you only want to run one of the projects, refer to the scripts in `package.json`.

Note: `npm run dev` can run into some concurrency issues. 
Either try rerunning it or running the individual project scripts.
