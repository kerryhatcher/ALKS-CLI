# ALKS CLI

[![NPM](https://nodei.co/npm/alks.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/alks/)

[![Build Status](https://travis-ci.org/Cox-Automotive/alks-cli.svg?branch=master)](https://travis-ci.org/Cox-Automotive/alks-cli)

## About
CLI for working with the ALKS service.

### Prerequisites

To install and use the ALKS CLI, you will need Node.js (version 4 or greater) and NPM ([nodejs.org](https://nodejs.org/en/download/package-manager)).

## Installing

ALKS CLI is meant to be installed via NPM.

```
npm install -g alks
```

## Configuring

The ALKS CLI requires some basic environment information to get started. Simply run the configuration command and you'll be prompted for the necessary configuration settings.

    alks developer configure

* ALKS Server: The full URL to your ALKS server (ex: https://alks.company.com/rest)
* Network Username: Your network username
* Network Password: Your network password (needed for loading list of accounts/roles)
* Save Network Password: Whether or not to save your network password, we suggest saving your password for ease of use
* Default Account/Role: Select the default ALKS account/role to use

## Running

After installing the ALKS CLI it will be available on your path. Simply run the following to see a list of supported commands:

    alks

### Options

To see a what options are available to a command ask for help on it:

    alks sessions help open

### Password

Since ALKS requires you to pass your credentials, we've made the CLI provide multiple ways of handling this.

1. **Recommended:** Store your password in the keychain. We offer the ability to store your password securely using build in OS functionality. On OS X we use Keychain, on Windows we use Credential Vault and on Linux we use netrc. To store your password simply run `alks developer login` and follow the prompt. You can remove your password at any time by running `alks developer logout`.
2. Provide your password as an argument, simply pass `-p 'my pass!'`. Note this will appear in your Bash history.
3. Create an environment variable called `ALKS_PASSWORD` whose value is your password.
4. Type your password. If we do not find a password we will prompt you on each use.


#### Password Priority

We will attempt to lookup your password in the following order:

1. CLI argument
2. Environment variable
3. Keystore
4. Prompt user

## Docker

If you would rather run the ALKS CLI as a Docker container, simply run the following:

```
docker run -it -v ~:/root coxauto/alks-cli
```

If you are on a windows host and need SET instead of export then add a PLATFORM env:

```
docker run -it -e PLATFORM=windows -v %USERPROFILE%:/root coxauto/alks-cli sessions open -a %AWS_ACCT% -r %AWS_ROLE% -o env 
```

# Commands

## Developer

### `developer configure`

`alks developer configure` - Configures ALKS

### `developer switch`

`alks developer switch` - Switch the active ALKS account/role

### `developer login`

`alks developer login` - Store your login credentials in the OS keychain.

### `developer logout`

`alks developer logout` - Remove your login credentials from the OS keychain.

### `developer info`

`alks developer info` - Show your current developer configuration

### `developer accounts`

`alks developer accounts` - Show all available ALKS accounts (both Standard and IAM)

Arguments:

* `-e` Output as environment variables for creating account shortcuts


## Sessions

### `sessions open`

`alks sessions open` Creates/resumes an ALKS session, this is the preferred way of using ALKS as it automates the underlying ALKS session for you. If you don't provide an account/role you'll be prompted for the one you'd like to use. Alternative you can use your default account/role by passing `-d`.

This will create your sessions with the maximum life and automatically renew them when necessary. If you would like to do IAM/Admin work you'll need to pass the `-i` flag.

Arguments:

* `-p [password]` Your password
* `-a [account]` The ALKS account to use, be sure to wrap in quotes
* `-r [role]` The ALKS role to use, be sure to wrap in quotes
* `-i` Specifies you wish to work as an IAM/Admin user
* `-o [output]` Output format. Supports: `env`, `json`, `docker`, `creds`, `idea`
* `-n` If output is set to creds, use this named profile (defaults to default)
* `-N` Forces a new session to be generated
* `-d` Uses your default account from `alks developer configure`
* `-f` If output is set to creds, force overwriting of AWS credentials if they already exist

Output values:

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_SESSION_TOKEN`

Example: 

`alks sessions open -a "12345678909/dev - webdevact" -r "Dev"`

### `sessions console`

`alks sessions console` - Open the AWS console in the default browser for the specified ALKS session.

Arguments:

* `-p [password]` Your password
* `-a [account]` The ALKS account to use, be sure to wrap in quotes
* `-r [role]` The ALKS role to use, be sure to wrap in quotes
* `-i` Specifies you wish to work as an IAM/Admin user
* `-o [appName]` Open with an alternative app (safari, google-chrome, etc)
* `-N` Forces a new session to be generated
* `-d` Uses your default account from `alks developer configure`
* `-p [password]` Your password

### `sessions list`

`alks sessions list` - List active ALKS sessions, this includes both IAM and non-IAM sessions.

Arguments:

* `-p [password]` Your password

## IAM

### `iam createrole`

`alks iam createrole` Creates a new IAM role for the requested type in the specified AWS account.

Arguments:

* `-p [password]` Your password
* `-n [roleName]` The name of the role, be sure to wrap in quotes, alphanumeric including: `@+=._-`
* `-t [roleType]` The role type, to see available roles: `alks iam roletypes`, be sure to wrap in quotes
* `-d`: Include default policies, defaults to false

Outputs the created role's ARN.

### `iam deleterole`

`alks iam deleterole` Deletes a previously created IAM role in the specified AWS account. Note this only works for IAM roles that were created with ALKS.

Arguments:

* `-p [password]` Your password
* `-n [roleName]` The name of the role, be sure to wrap in quotes, alphanumeric including: `@+=._-`

### `iam roletypes`

`alks iam roletypes` - List the available IAM role types.

Arguments:

* `-o [output]` Output format. Supports: `json`, `list`

Outputs a list of available role types.

# Output Formats

ALKS CLI will output in a variety of formats:

* `env`: Outputs Bash/Windows environment variable string. You can wrap this call in an eval:  `eval $(alks sessions open -d)`
* `json`: Outputs a JSON object
* `docker`: Outputs environment arguments to pass to a Docker run call
* `creds`: Updates the AWS credentials file
	* By default this will update the default profile, to use another named profile supply: `-n namedProfile`
	* If the named profile already exists you'll need to supply the overwrite flag: `-f`
* `idea`: Outputs environment variables formatted for Intelli-J
