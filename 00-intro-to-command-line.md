# Intro to Command Line (for new WP-CLI Users)

Adopting new tools can be intimidating. 

Whether this is your first venture into a Terminal application or it's just well outside your comfort zone, remember it takes time to get comfortable with new software -- put on your thinking cap and be patient, curious, careful and confident you can figure it out!

There's a lot of lingo to learn using a CLI. Shells, streams, SSH, strange symbols and a sea of acronyms...

This guide is designed to get you going with the command line so you can get going with WP-CLI. 

This guide is far from comprehensive -- it's designed to get you going with WP-CLI, but in reality you don't need to be a CLI expert to make great use of WP-CLI or build custom WP-CLI commands using PHP. 

Check out the resource section at the end to learn more about the Terminal and Command Line interfaces!

This guide is meant for Linux/Unix based systems (which includes MacOS, ChromeOS, Ubuntu, etc). While it's possible to host WordPress on Windows, most WordPress servers run Unix-like operating systems.

#### Note for Windows Users

If you're a Windows user, this guide is likely still useful to you! Often when you're running WP-CLI, you'll be remotely-connected to an internet server or local, virtual server on your Windows machine that's running one of these Unix-like operating systems. Windows 10 also offers a system called Bash on Ubuntu on Windows (yes, a mouthful) which lets you use native Linux commands on your Windows machine.

### Why Learn The Command Line
* More flexibility 

### Under The Hood: `STDIN`, `STDOUT` & `STDERR`

Most modern programming languages like PHP, JavaScript and Python can be used to create a CLI application -- just as these languages can be used to generate HTML documents for a web browser.

In a command line, these languages wrap three underlying application streams to create the CLI:

1. **STDIN** - Standard Input
2. **STDOUT** - Standard Output
3. **STDERR** - Standard Error

Think of these technologies as the conveyer belts your data is carried on -- carrying data into your application, out of your application and a special error handler that stops the other belts and tells you the error for why they stopped.

When you type a command and hit `return`, you're sending that input to `STDIN`. Then, as processes are reporting progress or completion, they'll return output to `STDOUT`. If something goes wrong, your application will send errors to `STDERR` and break out of their process.

_**It's not super important to understand these underlying pieces deeply to use WP-CLI or write Custom WP-CLI Commands with PHP**_.

### Users & Directories

The are two critical concepts to understand while you're using a CLI in a Terminal window:

In a terminal session...
1. ... **you're always using a user account** with varying permission rights to read or write to parts of the filesystem.
2. ... **you're always inside a folder (aka directory)** somewhere on the filesystem.

When you open a Terminal window on a Mac or Linux machine you'll see something like this:
```bash
username@hostname:directory[separator]
```

In my case, this looks like...
```bash
dryan@mbp:~$
```

* `dryan` is my system user's username (which differs from my user's display name of `David Ryan`)
* `mbp` is the name of my host machine
* `~` is a symbolic shorthand for my user's directory (`/Users/dryan`)
* `$` is the "prompt" part of "command prompt" that separates your command from  lets you know the Terminal is ready for input (`>` on Windows) and has a cursor following it.

We'll get into more details below about how to navigate the filesystem, but for now know whenever you see that string, it means your terminal is awaiting input and no processes are running.

### Navigating Directories & Common Commands

As mentioned above, when you first open a Terminal window you are in your user's Home Directory.

* `~` is short for the user's Home Directory (i.e. `/Users/dryan`)
* `.` is short for the **current** directory.
* `..` is short for the directory **above** the current directory.
* `/` when at the beginning of a path stands for the root directory of the entire filesystem (elsewhere it denotes the separation between directories)

With paths, similar to URLs, there are **relative paths** and **absolute paths**. 

This is best explained by introducing the `cd` command, which is used to _change directory_.

Example: Changing into a folder called `playground` in my user's directory would be:
* `cd playground` but only if I'm in the root of my home directory. This relative path is relative to your current directory.
* `cd /Users/dryan/playground` from anywhere (including my home directory). This absolute path can be used anywhere and doesn't take the current directory into account.

As directories change, you'll see your prompt prefix change as well:
```bash
dryan@mbp:~/playground$
```

Then, if I wanted to "step up" one directory to return to my user directory, I could run...
```bash
cd ..
```

Or, if I wanted to step up two directories into the `/Users` directory from `/Users/dryan/playground`, I'd run

```bash
cd ../..
```

**TIP:** For a long directory name that's inside the current directory (i.e. `my-super-long-folder-name`) you can type the first few characters `cd my-` and hit the **`TAB`** key to auto-complete the name without typing it all out. This also works with absolute paths as-you-go:
* `cd /Users/dr` + **`TAB`** (to get `dryan`), and then...
* `cd /Users/dryan/my-` + **`TAB`** (to get `my-super-long-folder-name`)

**macOS TIP:** For a long directory path, you can type `cd ` (note the space) and then drag-and-drop a folder from Finder into the terminal window, which will insert the absolute path to that directory. It can save a ton of time over typing a path like `cd /Users/dryan/Sites/VVV/wordpress/public_html/wp-content/plugins/src/scss`.

#### Common Commands & Operators
* `cd` - Change directories
* `ls` - List files
* `touch` - Create a file
* `mkdir` - Create a folder
* `rm` - Remove a file or folder
* `&&` - Chain commands together

##### Some examples...

###### Change into the themes directory and make a folder called custom
```bash
cd wp-content/themes && mkdir custom
```

###### Change into the plugins directory and force remove Hello Dolly, and recursively remove any subfolders and nested files inside hello
```bash
cd wp-content/plugins && rm -rf hello
```

###### Change up one level from the current directory and make an empty file called LICENSE.md
```bash
cd .. && touch README.md
```

###### Change into the public_html folder (where WordPress lives) and list files, including hidden system files/folders that begin with a period
```bash
cd ~/public_html && ls -a
```

### Understanding Remote Connections with SSH

SSH stands for Secure Shell and works like a remote desktop for the Terminal.

SSH lets you connect to a shell session on a remote server with credentials. The server could be elsewhere on the internet, or simply a virtual server running on your local OS like VVV or Lando. 

To access WP-CLI you will often need to "ssh in" to your server to begin running commands.

To use SSH you'll need:
1. A **username** for a user on the remote server
2. A **hostname** for pointing your computer at the server (either an IP address or domain like something.mydomain.com)
3. A **password or key file** used to authenticate as that user.

(You may also need a port number, if the default SSH port 22 is blocked)

```bash
dryan@mbp:~$ ssh dave@192.168.32.145
Password: **********

dave@192.168.32.145:~$
```

If you've set a default User and key file in your ssh config, you can...
```bash
dryan@mbp:~$ ssh 192.168.32.145

dave@192.168.32.145:~$
```

A few things to keep in-mind when you're SSH'd into a remote server:
* You don't have access to command history from your local system.
* You don't have access to any aliases in your local system's `~/.bash_profile` -- you have your remote user's `~/.bash_profile`
* You only have access to software installed on the server, not on your local machine _(i.e. you may be able to `npm install` on your local machine because you installed `npm` on there, but that command may not work on your remote server unless `npm` is installed there.)_
