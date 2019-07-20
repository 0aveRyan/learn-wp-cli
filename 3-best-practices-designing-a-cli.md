# Best Practices Designing A Command-Line Interface with WP-CLI

When crafting WP-CLI Commands, these are some best practices to make your commands resilient, professional-grade and the best they can be.

## Commands Should Do ONE Thing

Commands should aim to accomplish one task with flexability and grace.

If you need to make a command that does multiple things, consider making multiple discrete commands and then a wrapper command that executes those commands. Not only are these easier to test and debug, but easier to evolve in the future.

## Commands Should Be Verbose

Ideal WP-CLI commands find a balance between being overly-descriptive and too quiet.

If commands overwhelm a user with unnecessary information, they're more likely to read in a quick scan (if even) and perhaps miss important information. If commands don't provide enough necessary information, users can be impatient and distrustful.

Each time you introduce conditional logic or run an operation that could error-out or timeout like an API call or database query, consider adding a `WP_CLI::log()`.

Writing descriptive logs is the equivalent of adding loading spinners and notifications in a GUI -- they give users clarity and confidence in the application. Logging also makes debugging a heck of a lot easier.

## Design Commands for Human & Machine Usage

WP-CLI comes with helpful interactive tools like `WP_CLI::confirm()` and adding more advanced interactivity can really help your commands shine.

Interactivity is great for humans, but not fun for machines or automation.

Anytime you throw up an interactivity hurdle, make sure there is a way to set the value with an associative argument and set a flag to disable prompts.

## Using Consistient, Parallel Language Construction

This is a linguistic concept, not a technical concept.

Naming things is hard.

It's fine to have a preference on `create` vs. `add` vs. `new` vs. `insert` -- just stick with your preference and it's parallel.

You start, then stop (like a car).
You begin, then end (like a book).
You connect, then disconnect.
You login, then logout.

Don't intermix things like: connect + stop, start + end, etc.

## Don't Reinvent The Wheel

There's lots of "prior art" out there for CLI's. Try to use commonly-used language instead of inventing your own -- it will help people learn your CLI faster!

So if `clone` is a natural fit, don't name something `replicate` or `materialize-duplicate`.

If `start` is a natural fit, don't name something `powerup` or `lets-get-this-party-started` -- be cute inside your command, not in the naming of it.

Also consider the audience -- if your teammates are the primary audience, keep in mind other CLI tools you already share and if there's sensible command keywords to adopt.

## Brevity & Predictability Are Vital

WP-CLI discourages brevity for the sake of brevity -- clarity is more important than saving a few chars ;-)

Verbosity is good! Except when it's not. Brevity is good! Except when it's not.

Aim for _clarity_.

Aim to avoid abbrevs and words with multiple meanings like:
* arr
* obj
* opts
* str
* atts
* meta or metadata

Also watch out for jargon-y or overly-verbose command names.
* Don't include data types in command names (i.e. `wp process-billing-objects`) -- that's what type docs are for!
* Don't include API names in command names (i.e. `wp optimize-billing-post-meta`) -- that's what descriptions are for!
* Don't describe what the command is doing (i.e. `wp http_post_to_billing_api_and_store_processed_results`) -- that's what descriptions are for!

For the above, consider alternatives like `wp billing process`, `wp invoices fetch` or `wp invoices optimize`.

## Reuse Internal Commands Where Possible

Frankly, some of my favorite Custom WP-CLI Commands I've built are just callbacks using tons of `WP_CLI::runcommand()`'s without anything custom.

You don't need to invent magic to create magic!

Whenever possible, try to use a WP-CLI command over a WordPress Core function, so your command doesn't error out when WordPress isn't installed yet or is undergoing a fatal error.

## Prepare Responses In Multiple Data Formats

Many core commands use the `--format` flag and internal helper utility to return data in a table, CSV, YAML, JSON or a list of IDs. This greatly aids in their composability.

## Flags and Associative Args Are Generally Better Than Positional Args

For the same reason a single array of args is better for a WordPress Hook than defining multiple positional variables, try to stick to flags over positional args.

Nothing is more frustrating than knowing keys for data you're manipulating, but not remembering the order of arguments.

If you want to use a positional argument, allow for it to fail gracefully and check for an associative argument as well before falling back on a default value.

## Consider Scale Issues

Make sure you test your commands on large datasets, not just a simple install with `Hello World!` and a few dozen options.

* Don't use unbounded queries.
* Throw up confirmation dialogs before running an operation that could lock-up a large database.
* Consider whether an operation should be run immediately, tied to WordPress Cron or a true task queue system.

## Potentially Destructive Commands Default to Dry-Run

Humans make mistakes. We think `deploy` and type `destroy`. We miss a semicolon and bring down all of AWS.

Core WP-CLI commands have an optional `--dry-run` mode, however in my own commands I often prefer the default to use dry-run if the operation is potentially destructive or dangerous. It's kind of like "Undo Send" in Gmail.

Adding a tiny bit of friction can save you from yourself. Add too much friction and you'll just `return` through everything out of habit and still end up destroying everything.

So try to find a balance between being annoying and being helpful.

## Long & Short Flags

While mostly a bonus, having short flags and long flags that both do the same thing is super helpful.

If you're trying to skip loading Plugins, Themes and WP-CLI Packages `--skip-plugins --skip-themes --skip-packages` is a fair number of keystrokes for "skip them things."

`--sp --st --spkg` or even `--skip-all` would be too much appreviation/obfuscation if they were the only args available.

But if they were secondary triggers in addition to the verbose versions, it gives the best of both worlds.

Not every command needs long and short variants -- muddies docs -- but keep your eyes out for opportunities where having a select few short equivalents would be helpful.
