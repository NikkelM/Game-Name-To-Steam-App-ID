# Steam App ID Finder

![Steam Banner](./images/SteamAppIdFinderBanner.png)

This project is a collection of utilities that can be used to find Steam App IDs from a variety of sources.

Choose from one of the `modes` described in the [section below](#modes) to find the Steam App IDs from the source you are interested in.

## Setup

Run `npm install` to install the required dependencies first.

Following this, create a `config.json` file in the `config` directory of the project and fill it with your desired [configuration](#configuration).
Use the default configuration file (`config.<mode>.default.json`) for the mode in which you wish to run the utility as a template if you wish.

## Usage

After providing the `config.json` [configuration](#configuration) file and any additional input files if required by the mode, you can run the script using

```bash
node index.js
```

## Output

All modes will save their respective output in the `output/<mode>` folder.
You can find additional information about the output of each mode in the respective section below.

## Configuration

The different modes have different requirements when it comes to the kind of input they require, and each mode gives you different options to configure the script's behaviour.
For each `mode`, a default configuration is provided in the `config.<mode>.default.json` file, which you can use as a template for your own configuration file.

### Schema validation

The project provides an extensive JSON validation schema, which can help you with formatting your input and give you an idea of the options you have.

The schema can be found in the `config.schema.json` file and used within your `config.json` by adding the following property:

```json
"$schema": "config.schema.json"
```

*NOTE: The script will always test your provided `config.json` against this schema, so make sure your configuration is valid.*
*By adding the schema to your `config.json`, you will receive information about mistakes in the configuration.*

### Properties

The following is a list of all configuration items that are required no matter which `mode` you choose.

If any given property is not present in the configuration file, it will automatically be assumed to have a value of `false` (or equivalent, depending on the property type).

<details>
<summary><code>mode</code></summary>

The mode in which the script should run. Choose from any of the supported modes listed in the sections below.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `gameNames` | `gameNames`, `steamAccount` | Yes |
</details>

# Modes

You can choose from any of the following modes when running the script:

- [`gameNames`](#mode-gamenames): Do you have a list of game names and want to know which Steam App IDs they correspond to?
This mode is able to find the Steam App IDs for any number of provided game names, even if the provided name is not an exact match to the game in the Steam database.
- [`steamAccount`](#mode-steamaccount): This mode will fetch all apps (this includes games, but also e.g. soundtracks or movies) from a given Steam account and save them to a file.
Make sure that the account's game library is public, otherwise the script will not be able to access it.
- [`gogAccount`](#mode-gogaccount): With this mode, you can get the names for all games in your GOG account. You can then use this output as input for the [`gameNames`](#mode-gamenames) mode to find the Steam App IDs for the games.

## Mode: `gameNames`

Do you have a list of game names and want to know which Steam App IDs they correspond to?
This mode is able to find the Steam App IDs for any number of provided game names, even if the provided name is not an exact match to the game in the Steam database.

When running the scipt in the `gameNames` mode, it will find the Steam App IDs for all game names provided in the specified input file, starting with those that have an exact match in the Steam database.
If no exact match is found, the script will match the resulting game names to the Steam App ID of the game with the highest similarity score.

Note that the script will *always* find a match, even if the provided game name is not on Steam.
You can correct the number of false positives this may produce by providing a `partialMatchThreshold` in the [configuration](#configuration-gamenames) file.
Through testing, I found that a threshold of 0.65 finds almost all correct matches (i.e. games with a lower score are probably not on Steam) without too much clutter, but you can adjust this depending on your list of games.

### Output

You will find the resulting data in the created `output/gameNames` folder.

For games with a single full match, the results will be saved in a file called `steamAppIds_fullMatches.json`, with the games' names as keys and their corresponding Steam App IDs as values.

For games with multiple full matches, the results will be saved in a file called `steamAppIds_multipleFullMatches.json`, with the games' names as keys and an array of the corresponding Steam App IDs as values.
You can use tools such as [steamDB](https://steamdb.info/) to find out which of the found Steam App IDs is the correct one for your game.

For partial matches, the results will be saved in a file called `steamAppIds_partialMatches.json`, with the games' names as keys and the name and Steam App ID of the most similar game name as well as the similarity score as value.

Games for which no satisfying match was found (due to the similarity score being below the `partialMatchThreshold`) will be saved in a file called `steamAppIds_noMatch.json` for your convenience.

Find some examples of the output format for a partial match below:

#### Examples

```json
{
  "Fallout Tactics: Brotherhood of Steel": {
    "appId": 38420,
    "similarity": 0.5777777777777777,
    "steamName": "Fallout Tactics"
  },
}
```

In this case, the matched game was correct, but this is not always the case:

```json
{
  "DEATH STRANDING": {
    "appId": 1507790,
    "similarity": 0.6666666666666666,
    "steamName": "Witch Strandings"
  },
}
```

Such a mismatch can have a number of reasons, in the case of `DEATH STRANDING` it is that Steam only has the `DEATH STRANDING DIRECTOR'S CUT` in its database, but not the original game, which leads to `Witch Strandings` being the closest match.

These examples also show well how s higher similarity score does not necessarily mean a better match.

#### Matches with a similarity score of 1

It is possible for a game to have a partial match with a similarity score of 1.
This can happen as the search for full matches is case sensitive, but the search for partial matches is not, meaning that the following two game names would not be considered a full match, but their similarity score would be 1:

- `My Time at Portia`
- `My Time At Portia`

### Configuration: `gameNames`

#### Properties

The following is a list of all configuration items, their defaults and the values they can take.

<details>
<summary><code>inputFile</code></summary>

The file to read the game names from.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `object` | See item below | See sections below | Yes |

```json
"inputFile": {
  "fileName": "gameNames",
  "fileType": "txt",
  "delimiter": ","
}
```

<h3>Possible values</h3>

<h4><code>fileName</code></h4>

The name of the file to read the game names from.

Note: Do *not* provide the file's type/extension here.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `gameNames` | Any valid file name | Yes |

<h4><code>fileType</code></h4>

The file type of the input file.
Choose from any of the supported file types listed.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `txt` | `txt`, `csv` | Yes |

<h4><code>delimiter</code></h4>

The delimiter to use when parsing the input file.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `,` | Any valid delimiter | Yes, if the file type requires it. (true for all currently supported file types) |
</details>


<details>
<summary><code>onlyFullMatches</code></summary>

Whether to only get Steam App IDs for full matches or also for partial matches.
If set to `false`, partial matches will be saved to a different output file.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `false` | `true`, `false` | No |
</details>

<details>
<summary><code>partialMatchThreshold</code></summary>

The threshold for partial matches.
This means that the most similar game name must have a similarity score of at least this threshold to be added to the output.
The threshold must be between 0 and 1.
Use 0 to get a match for every game and 1 to only get full matches.
If the value is omitted, a match will be found for every game.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `number` | `0.8` | Number between `0` and `1` | No |
</details>

## Mode: `steamAccount`

This mode will fetch all apps (this includes games, but also e.g. soundtracks or movies) from a given Steam account and save them to a file.
Make sure that the account's game library is public, otherwise the script will not be able to access it.

You can check if the game library for a given `accountName` is public by logging out of Steam (or opening a private browsing session) and visiting this link: [https://steamcommunity.com/id/accountName/games](https://steamcommunity.com/id/accountName/games)

To set your game library to public, visit this link while logged in: [https://steamcommunity.com/my/edit/settings](https://steamcommunity.com/my/edit/settings)

### Output

You will find the resulting data in the created `output/gameNames` folder in a file named after your Steam account name.

### Configuration: `steamAccount`

#### Properties

The following is a list of all configuration items, their defaults and the values they can take.

<details>
<summary><code>steamAccountName</code></summary>

The name of the Steam account for which the App IDs should be fetched.
Your account's game library must be set to public for the script to work.
Check via this link: https://steamcommunity.com/id/accountName/games while not logged in.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `accountName` | Any valid Steam account name | Yes |
</details>

<details>
<summary><code>outputProperties</code></summary>

Which of the properties provided by the Steam API should be included in the resulting JSON object. Properties that are not available for an app will be omitted in the output.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `object` | See item below | See sections below | Yes, and at least one property defined |

```json
{
	"appId": true,
	"name": true,
	"logo": false,
	"storeLink": false,
	"statsLink": false,
	"globalStatsLink": false
}
```
</details>

#### outputProperties

You can choose any combination (at least one) of the following properties to be included in the output file:

<details>
<summary><code>appID</code></summary>

The App ID of the game.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `true` | `true`, `false` | No |
</details>

<details>
<summary><code>name</code></summary>

The name of the game.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `true` | `true`, `false` | No |
</details>

<details>
<summary><code>logo</code></summary>

The URL to the game's logo.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `false` | `true`, `false` | No |
</details>

<details>
<summary><code>storeLink</code></summary>

The URL to the game's store page.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `false` | `true`, `false` | No |
</details>

<details>
<summary><code>statsLink</code></summary>

The URL to this users stats page for this game.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `false` | `true`, `false` | No |
</details>

<details>
<summary><code>globalStatsLink</code></summary>

The URL to the global stats page for this game.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `boolean` | `false` | `true`, `false` | No |
</details>

## Mode: `gogAccount`

With this mode, you can get the names for all games in your GOG account. You can then use this output as input for the [`gameNames`](#mode-gamenames) mode to find the Steam App IDs for the games.

Unfortunately, due to limitations of the GOG API, setup for this mode is a bit more complicated than for the other modes:

### Setup

Start setting up as usual by creating a `config.json` file in the `config` folder with the `mode` set to `gogAccount`.

If you are using this mode for the first time, you will need to log in to your GOG account using the following link: [https://auth.gog.com/auth?client_id=46899977096215655&redirect_uri=https%3A%2F%2Fembed.gog.com%2Fon_login_success%3Forigin%3Dclient&response_type=code&layout=client2](https://auth.gog.com/auth?client_id=46899977096215655&redirect_uri=https%3A%2F%2Fembed.gog.com%2Fon_login_success%3Forigin%3Dclient&response_type=code&layout=client2).
This is the official login page for GOG, so you do not need to set your credentials in the `config.json` file.

After logging in, you will be redirected to a blank page with a URL that looks something like this: 

```text
https://embed.gog.com/on_login_success?origin=client&code=1234567890abcdef
```

Copy the value of the `code` parameter from this URL and set it as the value of the `gogLoginCode` property in the `config.json` file.
This allows the script to generate an access token for your GOG account, which authenticates you for the GOG API.

After setting the `gogLoginCode` property, run the script immediately - the login code is only valid for 60 seconds, after which it will expire and you would need to log in again.

After running the script once, you will find a file named `gogRefreshToken.json` in the `output/gogAccount` folder.
This file contains the refresh token for your GOG account, which allows the script to generate a new access token when the current one expires.
To run the script with this refresh token in use, set it as the value of the `gogRefreshToken` property in the `config.json` file.
You can then remove the `gogLoginCode` property from the `config.json` file.

### Output

You will find the list of games in your GOG account in the `output/gogAccount` folder as a `.txt` file named `gogGameNames.txt`.
You can use this file as input for the [`gameNames`](#mode-gamenames) mode to find the Steam App IDs for the games.
To do this, simply set the following as the value of the `inputFile` property in the `config.json` file, along with the other configuration options:

```json
"inputFile": {
	"fileName": "output/gogAccount/gogGameNames.txt",
	"fileType": "txt",
	"delimiter": "\n"
}
```

The script will also save the current refresh token to the `gogRefreshToken.json` file in the `output/gogAccount` folder.
You can use this token to avoid needing to log in again when using the script in the (near) future.

### Configuration: `gogAccount`

#### Properties

The following is a list of all configuration items, their defaults and the values they can take.

<details>
<summary><code>gogLoginCode</code></summary>

The code you received after logging in (consult the README.md for more information).
Be fast, this code is only valid for one minute.
If you don't start the script within that time, it will not be able to generate an access token from it.
If a refresh token is also provided, this option is ignored.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `"gogLoginCodeHereIfAvailable"` | A valid login code for your account | Yes, if no `gogRefreshToken` is provided |
</details>

<details>
<summary><code>gogRefreshToken</code></summary>

If you have used the script before and have a refresh token, enter it here to avoid having to log in again to generate a login code.
After running the script, the refresh token will be saved to "output/gogAccount/gogRefreshToken.txt".
You can then use it here to avoid having to log in again.

| Type | Default value | Possible values | Required |
| --- | --- | --- | --- |
| `string` | `"gogRefreshTokenHereIfAvailable"` | A valid refresh token for your account | Yes, if no `gogLoginCode` is provided |
</details>

## Related projects

Wondering what to do with the Steam App IDs you just found? You could try organizing all of your Steam games in Notion using the [Notion Steam API integration](https://github.com/NikkelM/Notion-Steam-API-Integration).

Would you like to know what games are currently available on Game Pass for your platform? Check out the [Game Pass API](https://github.com/NikkelM/Game-Pass-API) project.

## Feedback

If you have any question, feedback or feature requests, feel free to open an [issue](https://github.com/NikkelM/Steam-App-ID-Finder/issues/new).