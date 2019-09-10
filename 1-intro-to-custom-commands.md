# Introduction to WP-CLI Custom Commands

> Any sufficiently advanced technology is indistinguishable from magic. - Arthur C. Clarke

For a long time, I used WP-CLI and thought "the core commands are _so powerful_, they must be _so challenging_ to create. 

In reality, working on WP-CLI commands is more like creating an electrical circuit than a rocket engine. Which isn't to say it's always "quick" or "easy," but if you think it's magic you may be surprised... and it could very well be quicker and easier than you expect.

However, even with the knowledge that building commands isn't rocket science, it's easy to psyche yourself out: _I don't have a genius idea for a custom command to change the world, my products or even my workflow..._

### Some benefits of creating Custom WP-CLI Commands:
* A set of tools accessible from anywhere and easily shared with others.
* Create powerful tools without the overhead and distraction of designing and maintaining GUIs.
* Aim to mitigate human error. Chaining and composing commands can be super powerful -- also super dangerous. A typo can have disasterous consequences, particularly in production. Putting long chains and compound commands inside a custom commands can give you the confidence to run otherwise dangerous operations or sequences with extra validation and shorter strings to type.
* Add technical value for your technical customers.

### Some ideas for when to make custom WP-CLI commands:
* Setup WordPress with baseline dependencies and settings (local, staging or production environments).
* Custom import/export tool for a product.
* Custom diagnostic tool that scans for errors
* Custom tools to connects your data/files with an internet service or API.
* A custom scaffolding tool for creating Plugins, Themes, Blocks or WP-CLI Packages from standardized templates.
* CRUD operations for interacting with a product.
* Setting WordPress/an extending product to a specific state, perhaps with dummy content or to step 5 of 12 in a complex flow.
* Flows for common issues that fill your help desk system.
* A simple wrapper around a `wp option update` command, so users don't need to remember the option key, run a `wp option list` and scan a large list of options to find the key and so you can run validation prior to injecting an value. (i.e. `wp my-coming-soon disable` could run `wp option update my_cs_is_enabled 0`)

## Custom Commands Can Live in Plugins or WP-CLI Packages

Custom WP-CLI Commands can be registered in three locations:
* A traditional WordPress Plugin
* A must-use WordPress Plugin
* A WP-CLI Package

WP-CLI Packages are effectively a WP-CLI-only plugin that can be installed from a Git repo or an the official WP-CLI package repository (that official repository is now closed to submissions). 

Packages are nice because WP-CLI uses Composer to autoload and handle code updates!

#### Both Plugins and Packages have Pros & Cons

Plugins are great because custom commands can be updated by pushing a Plugin update, so they're often the best option if you're trying to distribute a CLI with a custom Plugin or Theme to environments you don't control.

Packages require `wp package update` to be run for updates to deploy. If you control your environments, you can run this as-needed or on a system cron job.

But Packages are great because they don't clutter your Plugins screen, prevent users from disabling unless they have permission to use WP-CLI and commands in packages can be run without WordPress already installed (so long as they don't require the WordPress database and Core functions).

## Registering A Custom Command

Before you name and register a new command, it's important to consider a few things:
1. Command name strings are a global namespace -- make sure your name string is unique.
   1. Run `wp` on your system and see what Core/Custom commands are used (your host may add their own commands).
   2. Particularly if you're distributing a product with custom commands, check [this list](https://make.wordpress.org/cli/handbook/tools/) **_(not exhaustive!)_** and search the web for "wp-cli wp [name]" to try to avoid collision.
2. Do you need an entirely new namespace? If you're building a generic custom command, say to scaffold a React-based UI for the WordPress Admin, it may make sense to register a new subcommand under `wp scaffold` -- instead of `wp scaffold-react-admin` you could put under `wp scaffold react-admin`.

Now, it's time to get building!

### WP_CLI::add_command()

Using a static method on the `WP_CLI` class called `add_command()` you can register new top-level commands and subcommands.

When using `WP_CLI::add_command()`, you first need to check if the `WP_CLI` class is available, otherwise you can get some nasty errors.

You can either check the class is available before using the static method or hook your code to load on `cli_init` action, so it only instantiates if WP-CLI is available.

```php
// used anywhere
if ( ! class_exists('WP_CLI' ) ) {
    return;
}

WP_CLI::add_command()
```

```php
// safe to directly use `WP_CLI` without checking if the class exists
add_action( 'cli_init', 'my_custom_command' );
function my_custom_command() {
    WP_CLI::add_command();
}
```

`WP_CLI::add_command()` has two required arguments and a third, optional argument that accepts a configuration array.

```php
WP_CLI::add_command( $name, $callable, $args = array() );
```

##### $name _(string)_
This argument is where the keyword name is set used to trigger your command.

```php
/** 
 * You can register one callback that handles subcommands, register each subcommand separately or hook into an existing core command.
 */ 
WP_CLI::add_command( 'magic', $callback_handles_start_and_stop );
// OR
WP_CLI::add_command( 'magic start', $callback_handles_start );
WP_CLI::add_command( 'magic stop', $callback_handles_stop );
// OR EXTEND CORE COMMANDS LIKE 'scaffold'
WP_CLI::add_command( 'scaffold magic', $scaffold_magic );
```

##### $callable _(callable)_
This argument can be a function, a closure function or a class.

##### $args _(array)_
This optional arguments array can be used to define a WP-CLI Hook to execute on, trigger code to invoke before/after the command fires and has string description fields as an alternative to PHPDoc to populate the `help` command results.

* _(callable)_ $before_invoke Callback to execute before invoking the command.
* _(callable)_ $after_invoke Callback to execute after invoking the command.
* _(string)_ $shortdesc Short description (80 char or less) for the command.
* _(string)_ $longdesc Description of arbitrary length for examples, etc.
* _(string)_ $synopsis The synopsis for the command (string or array).
* _(string)_ $when Execute callback on a named WP-CLI hook (e.g. before_wp_load).
* _(bool)_ $is_deferred Whether the command addition had already been deferred.


### Writing Command Callbacks

In the next few chapters I go into more detail about _what_ to do in your callback and WP_CLI's internal API and helper functions, but first we need to talk about what happens to `$callable` from above.

No matter whether you're passing a function name, qualified class name or closure, in the end you'll end up with a function that's passed two arguments -- both arrays that default to empty.

```php
/**
 *
 * @param array $args - Positional Arguments
 * @param array $assoc_args - Flags & Associative Arguments
 */
function( $args, $assoc_args ) {
    // doing command stuff
}
```

#### $args
These are your **Positional Args** -- if you have any. The first positional argument would be `$args[0]`, the second would be `$args[1]`, etc. 

_Remember to use good code hygene and use `isset()` or `! empty()` to check for a value before trying to access it._

#### $assoc_args
These are your **Associative Args** -- if you have any. The flag `--bam=pow` would be `$assoc_args['bam'] = 'pow'`. _Note that you don't need the double-dash in the key string._

However, instead of running an `isset()` or `! empty()` check and accessing directly, I recommend using the utility function to extract associative args, because they can be negated using `--no-quiet`.

```php
// to use, pass the entire array of $assoc_args and use $flag to set your key
WP_CLI\Utils\get_flag_value( $assoc_args, $flag, $default = null)

// example - If 'bam' is set to 'pow', $flag = 'pow'. If not set $flag = 'smack'.
$flag = WP_CLI\Utils\get_flag_value( $assoc_args, 'bam', 'smack' );
```

### Notes on Code Structure

I prefer the flexibility that PHP classes offer over functions.

When a qualified class name is passed for `$callable`, any `public function` in the class immediately becomes a subcommand.

```php
WP_CLI::add_command( 'cool', 'My_Cool_Command' );

class My_Cool_Command {
    public function start( $args, $assoc_args ) {
        // 'wp cool start' will execute this method
        $this->helper();
    }
    public function stop( $args, $assoc_args ) {
        // 'wp cool stop' will execute this method
        $this->helper();
    }
    protected function helper() {
        // this method isn't registered as a command
    }
}
```

Another benefit of using classes is that its easier to reuse code, map multiple subcommands to the same callback and organize code in general.
```php
WP_CLI::add_command( 'cool', 'My_Cool_Command' );

class My_Cool_Command {
    /**
     * Note use of __invoke(). __construct() won't work!
     */
    public function __invoke( $args, $assoc_args ) {
        $subcommand = ! empty( $args[0] ) ? $args[0] : '';
        switch( $subcommand ) {
            case 'start':
            case 'begin':
                $this->start( $args, $assoc_args );
                break;
            case 'stop':
            case 'end':
                $this->stop( $args, $assoc_args );
                break;
            default:
                WP_CLI::warning( '' );
                break;
        }

    }
    protected function start( $args, $assoc_args ) {
        /**
         * wp cool start OR wp cool begin
         */
         $this->helper( $assoc_args );
         // now do start stuff
    }
    protected function stop( $args, $assoc_args ) {
         /**
         * wp cool stop OR wp cool end
         */
         $this->helper( $assoc_args );
         // now do stop stuff
    }
    protected function helper( $assoc_args ) {
        // this method is shared by the command functions above
        // it's not registered because it's 'protected'
    }
}
```

If you're developing multiple commands, you may want to re-use helper methods across multiple classes, another place where using classes over functions really shines.

```php
class My_CLI_Commands {
    function my_helper( $input ) {
        // do stuff to $input
        return $input
    }
}

class Cool_Command extends My_CLI_Commands {
    public function start( $args ) {
        $my_input = ! empty( $args[0] ) ? $args[0] : 'default-value';
        $parsed = $this->my_helper( $my_input );
        // do stuff
    }
}
```

## Understanding WP-CLI Hooks

WP-CLI hooks are quite similar to WordPress Action Hooks.

Hooks control when the code is executed:
* Before/After a specific command.
* Before/After WordPress is loaded (Ex. you cannot use `wp core download` inside a custom command unless you use `before_wp_load`).
* Before/After the wp-config.php is loaded.

Defining a Hook for a command is done during `WP_CLI::add_command()` registration by specifying the `when` parameter in the $args array.

```php
WP_CLI::add_command(
    'magic',
    function() {
        WP_CLI::success( "It's magic!" );
    },
    array(
        'when' => 'before_wp_load'
    )
);
```

### Available WP-CLI Hooks
* before_add_command:<command> – Before the command is added.
* after_add_command:<command> – After the command was added.
* before_invoke:<command> – Just before a command is invoked.
* after_invoke:<command> – Just after a command is invoked.
* find_command_to_run_pre – Just before WP-CLI finds the command to run.
* before_wp_load – Just before the WP load process begins.
* before_wp_config_load – After wp-config.php has been located.
* after_wp_config_load – After wp-config.php has been loaded into scope.
* after_wp_load – Just after the WP load process has completed.
* before_run_command – Just before the command is executed.

## Using PHPDoc to Register In-CLI Help Documentation

Perhaps the most magical feature of WP-CLI is how it slurps PHPdoc comments above classes and functions to magically build the `help` results for commands.

No one likes going to code to check arguments -- so add simple PHPdoc comments.

The best artists steal, so I recommend copying + pasting an existing PHPdoc **(just make sure to edit/cut it down properly!)**.

There's great boilerplates on the [WP-CLI Documentation Standards](https://make.wordpress.org/cli/handbook/documentation-standards/) page.

## Why a custom command when I can use a shell script/alias or the WP-Admin?

### Advantages over `my-awesome-script.sh` & shell aliases
* Shareable with others and accessible no matter the machine you're using.
* Remotely installable with `wp plugin install` and `wp package install`
* Easier to write defensive checks in PHP than in Bash -- so more likely you'll do it and write safer, more resiliant code.
* Commands are less-likely to run into file permission/executable issues.
* More likely commands are in Git AND versioned than a random cowboy or config file.
* Easier to execute both PHP and shell commands together.
* Execute on a hook, to properly time the execution of your code with something happening elsewhere in WordPress.

### Advantages over WP-Admin Interface
* Expectation of a certain level of technical know-how from user consuming code.
* Less worry about scoping user privleges, nonces, etc.
* No need to write and maintain HTML, CSS or JavaScript.
* No obsessing over design decisions and browser inconsistiencies when the goal is a technical feature.
* No waiting for page refreshes.
* Reuse other powerful WP-CLI commands within your command that go beyond WordPress Core functions.
* Command is composable with other commands during execution.

### Multiple Ways To Do The Same Thing

One of the great things about WP-CLI is there is often a few ways to do the same thing. 

##### Example: Updating a database option

* `wp option update blogdescription "WP-CLI is awesome!"`
* `wp eval 'update_option( "blogdescription", "WP-CLI is awesome!" );'`
* `wp db query 'UPDATE wp_options SET option_value="WP-CLI is awesome!" WHERE option_name="blogdescription";'`
