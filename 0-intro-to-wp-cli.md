# Introduction to WP-CLI

The WordPress Command Line interface is a robust tool for interacting with WordPress sites.

WP-CLI uses a text-based syntax of commands (combinations of words, numbers, abbreviations and punctuation) to interact with a WordPress site instead of the visual WP Admin interface in a web browser.

Using a Terminal app (a.k.a "shell") you... 
1. Change directories (`cd`) the folder where your WordPress site is installed
2. Type a command using WP-CLI's syntax _(i.e. `wp maintenance-mode activate` )_
3. Hit `return`
4. When the command is done running, WP-CLI will show a written response explaining what operations it ran or errors it encountered.
    
This interaction is kind of like sending a text message to your favorite restaurant... **`subscribe`** to receive coupons (receiving an text back saying _"Great! We'll send you discounts. Type STOP to unsubscribe."_) or **`stop`** (receiving _"Okay, you've unsubscribed from sweet discounts."_). The system on the other end is designed to run operations based on receiving specific commands.. 

In WP-CLI, instead of SMS commands you'd text to a restaurant like `subscribe`, `stop`, `hours` or `reservation`, commands map to different features of WordPress like `post`, `option`, `maintenance-mode` and `user` -- there are 44 commands that come with WP-CLI out-of-the-box, many with subcommands and flexible options.

At the end of the day, WP-CLI is just a virtual assistant -- kinda like Slackbot or Siri -- you can direct message with to manage WordPress sites.

## Getting Started with WP-CLI

### WP-CLI is likely already installed on your web host

The [vast majority of WordPress hosts](https://make.wordpress.org/cli/handbook/hosting-companies/) pre-install WP-CLI on their servers, so it's ready-to-use whenever you're ready. It's also bundled with popular local development tools like VVV, DesktopServer, Lando and Local by Flywheel. _(Using MAMP? Here are [some good instructions](https://tommcfarlin.com/installing-wp-cli-with-mamp/) for installing WP-CLI).

### Checking for WP-CLI and where to start

To start using WP-CLI, you'll likely need to connect to your webserver or virtual server using SSH.

1. Sign-in to your server.
2. `cd` into the WordPress root directory _(the folder with `/wp-admin`, `/wp-content`, etc)_.
3. Type **`wp`** and press **`return`** on your keyboard.
4. If WP-CLI is installed correctly, you should see the help guide with a list of the available commands on the server.
5. Type **`q`** to quit the help documentation.

![example of WP-CLI help readout](assets/wp.gif)

## Commands & Subcommands

At the center of WP-CLI are **Commands**.

At time of writing, there are 44 top-level commands included with WP-CLI to run operations on data in a WordPress database, generate WordPress starter code and more!

Commands are pretty easy to identify -- they're the first word following `wp` in a command string, such as `wp `**`post`**. 

When commands are registered with WP-CLI, this keyword string is mapped to a callback to execute when that command is used. This mapping is similar to an automated phone tree, such as the one from your bank:

> Press `2` for account information, press `3` to file a claim, press `0` to speak to an operator...

Where WP-CLI is more like...

> Type `wp post` to work with Posts, type `wp option` to work with WordPress Options and type `wp help` to learn about available commands...

**Subcommands** are registered beneath a primary command, such as `wp post `**`delete`**. Subcommands are commands that aren't top-level (top-level commands are immediately right-adjacent to the `wp` keyword), but otherwise have all the same potential features as a top-level command.

It's not a hard rule, but in general the primary command is a _thing_ (post, user, cron, config etc) while subcommands are _operations_ that run on that thing (create, delete, start, install, etc).

It's also important to note that while `wp post create` and `wp user create` both have a `create` subcommand, these are two _distinct_ subcommands with different arguments and functionality.

## Positional Arguments, Associative Arguments and Flags

Many WP-CLI commands have required or optional arguments that change how the commands work, scope what the operation will modify and how the command will return a response.

### Positional Arguments

Positional arguments are values that immediately follow the command/subcommand keyword, separated by a space. The order of positional arguments matters, hence the name positional.

_Ex. `wp post delete `**`100`** -- `100` is the **first** positional argument following the `delete` subcommand. In a simple, top-level command like `wp do-something `**`my-plugin-name`**, `my-plugin-name` isn't a subcommand, it's the first positional argument.`wp do-something-else subcommand arg1 arg2 arg3` is an example of multiple positional arguments on a subcommand._

### Flags

Flags are similar to checkboxes in a visual interface -- they're used to denote true/false and are used to enable or disable part of a command's functionality. They're called flags because their syntax looks like a flag pole the word is "flying from" with two preceeding dashes.

_Ex. In `wp post delete 100 `**`--force`**`, --force` is the flag._

### Associative Arguments

Associative arguments are values that are associated with a string key. These key-value pairs that start with a flag syntax, but an `=` is used to set a value. Unlike positional args, associative args are always optional and can be used in any order. Associative is often abbreviated `assoc` in code.

_Ex. `wp post delete 100 --method=via-sql-query`_

## Global Arguments

At writing, there are 13 global arguments that can be used with any command included in WP-CLI or a Custom Command.

All global arguments are associative args, as they're all optional.

Two critical clobal arguments to be aware of, that greatly expedite learning and use of WP-CLI are:

* `--prompt` - Creates an interactive prompt that asks a user for values for each positional argument, associative argument and flag, meaning **you don't need to memorize argument keys OR the order of arguments**. Hitting `return` with an empty value in `--prompt` will fallback to the default value for the argument, as if you didn't type it.
* `--help` - Will return the documentation for any command or subcommand. `wp post --help`, `wp post list --help` and `wp maintenance-mode --help` will all return different help responses. **No need to go search the web for information about commands!**

## Remote Management & WordPress Multisite

### Using WP-CLI's built-in Remote Management Features

Ready for inception? You can use one install of WP-CLI to remotely control another!

This means you can use WP-CLI running on your local development environment to control your remote production and staging environments -- so long as WP-CLI is installed on both remote servers. This can be done using the `--ssh` or `--http` global arguments, or better yet using a `wp-cli.yml` configuration file where you identify an environment with an alias.

A common use case is mapping aliases for staging and production environments to a local development environment where WP-CLI is installed.

Another practical use is for a developer or agency that manages dozens or hundreds of sites. Creating a YAML config file on a local environment that points to all the remote servers for clients can make things like updating plugins for all clients a breeze. The config allows an alias to be set like "client1" or "candy-store". Note that "all" is a reserved alias -- using @all will run the command against the current environment and all remotes registered in the config file.


```bash
wp @candy-store plugin update --all
```

```bash
wp @staging plugin update --all && wp @production plugin update --all
```

```bash
wp @all plugin update --all
```

### Working with WordPress Multisite

By default, all WP-CLI commands on a Multisite installation are run against the *first* site in the network.

To run a command against another site, the global argument `--url` must be set -- i.e. `wp post delete 100 --url=https://site2.com` -- otherwise WP-CLI will try to delete Post ID 100 from the primary site in the network, so please use extra caution when running commands for Multisite installs.

**NOTE:** This must be the _fully-qualified Site URL_, not the numerical Site ID.

## Chaining Commands Together

Commands don't run in parallel, so they can be chained together and expected to run sequentially.

```bash
# Update All The Things
wp cli update && wp core update && wp theme update --all && wp plugin update --all
```

One risk of running commands in a sequence like this, is that if one command errors-out, subsequent commands are never run. So if you set a cron job to "update all the things" each day, but each day the WP-CLI update command fails... none of the other commands will run... so when chaining commands that aren't related like above, consider the order of importance and put the most stable/reliable commands first.

## Composing Commands Together

Some of the real power of WP-CLI is composing commands together in a sequence where subsequent commands use the dynamic response(s) from preceeding commands. 

By making commands that focus on doing one thing, the results of one command can be passed into arguments for another command, making them much more versitile and composible.

There are two primary ways to compose commands together:
1. Using a subshell to use the results of a command inside another
1. Using `xargs` to pass response items to the second command.

In general, I find the `xargs` syntax easier on the eyes, but there isn't a right way to do it, and sometimes there are benefits to subshells and loops.

### Scenario: Delete All Drafts For a Specific User

The WordPress Admin is a fairly powerful interface, but it does have limitations. Using the filters on the All Posts screen, it's possible to get a list of all draft posts or posts by a specific author, but not a list of all drafts by a specific author.

On a news website, a user might have thousands of posts and hundreds of drafts, which would make cleaning drafts out for that user an exhaustively manual task in the WordPress Admin.

Enter WP-CLI!

##### Using a subshell
```bash
wp post delete --force $(wp post list --post_status=draft --post_author=1 --format=ids)
```

##### Using `xargs`
```bash
wp post list --post_status=draft --post_author=1 --field=ID | xargs -n1 -I % wp post delete % --force
```

### Scenario: Add User Privileges to All Sites In a Multisite

Management of a WordPress Multisite is one area where WP-CLI really shines. Where WP-CLI will often save minutes on a task for a single site, it can save hours for a task that needs to happen in a large Multisite.

When I worked in publishing, one frequent task would be granting a new editor privileges on each site within a Multisite that contained hundreds of sites (or de-escalating their privileges when they left the company). Unlike a super-admin user, an editor-level user needs to get opt-in access to each site from an administrator.

In this example, we're looping over all sites in a multisite, and setting user "1" as an editor, with a custom capability called `custom_homepage_builder`.

##### Using For Loop
_(Here we use a for loop instad of using a subshell so we can echo-out the name of the site before setting user privleges, so the output can be copied and saved for reference.)_
```bash
for url in $(wp site list --format=csv --fields=url | tail -n +2); do echo "$url: "; wp user set-role 1 editor --url=$url; wp user add-cap 1 custom_homepage_builder --url=$url; done
```

##### Using `xargs`
```bash
wp site list --fields=url | xargs -n1 -I % wp user set-role 1 editor --url=% && wp user add-cap 1 custom_homepage_builder --url=%
```
