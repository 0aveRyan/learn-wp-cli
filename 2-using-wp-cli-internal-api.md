# Using WP-CLI's Internal API & Utility Functions

## Introduction

WP-CLI isn't just a robust tool, it has robust and reusable internal utilities that greatly expedite building WP-CLI commands just as powerful as the Core commands.

The official documentation is fantastic and extensive, but if it has one shortcoming it's that it's organized alphabetically instead of assigning a hiearchy of importance or grouping like items.

This chapter aims to surface some of the most commonly-used functions and utilities.

## Key API Functions

### `WP_CLI::log()` and `WP_CLI::line()`

Just as `echo` is used to output strings to an HTML document, it can be used to output strings in a CLI application. Both these methods can be used to output strings to a new line.

In general, `WP_CLI::log()` is preferred to `WP_CLI::line()` for a few reasons.

While _very_ similar methods, `WP_CLI::log()` is more resilient and will respect the `--quiet` flag.

`WP_CLI::log()` also allows users to swap the default logger with a custom one, so try to use it over `WP_CLI::line()` or a simple `echo`.

### `WP_CLI::success()`, `WP_CLI::warning()` and `WP_CLI::error()`

These methods are very straightforward -- they're mostly just `WP_CLI::log()` but with colored prefixes like `Success: $msg` in green, `Warning: $msg` in orange and `Error: $msg` in red.

Both success and warning messages are sent to `STDOUT`, so subsequent code in your callback will be executed. However, error messages are sent to `STDERR` and it **will also stop the command from running.**

Another consideration is you'll commonly use `WP_CLI::error()` during validation of arguments. If a user didn't include a required parameter, or included an invalid value, don't just say `You forgot X, stopping everything.`! Instead, help guide the user with `X parameter is required. Try "wp cmd X" with "Y" or "Z."`

### `WP_CLI::confirm()`

The confirm method is also similar to `WP_CLI::log()` in that you pass a message string (a yes/no question) and it stops the callback from continuing until a user types `y`/`yes` or `n`/`no`.

##### Ex. This callback...
```php
function my_command( $args, $assoc_args ) {
  WP_CLI::confirm( 'Are you ready to continue?' );
  WP_CLI::success( 'Did it!' );
}
```

##### Will show...
```bash
$ wp my_command
Are you ready to continue? [y/n]: y
Success: Did it!
```
If you type `y` above, you'll see `Success: Did it!`, otherwise you'll break out of the callback and see nothing further after the question.

### `WP_CLI::colorize()`

Pops of color can add clarity, branding and flair to a command. This method uses PHP CLI Tool's color tokens to style a string.

When styling a string, you need a beginning token and "reset" token to return to standard output. Tokens can be used to set both a background color and a text color. A full reference of these tokens is available here.

##### Important Note
Whenever you're using colors, remember that Users can customize their Terminal themes and may have a black, white or color background. If you're not careful, your text may be illegible or invisible. Because of this, I recommend using color sparingly and on non-critical text (i.e. branding of your company/product name). If you want to add color for a command you're distributing with a product or other developers will use, I recommend testing with a few Terminal themes ranging from light to dark backgrounds -- just as you'd test browsers for HTML.

### `WP_CLI::debug()`

One great way to find a balance between highly verbose output and streamlined output is stuffing data into `WP_CLI::debug()`.

`WP_CLI::debug()` is only shown when the `--debug` flag is used, and will write to `STDERR`.

One other benefit of `WP_CLI::debug()` is that it will append the script execution time to the output.

## Advanced API Functions

### `WP_CLI::error_multi_line()`

Handy error styler that can display multiple lines, wrapped in a red box. Unlike `WP_CLI::error()`, this method doesn't exit the script.

### `WP_CLI::add_hook()` and `WP_CLI::do_hook()`

Just like `add_action()` and `do_action()` in WordPress Core, `WP_CLI::add_hook()` lets you register a callback to a hook string and `WP_CLI::do_hook()` executes callbacks registered to a hook string.

### `WP_CLI::runcommand()`

Perhaps one of the most powerful methods in the internal API, `WP_CLI::runcommand()` can be used spawn a child process and run another WP-CLI command. Instead of `wp post list`, you drop the `wp` prefix and simply use `WP_CLI::runcommand( 'post list' )`.

Keep in mind, any `WP_CLI::error()` or other issue inside the command being wrapped will break out of the command where `WP_CLI::runcommand()` is being used.

_Also, remember to late-escape any variables you inject into the command string!_
```php
$format = 'csv';
WP_CLI::runcommand( sprintf( 'post list --format=%s', $format ) );
```
### `WP_CLI::launch()`

_Work-in-progress..._

## Key Utilitiy Functions

### `WP_CLI\Utils\format_items()`

This method is incredibly useful for outputting data in a variety of formats, including a CLI table, CSV, YAML and JSON.

### `WP_CLI\Utils\get_flag_value()`

Instead of accessing `$assoc_args` directly, use this method because flags can be negated with `--no-queit` or `--quiet`.

```php
// bad
function( $args, $assoc_args ) {
  $value = ! empty( $assoc_args['my-flag'] ) ? $assoc_args['my-value'] : 'default-value';
}
// good
function( $args, $assoc_args ) {
  $value = WP_CLI\Utils\get_flag_value( $assoc_args, 'my-value', 'default-value' );
}
```

### `WP_CLI\Utils\trailingslashit()`

_Work-in-progress..._

### `WP_CLI\Utils\get_temp_dir()`

_Work-in-progress..._

### `WP_CLI\Utils\get_home_dir()`

_Work-in-progress..._

### `WP_CLI\Utils\report_batch_operation_results()`

_Work-in-progress..._

## Advanced Utility Functions

### `WP_CLI\Utils\http_request()`

_Work-in-progress..._

### `WP_CLI\Utils\launch_editor_for_input()`

_Work-in-progress..._

### `WP_CLI\Utils\make_progress_bar()`

_Work-in-progress..._

### `WP_CLI\Utils\write_csv()`

_Work-in-progress..._
