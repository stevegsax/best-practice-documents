# Introduction to Nushell

Hello, and welcome to the Nushell project.
The goal of this project is to take the Unix philosophy of shells, where pipes connect simple commands together, and bring it to the modern style of development.
Thus, rather than being either a shell, or a programming language, Nushell connects both by bringing a rich programming language and a full-featured shell together into one package.

Nu takes cues from a lot of familiar territory: traditional shells like bash, object based shells like PowerShell, gradually typed languages like TypeScript, functional programming, systems programming, and more. But rather than trying to be a jack of all trades, Nu focuses its energy on doing a few things well:

- Being a flexible cross-platform shell with a modern feel
- Solving problems as a modern programming language that works with the structure of your data
- Giving clear error messages and clean IDE support

## This Book

The book is split into chapters which are further broken down into sections.
You can click on the chapter headers to get more information about it.

- [Installation](installation.md), of course, helps you get Nushell onto your system.
- [Getting Started](getting_started.md) shows you the ropes. It also explains some of the design principles where Nushell differs from typical shells, such as Bash.
- [Nu Fundamentals](nu_fundamentals.md) explains basic concepts of the Nushell language.
- [Programming in Nu](programming_in_nu.md) dives more deeply into the language features and shows several ways how to organize and structure your code.
- [Nu as a Shell](nu_as_a_shell.md) focuses on the shell features, most notably the configuration and environment.
- [Coming to Nu](coming_to_nu.md) is intended to give a quick start for users coming from other shells or languages.
- [Design Notes](design_notes.md) has in-depth explanation of some of the Nushell's design choices.
- [(Not So) Advanced](advanced.md) includes some more advanced topics (they are not _so_ advanced, make sure to check them out, too!).

## The Many Parts of Nushell

The Nushell project consists of multiple different repositories and subprojects.
You can find all of them under [our organization on GitHub](https://github.com/nushell).

- The main Nushell repository can be found [here](https://github.com/nushell/nushell). It is broken into multiple crates that can be used as independent libraries in your own project, if you wish so.
- The repository of our [nushell.sh](https://www.nushell.sh) page, including this book, can be found [here](https://github.com/nushell/nushell.github.io).
- Nushell has its own line editor which [has its own repository](https://github.com/nushell/reedline)
- [`nu_scripts`](https://github.com/nushell/nu_scripts) is a place to share scripts and modules with other users until we have some sort of package manager.
- [Nana](https://github.com/nushell/nana) is an experimental effort to explore graphical user interface for Nushell.
- [Awesome Nu](https://github.com/nushell/awesome-nu) contains a list of tools that work with the Nushell ecosystem: plugins, scripts, editor extension, 3rd party integrations, etc.
- [Nu Showcase](https://github.com/nushell/showcase) is a place to share works about Nushell, be it blogs, artwork or something else.
- [Request for Comment (RFC)](https://github.com/nushell/rfcs) serves as a place to propose and discuss major design changes. While currently under-utilized, we expect to use it more as we get closer to and beyond 1.0.

## Contributing

We welcome contributions!
[As you can see](#the-many-parts-of-nushell), there are a lot of places to contribute to.
Most repositories contain `CONTRIBUTING.md` file with tips and details that should help you get started (if not, consider contributing a fix!).

Nushell itself is written in [Rust](https://www.rust-lang.org).
However, you do not have to be a Rust programmer to help.
If you know some web development, you can contribute to improving this website or the Nana project.
[Dataframes](dataframes.md) can use your data processing expertise.

If you wrote a cool script, plugin or integrated Nushell somewhere, we'd welcome your contribution to `nu_scripts` or Awesome Nu.
Discovering bugs with reproduction steps and filing GitHub issues for them is a valuable help, too!
You can contribute to Nushell just by using Nushell!

Since Nushell evolves fast, this book is in a constant need of updating.
Contributing to this book does not require any special skills aside from a basic familiarity with Markdown.
Furthermore, you can consider translating parts of it to your language.

## Community

The main place to discuss anything Nushell is our [Discord](https://discord.com/invite/NtAbbGn).
You can also follow us on [Twitter](https://twitter.com/nu_shell) for news and updates.
Finally, you can use the GitHub discussions or file GitHub issues.
# (Not so) Advanced

While the "Advanced" title might sound daunting and you might be tempted to skip this chapter, in fact, some of the most interesting and powerful features can be found here.

Besides the built-in commands, Nushell has a [standard library](standard_library.md).

Nushell operates on _structured data_.
You could say that Nushell is a "data-first" shell and a programming language.
To further explore the data-centric direction, Nushell includes a full-featured dataframe processing engine using [Polars](https://github.com/pola-rs/polars) as the backend.
Make sure to check the [Dataframes documentation](dataframes.md) if you want to process large data efficiently directly in your shell.

Values in Nushell contain some extra [metadata](metadata.md).
This metadata can be used, for example, to [create custom errors](creating_errors.md).

Thanks to Nushell's strict scoping rules, it is very easy to [iterate over collections in parallel](parallelism.md) which can help you speed up long-running scripts by just typing a few characters.

You can [interactively explore data](explore.md) with the [`explore`](/commands/docs/explore.md) command.

Finally, you can extend Nushell's functionality with [plugins](plugins.md).
Almost anything can be a plugin as long as it communicates with Nushell in a protocol that Nushell understands.
# Aliases

Aliases in Nushell offer a way of doing a simple replacement of command calls (both external and internal commands). This allows you to create a shorthand name for a longer command, including its default arguments.

For example, let's create an alias called `ll` which will expand to `ls -l`.

```nu
alias ll = ls -l
```

We can now call this alias:

```nu
ll
```

Once we do, it's as if we typed `ls -l`. This also allows us to pass in flags or positional parameters. For example, we can now also write:

```nu
ll -a
```

And get the equivalent to having typed `ls -l -a`.

## List All Loaded Aliases

Your useable aliases can be seen in `scope aliases` and `help aliases`.

## Persisting

To make your aliases persistent they must be added to your _config.nu_ file by running `config nu` to open an editor and inserting them, and then restarting nushell.
e.g. with the above `ll` alias, you can add `alias ll = ls -l` anywhere in _config.nu_

```nu
$env.config = {
    # main configuration
}

alias ll = ls -l

# some other config and script loading
```

## Piping in Aliases

Note that `alias uuidgen = uuidgen | tr A-F a-f` (to make uuidgen on mac behave like linux) won't work.
The solution is to define a command without parameters that calls the system program `uuidgen` via `^`.

```nu
def uuidgen [] { ^uuidgen | tr A-F a-f }
```

See more in the [custom commands](custom_commands.md) section of this book.

Or a more idiomatic example with nushell internal commands

```nu
def lsg [] { ls | sort-by type name -i | grid -c | str trim }
```

displaying all listed files and folders in a grid.

## Replacing Existing Commands Using Aliases

::: warning Caution!
When replacing commands it is best to "back up" the command first and avoid recursion error.
:::

How to back up a command like `ls`:

```nu
alias core-ls = ls    # This will create a new alias core-ls for ls
```

Now you can use `core-ls` as `ls` in your nu-programming. You will see further down how to use `core-ls`.

The reason you need to use alias is because, unlike `def`, aliases are position-dependent. So, you need to "back up" the old command first with an alias, before re-defining it.
If you do not backup the command and you replace the command using `def` you get a recursion error.

```nu
def ls [] { ls }; ls    # Do *NOT* do this! This will throw a recursion error

#output:
#Error: nu::shell::recursion_limit_reached
#
#  × Recursion limit (50) reached
#     ╭─[C:\Users\zolodev\AppData\Roaming\nushell\config.nu:807:1]
# 807 │
# 808 │ def ls [] { ls }; ls
#     ·           ───┬──
#     ·              ╰── This called itself too many times
#     ╰────
```

The recommended way to replace an existing command is to shadow the command.
Here is an example shadowing the `ls` command.

```nu
# An escape hatch to have access to the original ls command
alias core-ls = ls

# Call the built-in ls command with a path parameter
def old-ls [path] {
  core-ls $path | sort-by type name -i
}

# Shadow the ls command so that you always have the sort type you want
def ls [path?] {
  if $path == null {
    old-ls .
  } else {
    old-ls $path
  }
}
```
# Background Tasks with Nu

Currently, Nushell doesn't have built-in background task management feature, but you can make it "support" background task with some tools, here are some examples:

1. Using a third-party task management tools, like [pueue](https://github.com/Nukesor/pueue)
2. Using a terminal multiplexer, like [tmux](https://github.com/tmux/tmux/wiki) or [zellij](https://zellij.dev/)

## Using Nu With Pueue

The module borrows the power of [pueue](https://github.com/Nukesor/pueue), it is possible to schedule background tasks to pueue, and manage those tasks (such as viewing logs, killing tasks, or getting the running status of all tasks, creating groups, pausing tasks etc etc)

Unlike terminal multiplexer, you don't need to attach to multiple tmux sessions, and get task status easily.

Here we provide a [nushell module](https://github.com/nushell/nu_scripts/tree/main/modules/background_task) that makes working with pueue easier.

Here is a setup example to make Nushell "support" background tasks:

1. Install pueue
2. run `pueued`, you can refer to [start-the-daemon page](https://github.com/Nukesor/pueue/wiki/Get-started#start-the-daemon) for more information.
3. Put the [task.nu](https://github.com/nushell/nu_scripts/blob/main/modules/background_task/task.nu) file under `$env.NU_LIB_DIRS`.
4. Add a line to the `$nu.config-path` file: `use task.nu`
5. Restart Nushell.

Then you will get some commands to schedule background tasks. (e.g: `task spawn`, `task status`, `task log`)

Cons: It spawns a new Nushell interpreter to execute every single task, so it doesn't inherit current scope's variables, custom commands, alias definition.
It only inherits environment variables whose value can be converted to a string.
Therefore, if you want to use custom commands or variables, you have to [`use`](/commands/docs/use.md) or [`def`](/commands/docs/def.md) them within the given block.

## Using Nu With Terminal Multiplexer

You can choose and install a terminal multiplexer and use it.

It allows you to easily switch between multiple programs in one terminal, detach them (they continue to run in the background) and reconnect them to a different terminal.  As a result, it is very flexible and usable.
# Nushell Cheat Sheet

## Data Types

convert string to integer:

```nu
"12" | into int
```

convert present date to provided time zone:

```nu
date now | date to-timezone "Europe/London"
```

update a record's language and if none is specified insert provided value:

```nu
{'name': 'nu', 'stars': 5, 'language': 'Python'} | upsert language 'Rust'
```

convert list of strings to yaml:

```nu
[one two three] | to yaml
```

print table data:

```nu
[[framework, language]; [Django, Python] [Laravel, PHP]]
```

select two named columns from the table and print their values:

```nu
[{name: 'Robert' age: 34 position: 'Designer'}
 {name: 'Margaret' age: 30 position: 'Software Developer'}
 {name: 'Natalie' age: 50 position: 'Accountant'}
] | select name position
```

## Strings

interpolate text:

```nu
let name = "Alice"
$"greetings, ($name)!"
# => greetings, Alice!
```

split text on comma delimiter and save the list to `string_list` variable:

```nu
let string_list = "one,two,three" | split row ","
$string_list
# => ╭───┬───────╮
# => │ 0 │ one   │
# => │ 1 │ two   │
# => │ 2 │ three │
# => ╰───┴───────╯
```

check if a string contains a substring:

```nu
"Hello, world!" | str contains "o, w"
# => true
```

join multiple strings with delimiter:

```nu
let str_list = [zero one two]
$str_list | str join ','
# => zero,one,two
```

slice text by indices:

```nu
'Hello World!' | str substring 4..8
# => o Wor
```

parse string into named columns:

```nu
'Nushell 0.80' | parse '{shell} {version}'
# => ╭───┬─────────┬─────────╮
# => │ # │  shell  │ version │
# => ├───┼─────────┼─────────┤
# => │ 0 │ Nushell │ 0.80    │
# => ╰───┴─────────┴─────────╯
```

parse comma separated values (csv):

```nu
"acronym,long\nAPL,A Programming Language" | from csv
# => ╭───┬─────────┬────────────────────────╮
# => │ # │ acronym │          long          │
# => ├───┼─────────┼────────────────────────┤
# => │ 0 │ APL     │ A Programming Language │
# => ╰───┴─────────┴────────────────────────╯
```

color text in command-line terminal:

```nu
$'(ansi purple_bold)This text is a bold purple!(ansi reset)'
# => This text is a bold purple!
```

## Lists

insert list value at index:

```nu
[foo bar baz] | insert 1 'beeze'
# => ╭───┬───────╮
# => │ 0 │ foo   │
# => │ 1 │ beeze │
# => │ 2 │ bar   │
# => │ 3 │ baz   │
# => ╰───┴───────╯
```

update list value by index:

```nu
[1, 2, 3, 4] | update 1 10
# => ╭───┬────╮
# => │ 0 │  1 │
# => │ 1 │ 10 │
# => │ 2 │  3 │
# => │ 3 │  4 │
# => ╰───┴────╯
```

prepend list value:

```nu
let numbers = [1, 2, 3]
$numbers | prepend 0
# => ╭───┬───╮
# => │ 0 │ 0 │
# => │ 1 │ 1 │
# => │ 2 │ 2 │
# => │ 3 │ 3 │
# => ╰───┴───╯
```

append list value:

```nu
let numbers = [1, 2, 3]
$numbers | append 4
# => ╭───┬───╮
# => │ 0 │ 1 │
# => │ 1 │ 2 │
# => │ 2 │ 3 │
# => │ 3 │ 4 │
# => ╰───┴───╯
```

slice first list values:

```nu
[cammomile marigold rose forget-me-not] | first 2
# => ╭───┬───────────╮
# => │ 0 │ cammomile │
# => │ 1 │ marigold  │
# => ╰───┴───────────╯
```

iterate over a list; `elt` is current list value:

```nu
let planets = [Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune]
$planets | each { |elt| $"($elt) is a planet of the solar system" }
# => ╭───┬─────────────────────────────────────────╮
# => │ 0 │ Mercury is a planet of the solar system │
# => │ 1 │ Venus is a planet of the solar system   │
# => │ 2 │ Earth is a planet of the solar system   │
# => │ 3 │ Mars is a planet of the solar system    │
# => │ 4 │ Jupiter is a planet of the solar system │
# => │ 5 │ Saturn is a planet of the solar system  │
# => │ 6 │ Uranus is a planet of the solar system  │
# => │ 7 │ Neptune is a planet of the solar system │
# => ╰───┴─────────────────────────────────────────╯
```

iterate over a list with an index and value:

```nu
$planets | enumerate | each { |elt| $"($elt.index + 1) - ($elt.item)" }
# => ╭───┬─────────────╮
# => │ 0 │ 1 - Mercury │
# => │ 1 │ 2 - Venus   │
# => │ 2 │ 3 - Earth   │
# => │ 3 │ 4 - Mars    │
# => │ 4 │ 5 - Jupiter │
# => │ 5 │ 6 - Saturn  │
# => │ 6 │ 7 - Uranus  │
# => │ 7 │ 8 - Neptune │
# => ╰───┴─────────────╯
```

reduce the list to a single value; `reduce` gives access to accumulator that is applied to each element in the list:

```nu
let scores = [3 8 4]
$"total = ($scores | reduce { |elt, acc| $acc + $elt })"
# => total = 15
```

reduce with an initial value (`--fold`):

```nu
let scores = [3 8 4]
$"total = ($scores | reduce --fold 1 { |elt, acc| $acc * $elt })"
# => total = 96
```

give access to the 3rd item in the list:

```nu
let planets = [Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune]
$planets.2
# => Earth
```

check if any string in the list starts with `E`:

```nu
let planets = [Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune]
$planets | any {|elt| $elt | str starts-with "E" }
# => true
```

slice items that satisfy provided condition:

```nu
let cond = {|x| $x < 0 }; [-1 -2 9 1] | take while $cond
# => ╭───┬────╮
# => │ 0 │ -1 │
# => │ 1 │ -2 │
# => ╰───┴────╯
```

## Tables

sort table:

```nu
ls | sort-by size
```

sort table, get first rows:

```nu
ls | sort-by size | first 5
```

concatenate two tables with same columns:

```nu
let $a = [[first_column second_column third_column]; [foo bar snooze]]
let $b = [[first_column second_column third_column]; [hex seeze feeze]]
$a | append $b
# => ╭───┬──────────────┬───────────────┬──────────────╮
# => │ # │ first_column │ second_column │ third_column │
# => ├───┼──────────────┼───────────────┼──────────────┤
# => │ 0 │ foo          │ bar           │ snooze       │
# => │ 1 │ hex          │ seeze         │ feeze        │
# => ╰───┴──────────────┴───────────────┴──────────────╯
```

remove the last column of a table:

```nu
let teams_scores = [[team score plays]; ['Boston Celtics' 311 3] ['Golden State Warriors', 245 2]]
$teams_scores | drop column
# => ╭───┬───────────────────────┬───────╮
# => │ # │         team          │ score │
# => ├───┼───────────────────────┼───────┤
# => │ 0 │ Boston Celtics        │   311 │
# => │ 1 │ Golden State Warriors │   245 │
# => ╰───┴───────────────────────┴───────╯
```

## Files and Filesystem

open a text file with the default text editor:

```nu
start file.txt
```

save a string to text file:

```nu
'lorem ipsum ' | save file.txt
```

append a string to the end of a text file:

```nu
'dolor sit amet' | save --append file.txt
```

save a record to file.json:

```nu
{ a: 1, b: 2 } | save file.json
```

recursively search for files by file name:

```nu
glob **/*.{rs,toml} --depth 2
```

watch a file, run command whenever it changes:

```nu
watch . --glob=**/*.rs {|| cargo test }
```

## Custom Commands

custom command with parameter type set to string:

```nu
def greet [name: string] {
    $"hello ($name)"
}
```

custom command with default parameter set to nushell:

```nu
def greet [name = "nushell"] {
    $"hello ($name)"
}
```

passing named parameter by defining flag for custom commands:

```nu
def greet [
    name: string
    --age: int
] {
    [$name $age]
}

greet world --age 10
```

using flag as a switch with a shorthand flag (-a) for the age:

```nu
def greet [
    name: string
    --age (-a): int
    --twice
] {
    if $twice {
        [$name $age $name $age]
    } else {
        [$name $age]
    }
}
greet -a 10 --twice hello
```

custom command which takes any number of positional arguments using rest params:

```nu
def greet [...name: string] {
    print "hello all:"
    for $n in $name {
        print $n
    }
}
greet earth mars jupiter venus
# => hello all:
# => earth
# => mars
# => jupiter
# => venus
```

## Variables

an immutable variable cannot change its value after declaration:

```nu
let val = 42
print $val
# => 42
```

shadowing variable (declaring variable with the same name in a different scope):

```nu
let val = 42
do { let val = 101;  $val }
# => 101
$val
# => 42
```

declaring a mutable variable with mut key word:

```nu
mut val = 42
$val += 27
$val
# => 69
```

closures and nested defs cannot capture mutable variables from their environment (errors):

```nu
mut x = 0
[1 2 3] | each { $x += 1 }
# => Error: nu::parser::expected_keyword
# => 
# =>   × Capture of mutable variable.
# =>    ╭─[entry #83:1:18]
# =>  1 │ [1 2 3] | each { $x += 1 }
# =>    ·                  ─┬
# =>    ·                   ╰── capture of mutable variable
# =>    ╰────
```

a constant variable is immutable and is fully evaluated at parse-time:

```nu
const file = 'path/to/file.nu'
source $file
```

use question mark operator `?` to return null instead of error if provided path is incorrect:

```nu
let files = (ls)
$files.name?.0?
```

assign the result of a pipeline to a variable:

```nu
let big_files = (ls | where size > 10kb)
$big_files
```

## Modules

use an inline module:

```nu
module greetings {
    export def hello [name: string] {
        $"hello ($name)!"
    }

    export def hi [where: string] {
        $"hi ($where)!"
    }
}
use greetings hello
hello "world"
```

import module from file and use its environment in current scope:

```nu
# greetings.nu
export-env {
    $env.MYNAME = "Arthur, King of the Britons"
}
export def hello [] {
    $"hello ($env.MYNAME)"
}

use greetings.nu
$env.MYNAME
# => Arthur, King of the Britons
greetings hello
# => hello Arthur, King of the Britons!
```

use main command in module:

```nu
# greetings.nu
export def hello [name: string] {
    $"hello ($name)!"
}

export def hi [where: string] {
    $"hi ($where)!"
}

export def main [] {
    "greetings and salutations!"
}

use greetings.nu
greetings
# => greetings and salutations!
greetings hello world
# => hello world!
```
# Coloring and Theming in Nu

Many parts of Nushell's interface can have their color customized. All of these can be set in the `config.nu` configuration file. If you see the hash/hashtag/pound mark `#` in the config file it means the text after it is commented out.

1. table borders
2. primitive values
3. shapes (this is the command line syntax)
4. prompt
5. LS_COLORS

## Table Borders

Table borders are controlled by the `$env.config.table.mode` setting in `config.nu`. Here is an example:

```nu
$env.config = {
    table: {
        mode: rounded
    }
}
```

Here are the current options for `$env.config.table.mode`:

- `rounded` # of course, this is the best one :)
- `basic`
- `compact`
- `compact_double`
- `light`
- `thin`
- `with_love`
- `reinforced`
- `heavy`
- `none`
- `other`

### Color Symbologies

---

- `r` - normal color red's abbreviation
- `rb` - normal color red's abbreviation with bold attribute
- `red` - normal color red
- `red_bold` - normal color red with bold attribute
- `"#ff0000"` - "#hex" format foreground color red (quotes are required)
- `{ fg: "#ff0000" bg: "#0000ff" attr: b }` - "full #hex" format foreground red in "#hex" format with a background of blue in "#hex" format with an attribute of bold abbreviated.

### Attributes

---

| code | meaning             |
| ---- | ------------------- |
| l    | blink               |
| b    | bold                |
| d    | dimmed              |
| h    | hidden              |
| i    | italic              |
| r    | reverse             |
| s    | strikethrough       |
| u    | underline           |
| n    | nothing             |
|      | defaults to nothing |

### Normal Colors and Abbreviations

| code   | name                   |
| ------ | ---------------------- |
| g      | green                  |
| gb     | green_bold             |
| gu     | green_underline        |
| gi     | green_italic           |
| gd     | green_dimmed           |
| gr     | green_reverse          |
| gbl    | green_blink            |
| gst    | green_strike           |
| lg     | light_green            |
| lgb    | light_green_bold       |
| lgu    | light_green_underline  |
| lgi    | light_green_italic     |
| lgd    | light_green_dimmed     |
| lgr    | light_green_reverse    |
| lgbl   | light_green_blink      |
| lgst   | light_green_strike     |
| r      | red                    |
| rb     | red_bold               |
| ru     | red_underline          |
| ri     | red_italic             |
| rd     | red_dimmed             |
| rr     | red_reverse            |
| rbl    | red_blink              |
| rst    | red_strike             |
| lr     | light_red              |
| lrb    | light_red_bold         |
| lru    | light_red_underline    |
| lri    | light_red_italic       |
| lrd    | light_red_dimmed       |
| lrr    | light_red_reverse      |
| lrbl   | light_red_blink        |
| lrst   | light_red_strike       |
| u      | blue                   |
| ub     | blue_bold              |
| uu     | blue_underline         |
| ui     | blue_italic            |
| ud     | blue_dimmed            |
| ur     | blue_reverse           |
| ubl    | blue_blink             |
| ust    | blue_strike            |
| lu     | light_blue             |
| lub    | light_blue_bold        |
| luu    | light_blue_underline   |
| lui    | light_blue_italic      |
| lud    | light_blue_dimmed      |
| lur    | light_blue_reverse     |
| lubl   | light_blue_blink       |
| lust   | light_blue_strike      |
| b      | black                  |
| bb     | black_bold             |
| bu     | black_underline        |
| bi     | black_italic           |
| bd     | black_dimmed           |
| br     | black_reverse          |
| bbl    | black_blink            |
| bst    | black_strike           |
| ligr   | light_gray             |
| ligrb  | light_gray_bold        |
| ligru  | light_gray_underline   |
| ligri  | light_gray_italic      |
| ligrd  | light_gray_dimmed      |
| ligrr  | light_gray_reverse     |
| ligrbl | light_gray_blink       |
| ligrst | light_gray_strike      |
| y      | yellow                 |
| yb     | yellow_bold            |
| yu     | yellow_underline       |
| yi     | yellow_italic          |
| yd     | yellow_dimmed          |
| yr     | yellow_reverse         |
| ybl    | yellow_blink           |
| yst    | yellow_strike          |
| ly     | light_yellow           |
| lyb    | light_yellow_bold      |
| lyu    | light_yellow_underline |
| lyi    | light_yellow_italic    |
| lyd    | light_yellow_dimmed    |
| lyr    | light_yellow_reverse   |
| lybl   | light_yellow_blink     |
| lyst   | light_yellow_strike    |
| p      | purple                 |
| pb     | purple_bold            |
| pu     | purple_underline       |
| pi     | purple_italic          |
| pd     | purple_dimmed          |
| pr     | purple_reverse         |
| pbl    | purple_blink           |
| pst    | purple_strike          |
| lp     | light_purple           |
| lpb    | light_purple_bold      |
| lpu    | light_purple_underline |
| lpi    | light_purple_italic    |
| lpd    | light_purple_dimmed    |
| lpr    | light_purple_reverse   |
| lpbl   | light_purple_blink     |
| lpst   | light_purple_strike    |
| c      | cyan                   |
| cb     | cyan_bold              |
| cu     | cyan_underline         |
| ci     | cyan_italic            |
| cd     | cyan_dimmed            |
| cr     | cyan_reverse           |
| cbl    | cyan_blink             |
| cst    | cyan_strike            |
| lc     | light_cyan             |
| lcb    | light_cyan_bold        |
| lcu    | light_cyan_underline   |
| lci    | light_cyan_italic      |
| lcd    | light_cyan_dimmed      |
| lcr    | light_cyan_reverse     |
| lcbl   | light_cyan_blink       |
| lcst   | light_cyan_strike      |
| w      | white                  |
| wb     | white_bold             |
| wu     | white_underline        |
| wi     | white_italic           |
| wd     | white_dimmed           |
| wr     | white_reverse          |
| wbl    | white_blink            |
| wst    | white_strike           |
| dgr    | dark_gray              |
| dgrb   | dark_gray_bold         |
| dgru   | dark_gray_underline    |
| dgri   | dark_gray_italic       |
| dgrd   | dark_gray_dimmed       |
| dgrr   | dark_gray_reverse      |
| dgrbl  | dark_gray_blink        |
| dgrst  | dark_gray_strike       |

### `"#hex"` Format

---

The "#hex" format is one way you typically see colors represented. It's simply the `#` character followed by 6 characters. The first two are for `red`, the second two are for `green`, and the third two are for `blue`. It's important that this string be surrounded in quotes, otherwise Nushell thinks it's a commented out string.

Example: The primary `red` color is `"#ff0000"` or `"#FF0000"`. Upper and lower case in letters shouldn't make a difference.

This `"#hex"` format allows us to specify 24-bit truecolor tones to different parts of Nushell.

## Full `"#hex"` Format

---

The `full "#hex"` format is a take on the `"#hex"` format but allows one to specify the foreground, background, and attributes in one line.

Example: `{ fg: "#ff0000" bg: "#0000ff" attr: b }`

- foreground of red in "#hex" format
- background of blue in "#hex" format
- attribute of bold abbreviated

## Primitive Values

---

Primitive values are things like `int` and `string`. Primitive values and shapes can be set with a variety of color symbologies seen above.

This is the current list of primitives. Not all of these are configurable. The configurable ones are marked with \*.

| primitive    | default color         | configurable |
| ------------ | --------------------- | ------------ |
| `any`        |                       |              |
| `binary`     | Color::White.normal() | \*           |
| `block`      | Color::White.normal() | \*           |
| `bool`       | Color::White.normal() | \*           |
| `cellpath`   | Color::White.normal() | \*           |
| `condition`  |                       |              |
| `custom`     |                       |              |
| `date`       | Color::White.normal() | \*           |
| `duration`   | Color::White.normal() | \*           |
| `expression` |                       |              |
| `filesize`   | Color::White.normal() | \*           |
| `float`      | Color::White.normal() | \*           |
| `glob`       |                       |              |
| `import`     |                       |              |
| `int`        | Color::White.normal() | \*           |
| `list`       | Color::White.normal() | \*           |
| `nothing`    | Color::White.normal() | \*           |
| `number`     |                       |              |
| `operator`   |                       |              |
| `path`       |                       |              |
| `range`      | Color::White.normal() | \*           |
| `record`     | Color::White.normal() | \*           |
| `signature`  |                       |              |
| `string`     | Color::White.normal() | \*           |
| `table`      |                       |              |
| `var`        |                       |              |
| `vardecl`    |                       |              |
| `variable`   |                       |              |

#### Special "primitives" (not really primitives but they exist solely for coloring)

| primitive                   | default color              | configurable |
| --------------------------- | -------------------------- | ------------ |
| `leading_trailing_space_bg` | Color::Rgb(128, 128, 128)) | \*           |
| `header`                    | Color::Green.bold()        | \*           |
| `empty`                     | Color::Blue.normal()       | \*           |
| `row_index`                 | Color::Green.bold()        | \*           |
| `hints`                     | Color::DarkGray.normal()   | \*           |

Here's a small example of changing some of these values.

```nu
let config = {
    color_config: {
        separator: purple
        leading_trailing_space_bg: "#ffffff"
        header: gb
        date: wd
        filesize: c
        row_index: cb
        bool: red
        int: green
        duration: blue_bold
        range: purple
        float: red
        string: white
        nothing: red
        binary: red
        cellpath: cyan
        hints: dark_gray
    }
}
```

Here's another small example using multiple color syntaxes with some comments.

```nu
let config = {
    color_config: {
        separator: "#88b719" # this sets only the foreground color like PR #486
        leading_trailing_space_bg: white # this sets only the foreground color in the original style
        header: { # this is like PR #489
            fg: "#B01455", # note, quotes are required on the values with hex colors
            bg: "#ffb900", # note, commas are not required, it could also be all on one line
            attr: bli # note, there are no quotes around this value. it works with or without quotes
        }
        date: "#75507B"
        filesize: "#729fcf"
        row_index: {
            # note, that this is another way to set only the foreground, no need to specify bg and attr
            fg: "#e50914"
        }
    }
}
```

## Shape Values

As mentioned above, `shape` is a term used to indicate the syntax coloring.

Here's the current list of flat shapes.

| shape                        | default style                          | configurable |
| ---------------------------- | -------------------------------------- | ------------ |
| `shape_block`                | fg(Color::Blue).bold()                 | \*           |
| `shape_bool`                 | fg(Color::LightCyan)                   | \*           |
| `shape_custom`               | bold()                                 | \*           |
| `shape_external`             | fg(Color::Cyan)                        | \*           |
| `shape_externalarg`          | fg(Color::Green).bold()                | \*           |
| `shape_filepath`             | fg(Color::Cyan)                        | \*           |
| `shape_flag`                 | fg(Color::Blue).bold()                 | \*           |
| `shape_float`                | fg(Color::Purple).bold()               | \*           |
| `shape_garbage`              | fg(Color::White).on(Color::Red).bold() | \*           |
| `shape_globpattern`          | fg(Color::Cyan).bold()                 | \*           |
| `shape_int`                  | fg(Color::Purple).bold()               | \*           |
| `shape_internalcall`         | fg(Color::Cyan).bold()                 | \*           |
| `shape_list`                 | fg(Color::Cyan).bold()                 | \*           |
| `shape_literal`              | fg(Color::Blue)                        | \*           |
| `shape_nothing`              | fg(Color::LightCyan)                   | \*           |
| `shape_operator`             | fg(Color::Yellow)                      | \*           |
| `shape_range`                | fg(Color::Yellow).bold()               | \*           |
| `shape_record`               | fg(Color::Cyan).bold()                 | \*           |
| `shape_signature`            | fg(Color::Green).bold()                | \*           |
| `shape_string`               | fg(Color::Green)                       | \*           |
| `shape_string_interpolation` | fg(Color::Cyan).bold()                 | \*           |
| `shape_table`                | fg(Color::Blue).bold()                 | \*           |
| `shape_variable`             | fg(Color::Purple)                      | \*           |

Here's a small example of how to apply color to these items. Anything not specified will receive the default color.

```nu
$env.config = {
    color_config: {
        shape_garbage: { fg: "#FFFFFF" bg: "#FF0000" attr: b}
        shape_bool: green
        shape_int: { fg: "#0000ff" attr: b}
    }
}
```

## Prompt Configuration and Coloring

The Nushell prompt is configurable through these environment variables and config items:

- `PROMPT_COMMAND`: Code to execute for setting up the prompt (block)
- `PROMPT_COMMAND_RIGHT`: Code to execute for setting up the _RIGHT_ prompt (block) (see oh-my.nu in nu_scripts)
- `PROMPT_INDICATOR` = "〉": The indicator printed after the prompt (by default ">"-like Unicode symbol)
- `PROMPT_INDICATOR_VI_INSERT` = ": "
- `PROMPT_INDICATOR_VI_NORMAL` = "v "
- `PROMPT_MULTILINE_INDICATOR` = "::: "
- `render_right_prompt_on_last_line`: Bool value to enable or disable the right prompt to be rendered on the last line of the prompt

Example: For a simple prompt one could do this. Note that `PROMPT_COMMAND` requires a `block` whereas the others require a `string`.

```nu
$env.PROMPT_COMMAND = { build-string (date now | format date '%m/%d/%Y %I:%M:%S%.3f') ': ' (pwd | path basename) }
```

If you don't like the default `PROMPT_INDICATOR` you could change it like this.

```nu
$env.PROMPT_INDICATOR = "> "
```

If you're using `starship`, you'll most likely want to show the right prompt on the last line of the prompt, just like zsh or fish. You could modify the `config.nu` file, just set `render_right_prompt_on_last_line` to true:

```nu
config {
    render_right_prompt_on_last_line = true
    ...
}
```

Coloring of the prompt is controlled by the `block` in `PROMPT_COMMAND` where you can write your own custom prompt. We've written a slightly fancy one that has git statuses located in the [nu_scripts repo](https://github.com/nushell/nu_scripts/blob/main/modules/prompt/oh-my.nu).

### Transient Prompt

If you want a different prompt displayed for previously entered commands, you can use Nushell's transient prompt feature. This can be useful if your prompt has lots of information that is unnecessary to show for previous lines (e.g. time and Git status), since you can make it so that previous lines show with a shorter prompt.

Each of the `PROMPT_*` variables has a corresponding `TRANSIENT_PROMPT_*` variable to be used for changing that segment when displaying past prompts: `TRANSIENT_PROMPT_COMMAND`, `TRANSIENT_PROMPT_COMMAND_RIGHT`, `TRANSIENT_PROMPT_INDICATOR`, `TRANSIENT_PROMPT_INDICATOR_VI_INSERT`, `TRANSIENT_PROMPT_INDICATOR_VI_NORMAL`, `TRANSIENT_PROMPT_MULTILINE_INDICATOR`. By default, the `PROMPT_*` variables are used for displaying past prompts.

For example, if you want to make past prompts show up without a left prompt entirely and leave only the indicator, you can use:

```nu
$env.TRANSIENT_PROMPT_COMMAND = ""
```

If you want to go back to the normal left prompt, you'll have to unset `TRANSIENT_PROMPT_COMMAND`:

```nu
hide-env TRANSIENT_PROMPT_COMMAND
```

## `LS_COLORS` Colors for the [`ls`](/commands/docs/ls.md) Command

Nushell will respect and use the `LS_COLORS` environment variable setting on Mac, Linux, and Windows. This setting allows you to define the color of file types when you do a [`ls`](/commands/docs/ls.md). For instance, you can make directories one color, _.md markdown files another color, _.toml files yet another color, etc. There are a variety of ways to color your file types.

There's an exhaustive list [here](https://github.com/trapd00r/LS_COLORS), which is overkill, but gives you an rudimentary understanding of how to create a ls_colors file that `dircolors` can turn into a `LS_COLORS` environment variable.

[This](https://www.linuxhowto.net/how-to-set-colors-for-ls-command/) is a pretty good introduction to `LS_COLORS`. I'm sure you can find many more tutorials on the web.

I like the `vivid` application and currently have it configured in my `config.nu` like this. You can find `vivid` [here](https://github.com/sharkdp/vivid).

`$env.LS_COLORS = (vivid generate molokai | str trim)`

If `LS_COLORS` is not set, nushell will default to a built-in `LS_COLORS` setting, based on 8-bit (extended) ANSI colors.

## Theming

Theming combines all the coloring above. Here's a quick example of one we put together quickly to demonstrate the ability to theme. This is a spin on the `base16` themes that we see so widespread on the web.

The key to making theming work is to make sure you specify all themes and colors you're going to use in the `config.nu` file _before_ you declare the `let config = ` line.

```nu
# let's define some colors

let base00 = "#181818" # Default Background
let base01 = "#282828" # Lighter Background (Used for status bars, line number and folding marks)
let base02 = "#383838" # Selection Background
let base03 = "#585858" # Comments, Invisibles, Line Highlighting
let base04 = "#b8b8b8" # Dark Foreground (Used for status bars)
let base05 = "#d8d8d8" # Default Foreground, Caret, Delimiters, Operators
let base06 = "#e8e8e8" # Light Foreground (Not often used)
let base07 = "#f8f8f8" # Light Background (Not often used)
let base08 = "#ab4642" # Variables, XML Tags, Markup Link Text, Markup Lists, Diff Deleted
let base09 = "#dc9656" # Integers, Boolean, Constants, XML Attributes, Markup Link Url
let base0a = "#f7ca88" # Classes, Markup Bold, Search Text Background
let base0b = "#a1b56c" # Strings, Inherited Class, Markup Code, Diff Inserted
let base0c = "#86c1b9" # Support, Regular Expressions, Escape Characters, Markup Quotes
let base0d = "#7cafc2" # Functions, Methods, Attribute IDs, Headings
let base0e = "#ba8baf" # Keywords, Storage, Selector, Markup Italic, Diff Changed
let base0f = "#a16946" # Deprecated, Opening/Closing Embedded Language Tags, e.g. <?php ?>

# we're creating a theme here that uses the colors we defined above.

let base16_theme = {
    separator: $base03
    leading_trailing_space_bg: $base04
    header: $base0b
    date: $base0e
    filesize: $base0d
    row_index: $base0c
    bool: $base08
    int: $base0b
    duration: $base08
    range: $base08
    float: $base08
    string: $base04
    nothing: $base08
    binary: $base08
    cellpath: $base08
    hints: dark_gray

    # shape_garbage: { fg: $base07 bg: $base08 attr: b} # base16 white on red
    # but i like the regular white on red for parse errors
    shape_garbage: { fg: "#FFFFFF" bg: "#FF0000" attr: b}
    shape_bool: $base0d
    shape_int: { fg: $base0e attr: b}
    shape_float: { fg: $base0e attr: b}
    shape_range: { fg: $base0a attr: b}
    shape_internalcall: { fg: $base0c attr: b}
    shape_external: $base0c
    shape_externalarg: { fg: $base0b attr: b}
    shape_literal: $base0d
    shape_operator: $base0a
    shape_signature: { fg: $base0b attr: b}
    shape_string: $base0b
    shape_filepath: $base0d
    shape_globpattern: { fg: $base0d attr: b}
    shape_variable: $base0e
    shape_flag: { fg: $base0d attr: b}
    shape_custom: {attr: b}
}

# now let's apply our regular config settings but also apply the "color_config:" theme that we specified above.

let config = {
  filesize_metric: true
  table_mode: rounded # basic, compact, compact_double, light, thin, with_love, rounded, reinforced, heavy, none, other
  use_ls_colors: true
  color_config: $base16_theme # <-- this is the theme
  use_grid_icons: true
  footer_mode: always #always, never, number_of_rows, auto
  animate_prompt: false
  float_precision: 2
  use_ansi_coloring: true
  filesize_format: "b" # b, kb, kib, mb, mib, gb, gib, tb, tib, pb, pib, eb, eib, auto
  edit_mode: emacs # vi
  max_history_size: 10000
  log_level: error
}
```

if you want to go full-tilt on theming, you'll want to theme all the items I mentioned at the very beginning, including LS_COLORS, and the prompt. Good luck!

### Working on Light Background Terminal

Nushell's default config file contains a light theme definition, if you are working on a light background terminal, you can apply the light theme easily.

```nu
# in $nu.config-path
$env.config = {
  ...
  color_config: $dark_theme   # if you want a light theme, replace `$dark_theme` to `$light_theme`
  ...
}
```

You can just change it to light theme by replacing `$dark_theme` to `$light_theme`

```nu
# in $nu.config-path
$env.config = {
  ...
  color_config: $light_theme   # if you want a light theme, replace `$dark_theme` to `$light_theme`
  ...
}
```

## Accessibility

It's often desired to have the minimum amount of decorations when using a screen reader. In those cases, it's possible to disable borders and other decorations for both table and errors with the following options:

```nu
# in $nu.config-path
$env.config = {
  ...
  table: {
   ...
    mode: "none"
   ...
  }
  error_style: "plain"
  ...
}

```

## Line Editor Menus (completion, history, help…)

Reedline (Nu’s line editor) style is not using the `color_config` key.
Instead, each menu has its own style to be configured separately.
See the [section dedicated to Reedline’s menus configuration](line_editor.md#menus) to learn more on this.
# Coming from Bash

::: tip
If you're coming from `Git Bash` on Windows, then the external commands you're used to (e.g, `ln`, `grep`, `vi`, etc) will not be available in Nushell by default unless you have already explicitly made them available in the Windows Path environment variable.
To make these commands available in Nushell as well, add the following line to your `config.nu` with either `append` or `prepend`.

```
$env.Path = ($env.Path | prepend 'C:\Program Files\Git\usr\bin')
```
:::

## Command Equivalents:

| Bash                                 | Nu                                                            | Task                                                              |
| ------------------------------------ | ------------------------------------------------------------- | ----------------------------------------------------------------- |
| `ls`                                 | `ls`                                                          | Lists the files in the current directory                          |
| `ls <dir>`                           | `ls <dir>`                                                    | Lists the files in the given directory                            |
| `ls pattern*`                        | `ls pattern*`                                                 | Lists files that match a given pattern                            |
| `ls -la`                             | `ls --long --all` or `ls -la`                                 | List files with all available information, including hidden files |
| `ls -d */`                           | `ls \| where type == dir`                                     | List directories                                                  |
| `find . -name *.rs`                  | `ls **/*.rs`                                                  | Find recursively all files that match a given pattern             |
| `find . -name Makefile \| xargs vim` | `ls **/Makefile \| get name \| vim ...$in`                    | Pass values as command parameters                                 |
| `cd <directory>`                     | `cd <directory>`                                              | Change to the given directory                                     |
| `cd`                                 | `cd`                                                          | Change to the home directory                                      |
| `cd -`                               | `cd -`                                                        | Change to the previous directory                                  |
| `mkdir <path>`                       | `mkdir <path>`                                                | Creates the given path                                            |
| `mkdir -p <path>`                    | `mkdir <path>`                                                | Creates the given path, creating parents as necessary             |
| `touch test.txt`                     | `touch test.txt`                                              | Create a file                                                     |
| `> <path>`                           | `out> <path>` or `o> <path>`                                  | Save command output to a file                                     |
|                                      | `\| save <path>`                                              | Save command output to a file as structured data                  |
| `>> <path>`                          | `out>> <path>` or `o>> <path>`                                | Append command output to a file                                   |
|                                      | `\| save --append <path>`                                     | Append command output to a file as structured data                |
| `> /dev/null`                        | `\| ignore`                                                   | Discard command output                                            |
| `> /dev/null 2>&1`                   | `out+err>\| ignore` or `o+e>\| ignore`                        | Discard command output, including stderr                          |
| `command 2>&1 \| less`               | `command out+err>\| less` or `command o+e>\| less`            | Pipe stdout and stderr of an external command into less (NOTE: use [explore](explore.html) for paging output of internal commands) |
| `cmd1 \| tee log.txt \| cmd2`        | `cmd1 \| tee { save log.txt } \| cmd2`                        | Tee command output to a log file                                  |
| `command \| head -5`                 | `command \| first 5`                                          | Limit the output to the first 5 rows of an internal command (see also `last` and `skip`) |
| `cat <path>`                         | `open --raw <path>`                                           | Display the contents of the given file                            |
|                                      | `open <path>`                                                 | Read a file as structured data                                    |
| `mv <source> <dest>`                 | `mv <source> <dest>`                                          | Move file to new location                                         |
| `for f in *.md; do echo $f; done`    | `ls *.md \| each { $in.name }`                                | Iterate over a list and return results                            |
| `for i in $(seq 1 10); do echo $i; done` | `for i in 1..10 { print $i }`                             | Iterate over a list and run a command on results                  |
| `cp <source> <dest>`                 | `cp <source> <dest>`                                          | Copy file to new location                                         |
| `cp -r <source> <dest>`              | `cp -r <source> <dest>`                                       | Copy directory to a new location, recursively                     |
| `rm <path>`                          | `rm <path>`                                                   | Remove the given file                                             |
|                                      | `rm -t <path>`                                                | Move the given file to the system trash                           |
| `rm -rf <path>`                      | `rm -r <path>`                                                | Recursively removes the given path                                |
| `date -d <date>`                     | `"<date>" \| into datetime -f <format>`                       | Parse a date ([format documentation](https://docs.rs/chrono/0.4.15/chrono/format/strftime/index.html)) |
| `sed`                                | `str replace`                                                 | Find and replace a pattern in a string                            |
| `grep <pattern>`                     | `where $it =~ <substring>` or `find <substring>`              | Filter strings that contain the substring                         |
| `man <command>`                      | `help <command>`                                              | Get the help for a given command                                  |
|                                      | `help commands`                                               | List all available commands                                       |
|                                      | `help --find <string>`                                        | Search for match in all available commands                        |
| `command1 && command2`               | `command1; command2`                                          | Run a command, and if it's successful run a second                |
| `stat $(which git)`                  | `stat ...(which git).path`                                    | Use command output as argument for other command                  |
| `echo /tmp/$RANDOM`                  | `$"/tmp/(random int)"`                                        | Use command output in a string                                    |
| `cargo b --jobs=$(nproc)`            | `cargo b $"--jobs=(sys cpu \| length)"`                       | Use command output in an option                                   |
| `echo $PATH`                         | `$env.PATH` (Non-Windows) or `$env.Path` (Windows)            | See the current path                                              |
| `echo $?`                            | `$env.LAST_EXIT_CODE`                                         | See the exit status of the last executed command                  |
| `<update ~/.bashrc>`                 | `vim $nu.config-path`                                         | Update PATH permanently                                           |
| `export PATH = $PATH:/usr/other/bin` | `$env.PATH = ($env.PATH \| append /usr/other/bin)`            | Update PATH temporarily                                           |
| `export`                             | `$env`                                                        | List the current environment variables                            |
| `<update ~/.bashrc>`                 | `vim $nu.config-path`                                         | Update environment variables permanently                          |
| `FOO=BAR ./bin`                      | `FOO=BAR ./bin`                                               | Update environment temporarily                                    |
| `export FOO=BAR`                     | `$env.FOO = BAR`                                              | Set environment variable for current session                      |
| `echo $FOO`                          | `$env.FOO`                                                    | Use environment variables                                         |
| `echo ${FOO:-fallback}`              | `$env.FOO? \| default "ABC"`                                  | Use a fallback in place of an unset variable                      |
| `unset FOO`                          | `hide-env FOO`                                                | Unset environment variable for current session                    |
| `alias s="git status -sb"`           | `alias s = git status -sb`                                    | Define an alias temporarily                                       |
| `type FOO`                           | `which FOO`                                                   | Display information about a command (builtin, alias, or executable) |
| `<update ~/.bashrc>`                 | `vim $nu.config-path`                                         | Add and edit alias permanently (for new shells)                   |
| `bash -c <commands>`                 | `nu -c <commands>`                                            | Run a pipeline of commands                                        |
| `bash <script file>`                 | `nu <script file>`                                            | Run a script file                                                 |
| `\`                                  | `( <command> )`                                               | A command can span multiple lines when wrapped with `(` and `)`   |
| `pwd`                                | `$env.PWD`                                                    | Display the current directory                                     |
| `read var`                           | `let var = input`                                             | Get input from the user                                           |
| `read -s secret`                     | `let secret = input -s`                                       | Get a secret value from the user without printing keystrokes      |

## History Substitutions and Default Keybindings:

| Bash                                 | Nu                                                            | Task                                                              |
| ------------------------------------ | ------------------------------------------------------------- | ----------------------------------------------------------------- |
| `!!`                                 | `!!`                                                          | Insert last command-line from history                              |
| `!$`                                 | `!$`                                                          | Insert last spatially separated token from history                |
| `!<n>` (e.g., `!5`)                  | `!<n>`                                                        | Insert \<n\>th command from the beginning of the history          |
|                                      |                                                               | Tip: `history \| enumerate \| last 10` to show recent positions   |
| `!<-n>` (e.g., `!-5`)                | `!<-n>`                                                       | Insert \<n\>th command from the end of the history                |
| `!<string>` (e.g., `!ls`)            | `!<string>`                                                   | Insert the most recent history item that begins with the string   |
| <kbd>Ctrl/Cmd</kbd>+<kbd>R</kbd>     | <kbd>Ctrl/Cmd</kbd>+<kbd>R</kbd>                              | Reverse history search                                            |
| (Emacs Mode) <kbd>Ctrl</kbd>+<kbd>X</kbd><kbd>Ctrl</kbd>+<kbd>E</kbd> | <kbd>Ctrl/Cmd</kbd>+<kbd>O</kbd> | Edit the command-line in the editor defined by `$env.EDITOR`   |
| (Vi Command Mode) <kbd>V</kbd>       | <kbd>Ctrl/Cmd</kbd>+<kbd>O</kbd>                              | Edit the command-line in the editor defined by `$env.EDITOR`       |

Most common Emacs-mode and Vi-mode keybindings are also available. See the [Reedline Chapter](line_editor.html#editing-mode).

::: tip
In Bash, history substitution occurs immediately after pressing <kbd>Enter</kbd> 
to execute the command-line. Nushell, however, *inserts* the substitution into
the command-line after pressing <kbd>Enter</kbd>. This allows you to confirm
the substitution and, if needed, make additional edits before execution.

This behavior extends to "Edit command-line in Editor" as well. While Bash immediately
executes the command after exiting the editor, Nushell (like other, more modern shells
such as Fish and Zsh) inserts the editor contents into the command-line, allowing you
to review and make changes before committing it to execution.
:::
# Coming from CMD.EXE

This table was last updated for Nu 0.67.0.

| CMD.EXE                              | Nu                                                                                  | Task                                                                  |
| ------------------------------------ | ----------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `ASSOC`                              |                                                                                     | Displays or modifies file extension associations                      |
| `BREAK`                              |                                                                                     | Trigger debugger breakpoint                                           |
| `CALL <filename.bat>`                | `<filename.bat>`                                                                    | Run a batch program                                                   |
|                                      | `nu <filename>`                                                                     | Run a nu script in a fresh context                                    |
|                                      | `source <filename>`                                                                 | Run a nu script in this context                                       |
|                                      | `use <filename>`                                                                    | Run a nu script as a module                                           |
| `CD` or `CHDIR`                      | `$env.PWD`                                                                          | Get the present working directory                                     |
| `CD <directory>`                     | `cd <directory>`                                                                    | Change the current directory                                          |
| `CD /D <drive:directory>`            | `cd <drive:directory>`                                                              | Change the current directory                                          |
| `CLS`                                | `clear`                                                                             | Clear the screen                                                      |
| `COLOR`                              |                                                                                     | Set the console default foreground/background colors                  |
|                                      | `ansi {flags} (code)`                                                               | Output ANSI codes to change color                                     |
| `COPY <source> <destination>`        | `cp <source> <destination>`                                                         | Copy files                                                            |
| `COPY <file1>+<file2> <destination>` | `[<file1>, <file2>] \| each { open --raw } \| str join \| save --raw <destination>` | Append multiple files into one                                        |
| `DATE /T`                            | `date now`                                                                          | Get the current date                                                  |
| `DATE`                               |                                                                                     | Set the date                                                          |
| `DEL <file>` or `ERASE <file>`       | `rm <file>`                                                                         | Delete files                                                          |
| `DIR`                                | `ls`                                                                                | List files in the current directory                                   |
| `ECHO <message>`                     | `print <message>`                                                                   | Print the given values to stdout                                      |
| `ECHO ON`                            |                                                                                     | Echo executed commands to stdout                                      |
| `ENDLOCAL`                           | `export-env`                                                                        | Change env in the caller                                              |
| `EXIT`                               | `exit`                                                                              | Close the prompt or script                                            |
| `FOR %<var> IN (<set>) DO <command>` | `for $<var> in <set> { <command> }`                                                 | Run a command for each item in a set                                  |
| `FTYPE`                              |                                                                                     | Displays or modifies file types used in file extension associations   |
| `GOTO`                               |                                                                                     | Jump to a label                                                       |
| `IF ERRORLEVEL <number> <command>`   | `if $env.LAST_EXIT_CODE >= <number> { <command> }`                                  | Run a command if the last command returned an error code >= specified |
| `IF <string> EQU <string> <command>` | `if <string> == <string> { <command> }`                                             | Run a command if strings match                                        |
| `IF EXIST <filename> <command>`      | `if (<filename> \| path exists) { <command> }`                                      | Run a command if the file exists                                      |
| `IF DEFINED <variable> <command>`    | `if '$<variable>' in (scope variables).name { <command> }`                          | Run a command if the variable is defined                              |
| `MD` or `MKDIR`                      | `mkdir`                                                                             | Create directories                                                    |
| `MKLINK`                             |                                                                                     | Create symbolic links                                                 |
| `MOVE`                               | `mv`                                                                                | Move files                                                            |
| `PATH`                               | `$env.Path`                                                                         | Display the current path variable                                     |
| `PATH <path>;%PATH%`                 | `$env.Path = ($env.Path \| append <path>`)                                          | Edit the path variable                                                |
| `PATH %PATH%;<path>`                 | `$env.Path = ($env.Path \| prepend <path>`)                                         | Edit the path variable                                                |
| `PAUSE`                              | `input "Press any key to continue . . ."`                                           | Pause script execution                                                |
| `PROMPT <template>`                  | `$env.PROMPT_COMMAND = { <command> }`                                               | Change the terminal prompt                                            |
| `PUSHD <path>`/`POPD`                | `enter <path>`/`dexit`                                                              | Change working directory temporarily                                  |
| `REM`                                | `#`                                                                                 | Comments                                                              |
| `REN` or `RENAME`                    | `mv`                                                                                | Rename files                                                          |
| `RD` or `RMDIR`                      | `rm`                                                                                | Remove directory                                                      |
| `SET <var>=<string>`                 | `$env.<var> = <string>`                                                             | Set environment variables                                             |
| `SETLOCAL`                           | (default behavior)                                                                  | Localize environment changes to a script                              |
| `START <path>`                       | Partially covered by `start <path>`                                                 | Open the path in the system-configured default application            |
| `START <internal command>`           |                                                                                     | Start a separate window to run a specified internal command           |
| `START <batch file>`                 |                                                                                     | Start a separate window to run a specified batch file                 |
| `TIME /T`                            | `date now \| format date "%H:%M:%S"`                                                | Get the current time                                                  |
| `TIME`                               |                                                                                     | Set the current time                                                  |
| `TITLE`                              |                                                                                     | Set the cmd.exe window name                                           |
| `TYPE`                               | `open --raw`                                                                        | Display the contents of a text file                                   |
|                                      | `open`                                                                              | Open a file as structured data                                        |
| `VER`                                |                                                                                     | Display the OS version                                                |
| `VERIFY`                             |                                                                                     | Verify that file writes happen                                        |
| `VOL`                                |                                                                                     | Show drive information                                                |

## Forwarded CMD.EXE commands

Nu accepts and runs *some* of CMD.EXE's internal commands through `cmd.exe`.

The internal commands are: `ASSOC`, `CLS`, `ECHO`, `FTYPE`, `MKLINK`, `PAUSE`, `START`, `VER`, `VOL`

These internal commands take precedence over external commands.

For example, with a `ver.bat` file in the current working directory, executing `^ver` executes CMD.EXE's internal `VER` command, *NOT* the `ver.bat` file.

Executing `./ver` or `ver.bat` *will* execute the local bat file though.

Note that Nushell has its own [`start` command](/commands/docs/start.md) which takes precedence.
You can call the CMD.EXE's internal `START` command with the external command syntax `^start`.
# Coming to Nu

If you are familiar with other shells or programming languages, you might find this chapter useful to get up to speed.

[Coming from Bash](coming_from_bash.md) shows how some patterns typical for Bash, or POSIX shells in general, can be mapped to Nushell.
Similarly, [Coming from CMD.EXE](coming_from_cmd.md) shows how built-in commands in the Windows Command Prompt can be mapped to Nushell.

Similar comparisons are made for some [other shells and domain-specific languages](nushell_map.md), [imperative languages](nushell_map_imperative.md), and [functional languages](nushell_map_functional.md).
A separate comparison is made specifically for [operators](nushell_operator_map.md).
# Configuration

## Quickstart

While Nushell provides many options for managing its startup and configuration, new users
can get started with just a few simple steps:

1. Tell Nushell what editor to use:

   ```nu
   $env.config.buffer_editor = <path_to_your_preferred_editor>
   ```

   For example:

   ```nu
   $env.config.buffer_editor = "code"
   # or
   $env.config.buffer_editor = "nano"
   # or
   $env.config.buffer_editor = "hx"
   # or
   $env.config.buffer_editor = "vi"
   # etc.
   ```

2. Edit `config.nu` using:

   ```nu
   config nu
   ```

   This will open the current `config.nu` in the editor defined above.

3. Add commands to this file that should run each time Nushell starts. A good first example might be the `buffer_editor` setting above.

   You can find a detailed list of available settings using:

   ```nu
    config nu --doc | nu-highlight | less -R
   ```

4. Save, exit the editor, and start a new Nushell session to load these settings.

That's it! More details are below when you need them ...

---

[[toc]]

::: tip
To view a simplified version of this documentation from inside Nushell, run:

```nu
config nu --doc | nu-highlight | less -R
```

:::

## Configuration Overview

Nushell uses multiple, optional configuration files. These files are loaded in the following order:

1. The first file loaded is `env.nu`, which was historically used to override environment variables. However, the current "best-practice" recommendation is to set all environment variables (and other configuration) using `config.nu` and the autoload directories below.

2. `config.nu` is typically used to override default Nushell settings, define (or import) custom commands, or run any other startup tasks.
3. Files in `$nu.vendor-autoload-dirs` are loaded. These files can be used for any purpose and are a convenient way to modularize a configuration.
4. `login.nu` runs commands or handles configuration that should only take place when Nushell is running as a login shell.

By default, `env.nu`, `config.nu`, and `login.nu` are read from the `$nu.default-config-dir` directory. For example:

```nu
$nu.default-config-dir
# macOS
# => /Users/me/Library/Application Support/nushell
# Linux
# => /home/me/.config/nushell
# Windows
# => C:\Users\me\AppData\Roaming\nushell
```

The first time Nushell is launched, it will create the configuration directory and an empty (other than comments) `env.nu` and `config.nu`.

::: tip
You can quickly open `config.nu` in your default text editor using the `config nu` command. Likewise, the `config env` command will open `env.nu`.

This requires that you have configured a default editor using either:

- Nushell's `$env.config.buffer_editor` setting
- The `$env.VISUAL` or `$env.EDITOR` environment variables

For example, place this in your `config.nu` to edit your files in Visual Studio Code:

```nu
$env.config.buffer_editor = 'code'
```

:::

## Common Configuration Tasks in `config.nu`:

::: tip
Some users will prefer a "monolithic" configuration file with most or all startup tasks in one place. `config.nu` can be used for this purpose.

Other users may prefer a "modular" configuration where each file handles a smaller, more focused set of tasks. Files in the autoload dirs can be used to create this experience.
:::

`config.nu` is commonly used to:

- Set [environment variables](#set-environment-variables) for Nushell and other applications
- Set Nushell settings in [`$env.config`](#nushell-settings-in-the-envconfig-record)
- Load modules or source files so that their commands are readily available
- Run any other applications or commands at startup

## Set Environment Variables

::: tip See Also
The [Environment](./environment.md) Chapter covers additional information on how to set and access environment variables.
:::

### Path Configuration

As with most shells, Nushell searches the environment variable named `PATH` (or variants).

:::tip
Unlike some shells, Nushell attempts to be "case agnostic" with environment variables. `Path`, `path`, `PATH`, and even `pAtH` are all allowed variants of the same environment variable. See [Environment - Case Sensitivity](./environment.md#case-sensitivity) for details.
:::

When Nushell is launched, it usually inherits the `PATH` as a string. However, Nushell automatically converts this to a Nushell list for easy access. This means that you can _append_ to the path using, for example:

```nu
$env.path ++= ["~/.local/bin"]
```

The Standard Library also includes a helper command. The default `path add` behavior is to _prepend_
a directory so that it has higher precedence than the rest of the path. For example, the following can be
added to your startup config:

```nushell
use std/util "path add"
path add "~/.local/bin"
path add ($env.CARGO_HOME | path join "bin")
```

::: tip
Notice the use of `path join` in the example above. This command properly joins two path
components regardless of whether or not the path separator is present. See `help path` for
more commands in this category.
:::

### Prompt Configuration

Nushell provides a number of prompt configuration options. By default, Nushell includes:

- A prompt which includes the current directory, abbreviated using `~` if it is (or is under)
  the home directory.
- A prompt indicator which appears immediately to the right of the prompt. This defaults to `> ` when in normal editing mode, or a `: ` when in Vi-insert mode. Note extra space after the character to provide separation of the command from the prompt.
- A right-prompt with the date and time
- An indicator which is displayed when the current commandline extends over multiple lines - `::: ` by default

The environment variables which control these prompt components are:

- `$env.PROMPT_COMMAND`: The prompt itself
- `$env.PROMPT_COMMAND_RIGHT`: A prompt which can appear on the right side of the terminal
- `$env.PROMPT_INDICATOR`: Emacs mode indicator
- `$env.PROMPT_INDICATOR_VI_NORMAL`: Vi-normal mode indicator
- `$env.PROMPT_INDICATOR_VI_INSERT`: Vi-insert mode indicator
- `$env.PROMPT_MULTILINE_INDICATOR`: The multi-line indicator

Each of these variables accepts either:

- A string, in which case the component will be statically displayed as that string.
- A closure (with no parameters), in which case the component will be dynamically displayed based on the closure's code.
- `null`, in which case the component will revert to its internal default value.

::: tip
To disable the right-prompt, for instance, add the following to your startup config:

```nu
$env.PROMPT_COMMAND_RIGHT = ""
# or
$env.PROMPT_COMMAND_RIGHT = {||}
```

:::

#### Transient Prompts

Nushell also supports transient prompts, which allow a different prompt to be shown _after_ a commandline has been executed. This can be useful in several situations:

- When using a multi-line prompt, the transient prompt can be a more condensed version.
- Removing the transient multiline indicator and right-prompt can simplify copying from the terminal.

As with the normal prompt commands above, each transient prompt can accept a (static) string, a (dynamic) closure, or a `null` to use the Nushell internal defaults.

The environment variables which control the transient prompt components are:

- `$env.TRANSIENT_PROMPT_COMMAND`: The prompt itself after the commandline has been executed
- `$env.TRANSIENT_PROMPT_COMMAND_RIGHT`: A prompt which can appear on the right side of the terminal
- `$env.TRANSIENT_PROMPT_INDICATOR`: Emacs mode indicator
- `$env.TRANSIENT_PROMPT_INDICATOR_VI_NORMAL`: Vi-normal mode indicator
- `$env.TRANSIENT_PROMPT_INDICATOR_VI_INSERT`: Vi-insert mode indicator
- `$env.TRANSIENT_PROMPT_MULTILINE_INDICATOR`: The multi-line indicator

::: tip
Nushell sets `TRANSIENT_PROMPT_COMMAND_RIGHT` and `TRANSIENT_PROMPT_MULTILINE_INDICATOR` to an empty string (`""`) so that each disappears after the previous command is entered. This simplifies copying and pasting from the terminal.

To disable this feature and always show those items, set:

```nu
$env.TRANSIENT_PROMPT_COMMAND_RIGHT = null
$env.TRANSIENT_PROMPT_MULTILINE_INDICATOR = null
```

:::

### ENV_CONVERSIONS

Certain variables, such as those containing multiple paths, are often stored as a
colon-separated string in other shells. Nushell can convert these automatically to a
more convenient Nushell list. The ENV_CONVERSIONS variable specifies how environment
variables are:

- converted from a string to a value on Nushell startup (from_string)
- converted from a value back to a string when running external commands (to_string)

`ENV_CONVERSIONS` is a record, where:

- each key is an environment variable to be converted
- each value is another record containing a:
  ```nu
  {
    from_string: <closure>
    to_string: <closure>
  }
  ```

::: tip
As mentioned above, the OS Path variable is automatically converted by Nushell. As a result, it can be treated as a list within your startup config without needing to be present in `ENV_CONVERSIONS`. Other colon-separated paths, like `XDG_DATA_DIRS`, are not automatically converted.
:::

To add an additional conversion, [`merge`](/commands/docs/merge.md) it into the `$env.ENV_CONVERSIONS` record. For example, to add a conversion for the `XDG_DATA_DIRS` variable:

```nu
$env.ENV_CONVERSIONS = $env.ENV_CONVERSIONS | merge {
    "XDG_DATA_DIRS": {
        from_string: {|s| $s | split row (char esep) | path expand --no-symlink }
        to_string: {|v| $v | path expand --no-symlink | str join (char esep) }
    }
}
```

### `LS_COLORS`

As with many `ls`-like utilities, Nushell's directory listings make use of the `LS_COLORS` environment variable for defining styles/colors to apply to certain file types and patterns.

## Nushell Settings in the `$env.config` Record

### Changing Settings in the `$env.config` Record

The primary mechanism for changing Nushell's behavior is the `$env.config` record. While this record is accessed as an environment variable, unlike most other variables it is:

- Not inherited from the parent process. Instead, it is populated by Nushell itself with certain defaults.
- Not exported to child processes started by Nushell.

To examine the current settings in `$env.config`, just type the variable name:

```nu
$env.config
```

::: tip
Since Nushell provides so many customization options, it may be better to send this to a pager like:

```nu
$env.config | table -e | less -R
# or, if bat is installed:
$env.config | table -e | bat -p
```

:::

An appendix documenting each setting will be available soon. In the meantime, abbreviated documentation on each setting can be viewed in Nushell using:

```nu
config nu --doc | nu-highlight | bat
# or
config nu --doc | nu-highlight | less -R
```

To avoid overwriting existing settings, it's best to simply assign updated values to the desired configuration keys, rather than the entire `config` record. In other words:

::: warning Wrong

```nu
$env.config = {
  show_banner: false
}
```

This would reset any _other_ settings that had been changed, since the entire record would be overwritten.
:::

::: tip Right

```nu
$env.config.show_banner = false
```

This changes _only_ the `show_banner` key/value pair, leaving all other keys with their existing values.
:::

Certain keys are themselves also records. It's okay to overwrite these records, but it's best-practice
to set all values when doing so. For example:

```nu
$env.config.history = {
  file_format: sqlite
  max_size: 1_000_000
  sync_on_enter: true
  isolation: true
}
```

### Remove Welcome Message

:::note
This section is linked directly from the banner message, so it repeats some information from above.
:::

To remove the welcome message that displays each time Nushell starts:

1. Type `config nu` to edit your configuration file.
2. If you receive an error regarding the editor not being defined:

   ```nu
   $env.config.buffer_editor = <path to your preferred editor>
   # Such as:
   $env.config.buffer_editor = "code"
   $env.config.buffer_editor = "vi"
   # Or with editor arguments:
   $env.config.buffer_editor = ["emacsclient", "-s", "light", "-t"]
   ```

   Then repeat step 1.

3. Add the following line to the end of the file:

   ```nu
   $env.config.show_banner = false
   ```

4. Save and exit your editor.
5. Restart Nushell to test the change.

## Additional Startup Configuration

### Changing default directories

::: warning Important
As discussed below, variables in this section must be set **before** Nushell is launched.
:::

Some variables that control Nushell startup file locations must be set **before** Nushell is loaded. This is often done by a parent process such as:

- The terminal application in which Nushell is run

- The operating system or window manager. When running Nushell as a login shell, this will likely be the only mechanism available.

  For example, on Windows, you can set environment variables through the Control Panel. Choose the Start Menu and search for _"environment variables"_.

  On Linux systems using PAM, `/etc/environment` (and other system-specific mechanisms) can be used.

- A parent shell. For example, exporting the value from `bash` before running `nu`.

### Startup Variables

The variables that affect Nushell file locations are:

- `$env.XDG_CONFIG_HOME`: If this environment variable is set, it is used to change the directory that Nushell searches for its configuration files such as `env.nu`, `config.nu`, and `login.nu`. The history and plugin files are also stored in this directory by default.

  Once Nushell starts, this value is stored in the `$nu.default-config-path` constant. See [Using Constants](#using-constants) below.

- `$env.XDG_DATA_HOME`: If this environment variable is set, Nushell sets the `$nu.data-dir` constant to this value. The `data-dir` is used in several startup tasks:

  - `($nu.data-dir)/nushell/completions` is added to the `$env.NU_LIB_DIRS` search path.
  - `($nu.data-dir)/vendor/autoload` is added as the last path in `nu.vendor-autoload-dirs`. This means that files in this directory will be read last during startup (and thus override any definitions made in earlier files).

  Note that the directory represented by `$nu.data-dir`, nor any of its subdirectories, are created by default. Creation and use of these directories is up to the user.

- `$env.XDG_DATA_DIRS` _(Unix Platforms Only)_: If this environment variable is set, it is used to populate the `$nu.vendor-auto-load` directories in the order listed. The first directory in the list is processed first, meaning the last one read will have the ability to override previous definitions.

::: warning
The `XDG_*` variables are **not** Nushell-specific and should not be set to a directory with only Nushell files. Instead, set the environment variable to the directory _above_ the one with the `nushell` directory.

For example, if you set `$env.XDG_CONFIG_HOME` to:

```
/users/username/dotfiles/nushell
```

... Nushell will look for config files in `/Users/username/dotfiles/nushell/nushell`. The proper setting would be:

```
/users/username/dotfiles
```

Also keep in mind that if the system has already set `XDG` variables, then there may already be files in use in those directories. Changing the location may require that you move other application's files to the new directory.
:::

::: tip
You can easily test out config changes in a "fresh" environment using the following recipe. The following is run from inside Nushell, but can be
adapted to other shells as well:

```nu
# Create an empty temporary directory
let temp_home = (mktemp -d)
# Set the configuration path to this directory
$env.XDG_CONFIG_HOME = $temp_home
# Set the data-dir to this directory
$env.XDG_DATA_HOME = $temp_home
# Remove other potential autoload directories
$env.XDG_DATA_DIRS = ""
# Run Nushell in this environment
nu

# Edit config
config nu
# Exit the subshell
exit
# Run the temporary config
nu
```

When done testing the configuration:

```nu
# Remove the temporary config directory, if desired
rm $temp_home
```

**Important:** Then exit the parent shell so that the `XDG` changes are not accidentally propagated to other processes.
:::

### Using Constants

Some important commands, like `source` and `use`, that help define custom commands (and other definitions) are parse-time keywords. Among other things, this means means that all arguments must be known at parse-time.

In other words, **_variable arguments are not allowed for parser keywords_**.

However, Nushell creates some convenience _constants_ that can be used to help identify common file locations. For instance, you can source a file in the default configuration directory using:

```
source ($nu.default-config-dir | path join "myfile.nu")
```

Because the constant value is known at parse-time, it can be used with parse-time keywords like `source` and `use`.

:::tip See Also
See [Parse-time Constant Evaluation](./how_nushell_code_gets_run.md#parse-time-constant-evaluation) for more details on this process.
:::

#### `$nu` Constant

To see a list of the built-in Nushell constants, examine the record constant using `$nu` (including the dollar sign).

#### `NU_LIB_DIRS` Constant

Nushell can also make use of a `NU_LIB_DIRS` _constant_ which can act like the `$env.NU_LIB_DIRS` variable mentioned above. However, unlike `$env.NU_LIB_DIRS`, it can be defined _and_ used in `config.nu`. For example:

```nu
# Define module and source search path
const NU_LIB_DIRS = [
  '~/myscripts'
]
# Load myscript.nu from the ~/myscripts directory
source myscript.nu
```

If both the variable `$env.NU_LIB_DIRS` and the const `NU_LIB_DIRS` are defined, both sets
of paths will be searched. The constant `NU_LIB_DIRS` will be searched _first_ and have
precedence. If a file matching the name is found in one of those directories, the search will
stop. Otherwise, it will continue into the `$env.NU_LIB_DIRS` search path.

#### `NU_PLUGIN_DIRS` Constant

`const NU_PLUGIN_DIRS` works in the same way for the plugin search path.

The following `NU_PLUGIN_DIRS` configuration will allow plugins to be loaded from;

- The directory where the `nu` executable is located. This is typically where plugins are located in release packages.
- A directory in `$nu.data-dirs` named after the version of Nushell running (e.g. `0.100.0`).
- A `plugins` directory in your `$nu.config-path`.

```nu
const NU_PLUGIN_DIRS = [
  ($nu.current-exe | path dirname)
  ($nu.data-dir | path join 'plugins' | path join (version).version)
  ($nu.config-path | path dirname | path join 'plugins')
]
```

### Colors, Theming, and Syntax Highlighting

You can learn more about setting up colors and theming in the [associated chapter](coloring_and_theming.md).

### Configuring Nu as a Login Shell

The login shell is often responsible for setting certain environment variables which will be inherited by subshells and other processes. When setting Nushell as a user's default login shell, you'll want to make sure that the `login.nu` handles this task.

Many applications will assume a POSIX or PowerShell login shell, and will either provide instructions for modifying the system or user `profile` that is loaded by POSIX login-shells (or `.ps1` file on PowerShell systems).

As you may have noticed by now, Nushell is not a POSIX shell, nor is it PowerShell, and it won't be able to process a profile written for these. You'll need to set these values in `login.nu` instead.

To find environment variables that may need to be set through `login.nu`, examine the inherited environment from your login shell by running `nu` from within your previous login shell. Run:

```nu
$env | reject config | transpose key val | each {|r| echo $"$env.($r.key) = '($r.val)'"} | str join (char nl)
```

Look for any values that may be needed by third-party applications and copy these to your `login.nu`. Many of these will not be needed. For instance, the `PS1` setting is the current prompt in POSIX shells and won't be useful in Nushell.

When ready, add Nushell to your `/etc/shells` (Unix) and `chsh` as discussed in [the Installation Chapter](./default_shell.md).

### macOS: Keeping `/usr/bin/open` as `open`

Some tools such as Emacs rely on an [`open`](/commands/docs/open.md) command to open files on Mac.

Since Nushell has its own [`open`](/commands/docs/open.md) command with a different meaning which shadows (overrides) `/usr/bin/open`, these tools will generate an error when trying to use the command.

One way to work around this is to define a custom command for Nushell's [`open`](/commands/docs/open.md) and create an alias for the system's [`open`](/commands/docs/open.md) in your `config.nu` file like this:

```nu
alias nu-open = open
alias open = ^open
```

Place this in your `config.nu` to make it permanent.

The `^` symbol tells Nushell to run the following command as an _external_ command, rather than as a Nushell built-in. After running these commands, `nu-open` will be the Nushell _internal_ version, and the `open` alias will call the Mac, external `open` instead.

For more information, see [Running System (External) Commands](./running_externals.md).

## Detailed Configuration Startup Process

This section contains a more detailed description of how different configuration (and flag) options can be used to
change Nushell's startup behavior.

### Launch Stages

The following stages and their steps _may_ occur during startup, based on the flags that are passed to `nu`. See [Flag Behavior](#flag-behavior) immediately following this table for how each flag impacts the process:

| Step | Stage                           | Nushell Action                                                                                                                                                                                                                                                                                                                                                                     |
| ---- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.   | (misc)                          | Sets internal defaults via its internal Rust implementation. In practice, this may not take place until "first use" of the setting or variable, but there will typically be a Rust default for most (but not all) settings and variables that control Nushell's behavior. These defaults can then be superseded by the steps below.                                                |
| 1.   | (main)                          | Inherits its initial environment from the calling process. These will initially be converted to Nushell strings, but can be converted to other structures later using `ENV_CONVERSIONS` (see below).                                                                                                                                                                               |
| 2.   | (main)                          | Gets the configuration directory. This is OS-dependent (see [dirs::config_dir](https://docs.rs/dirs/latest/dirs/fn.config_dir.html)), but can be overridden using `XDG_CONFIG_HOME` on all platforms as discussed [above](#changing-default-directories).                                                                                                                          |
| 3.   | (main)                          | Creates the initial `$env.NU_LIB_DIRS` variable. By default, it includes (1) the `scripts` directory under the configuration directory, and (2) `nushell/completions` under the default data directory (either `$env.XDG_DATA_HOME` or [the default provided by the dirs crate](https://docs.rs/dirs/latest/dirs/fn.data_dir.html)). These directories are not created by default. |
| 4.   | (main)                          | Creates the initial `$env.NU_PLUGIN_DIRS` variable. By default, this will include the configuration directory.                                                                                                                                                                                                                                                                     |
| 5.   | (main)                          | Initializes the in-memory SQLite database. This allows the `stor` family of commands to be used in the following configuration files.                                                                                                                                                                                                                                              |
| 6.   | (main)                          | Processes commandline arguments such as `--plugin-config <file>`, `--plugins <list>`, and others. See `nu --help` for a complete list.                                                                                                                                                                                                                                             |
| 7.   | (main)                          | Gets the path to `env.nu` and `config.nu`. By default, these are located in the config directory, but either or both can be overridden using the `--env-config <path>` and `--config <path>` flags.                                                                                                                                                                                |
| 8.   | (main)                          | If the `--include-path (-I)` flag was used, it overrides the default `$env.NU_LIB_DIRS` that was obtained above.                                                                                                                                                                                                                                                                   |
| 9.   | (main)                          | Loads the initial `$env.config` values from the internal defaults.                                                                                                                                                                                                                                                                                                                 |
| 10.  | (stdlib)                        | Loads the [Standard Library](./standard_library.md) into the virtual filesystem. It is not parsed or evaluated at this point.                                                                                                                                                                                                                                                      |
| 11.  | (stdlib)                        | Parses and evaluates `std/core`, which brings the `banner` and `pwd` commands into scope.                                                                                                                                                                                                                                                                                          |
| 12.  | (main)                          | Generates the initial [`$nu` record constant](#using-constants) so that items such as `$nu.default-config-dir` can be used in the following config files.                                                                                                                                                                                                                          |
| 13.  | (main)                          | Loads any plugins that were specified using the `--plugin` flag.                                                                                                                                                                                                                                                                                                                   |
| 14.  | (config files) (plugin)         | Processes the signatures in the user's `plugin.msgpackz` (located in the configuration directory) so that added plugins can be used in the following config files.                                                                                                                                                                                                                 |
| 15.  | (config files)                  | If this is the first time Nushell has been launched, then it creates the configuration directory. "First launch" is determined by whether or not the configuration directory exists.                                                                                                                                                                                               |
| 16.  | (config files)                  | Also, if this is the first time Nushell has been launched, creates a mostly empty (other than a few comments) `env.nu` and `config .nu` in that directory.                                                                                                                                                                                                                         |
| 17.  | (config files) (default_env.nu) | Loads default environment variables from the internal `default_env.nu`. This file can be viewed with: `nu config env --default \| nu-highlight \| less -R`.                                                                                                                                                                                                                        |
| 18.  | (config files) (env.nu)         | Converts the `PATH` variable into a list so that it can be accessed more easily in the next step.                                                                                                                                                                                                                                                                                  |
| 19.  | (config files) (env.nu)         | Loads (parses and evaluates) the user's `env.nu` (the path to which was determined above).                                                                                                                                                                                                                                                                                         |
| 20.  | (config files) (config.nu)      | Processes any `ENV_CONVERSIONS` that were defined in the user's `env.nu` so that those environment variables can be treated as Nushell structured data in the `config.nu`.                                                                                                                                                                                                         |
| 21.  | (config files) (config.nu)      | Loads a minimal `$env.config` record from the internal `default_config.nu`. This file can be viewed with: `nu config nu --default \| nu-highlight \| less -R`. Most values that are not defined in `default_config.nu` will be auto-populated into `$env.config` using their internal defaults as well.                                                                            |
| 22.  | (config files) (config.nu)      | Loads (parses and evaluates) the user's `config.nu` (the path to which was determined above).                                                                                                                                                                                                                                                                                      |
| 23.  | (config files) (login)          | When Nushell is running as a login shell, loads the user's `login.nu`.                                                                                                                                                                                                                                                                                                             |
| 24.  | (config files)                  | Loops through autoload directories and loads any `.nu` files found. The directories are processed in the order found in `$nu.vendor-autoload-directories`, and files in those directories are processed in alphabetical order.                                                                                                                                                     |
| 25.  | (repl)                          | Processes any additional `ENV_CONVERSIONS` that were defined in `config.nu` or the autoload files.                                                                                                                                                                                                                                                                                 |
| 26.  | (repl) and (stdlib)             | Shows the banner if configured.                                                                                                                                                                                                                                                                                                                                                    |
| 27.  | (repl)                          | Nushell enters the normal commandline (REPL).                                                                                                                                                                                                                                                                                                                                      |

### Flag Behavior

| Mode                | Command/Flags                              | Behavior                                                                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Normal Shell        | `nu` (no flags)                            | All launch steps **_except_** those marked with **_(login)_** occur.                                                                                                                                                                                                                         |
| Login Shell         | `nu --login/-l`                            | All launch steps occur.                                                                                                                                                                                                                                                                      |
| Command-string      | `nu --commands <command-string>` (or `-c`) | All Launch stages **_except_** those marked with **_(config files)_** or **_(repl)_** occur. However, **_(default_env)_** and **_(plugin)_** do occur. The first allows the path `ENV_CONVERSIONS` defined there can take place. The second allows plugins to be used in the command-string. |
| Script file         | `nu <script_file>`                         | Same as with Command-string.                                                                                                                                                                                                                                                                 |
| No config           | `nu -n`                                    | **_(config files)_** stages do **_not_** occur, regardless of other flags.                                                                                                                                                                                                                   |
| No Standard Library | `nu --no-std-lib`                          | Regardless of other flags, the steps marked **_(stdlib)_** will **_not_** occur.                                                                                                                                                                                                             |
| Force config file   | `nu --config <file>`                       | Forces steps marked with **_(config.nu)_** above to run with the provided config `<file>`, unless `-n` was also specified                                                                                                                                                                    |
| Force env file      | `nu --env-config <file>`                   | Forces steps marked with **_(default_env.nu)_** and **_(env.nu)_** above to run with the specified env `<file>`, unless `-n` was also specified                                                                                                                                              |

### Scenarios

- `nu`:

  - ✅ Makes the Standard Library available
  - ✅ Reads user's `plugin.msgpackz` file if it exists in the config directory
  - ✅ Sources the `default_env.nu` file internally
  - ✅ Sources the user's `env.nu` file if it exists in the config directory
  - ✅ Sources the `default_config.nu` file internally
  - ✅ Sources user's `config.nu` file if it exists if it exists in the config directory
  - ❌ Does not read `personal login.nu` file
  - ✅ Enters the REPL

- `nu -c "ls"`:

  - ✅ Makes the Standard Library available
  - ✅ Reads user's `plugin.msgpackz` file if it exists in the config directory
  - ✅ Sources the `default_env.nu` file internally
  - ❌ Does not source the user's `env.nu`
  - ❌ Does not read the internal `default_config.nu` file
  - ❌ Does not read the user's `config.nu` file
  - ❌ Does not read the user's `login.nu` file
  - ✅ Runs the `ls` command and exits
  - ❌ Does not enter the REPL

- `nu -l -c "ls"`:

  - ✅ Makes the Standard Library available
  - ✅ Reads user's `plugin.msgpackz` file if it exists in the config directory
  - ✅ Sources the `default_env.nu` file internally
  - ✅ Sources the user's `env.nu` file if it exists in the config directory
  - ✅ Sources the `default_config.nu` file internally
  - ✅ Sources user's `config.nu` file if it exists in the config directory
  - ✅ Sources the user's `login.nu` file if it exists in the config directory
  - ✅ Runs the `ls` command and exits
  - ❌ Does not enter the REPL

- `nu -l -c "ls" --config foo_config.nu`

  - Same as above, but reads an alternative config file named `foo_config.nu` from the config directory

- `nu -n -l -c "ls"`:

  - ✅ Makes the Standard Library available
  - ❌ Does not read user's `plugin.msgpackz`
  - ❌ Does not read the internal `default_env.nu`
  - ❌ Does not source the user's `env.nu`
  - ❌ Does not read the internal `default_config.nu` file
  - ❌ Does not read the user's `config.nu` file
  - ❌ Does not read the user's `login.nu` file
  - ✅ Runs the `ls` command and exits
  - ❌ Does not enter the REPL

- `nu test.nu`:

  - ✅ Makes the Standard Library available
  - ✅ Reads user's `plugin.msgpackz` file if it exists in the config directory
  - ✅ Sources the `default_env.nu` file internally
  - ❌ Does not source the user's `env.nu`
  - ❌ Does not read the internal `default_config.nu` file
  - ❌ Does not read the user's `config.nu` file
  - ❌ Does not read the user's `login.nu` file
  - ✅ Runs `test.nu` file as a script
  - ❌ Does not enter the REPL

- `nu --config foo_config.nu test.nu`

  - ✅ Makes the Standard Library available
  - ✅ Reads user's `plugin.msgpackz` file if it exists in the config directory
  - ✅ Sources the `default_env.nu` file internally
  - ❌ Does not source the user's `env.nu` (no `--env-config` was specified)
  - ✅ Sources the `default_config.nu` file internally. Note that `default_config.nu` is always handled before a user's config
  - ✅ Sources user's `config.nu` file if it exists in the config directory
  - ❌ Does not read the user's `login.nu` file
  - ✅ Runs `test.nu` file as a script
  - ❌ Does not enter the REPL

- `nu -n --no-std-lib` (fastest REPL startup):

  - ❌ Does not make the Standard Library available
  - ❌ Does not read user's `plugin.msgpackz`
  - ❌ Does not read the internal `default_env.nu`
  - ❌ Does not source the user's `env.nu`
  - ❌ Does not read the internal `default_config.nu` file
  - ❌ Does not read the user's `config.nu` file
  - ❌ Does not read the user's `login.nu` file
  - ✅ Enters the REPL

- `nu -n --no-std-lib -c "ls"` (fastest command-string invocation):

  - ❌ Does not make the Standard Library available
  - ❌ Does not read user's `plugin.msgpackz`
  - ❌ Does not read the internal `default_env.nu`
  - ❌ Does not source the user's `env.nu`
  - ❌ Does not read the internal `default_config.nu` file
  - ❌ Does not read the user's `config.nu` file
  - ❌ Does not read the user's `login.nu` file
  - ✅ Runs the `ls` command and exits
  - ❌ Does not enter the REPL
# Control Flow

Nushell provides several commands that help determine how different groups of code are executed. In programming languages this functionality is often referred to as _control flow_.

::: tip
One thing to note is that all of the commands discussed on this page use [blocks](/book/types_of_data.html#blocks). This means you can mutate [environmental variables](/book/environment.html) and other [mutable variables](/book/variables.html#mutable-variables) in them.
:::

## Already covered

Below we cover some commands related to control flow, but before we get to them, it's worthwhile to note there are several pieces of functionality and concepts that have already been covered in other sections that are also related to control flow or that can be used in the same situations. These include:

- Pipes on the [pipelines](/book/pipelines.html) page.
- Closures on the [types of data](/book/types_of_data.html) page.
- Iteration commands on the [working with lists](/book/working_with_lists.html) page. Such as:
  - [`each`](/commands/docs/each.html)
  - [`where`](/commands/docs/where.html)
  - [`reduce`](/commands/docs/reduce.html)

## Choice (Conditionals)

The following commands execute code based on some given condition.

::: tip
The choice/conditional commands are expressions so they return values, unlike the other commands on this page. This means the following works.

```nu
'foo' | if $in == 'foo' { 1 } else { 0 } | $in + 2
# => 3
```

:::

### `if`

[`if`](/commands/docs/if.html) evaluates branching [blocks](/book/types_of_data.html#blocks) of code based on the results of one or more conditions similar to the "if" functionality in other programming languages. For example:

```nu
if $x > 0 { 'positive' }
```

Returns `'positive`' when the condition is `true` (`$x` is greater than zero) and `null` when the condition is `false` (`$x` is less than or equal to zero).

We can add an `else` branch to the `if` after the first block which executes and returns the resulting value from the `else` block when the condition is `false`. For example:

```nu
if $x > 0 { 'positive' } else { 'non-positive' }
```

This time it returns `'positive'` when the condition is `true` (`$x` is greater than zero) and `'non-positive`' when the condition is `false` (`$x` is less than or equal to zero).

We can also chain multiple `if`s together like the following:

```nu
if $x > 0 { 'positive' } else if $x == 0 { 'zero' } else { "negative" }
```

When the first condition is `true` (`$x` is greater than zero) it will return `'positive'`, when the first condition is `false` and the next condition is `true` (`$x` equals zero) it will return `'zero'`, otherwise it will return `'negative'` (when `$x` is less than zero).

### `match`

[`match`](/commands/docs/match.html) executes one of several conditional branches based on the value given to match. You can also do some [pattern matching](/cookbook/pattern_matching.html) to unpack values in composite types like lists and records.

Basic usage of [`match`](/commands/docs/match.html) can conditionally run different code like a "switch" statement common in other languages. [`match`](/commands/docs/match.html) checks if the value after the word [`match`](/commands/docs/match.html) is equal to the value at the start of each branch before the `=>` and if it does, it executes the code after that branch's `=>`.

```nu
match 3 {
    1 => 'one',
    2 => {
        let w = 'w'
        't' + $w + 'o'
    },
    3 => 'three',
    4 => 'four'
}
# => three
```

The branches can either return a single value or, as shown in the second branch, can return the results of a [block](/book/types_of_data.html#blocks).

#### Catch all Branch

You can also have a catch all condition for when the given value doesn't match any of the other conditions by having a branch whose matching value is `_`.

```nu
let foo = match 7 {
    1 => 'one',
    2 => 'two',
    3 => 'three',
    _ => 'other number'
}
$foo
# => other number
```

(Reminder, [`match`](/commands/docs/match.html) is an expression which is why we can assign the result to `$foo` here).

#### Pattern Matching

You can "unpack" values from types like lists and records with [pattern matching](/cookbook/pattern_matching.html). You can then assign variables to the parts you want to unpack and use them in the matched expressions.

```nu
let foo = { name: 'bar', count: 7 }
match $foo {
    { name: 'bar', count: $it } => ($it + 3),
    { name: _, count: $it } => ($it + 7),
    _ => 1
}
# => 10
```

The `_` in the second branch means it matches any record with field `name` and `count`, not just ones where `name` is `'bar'`.

#### Guards

You can also add an additional condition to each branch called a "guard" to determine if the branch should be matched. To do so, after the matched pattern put `if` and then the condition before the `=>`.

```nu
let foo = { name: 'bar', count: 7 }
match $foo {
    { name: 'bar', count: $it } if $it < 5 => ($it + 3),
    { name: 'bar', count: $it } if $it >= 5 => ($it + 7),
    _ => 1
}
# => 14
```

---

You can find more details about [`match`](/commands/docs/match.html) in the [pattern matching cookbook page](https://www.nushell.sh/cookbook/pattern_matching.html).

## Loops

The loop commands allow you to repeat a block of code multiple times.

### Loops and Other Iterating Commands

The functionality of the loop commands is similar to commands that apply a closure over elements in a list or table like [`each`](/commands/docs/each.html) or [`where`](/commands/docs/where.html) and many times you can accomplish the same thing with either. For example:

```nu
mut result = []
for $it in [1 2 3] { $result = ($result | append ($it + 1)) }
$result
# => ╭───┬───╮
# => │ 0 │ 2 │
# => │ 1 │ 3 │
# => │ 2 │ 4 │
# => ╰───┴───╯


[1 2 3] | each { $in + 1 }
# => ╭───┬───╮
# => │ 0 │ 2 │
# => │ 1 │ 3 │
# => │ 2 │ 4 │
# => ╰───┴───╯
```

While it may be tempting to use loops if you're familiar with them in other languages, it is considered more in the [Nushell-style](book/thinking_in_nu.html) (idiomatic) to use commands that apply closures when you can solve a problem either way. The reason for this is because of a pretty big downside with using loops.

#### Loop Disadvantages

The biggest downside of loops is that they are statements, unlike [`each`](/commands/docs/each.html) which is an expression. Expressions, like [`each`](/commands/docs/each.html) always result in some output value, however statements do not.

This means that they don't work well with immutable variables and using immutable variables is considered a more [Nushell-style](/book/thinking_in_nu.html#variables-are-immutable). Without a mutable variable declared beforehand in the example in the previous section, it would be impossible to use [`for`](/commands/docs/each.html) to get the list of numbers with incremented numbers, or any value at all.

Statements also don't work in Nushell pipelines which require some output. In fact Nushell will give an error if you try:

```nu
[1 2 3] | for x in $in { $x + 1 } | $in ++ [5 6 7]
# => Error: nu::parser::unexpected_keyword
# => 
# =>   × Statement used in pipeline.
# =>    ╭─[entry #5:1:1]
# =>  1 │ [1 2 3] | for x in $in { $x + 1 } | $in ++ [5 6 7]
# =>    ·           ─┬─
# =>    ·            ╰── not allowed in pipeline
# =>    ╰────
# =>   help: 'for' keyword is not allowed in pipeline. Use 'for' by itself, outside of a pipeline.
```

Because Nushell is very pipeline oriented, this means using expression commands like [`each`](/commands/docs/each.html) is typically more natural than loop statements.

#### Loop Advantages

If loops have such a big disadvantage, why do they exist? Well, one reason is that closures, like [`each`](/commands/docs/each.html) uses, can't modify mutable variables in the surrounding environment. If you try to modify a mutable variable in a closure you will get an error:

```nu
mut foo = []
[1 2 3] | each { $foo = ($foo | append ($in + 1)) }
# => Error: nu::parser::expected_keyword
# => 
# =>   × Capture of mutable variable.
# =>    ╭─[entry #8:1:1]
# =>  1 │ [1 2 3] | each { $foo = ($foo | append ($in + 1)) }
# =>    ·                  ──┬─
# =>    ·                    ╰── capture of mutable variable
# =>    ╰────
```

If you modify an environmental variable in a closure, you can, but it will only modify it within the scope of the closure, leaving it unchanged everywhere else. Loops, however, use [blocks](/book/types_of_data.html#blocks) which means they can modify a regular mutable variable or an environmental variable within the larger scope.

```nu
mut result = []
for $it in [1 2 3] { $result = ($result | append ($it + 1)) }
$result
# => ╭───┬───╮
# => │ 0 │ 2 │
# => │ 1 │ 3 │
# => │ 2 │ 4 │
# => ╰───┴───╯
```

### `for`

[`for`](/commands/docs/for.html) loops over a range or collection like a list or a table.

```nu
for x in [1 2 3] { $x * $x | print }
# => 1
# => 4
# => 9
```

#### Expression Command Alternatives

- [`each`](/commands/docs/each.html)
- [`par-each`](/commands/docs/par-each.html)
- [`where`](/commands/docs/where.html)/[`filter`](/commands/docs/filter.html)
- [`reduce`](/commands/docs/reduce.html)

### `while`

[`while`](/commands/docs/while.html) loops the same block of code until the given condition is `false`.

```nu
mut x = 0; while $x < 10 { $x = $x + 1 }; $x
# => 10
```

#### Expression Command Alternatives

The "until" and other "while" commands

- [`take until`](/commands/docs/take_until.html)
- [`take while`](/commands/docs/take_while.html)
- [`skip until`](/commands/docs/skip_until.html)
- [`skip while`](/commands/docs/skip_while.html)

### `loop`

[`loop`](/commands/docs/loop.html) loops a block infinitely. You can use [`break`](/commands/docs/break.html) (as described in the next section) to limit how many times it loops. It can also be handy for continuously running scripts, like an interactive prompt.

```nu
mut x = 0; loop { if $x > 10 { break }; $x = $x + 1 }; $x
# => 11
```

### `break`

[`break`](/commands/docs/break.html) will stop executing the code in a loop and resume execution after the loop. Effectively "break"ing out of the loop.

```nu
for x in 1..10 { if $x > 3 { break }; print $x }
# => 1
# => 2
# => 3
```

### `continue`

[`continue`](/commands/docs/continue.html) will stop execution of the current loop, skipping the rest of the code in the loop, and will go to the next loop. If the loop would normally end, like if [`for`](/commands/docs/for.html) has iterated through all the given elements, or if [`while`](/commands/docs/while.html)'s condition is now false, it won't loop again and execution will continue after the loop block.

```nu
mut x = -1; while $x <= 6 { $x = $x + 1; if $x mod 3 == 0 { continue }; print $x }
# => 1
# => 2
# => 4
# => 5
# => 7
```

## Errors

### `error make`

[`error make`](/commands/docs/error_make.html) creates an error that stops execution of the code and any code that called it, until either it is handled by a [`try`](/commands/docs/try.html) block, or it ends the script and outputs the error message. This functionality is the same as "exceptions" in other languages.

```nu
print 'printed'; error make { msg: 'Some error info' }; print 'unprinted'
# => printed
# => Error:   × Some error info
# =>    ╭─[entry #9:1:1]
# =>  1 │ print 'printed'; error make { msg: 'Some error info' }; print 'unprinted'
# =>    ·                  ─────┬────
# =>    ·                       ╰── originates from here
# =>    ╰────
```

The record passed to it provides some information to the code that catches it or the resulting error message.

You can find more information about [`error make`](/commands/docs/error_make.html) and error concepts on the [Creating your own errors page](/book/creating_errors.html).

### `try`

[`try`](/commands/docs/try.html) will catch errors created anywhere in the [`try`](/commands/docs/try.html)'s code block and resume execution of the code after the block.

```nu
try { error make { msg: 'Some error info' }}; print 'Resuming'
# => Resuming
```

This includes catching built in errors.

```nu
try { 1 / 0 }; print 'Resuming'
# => Resuming
```

The resulting value will be `nothing` if an error occurs and the returned value of the block if an error did not occur.

If you include a `catch` block after the [`try`](/commands/docs/try.html) block, it will execute the code in the `catch` block if an error occurred in the [`try`](/commands/docs/try.html) block.

```nu
try { 1 / 0 } catch { 'An error happened!' } | $in ++ ' And now I am resuming.'
# => An error happened! And now I am resuming.
```

It will not execute the `catch` block if an error did not occur.

## Other

### `return`

[`return`](/commands/docs/return.html) Ends a closure or command early where it is called, without running the rest of the command/closure, and returns the given value. Not often necessary since the last value in a closure or command is also returned, but it can sometimes be convenient.

```nu
def 'positive-check' [it] {
    if $it > 0 {
        return 'positive'
    };

    'non-positive'
}
```

```nu
positive-check 3
# => positive

positive-check (-3)
# => non-positive

let positive_check = {|elt| if $elt > 0 { return 'positive' }; 'non-positive' }

do $positive_check 3
# => positive

do $positive_check (-3)
# => non-positive
```
# Creating Your Own Errors

Using the [metadata](metadata.md) information, you can create your own custom error messages. Error messages are built of multiple parts:

- The title of the error
- The label of error message, which includes both the text of the label and the span to underline

You can use the [`error make`](/commands/docs/error_make.md) command to create your own error messages. For example, let's say you had your own command called `my-command` and you wanted to give an error back to the caller about something wrong with a parameter that was passed in.

First, you can take the span of where the argument is coming from:

```nu
let span = (metadata $x).span;
```

Next, you can create an error using the [`error make`](/commands/docs/error_make.md) command. This command takes in a record that describes the error to create:

```nu
error make {msg: "this is fishy", label: {text: "fish right here", span: $span } }
```

Together with your custom command, it might look like this:

```nu
def my-command [x] {
    let span = (metadata $x).span;
    error make {
        msg: "this is fishy",
        label: {
            text: "fish right here",
            span: $span
        }
    }
}
```

When called with a value, we'll now see an error message returned:

```nu
my-command 100
# => Error:
# =>   × this is fishy
# =>    ╭─[entry #5:1:1]
# =>  1 │ my-command 100
# =>    ·            ─┬─
# =>    ·             ╰── fish right here
# =>    ╰────
```
# Custom Commands

As with any programming language, you'll quickly want to save longer pipelines and expressions so that you can call them again easily when needed.

This is where custom commands come in.

::: tip Note
Custom commands are similar to functions in many languages, but in Nushell, custom commands _act as first-class commands themselves_. As you'll see below, they are included in the Help system along with built-in commands, can be a part of a pipeline, are parsed in real-time for type errors, and much more.
:::

[[toc]]

## Creating and Running a Custom Command

Let's start with a simple `greet` custom command:

```nu
def greet [name] {
  $"Hello, ($name)!"
}
```

Here, we define the `greet` command, which takes a single parameter `name`. Following this parameter is the block that represents what will happen when the custom command runs. When called, the custom command will set the value passed for `name` as the `$name` variable, which will be available to the block.

To run this command, we can call it just as we would call built-in commands:

```nu
greet "World"
# => Hello, World!
```

## Returning Values from Commands

You might notice that there isn't a `return` or `echo` statement in the example above.

Like some other languages, such as PowerShell and JavaScript (with arrow functions), Nushell features an _implicit return_, where the value of the final expression in the command becomes its return value.

In the above example, there is only one expression—The string. This string becomes the return value of the command.

```nu
greet "World" | describe
# => string
```

A typical command, of course, will be made up of multiple expressions. For demonstration purposes, here's a non-sensical command that has 3 expressions:

```nu
def eight [] {
  1 + 1
  2 + 2
  4 + 4
}

eight
# => 8
```

The return value, again, is simply the result of the _final_ expression in the command, which is `4 + 4` (8).

Additional examples:

::: details Early return
Commands that need to exit early due to some condition can still return a value using the [`return` statement](/commands/docs/return.md).

```nu
def process-list [] {
  let input_length = length
  if $input_length > 10_000 {
    print "Input list is too long"
    return null
  }

  $in | each {|i|
    # Process the list
    $i * 4.25
  }
}
```

:::

::: details Suppressing the return value
You'll often want to create a custom command that acts as a _statement_ rather than an expression, and doesn't return a a value.

You can use the `ignore` keyword in this case:

```nu
def create-three-files [] {
  [ file1 file2 file3 ] | each {|filename|
    touch $filename
  } | ignore
}
```

Without the `ignore` at the end of the pipeline, the command will return an empty list from the `each` statement.

You could also return a `null` as the final expression. Or, in this contrived example, use a `for` statement, which doesn't return a value (see next example).
:::

::: details Statements which don't return a value
Some keywords in Nushell are _statements_ which don't return a value. If you use one of these statements as the final expression of a custom command, the _return value_ will be `null`. This may be unexpected in some cases. For example:

```nu
def exponents-of-three [] {
  for x in [ 0 1 2 3 4 5 ] {
    3 ** $x
  }
}
exponents-of-three
```

The above command will not display anything, and the return value is empty, or `null` because `for` is a _statement_ which doesn't return a value.

To return a value from an input list, use a filter such as the `each` command:

````nu
def exponents-of-three [] {
  [ 0 1 2 3 4 5 ] | each {|x|
    3 ** $x
  }
}

exponents-of-three

# => ╭───┬─────╮
# => │ 0 │   1 │
# => │ 1 │   3 │
# => │ 2 │   9 │
# => │ 3 │  27 │
# => │ 4 │  81 │
# => │ 5 │ 243 │
# => ╰───┴─────╯
:::

::: details Match expression
```nu
# Return a random file in the current directory
def "random file" [] {
  let files = (ls)
  let num_files = ($files | length)

  match $num_files {
    0 => null  # Return null for empty directory
    _ => {
      let random_file = (random int 0..($num_files - 1))
      ($files | get $random_file)
    }
  }
}
````

In this case, the final expression is the `match` statement which can return:

- `null` if the directory is empty
- Otherwise, a `record` representing the randomly chosen file
  :::

## Custom Commands and Pipelines

Just as with built-in commands, the return value of a custom command can be passed into the next command in a pipeline. Custom commands can also accept pipeline input. In addition, whenever possible, pipeline input and output is streamed as it becomes available.

::: tip Important!
See also: [Pipelines](./pipelines.html)
:::

### Pipeline Output

```nu
ls | get name
```

Let's move [`ls`](/commands/docs/ls.md) into a command that we've written:

```nu
def my-ls [] { ls }
```

We can use the output from this command just as we would [`ls`](/commands/docs/ls.md).

```nu
my-ls | get name
# => ╭───┬───────────────────────╮
# => │ 0 │ myscript.nu           │
# => │ 1 │ myscript2.nu          │
# => │ 2 │ welcome_to_nushell.md │
# => ╰───┴───────────────────────╯
```

This lets us easily build custom commands and process their output. Remember that we don't use return statements like other languages. Instead, the [implicit return](#returning-values-from-a-command) allows us to build pipelines that output streams of data that can be connected to other pipelines.

::: tip Note
The `ls` content is still streamed in this case, even though it is in a separate command. Running this command against a long-directory on a slow (e.g., networked) filesystem would return rows as they became available.
:::

### Pipeline Input

Custom commands can also take input from the pipeline, just like other commands. This input is automatically passed to the custom command's block.

Let's make our own command that doubles every value it receives as input:

```nu
def double [] {
  each { |num| 2 * $num }
}
```

Now, if we call the above command later in a pipeline, we can see what it does with the input:

```nu
[1 2 3] | double
# => ╭───┬───╮
# => │ 0 │ 2 │
# => │ 1 │ 4 │
# => │ 2 │ 6 │
# => ╰───┴───╯
```

::: tip Cool!
This command demonstrates both input and output _streaming_. Try running it with an infinite input:

```nu
1.. | each {||} | double
```

Even though the input command hasn't ended, the `double` command can still receive and output values as they become available.

Press <kbd>Ctrl</kbd>+<kbd>C</kbd> to stop the command.
:::

We can also store the input for later use using the [`$in` variable](pipelines.html#pipeline-input-and-the-special-in-variable):

```nu
def nullify [...cols] {
  let start = $in
  $cols | reduce --fold $start { |col, table|
    $table | upsert $col null
  }
}

ls | nullify name size
# => ╭───┬──────┬──────┬──────┬───────────────╮
# => │ # │ name │ type │ size │   modified    │
# => ├───┼──────┼──────┼──────┼───────────────┤
# => │ 0 │      │ file │      │ 8 minutes ago │
# => │ 1 │      │ file │      │ 8 minutes ago │
# => │ 2 │      │ file │      │ 8 minutes ago │
# => ╰───┴──────┴──────┴──────┴───────────────╯
```

## Naming Commands

In Nushell, a command name can be a string of characters. Here are some examples of valid command names: `greet`, `get-size`, `mycommand123`, `my command`, `命令` (English translation: "command"), and even `😊`.

Strings which might be confused with other parser patterns should be avoided. For instance, the following command names might not be callable:

- `1`, `"1"`, or `"1.5"`: Nushell will not allow numbers to be used as command names
- `4MiB` or `"4MiB"`: Nushell will not allow filesizes to be use₫ as command names
- `"number#four"` or `"number^four"`: Carets and hash symbols are not allowed in command names
- `-a`, `"{foo}"`, `"(bar)"`: Will not be callable, as Nushell will interpret them as flags, closures, or expressions.

While names like `"+foo"` might work, they are best avoided as the parser rules might change over time. When in doubt, keep command names as simple as possible.

::: tip
It's common practice in Nushell to separate the words of the command with `-` for better readability. For example `get-size` instead of `getsize` or `get_size`.
:::

::: tip
Because `def` is a parser keyword, the command name must be known at parse time. This means that command names may not be a variable or constant. For example, the following is _not allowed_:

```nu
let name = "foo"
def $name [] { foo }
```

:::

### Subcommands

You can also define subcommands of commands using a space. For example, if we wanted to add a new subcommand to [`str`](/commands/docs/str.md), we can create it by naming our subcommand starting with "str ". For example:

```nu
def "str mycommand" [] {
  "hello"
}
```

Now we can call our custom command as if it were a built-in subcommand of [`str`](/commands/docs/str.md):

```nu
str mycommand
```

Of course, commands with spaces in their names are defined in the same way:

```nu
def "custom command" [] {
  "This is a custom command with a space in the name!"
}
```

## Parameters

### Multiple parameters

In the `def` command, the parameters are defined in a [`list`](./types_of_data.md#lists). This means that multiple parameters can be separated with spaces, commas, or line-breaks.

For example, here's a version of `greet` that accepts two names. Any of these three definitions will work:

```nu
# Spaces
def greet [name1 name2] {
  $"Hello, ($name1) and ($name2)!"
}

# Commas
def greet [name1, name2] {
  $"Hello, ($name1) and ($name2)!"
}

# Linebreaks
def greet [
  name1
  name2
] {
  $"Hello, ($name1) and ($name2)!"
}
```

### Required positional parameters

The basic argument definitions used above are _positional_. The first argument passed into the `greet` command above is assigned to the `name1` parameter (and, as mentioned above, the `$name1` variable). The second argument becomes the `name2` parameter and the `$name2` variable.

By default, positional parameters are _required_. Using our previous definition of `greet` with two required, positional parameters:

```nu
def greet [name1, name2] {
  $"Hello, ($name1) and ($name2)!"
}

greet Wei Mei
# => Hello, Wei and Mei!

greet Wei
# => Error: nu::parser::missing_positional
# =>
# =>
# => × Missing required positional argument.
# => ╭─[entry #10:1:10]
# => 1 │ greet Mei
# => ╰────
# => help: Usage: greet <name1> <name2> . Use `--help` for more information.
```

::: tip
Try typing a third name after this version of `greet`. Notice that the parser automatically detects the error and highlights the third argument as an error even before execution.
:::

### Optional Positional Parameters

We can define a positional parameter as optional by putting a question mark (`?`) after its name. For example:

```nu
def greet [name?: string] {
  $"Hello, ($name | default 'You')"
}

greet

# => Hello, You
```

::: tip
Notice that the name used to access the variable does not include the `?`; only its definition in the command signature.
:::

When an optional parameter is not passed, its value in the command body is equal to `null`. The above example uses the `default` command to provide a default of "You" when `name` is `null`.

You could also compare the value directly:

```nu
def greet [name?: string] {
  match $name {
    null => "Hello! I don't know your name!"
    _ => $"Hello, ($name)!"
  }
}

greet

# => Hello! I don't know your name!
```

If required and optional positional parameters are used together, then the required parameters must appear in the definition first.

#### Parameters with a Default Value

You can also set a default value for the parameter when it is missing. Parameters with a default value are also optional when calling the command.

```nu
def greet [name = "Nushell"] {
  $"Hello, ($name)!"
}
```

You can call this command either without the parameter or with a value to override the default value:

```nu
greet
# => Hello, Nushell!

greet world
# => Hello, World!
```

You can also combine a default value with a [type annotation](#parameter-types):

```nu
def congratulate [age: int = 18] {
  $"Happy birthday! You are ($age) years old now!"
}
```

### Parameter Types

For each parameter, you can optionally define its type. For example, you can write the basic `greet` command as:

```nu
def greet [name: string] {
  $"Hello, ($name)"
}
```

If a parameter is not type-annotated, Nushell will treat it as an [`any` type](./types_of_data.html#any). If you annotate a type on a parameter, Nushell will check its type when you call the function.

For example, let's say you wanted to only accept an `int` instead of a `string`:

```nu
def greet [name: int] {
  $"hello ($name)"
}

greet World
```

If we try to run the above, Nushell will tell us that the types don't match:

```nu
error: Type Error
  ┌─ shell:6:7
  │
5 │ greet world
  │       ^^^^^ Expected int
```

::: tip Cool!
Type checks are a parser feature. When entering a custom command at the command-line, the Nushell parser can even detect invalid argument types in real-time and highlight them before executing the command.

The highlight style can be changed using a [theme](https://github.com/nushell/nu_scripts/tree/main/themes) or manually using `$env.config.color_config.shape_garbage`.
:::

::: details List of Type Annotations
Most types can be used as type-annotations. In addition, there are a few "shapes" which can be used. For instance:

- `number`: Accepts either an `int` or a `float`
- `path`: A string where the `~` and `.` characters have special meaning and will automatically be expanded to the full-path equivalent. See [Path](/lang-guide/chapters/types/other_types/path.html) in the Language Reference Guide for example usage.
- `directory`: A subset of `path` (above). Only directories will be offered when using tab-completion for the parameter. Expansions take place just as with `path`.
- `error`: Available, but currently no known valid usage. See [Error](/lang-guide/chapters/types/other_types/error.html) in the Language Reference Guide for more information.

The following [types](./types_of_data.html) can be used for parameter annotations:

- `any`
- `binary`
- `bool`
- `cell-path`
- `closure`
- `datetime`
- `duration`
- `filesize`
- `float`
- `glob`
- `int`
- `list`
- `nothing`
- `range`
- `record`
- `string`
- `table`
  :::

### Flags

In addition to positional parameters, you can also define named flags.

For example:

```nu
def greet [
  name: string
  --age: int
] {
    {
      name: $name
      age: $age
    }
}
```

In this version of `greet`, we define the `name` positional parameter as well as an `age` flag. The positional parameter (since it doesn't have a `?`) is required. The named flag is optional. Calling the command without the `--age` flag will set `$age` to `null`.

The `--age` flag can go before or after the positional `name`. Examples:

```nu
greet Lucia --age 23
# => ╭──────┬───────╮
# => │ name │ Lucia │
# => │ age  │ 23    │
# => ╰──────┴───────╯

greet --age 39 Ali
# => ╭──────┬─────╮
# => │ name │ Ali │
# => │ age  │ 39  │
# => ╰──────┴─────╯

greet World
# => ╭──────┬───────╮
# => │ name │ World │
# => │ age  │       │
# => ╰──────┴───────╯
```

Flags can also be defined with a shorthand version. This allows you to pass a simpler flag as well as a longhand, easier-to-read flag.

Let's extend the previous example to use a shorthand flag for the `age` value:

```nu
def greet [
  name: string
  --age (-a): int
] {
    {
      name: $name
      age: $age
    }
  }
```

::: tip
The resulting variable is always based on the long flag name. In the above example, the variable continues to be `$age`. `$a` would not be valid.
:::

Now, we can call this updated definition using the shorthand flag:

```nu
greet Akosua -a 35
# => ╭──────┬────────╮
# => │ name │ Akosua │
# => │ age  │ 35     │
# => ╰──────┴────────╯
```

Flags can also be used as basic switches. When present, the variable based on the switch is `true`. When absent, it is `false`.

```nu
def greet [
  name: string
  --caps
] {
    let greeting = $"Hello, ($name)!"
    if $caps {
      $greeting | str upcase
    } else {
      $greeting
    }
}

greet Miguel --caps
# => HELLO, MIGUEL!

greet Chukwuemeka
# => Hello, Chukwuemeka!
```

You can also assign it to `true`/`false` to enable/disable the flag:

```nu
greet Giulia --caps=false
# => Hello, Giulia!

greet Hiroshi --caps=true
# => HELLO, HIROSHI!
```

::: tip
Be careful of the following mistake:

```nu
greet Gabriel --caps true
```

Typing a space instead of an equals sign will pass `true` as a positional argument, which is likely not the desired result!

To avoid confusion, annotating a boolean type on a flag is not allowed:

```nu
def greet [
    --caps: bool   # Not allowed
] { ... }
```

:::

Flags can contain dashes. They can be accessed by replacing the dash with an underscore in the resulting variable name:

```nu
def greet [
  name: string
  --all-caps
] {
    let greeting = $"Hello, ($name)!"
    if $all_caps {
      $greeting | str upcase
    } else {
      $greeting
    }
}
```

### Rest parameters

There may be cases when you want to define a command which takes any number of positional arguments. We can do this with a "rest" parameter, using the following `...` syntax:

```nu
def multi-greet [...names: string] {
  for $name in $names {
    print $"Hello, ($name)!"
  }
}

multi-greet Elin Lars Erik
# => Hello, Elin!
# => Hello, Lars!
# => Hello, Erik!
```

We could call the above definition of the `greet` command with any number of arguments, including none at all. All of the arguments are collected into `$names` as a list.

Rest parameters can be used together with positional parameters:

```nu
def vip-greet [vip: string, ...names: string] {
  for $name in $names {
    print $"Hello, ($name)!"
  }

  print $"And a special welcome to our VIP today, ($vip)!"
}

#         $vip          $name
#         ----- -------------------------
vip-greet Rahul Priya Arjun Anjali Vikram
# => Hello, Priya!
# => Hello, Arjun!
# => Hello, Anjali!
# => Hello, Vikram!
# => And a special welcome to our VIP today, Rahul!
```

To pass a list to a rest parameter, you can use the [spread operator](/book/operators#spread-operator) (`...`). Using the `vip-greet` command definition above:

```nu
let vip = "Tanisha"
let guests = [ Dwayne, Shanice, Jerome ]
vip-greet $vip ...$guests
# => Hello, Dwayne!
# => Hello, Shanice!
# => Hello, Jerome!
# => And a special welcome to our VIP today, Tanisha!
```

## Pipeline Input-Output Signature

By default, custom commands accept [`<any>` type](./types_of_data.md#any) as pipeline input and likewise can output `<any>` type. But custom commands can also be given explicit signatures to narrow the types allowed.

For example, the signature for [`str stats`](/commands/docs/str_stats.md) looks like this:

```nu
def "str stats" []: string -> record { }
```

Here, `string -> record` defines the allowed types of the _pipeline input and output_ of the command:

- It accepts a `string` as pipeline input
- It outputs a `record`

If there are multiple input/output types, they can be placed within brackets and separated with commas or newlines, as in [`str join`](/commands/docs/str_join.md):

```nu
def "str join" [separator?: string]: [
  list -> string
  string -> string
] { }
```

This indicates that `str join` can accept either a `list<any>` or a `string` as pipeline input. In either case, it will output a `string`.

Some commands don't accept or require data as pipeline input. In this case, the input type will be `<nothing>`. The same is true for the output type if the command returns `null` (e.g., [`rm`](/commands/docs/rm.md) or [`hide`](/commands/docs/hide.md)):

```nu
def xhide [module: string, members?]: nothing -> nothing { }
```

::: tip Note
The example above is renamed `xhide` so that copying it to the REPL will not shadow the built-in `hide` command.
:::

Input-output signatures are shown in the `help` for a command (both built-in and custom) and can also be introspected through:

```nu
help commands | where name == <command_name>
scope commands | where name == <command_name>
```

:::tip Cool!
Input-Output signatures allow Nushell to catch two additional categories of errors at parse-time:

- Attempting to return the wrong type from a command. For example:

  ```nu
    def inc []: int -> int {
    $in + 1
    print "Did it!"
  }

  # => Error: nu::parser::output_type_mismatch
  # =>
  # => × Command output doesn't match int.
  # => ╭─[entry #12:1:24]
  # => 1 │ ╭─▶ def inc []: int -> int {
  # => 2 │ │     $in + 1
  # => 3 │ │     print "Did it!"
  # => 4 │ ├─▶ }
  # => · ╰──── expected int, but command outputs nothing
  # => ╰────
  ```

- And attempting to pass an invalid type into a command:

  ```nu
  def inc []: int -> int { $in + 1 }
  "Hi" | inc
  # => Error: nu::parser::input_type_mismatch
  # =>
  # =>     × Command does not support string input.
  # =>      ╭─[entry #16:1:8]
  # =>    1 │ "Hi" | inc
  # =>      ·        ─┬─
  # =>      ·         ╰── command doesn't support string input
  # =>      ╰────
  ```

  :::

## Documenting Your Command

In order to best help users understand how to use your custom commands, you can also document them with additional descriptions for the commands and parameters.

Run `help vip-greet` to examine our most recent command defined above:

```text
Usage:
  > vip-greet <vip> ...(names)

Flags:
  -h, --help - Display the help message for this command

Parameters:
  vip <string>
  ...names <string>

Input/output types:
  ╭───┬───────┬────────╮
  │ # │ input │ output │
  ├───┼───────┼────────┤
  │ 0 │ any   │ any    │
  ╰───┴───────┴────────╯
```

::: tip Cool!
You can see that Nushell automatically created some basic help for the command simply based on our definition so far. Nushell also automatically adds a `--help`/`-h` flag to the command, so users can also access the help using `vip-greet --help`.
:::

We can extend the help further with some simple comments describing the command and its parameters:

```nu
# Greet guests along with a VIP
#
# Use for birthdays, graduation parties,
# retirements, and any other event which
# celebrates an event # for a particular
# person.
def vip-greet [
  vip: string        # The special guest
   ...names: string  # The other guests
] {
  for $name in $names {
    print $"Hello, ($name)!"
  }

  print $"And a special welcome to our VIP today, ($vip)!"
}
```

Now run `help vip-greet` again to see the difference:

```text
Greet guests along with a VIP

Use for birthdays, graduation parties,
retirements, and any other event which
celebrates an event # for a particular
person.

Category: default

This command:
- does not create a scope.
- is not a built-in command.
- is not a subcommand.
- is not part of a plugin.
- is a custom command.
- is not a keyword.

Usage:
  > vip-greet <vip>


Flags:


  -h, --help - Display the help message for this command

Signatures:

  <any> | vip-greet[ <string>] -> <any>

Parameters:

  vip: <string> The special guest
  ...rest: <string> The other guests
```

Notice that the comments on the lines immediately before the `def` statement become a description of the command in the help system. Multiple lines of comments can be used. The first line (before the blank-comment line) becomes the Help `description`. This information is also shown when tab-completing commands.

The remaining comment lines become its `extra_description` in the help data.

::: tip
Run:

```nu
scope commands
| where name == 'vip-greet'
| wrap help
```

This will show the Help _record_ that Nushell creates.
:::

The comments following the parameters become their description. Only a single-line comment is valid for parameters.

::: tip Note
A Nushell comment that continues on the same line for argument documentation purposes requires a space before the ` #` pound sign.
:::

## Changing the Environment in a Custom Command

Normally, environment variable definitions and changes are [_scoped_ within a block](./environment.html#scoping). This means that changes to those variables are lost when they go out of scope at the end of the block, including the block of a custom command.

```nu
def foo [] {
    $env.FOO = 'After'
}

$env.FOO = "Before"
foo
$env.FOO
# => Before
```

However, a command defined using [`def --env`](/commands/docs/def.md) or [`export def --env`](/commands/docs/export_def.md) (for a [Module](modules.md)) will preserve the environment on the caller's side:

```nu
def --env foo [] {
    $env.FOO = 'After'
}

$env.FOO = "Before"
foo
$env.FOO
# => After
```

### Changing Directories (cd) in a Custom Command

Likewise, changing the directory using the `cd` command results in a change of the `$env.PWD` environment variable. This means that directory changes (the `$env.PWD` variable) will also be reset when a custom command ends. The solution, as above, is to use `def --env` or `export def --env`.

```nu
def --env go-home [] {
  cd ~
}

cd /
go-home
pwd
# => Your home directory
```

## Persisting

To make custom commands available in future Nushell sessions, you'll want to add them to your startup configuration. You can add command definitions:

- Directly in your `config.nu`
- To a file sourced by your `config.nu`
- To a [module](./modules.html) imported by your `config.nu`

See the [configuration chapter](configuration.md) for more details.
# Custom Completions

Custom completions allow you to mix together two features of Nushell: custom commands and completions. With them, you're able to create commands that handle the completions for positional parameters and flag parameters. These custom completions work both for [custom commands](custom_commands.md) and [known external, or `extern`, commands](externs.md).

A completion is defined in two steps:

- Define a completion command (a.k.a. completer) that returns the possible values to suggest
- Attach the completer to the type annotation (shape) of another command's argument using `<shape>@<completer>`

Here's a simple example:

```nu
# Completion command
def animals [] { ["cat", "dog", "eel" ] }
# Command to be completed
def my-command [animal: string@animals] { print $animal }
my-command
# => cat                 dog                 eel
```

The first line defines a custom command which returns a list of three different animals. These are the possible values for the completion.

::: tip
To suppress completions for an argument (for example, an `int` that can accept any integer), define a completer that returns an empty list (`[ ]`). At the moment, invalid values such as `null` will also suppress completions, but this may change in the future.
:::

In the second line, `string@animals` tells Nushell two things—the shape of the argument for type-checking and the completer which will suggest possible values for the argument.

The third line is demonstration of the completion. Type the name of the custom command `my-command`, followed by a space, and then the <kbd>Tab</kbd> key. This displays a menu with the possible completions. Custom completions work the same as other completions in the system, allowing you to type `e` followed by the <kbd>Tab</kbd> key to complete "eel" automatically.

::: tip
When the completion menu is displayed, the prompt changes to include the `|` character by default. This can be changed using `$env.config.menus.marker`.
:::

## Options for Custom Completions

If you want to choose how your completions are filtered and sorted, you can also return a record rather than a list. The list of completion suggestions should be under the `completions` key of this record. Optionally, it can also have, under the `options` key, a record containing the following optional settings:

- `sort` - Set this to `false` to stop Nushell from sorting your completions. By default, this is `true`, and completions are sorted according to `$env.config.completions.sort`.
- `case_sensitive` - Set to `true` for the custom completions to be matched case sensitively, `false` otherwise. Used for overriding `$env.config.completions.case_sensitive`.
- `completion_algorithm` - Set this to either `prefix` or `fuzzy` to choose how your completions are matched against the typed text. Used for overriding `$env.config.completions.algorithm`.
- `positional` - When prefix matching is used, setting this to `false` will use substring matching instead. `true` by default.

Here's an example demonstrating how to set these options:

```nu
def animals [] {
    {
        options: {
            case_sensitive: false,
            completion_algorithm: prefix,
            positional: false,
            sort: false,
        },
        completions: [cat, rat, bat]
    }
}
def my-command [animal: string@animals] { print $animal }
```

Now, if you try to complete `A`, you get the following completions:

```nu
>| my-command A
cat                 rat                 bat
```

Because we made matching case-insensitive and used `positional: false`, Nushell will find the substring "a" in all of the completion suggestions. Additionally, because we set `sort: false`, the completions will be left in their original order. This is useful if your completions are already sorted in a particular order unrelated to their text (e.g. by date).

## Modules and Custom Completions

Since completion commands aren't meant to be called directly, it's common to define them in modules.

Extending the above example with a module:

```nu
module commands {
    def animals [] {
        ["cat", "dog", "eel" ]
    }

    export def my-command [animal: string@animals] {
        print $animal
    }
}
```

In this module, only the the custom command `my-command` is exported. The `animals` completion is not exported. This allows users of this module to call the command, and even use the custom completion logic, without having access to the completion command itself. This results in a cleaner and more maintainable API.

::: tip
Completers are attached to custom commands using `@` at parse time. This means that, in order for a change to the completion command to take effect, the public custom command must be reparsed as well. Importing a module satisfies both of these requirements at the same time with a single `use` statement.
:::

## Context Aware Custom Completions

It is possible to pass the context to the completion command. This is useful in situations where it is necessary to know previous arguments or flags to generate accurate completions.

Applying this concept to the previous example:

```nu
module commands {
    def animals [] {
        ["cat", "dog", "eel" ]
    }

    def animal-names [context: string] {
        match ($context | split words | last) {
            cat => ["Missy", "Phoebe"]
            dog => ["Lulu", "Enzo"]
            eel => ["Eww", "Slippy"]
        }
    }

    export def my-command [
        animal: string@animals
        name: string@animal-names
    ] {
        print $"The ($animal) is named ($name)."
    }
}
```

Here, the command `animal-names` returns the appropriate list of names. `$context` is a string where the value is the command-line that has been typed so far.

```nu
>| my-command
cat                 dog                 eel
>| my-command dog
Lulu                Enzo
>my-command dog enzo
The dog is named Enzo
```

On the second line, after pressing the <kbd>tab</kbd> key, the argument `"my-command dog"` is passed to the `animal-names` completer as context.

::: tip
Completers can also obtain the current cursor position on the command-line using:

```nu
def completer [context:string, position:int] {}
```

:::

## Custom Completion and [`extern`](/commands/docs/extern.md)

A powerful combination is adding custom completions to [known `extern` commands](externs.md). These work the same way as adding a custom completion to a custom command: by creating the custom completion and then attaching it with a `@` to the type of one of the positional or flag arguments of the `extern`.

If you look closely at the examples in the default config, you'll see this:

```nu
export extern "git push" [
    remote?: string@"nu-complete git remotes",  # the name of the remote
    refspec?: string@"nu-complete git branches" # the branch / refspec
    ...
]
```

Custom completions will serve the same role in this example as in the previous examples. The examples above call into two different custom completions, based on the position the user is currently in.

## Custom Descriptions and Styles

As an alternative to returning a list of strings, a completion function can also return a list of records with a `value` field as well as optional `description` and `style` fields. The style can be one of the following:

- A string with the foreground color, either a hex code or a color name such as `yellow`. For a list of valid color names, see `ansi --list`.
- A record with the fields `fg` (foreground color), `bg` (background color), and `attr` (attributes such as underline and bold). This record is in the same format that `ansi --escape` accepts. See the [`ansi`](/commands/docs/ansi) command reference for a list of possible values for the `attr` field.
- The same record, but converted to a JSON string.

```nu
def my_commits [] {
    [
        { value: "5c2464", description: "Add .gitignore", style: red },
        # "attr: ub" => underlined and bolded
        { value: "f3a377", description: "Initial commit", style: { fg: green, bg: "#66078c", attr: ub } }
    ]
}
```

::: tip Note
With the following snippet:

```nu
def my-command [commit: string@my_commits] {
    print $commit
}
```

... be aware that, even though the completion menu will show you something like

```ansi
>_ [36mmy-command[0m <TAB>
[1;31m5c2464[0m  [33mAdd .gitignore[0m
[1;4;48;2;102;7;140;32mf3a377  [0m[33mInitial commit[0m
```

... only the value (i.e., "5c2464" or "f3a377") will be used in the command arguments!
:::

## External Completions

External completers can also be integrated, instead of relying solely on Nushell ones.

For this, set the `external_completer` field in `config.nu` to a [closure](types_of_data.md#closures) which will be evaluated if no Nushell completions were found.

```nu
$env.config.completions.external = {
    enable: true
    max_results: 100
    completer: $completer
}
```

You can configure the closure to run an external completer, such as [carapace](https://github.com/rsteube/carapace-bin).

When the closure returns unparsable json (e.g., an empty string) it defaults to file completion.

An external completer is a function that takes the current command as a string list, and outputs a list of records with `value` and `description` keys, like custom completion functions.

::: tip Note
This closure will accept the current command as a list. For example, typing `my-command --arg1 <tab>` will receive `[my-command --arg1 " "]`.
:::

This example will enable carapace external completions:

```nu
let carapace_completer = {|spans|
    carapace $spans.0 nushell ...$spans | from json
}
```

[More examples of custom completers can be found in the cookbook](../cookbook/external_completers.md).
# Dataframes

::: warning Important!
This feature requires the Polars plugin.  See the
[Plugins Chapter](plugins.md) to learn how to install it.

To test that this plugin is properly installed, run `help polars`.
:::

As we have seen so far, Nushell makes working with data its main priority.
`Lists` and `Tables` are there to help you cycle through values in order to
perform multiple operations or find data in a breeze. However, there are
certain operations where a row-based data layout is not the most efficient way
to process data, especially when working with extremely large files. Operations
like group-by or join using large datasets can be costly memory-wise, and may
lead to large computation times if they are not done using the appropriate
data format.

For this reason, the `DataFrame` structure was introduced to Nushell. A
`DataFrame` stores its data in a columnar format using as its base the [Apache
Arrow](https://arrow.apache.org/) specification, and uses
[Polars](https://github.com/pola-rs/polars) as the motor for performing
extremely [fast columnar operations](https://h2oai.github.io/db-benchmark/).

You may be wondering now how fast this combo could be, and how could it make
working with data easier and more reliable. For this reason, we'll start this
chapter by presenting benchmarks on common operations that are done when
processing data.

[[toc]]

## Benchmark Comparisons

For this little benchmark exercise we will be comparing native Nushell
commands, dataframe Nushell commands and [Python
Pandas](https://pandas.pydata.org/) commands. For the time being don't pay too
much attention to the [`Dataframe` commands](/commands/categories/dataframe.md). They will be explained in later
sections of this page.

::: tip System Details
The benchmarks presented in this section were run using a Macbook with a processor M1 pro and 32gb of ram. All examples were run on Nushell version 0.97 using `nu_plugin_polars 0.97`.
:::

### File Information

The file that we will be using for the benchmarks is the
[New Zealand business demography](https://www.stats.govt.nz/assets/Uploads/New-Zealand-business-demography-statistics/New-Zealand-business-demography-statistics-At-February-2020/Download-data/Geographic-units-by-industry-and-statistical-area-2000-2020-descending-order-CSV.zip) dataset.
Feel free to download it if you want to follow these tests.

The dataset has 5 columns and 5,429,252 rows. We can check that by using the
`polars store-ls` command:

```nu
let df_0 = polars open --eager Data7602DescendingYearOrder.csv
polars store-ls | select key type columns rows estimated_size
# => ╭──────────────────────────────────────┬───────────┬─────────┬─────────┬────────────────╮
# => │                 key                  │   type    │ columns │  rows   │ estimated_size │
# => ├──────────────────────────────────────┼───────────┼─────────┼─────────┼────────────────┤
# => │ b2519dac-3b64-4e5d-a0d7-24bde9052dc7 │ DataFrame │       5 │ 5429252 │       184.5 MB │
# => ╰──────────────────────────────────────┴───────────┴─────────┴─────────┴────────────────╯
```

::: tip
As of nushell 0.97, `polars open` will open as a lazy dataframe instead of a eager dataframe.
To open as an eager dataframe, use the `--eager` flag.
:::

We can have a look at the first lines of the file using [`first`](/commands/docs/first.md):

```nu
$df_0 | polars first
# => ╭───┬──────────┬─────────┬──────┬───────────┬──────────╮
# => │ # │ anzsic06 │  Area   │ year │ geo_count │ ec_count │
# => ├───┼──────────┼─────────┼──────┼───────────┼──────────┤
# => │ 0 │ A        │ A100100 │ 2000 │        96 │      130 │
# => ╰───┴──────────┴─────────┴──────┴───────────┴──────────╯
```

...and finally, we can get an idea of the inferred data types:

```nu
$df_0 | polars schema
# => ╭───────────┬─────╮
# => │ anzsic06  │ str │
# => │ Area      │ str │
# => │ year      │ i64 │
# => │ geo_count │ i64 │
# => │ ec_count  │ i64 │
# => ╰───────────┴─────╯
```

### Group-by Comparison

To output more statistically correct timings, let's load and use the `std bench` command.

```nu
use std bench
```

We are going to group the data by year, and sum the column `geo_count`.

First, let's measure the performance of a Nushell native commands pipeline.

```nu
bench -n 10 --pretty {
    open 'Data7602DescendingYearOrder.csv'
    | group-by year --to-table
    | update items {|i|
        $i.items.geo_count
        | math sum
    }
}
```

Output

```
3sec 268ms +/- 50ms
```

So, 3.3 seconds to perform this aggregation.

Let's try the same operation in pandas:

```nu
('import pandas as pd

df = pd.read_csv("Data7602DescendingYearOrder.csv")
res = df.groupby("year")["geo_count"].sum()
print(res)'
| save load.py -f)
```

And the result from the benchmark is:

```nu
bench -n 10 --pretty {
    python load.py | complete | null
}
```

Output

```
1sec 322ms +/- 6ms
```

Not bad at all. Pandas managed to get it 2.6 times faster than Nushell.
And with bigger files, the superiority of Pandas should increase here.

To finish the comparison, let's try Nushell dataframes. We are going to put
all the operations in one `nu` file, to make sure we are doing the correct
comparison:

```nu
( 'polars open Data7602DescendingYearOrder.csv
    | polars group-by year
    | polars agg (polars col geo_count | polars sum)
    | polars collect'
| save load.nu -f )
```

and the benchmark with dataframes (together with loading a new nushell and `polars`
instance for each test in order of honest comparison) is:

```nu
bench -n 10 --pretty {
    nu load.nu | complete | null
}
```

Output

```
135ms +/- 4ms
```

The `polars` dataframes plugin managed to finish operation 10 times
faster than `pandas` with python. Isn't that great?

As you can see, the Nushell's `polars` plugin is performant like `polars` itself.
Coupled with Nushell commands and pipelines, it is capable of conducting sophisticated
analysis without leaving the terminal.

Let's clean up the cache from the dataframes that we used during benchmarking.
To do that, let's stop the `polars`.
When we execute our next commands, we will start a new instance of plugin.

```nu
plugin stop polars
```

## Working with Dataframes

After seeing a glimpse of the things that can be done with [`Dataframe` commands](/commands/categories/dataframe.md),
now it is time to start testing them. To begin let's create a sample
CSV file that will become our sample dataframe that we will be using along with
the examples. In your favorite file editor paste the next lines to create out
sample csv file.

```nu
("int_1,int_2,float_1,float_2,first,second,third,word
1,11,0.1,1.0,a,b,c,first
2,12,0.2,1.0,a,b,c,second
3,13,0.3,2.0,a,b,c,third
4,14,0.4,3.0,b,a,c,second
0,15,0.5,4.0,b,a,a,third
6,16,0.6,5.0,b,a,a,second
7,17,0.7,6.0,b,c,a,third
8,18,0.8,7.0,c,c,b,eight
9,19,0.9,8.0,c,c,b,ninth
0,10,0.0,9.0,c,c,b,ninth"
| save --raw --force test_small.csv)
```

Save the file and name it however you want to, for the sake of these examples
the file will be called `test_small.csv`.

Now, to read that file as a dataframe use the `polars open` command like
this:

```nu
let df_1 = polars open --eager test_small.csv
```

This should create the value `$df_1` in memory which holds the data we just
created.

::: tip
The `polars open` command can read files in formats: **csv**, **tsv**, **parquet**, **json(l)**, **arrow**, and **avro**.
:::

To see all the dataframes that are stored in memory you can use

```nu
polars store-ls | select key type columns rows estimated_size
# => ╭──────────────────────────────────────┬───────────┬─────────┬──────┬────────────────╮
# => │                 key                  │   type    │ columns │ rows │ estimated_size │
# => ├──────────────────────────────────────┼───────────┼─────────┼──────┼────────────────┤
# => │ e780af47-c106-49eb-b38d-d42d3946d66e │ DataFrame │       8 │   10 │          403 B │
# => ╰──────────────────────────────────────┴───────────┴─────────┴──────┴────────────────╯
```

As you can see, the command shows the created dataframes together with basic
information about them.

And if you want to see a preview of the loaded dataframe you can send the
dataframe variable to the stream

```nu
$df_1
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
# => │ 0 │     1 │    11 │    0.10 │    1.00 │ a     │ b      │ c     │ first  │
# => │ 1 │     2 │    12 │    0.20 │    1.00 │ a     │ b      │ c     │ second │
# => │ 2 │     3 │    13 │    0.30 │    2.00 │ a     │ b      │ c     │ third  │
# => │ 3 │     4 │    14 │    0.40 │    3.00 │ b     │ a      │ c     │ second │
# => │ 4 │     0 │    15 │    0.50 │    4.00 │ b     │ a      │ a     │ third  │
# => │ 5 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │
# => │ 6 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │
# => │ 7 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight  │
# => │ 8 │     9 │    19 │    0.90 │    8.00 │ c     │ c      │ b     │ ninth  │
# => │ 9 │     0 │    10 │    0.00 │    9.00 │ c     │ c      │ b     │ ninth  │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

With the dataframe in memory we can start doing column operations with the
`DataFrame`

::: tip
If you want to see all the dataframe commands that are available you
can use `scope commands | where category =~ dataframe`
:::

## Basic Aggregations

Let's start with basic aggregations on the dataframe. Let's sum all the columns
that exist in `df` by using the `aggregate` command

```nu
$df_1 | polars sum | polars collect
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬──────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │ word │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼──────┤
# => │ 0 │    40 │   145 │    4.50 │   46.00 │       │        │       │      │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴──────╯
```

As you can see, the aggregate function computes the sum for those columns where
a sum makes sense. If you want to filter out the text column, you can select
the columns you want by using the [`polars select`](/commands/docs/polars_select.md) command

```nu
$df_1 | polars sum | polars select int_1 int_2 float_1 float_2 | polars collect
# => ╭───┬───────┬───────┬─────────┬─────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │
# => ├───┼───────┼───────┼─────────┼─────────┤
# => │ 0 │    40 │   145 │    4.50 │   46.00 │
# => ╰───┴───────┴───────┴─────────┴─────────╯
```

You can even store the result from this aggregation as you would store any
other Nushell variable

```nu
let res = $df_1 | polars sum | polars select int_1 int_2 float_1 float_2
```

::: tip
Type `let res = !!` and press enter. This will auto complete the previously
executed command. Note the space between `=` and `!!`.
:::

And now we have two dataframes stored in memory

```nu
polars store-ls | select key type columns rows estimated_size
╭──────────────────────────────────────┬───────────┬─────────┬──────┬────────────────╮
│                 key                  │   type    │ columns │ rows │ estimated_size │
├──────────────────────────────────────┼───────────┼─────────┼──────┼────────────────┤
│ e780af47-c106-49eb-b38d-d42d3946d66e │ DataFrame │       8 │   10 │          403 B │
│ 3146f4c1-f2a0-475b-a623-7375c1fdb4a7 │ DataFrame │       4 │    1 │           32 B │
╰──────────────────────────────────────┴───────────┴─────────┴──────┴────────────────╯
```

Pretty neat, isn't it?

You can perform several aggregations on the dataframe in order to extract basic
information from the dataframe and do basic data analysis on your brand new
dataframe.

## Joining a DataFrame

It is also possible to join two dataframes using a column as reference. We are
going to join our mini dataframe with another mini dataframe. Copy these lines
in another file and create the corresponding dataframe (for these examples we
are going to call it `test_small_a.csv`)

```nu
"int_1,int_2,float_1,float_2,first
9,14,0.4,3.0,a
8,13,0.3,2.0,a
7,12,0.2,1.0,a
6,11,0.1,0.0,b"
| save --raw --force test_small_a.csv
```

We use the `polars open` command to create the new variable

```nu
let df_2 = polars open --eager test_small_a.csv
```

Now, with the second dataframe loaded in memory we can join them using the
column called `int_1` from the left dataframe and the column `int_1` from the
right dataframe

```nu
$df_1 | polars join $df_2 int_1 int_1
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────┬─────────┬───────────┬───────────┬─────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │ int_2_x │ float_1_x │ float_2_x │ first_x │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┼─────────┼───────────┼───────────┼─────────┤
# => │ 0 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │      11 │      0.10 │      0.00 │ b       │
# => │ 1 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │      12 │      0.20 │      1.00 │ a       │
# => │ 2 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight  │      13 │      0.30 │      2.00 │ a       │
# => │ 3 │     9 │    19 │    0.90 │    8.00 │ c     │ c      │ b     │ ninth  │      14 │      0.40 │      3.00 │ a       │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────┴─────────┴───────────┴───────────┴─────────╯
```

::: tip
In `Nu` when a command has multiple arguments that are expecting
multiple values we use brackets `[]` to enclose those values. In the case of
[`polars join`](/commands/docs/polars_join.md) we can join on multiple columns
as long as they have the same type.
:::

For example:

```nu
$df_1 | polars join $df_2 [int_1 first] [int_1 first]
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────┬─────────┬───────────┬───────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │ int_2_x │ float_1_x │ float_2_x │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┼─────────┼───────────┼───────────┤
# => │ 0 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │      11 │      0.10 │      0.00 │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────┴─────────┴───────────┴───────────╯
```

By default, the join command does an inner join, meaning that it will keep the
rows where both dataframes share the same value. You can select a left join to
keep the missing rows from the left dataframe. You can also save this result
in order to use it for further operations.

## DataFrame group-by

One of the most powerful operations that can be performed with a DataFrame is
the [`polars group-by`](/commands/docs/polars_group-by.md). This command will allow you to perform aggregation operations
based on a grouping criteria. In Nushell, a `GroupBy` is a type of object that
can be stored and reused for multiple aggregations. This is quite handy, since
the creation of the grouped pairs is the most expensive operation while doing
group-by and there is no need to repeat it if you are planning to do multiple
operations with the same group condition.

To create a `GroupBy` object you only need to use the [`polars_group-by`](/commands/docs/polars_group-by.md) command

```nu
let group = $df_1 | polars group-by first
$group
# => ╭─────────────┬──────────────────────────────────────────────╮
# => │ LazyGroupBy │ apply aggregation to complete execution plan │
# => ╰─────────────┴──────────────────────────────────────────────╯
```

When printing the `GroupBy` object we can see that it is in the background a
lazy operation waiting to be completed by adding an aggregation. Using the
`GroupBy` we can create aggregations on a column

```nu
$group | polars agg (polars col int_1 | polars sum)
# => ╭────────────────┬───────────────────────────────────────────────────────────────────────────────────────╮
# => │ plan           │ AGGREGATE                                                                             │
# => │                │     [col("int_1").sum()] BY [col("first")] FROM                                       │
# => │                │   DF ["int_1", "int_2", "float_1", "float_2"]; PROJECT */8 COLUMNS; SELECTION: "None" │
# => │ optimized_plan │ AGGREGATE                                                                             │
# => │                │     [col("int_1").sum()] BY [col("first")] FROM                                       │
# => │                │   DF ["int_1", "int_2", "float_1", "float_2"]; PROJECT 2/8 COLUMNS; SELECTION: "None" │
# => ╰────────────────┴───────────────────────────────────────────────────────────────────────────────────────╯
```

or we can define multiple aggregations on the same or different columns

```nu
$group
| polars agg [
    (polars col int_1 | polars n-unique)
    (polars col int_2 | polars min)
    (polars col float_1 | polars sum)
    (polars col float_2 | polars count)
] | polars sort-by first
```

Output

```
╭────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ plan           │ SORT BY [col("first")]                                                                              │
│                │   AGGREGATE                                                                                         │
│                │       [col("int_1").n_unique(), col("int_2").min(), col("float_1")                                  │
│                │ .sum(), col("float_2").count()] BY [col("first")] FROM                                              │
│                │     DF ["int_1", "int_2", "float_1", "float_2                                                       │
│                │ "]; PROJECT */8 COLUMNS; SELECTION: "None"                                                          │
│ optimized_plan │ SORT BY [col("first")]                                                                              │
│                │   AGGREGATE                                                                                         │
│                │       [col("int_1").n_unique(), col("int_2").min(), col("float_1")                                  │
│                │ .sum(), col("float_2").count()] BY [col("first")] FROM                                              │
│                │     DF ["int_1", "int_2", "float_1", "float_2                                                       │
│                │ "]; PROJECT 5/8 COLUMNS; SELECTION: "None"                                                          │
╰────────────────┴─────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

As you can see, the `GroupBy` object is a very powerful variable and it is
worth keeping in memory while you explore your dataset.

## Creating Dataframes

It is also possible to construct dataframes from basic Nushell primitives, such
as integers, decimals, or strings. Let's create a small dataframe using the
command `polars into-df`.

```nu
let df_3 = [[a b]; [1 2] [3 4] [5 6]] | polars into-df
$df_3
# => ╭───┬───┬───╮
# => │ # │ a │ b │
# => ├───┼───┼───┤
# => │ 0 │ 1 │ 2 │
# => │ 1 │ 3 │ 4 │
# => │ 2 │ 5 │ 6 │
# => ╰───┴───┴───╯
```

::: tip
For the time being, not all of Nushell primitives can be converted into
a dataframe. This will change in the future, as the dataframe feature matures
:::

We can append columns to a dataframe in order to create a new variable. As an
example, let's append two columns to our mini dataframe `$df_3`

```nu
let df_4 = $df_3 | polars with-column $df_3.a --name a2 | polars with-column $df_3.a --name a3
$df_4
# => ╭───┬───┬───┬────┬────╮
# => │ # │ a │ b │ a2 │ a3 │
# => ├───┼───┼───┼────┼────┤
# => │ 0 │ 1 │ 2 │  1 │  1 │
# => │ 1 │ 3 │ 4 │  3 │  3 │
# => │ 2 │ 5 │ 6 │  5 │  5 │
# => ╰───┴───┴───┴────┴────╯
```

Nushell's powerful piping syntax allows us to create new dataframes by
taking data from other dataframes and appending it to them. Now, if you list your
dataframes you will see in total five dataframes

```nu
polars store-ls | select key type columns rows estimated_size
# => ╭──────────────────────────────────────┬─────────────┬─────────┬──────┬────────────────╮
# => │                 key                  │    type     │ columns │ rows │ estimated_size │
# => ├──────────────────────────────────────┼─────────────┼─────────┼──────┼────────────────┤
# => │ e780af47-c106-49eb-b38d-d42d3946d66e │ DataFrame   │       8 │   10 │          403 B │
# => │ 3146f4c1-f2a0-475b-a623-7375c1fdb4a7 │ DataFrame   │       4 │    1 │           32 B │
# => │ 455a1483-e328-43e2-a354-35afa32803b9 │ DataFrame   │       5 │    4 │          132 B │
# => │ 0d8532a5-083b-4f78-8f66-b5e6b59dc449 │ LazyGroupBy │         │      │                │
# => │ 9504dfaf-4782-42d4-9110-9dae7c8fb95b │ DataFrame   │       2 │    3 │           48 B │
# => │ 37ab1bdc-e1fb-426d-8006-c3f974764a3d │ DataFrame   │       4 │    3 │           96 B │
# => ╰──────────────────────────────────────┴─────────────┴─────────┴──────┴────────────────╯
```

One thing that is important to mention is how the memory is being optimized
while working with dataframes, and this is thanks to **Apache Arrow** and
**Polars**. In a very simple representation, each column in a DataFrame is an
Arrow Array, which is using several memory specifications in order to maintain
the data as packed as possible (check [Arrow columnar
format](https://arrow.apache.org/docs/format/Columnar.html)). The other
optimization trick is the fact that whenever possible, the columns from the
dataframes are shared between dataframes, avoiding memory duplication for the
same data. This means that dataframes `$df_3` and `$df_4` are sharing the same two
columns we created using the `polars into-df` command. For this reason, it isn't
possible to change the value of a column in a dataframe. However, you can
create new columns based on data from other columns or dataframes.

## Working with Series

A `Series` is the building block of a `DataFrame`. Each Series represents a
column with the same data type, and we can create multiple Series of different
types, such as float, int or string.

Let's start our exploration with Series by creating one using the `polars into-df`
command:

```nu
let df_5 = [9 8 4] | polars into-df
$df_5
# => ╭───┬───╮
# => │ # │ 0 │
# => ├───┼───┤
# => │ 0 │ 9 │
# => │ 1 │ 8 │
# => │ 2 │ 4 │
# => ╰───┴───╯
```

We have created a new series from a list of integers (we could have done the
same using floats or strings)

Series have their own basic operations defined, and they can be used to create
other Series. Let's create a new Series by doing some arithmetic on the
previously created column.

```nu
let df_6 = $df_5 * 3 + 10
$df_6
# => ╭───┬────╮
# => │ # │ 0  │
# => ├───┼────┤
# => │ 0 │ 37 │
# => │ 1 │ 34 │
# => │ 2 │ 22 │
# => ╰───┴────╯
```

Now we have a new Series that was constructed by doing basic operations on the
previous variable.

::: tip
If you want to see how many variables you have stored in memory you can
use `scope variables`
:::

Let's rename our previous Series so it has a memorable name

```nu
let df_7 = $df_6 | polars rename "0" memorable
$df_7
# => ╭───┬───────────╮
# => │ # │ memorable │
# => ├───┼───────────┤
# => │ 0 │        37 │
# => │ 1 │        34 │
# => │ 2 │        22 │
# => ╰───┴───────────╯
```

We can also do basic operations with two Series as long as they have the same
data type

```nu
$df_5 - $df_7
# => ╭───┬─────────────────╮
# => │ # │ sub_0_memorable │
# => ├───┼─────────────────┤
# => │ 0 │             -28 │
# => │ 1 │             -26 │
# => │ 2 │             -18 │
# => ╰───┴─────────────────╯
```

And we can add them to previously defined dataframes

```nu
let df_8 = $df_3 | polars with-column $df_5 --name new_col
$df_8
# => ╭───┬───┬───┬─────────╮
# => │ # │ a │ b │ new_col │
# => ├───┼───┼───┼─────────┤
# => │ 0 │ 1 │ 2 │       9 │
# => │ 1 │ 3 │ 4 │       8 │
# => │ 2 │ 5 │ 6 │       4 │
# => ╰───┴───┴───┴─────────╯
```

The Series stored in a Dataframe can also be used directly, for example,
we can multiply columns `a` and `b` to create a new Series

```nu
$df_8.a * $df_8.b
# => ╭───┬─────────╮
# => │ # │ mul_a_b │
# => ├───┼─────────┤
# => │ 0 │       2 │
# => │ 1 │      12 │
# => │ 2 │      30 │
# => ╰───┴─────────╯
```

and we can start piping things in order to create new columns and dataframes

```nu
let df_9 = $df_8 | polars with-column ($df_8.a * $df_8.b / $df_8.new_col) --name my_sum
$df_9
# => ╭───┬───┬───┬─────────┬────────╮
# => │ # │ a │ b │ new_col │ my_sum │
# => ├───┼───┼───┼─────────┼────────┤
# => │ 0 │ 1 │ 2 │       9 │      0 │
# => │ 1 │ 3 │ 4 │       8 │      1 │
# => │ 2 │ 5 │ 6 │       4 │      7 │
# => ╰───┴───┴───┴─────────┴────────╯
```

Nushell's piping system can help you create very interesting workflows.

## Series and Masks

Series have another key use in when working with `DataFrames`, and it is the fact
that we can build boolean masks out of them. Let's start by creating a simple
mask using the equality operator

```nu
let mask_0 = $df_5 == 8
$mask_0
# => ╭───┬───────╮
# => │ # │   0   │
# => ├───┼───────┤
# => │ 0 │ false │
# => │ 1 │ true  │
# => │ 2 │ false │
# => ╰───┴───────╯
```

and with this mask we can now filter a dataframe, like this

```nu
$df_9 | polars filter-with $mask_0
# => ╭───┬───┬───┬─────────┬────────╮
# => │ # │ a │ b │ new_col │ my_sum │
# => ├───┼───┼───┼─────────┼────────┤
# => │ 0 │ 3 │ 4 │       8 │      1 │
# => ╰───┴───┴───┴─────────┴────────╯
```

Now we have a new dataframe with only the values where the mask was true.

The masks can also be created from Nushell lists, for example:

```nu
let mask_1 = [true true false] | polars into-df
$df_9 | polars filter-with $mask_1
# => ╭───┬───┬───┬─────────┬────────╮
# => │ # │ a │ b │ new_col │ my_sum │
# => ├───┼───┼───┼─────────┼────────┤
# => │ 0 │ 1 │ 2 │       9 │      0 │
# => │ 1 │ 3 │ 4 │       8 │      1 │
# => ╰───┴───┴───┴─────────┴────────╯
```

To create complex masks, we have the `AND`

```nu
$mask_0 and $mask_1
# => ╭───┬─────────╮
# => │ # │ and_0_0 │
# => ├───┼─────────┤
# => │ 0 │ false   │
# => │ 1 │ true    │
# => │ 2 │ false   │
# => ╰───┴─────────╯
```

and `OR` operations

```nu
$mask_0 or $mask_1
# => ╭───┬────────╮
# => │ # │ or_0_0 │
# => ├───┼────────┤
# => │ 0 │ true   │
# => │ 1 │ true   │
# => │ 2 │ false  │
# => ╰───┴────────╯
```

We can also create a mask by checking if some values exist in other Series.
Using the first dataframe that we created we can do something like this

```nu
let mask_2 = $df_1 | polars col first | polars is-in [b c]
$mask_2
# => ╭──────────┬─────────────────────────╮
# => │ input    │ [table 2 rows]          │
# => │ function │ Boolean(IsIn)           │
# => │ options  │ FunctionOptions { ... } │
# => ╰──────────┴─────────────────────────╯
```

and this new mask can be used to filter the dataframe

```nu
$df_1 | polars filter-with $mask_2
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
# => │ 0 │     4 │    14 │    0.40 │    3.00 │ b     │ a      │ c     │ second │
# => │ 1 │     0 │    15 │    0.50 │    4.00 │ b     │ a      │ a     │ third  │
# => │ 2 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │
# => │ 3 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │
# => │ 4 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight  │
# => │ 5 │     9 │    19 │    0.90 │    8.00 │ c     │ c      │ b     │ ninth  │
# => │ 6 │     0 │    10 │    0.00 │    9.00 │ c     │ c      │ b     │ ninth  │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

Another operation that can be done with masks is setting or replacing a value
from a series. For example, we can change the value in the column `first` where
the value is equal to `a`

```nu
$df_1 | polars get first | polars set new --mask ($df_1.first =~ a)
# => ╭───┬────────╮
# => │ # │ string │
# => ├───┼────────┤
# => │ 0 │ new    │
# => │ 1 │ new    │
# => │ 2 │ new    │
# => │ 3 │ b      │
# => │ 4 │ b      │
# => │ 5 │ b      │
# => │ 6 │ b      │
# => │ 7 │ c      │
# => │ 8 │ c      │
# => │ 9 │ c      │
# => ╰───┴────────╯
```

## Series as Indices

Series can be also used as a way of filtering a dataframe by using them as a
list of indices. For example, let's say that we want to get rows 1, 4, and 6
from our original dataframe. With that in mind, we can use the next command to
extract that information

```nu
let indices_0 = [1 4 6] | polars into-df
$df_1 | polars take $indices_0
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
# => │ 0 │     2 │    12 │    0.20 │    1.00 │ a     │ b      │ c     │ second │
# => │ 1 │     0 │    15 │    0.50 │    4.00 │ b     │ a      │ a     │ third  │
# => │ 2 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

The command [`polars take`](/commands/docs/polars_take.md) is very handy, especially if we mix it with other commands.
Let's say that we want to extract all rows for the first duplicated element for
column `first`. In order to do that, we can use the command `polars arg-unique` as
shown in the next example

```nu
let indices_1 = $df_1 | polars get first | polars arg-unique
$df_1 | polars take $indices_1
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
# => │ 0 │     1 │    11 │    0.10 │    1.00 │ a     │ b      │ c     │ first  │
# => │ 1 │     4 │    14 │    0.40 │    3.00 │ b     │ a      │ c     │ second │
# => │ 2 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight  │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

Or what if we want to create a new sorted dataframe using a column in specific.
We can use the `arg-sort` to accomplish that. In the next example we
can sort the dataframe by the column `word`

::: tip
The same result could be accomplished using the command [`sort`](/commands/docs/sort.md)
:::

```nu
let indices_2 = $df_1 | polars get word | polars arg-sort
$df_1 | polars take $indices_2
# => ╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
# => │ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
# => ├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
# => │ 0 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight  │
# => │ 1 │     1 │    11 │    0.10 │    1.00 │ a     │ b      │ c     │ first  │
# => │ 2 │     9 │    19 │    0.90 │    8.00 │ c     │ c      │ b     │ ninth  │
# => │ 3 │     0 │    10 │    0.00 │    9.00 │ c     │ c      │ b     │ ninth  │
# => │ 4 │     2 │    12 │    0.20 │    1.00 │ a     │ b      │ c     │ second │
# => │ 5 │     4 │    14 │    0.40 │    3.00 │ b     │ a      │ c     │ second │
# => │ 6 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │
# => │ 7 │     3 │    13 │    0.30 │    2.00 │ a     │ b      │ c     │ third  │
# => │ 8 │     0 │    15 │    0.50 │    4.00 │ b     │ a      │ a     │ third  │
# => │ 9 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │
# => ╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

And finally, we can create new Series by setting a new value in the marked
indices. Have a look at the next command

```nu
let indices_3 = [0 2] | polars into-df
$df_1 | polars get int_1 | polars set-with-idx 123 --indices $indices_3
# => ╭───┬───────╮
# => │ # │ int_1 │
# => ├───┼───────┤
# => │ 0 │   123 │
# => │ 1 │     2 │
# => │ 2 │   123 │
# => │ 3 │     4 │
# => │ 4 │     0 │
# => │ 5 │     6 │
# => │ 6 │     7 │
# => │ 7 │     8 │
# => │ 8 │     9 │
# => │ 9 │     0 │
# => ╰───┴───────╯
```

## Unique Values

Another operation that can be done with `Series` is to search for unique values
in a list or column. Lets use again the first dataframe we created to test
these operations.

The first and most common operation that we have is `value_counts`. This
command calculates a count of the unique values that exist in a Series. For
example, we can use it to count how many occurrences we have in the column
`first`

```nu
$df_1 | polars get first | polars value-counts
# => ╭───┬───────┬───────╮
# => │ # │ first │ count │
# => ├───┼───────┼───────┤
# => │ 0 │ a     │     3 │
# => │ 1 │ b     │     4 │
# => │ 2 │ c     │     3 │
# => ╰───┴───────┴───────╯
```

As expected, the command returns a new dataframe that can be used to do more
queries.

Continuing with our exploration of `Series`, the next thing that we can do is
to only get the unique unique values from a series, like this

```nu
$df_1 | polars get first | polars unique
# => ╭───┬───────╮
# => │ # │ first │
# => ├───┼───────┤
# => │ 0 │ a     │
# => │ 1 │ b     │
# => │ 2 │ c     │
# => ╰───┴───────╯
```

Or we can get a mask that we can use to filter out the rows where data is
unique or duplicated. For example, we can select the rows for unique values
in column `word`

```nu
$df_1 | polars filter-with ($in.word | polars is-unique)
```

Output

```
╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬───────╮
│ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │ word  │
├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼───────┤
│ 0 │     1 │    11 │    0.10 │    1.00 │ a     │ b      │ c     │ first │
│ 1 │     8 │    18 │    0.80 │    7.00 │ c     │ c      │ b     │ eight │
╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴───────╯
```

Or all the duplicated ones

```nu
$df_1 | polars filter-with ($in.word | polars is-duplicated)
```

Output

```
╭───┬───────┬───────┬─────────┬─────────┬───────┬────────┬───────┬────────╮
│ # │ int_1 │ int_2 │ float_1 │ float_2 │ first │ second │ third │  word  │
├───┼───────┼───────┼─────────┼─────────┼───────┼────────┼───────┼────────┤
│ 0 │     2 │    12 │    0.20 │    1.00 │ a     │ b      │ c     │ second │
│ 1 │     3 │    13 │    0.30 │    2.00 │ a     │ b      │ c     │ third  │
│ 2 │     4 │    14 │    0.40 │    3.00 │ b     │ a      │ c     │ second │
│ 3 │     0 │    15 │    0.50 │    4.00 │ b     │ a      │ a     │ third  │
│ 4 │     6 │    16 │    0.60 │    5.00 │ b     │ a      │ a     │ second │
│ 5 │     7 │    17 │    0.70 │    6.00 │ b     │ c      │ a     │ third  │
│ 6 │     9 │    19 │    0.90 │    8.00 │ c     │ c      │ b     │ ninth  │
│ 7 │     0 │    10 │    0.00 │    9.00 │ c     │ c      │ b     │ ninth  │
╰───┴───────┴───────┴─────────┴─────────┴───────┴────────┴───────┴────────╯
```

## Lazy Dataframes

Lazy dataframes are a way to query data by creating a logical plan. The
advantage of this approach is that the plan never gets evaluated until you need
to extract data. This way you could chain together aggregations, joins and
selections and collect the data once you are happy with the selected
operations.

Let's create a small example of a lazy dataframe

```nu
let lf_0 = [[a b]; [1 a] [2 b] [3 c] [4 d]] | polars into-lazy
$lf_0
# => ╭────────────────┬───────────────────────────────────────────────────────╮
# => │ plan           │ DF ["a", "b"]; PROJECT */2 COLUMNS; SELECTION: "None" │
# => │ optimized_plan │ DF ["a", "b"]; PROJECT */2 COLUMNS; SELECTION: "None" │
# => ╰────────────────┴───────────────────────────────────────────────────────╯
```

As you can see, the resulting dataframe is not yet evaluated, it stays as a
set of instructions that can be done on the data. If you were to collect that
dataframe you would get the next result

```nu
$lf_0 | polars collect
# => ╭───┬───┬───╮
# => │ # │ a │ b │
# => ├───┼───┼───┤
# => │ 0 │ 1 │ a │
# => │ 1 │ 2 │ b │
# => │ 2 │ 3 │ c │
# => │ 3 │ 4 │ d │
# => ╰───┴───┴───╯
```

as you can see, the collect command executes the plan and creates a nushell
table for you.

All dataframes operations should work with eager or lazy dataframes. They are
converted in the background for compatibility. However, to take advantage of
lazy operations if is recommended to only use lazy operations with lazy
dataframes.

To find all lazy dataframe operations you can use

```nu no-run
scope commands | where category =~ lazyframe | select name category usage
```

With your lazy frame defined we can start chaining operations on it. For
example this

```nu
$lf_0
| polars reverse
| polars with-column [
     ((polars col a) * 2 | polars as double_a)
     ((polars col a) / 2 | polars as half_a)
]
| polars collect
```

Output

```
╭───┬───┬───┬──────────┬────────╮
│ # │ a │ b │ double_a │ half_a │
├───┼───┼───┼──────────┼────────┤
│ 0 │ 4 │ d │        8 │      2 │
│ 1 │ 3 │ c │        6 │      1 │
│ 2 │ 2 │ b │        4 │      1 │
│ 3 │ 1 │ a │        2 │      0 │
╰───┴───┴───┴──────────┴────────╯
```

:::tip
You can use the line buffer editor to format your queries (`ctr + o`) easily
:::

This query uses the lazy reverse command to invert the dataframe and the
`polars with-column` command to create new two columns using `expressions`.
An `expression` is used to define an operation that is executed on the lazy
frame. When put together they create the whole set of instructions used by the
lazy commands to query the data. To list all the commands that generate an
expression you can use

```nu no-run
scope commands | where category =~ expression | select name category usage
```

In our previous example, we use the `polars col` command to indicate that column `a`
will be multiplied by 2 and then it will be aliased to the name `double_a`.
In some cases the use of the `polars col` command can be inferred. For example,
using the `polars select` command we can use only a string

```nu
$lf_0 | polars select a | polars collect
# => ╭───┬───╮
# => │ # │ a │
# => ├───┼───┤
# => │ 0 │ 1 │
# => │ 1 │ 2 │
# => │ 2 │ 3 │
# => │ 3 │ 4 │
# => ╰───┴───╯
```

or the `polars col` command

```nu
$lf_0 | polars select (polars col a) | polars collect
# => ╭───┬───╮
# => │ # │ a │
# => ├───┼───┤
# => │ 0 │ 1 │
# => │ 1 │ 2 │
# => │ 2 │ 3 │
# => │ 3 │ 4 │
# => ╰───┴───╯
```

Let's try something more complicated and create aggregations from a lazy
dataframe

```nu
let lf_1 =  [[name value]; [one 1] [two 2] [one 1] [two 3]] | polars into-lazy

$lf_1
| polars group-by name
| polars agg [
     (polars col value | polars sum | polars as sum)
     (polars col value | polars mean | polars as mean)
]
| polars collect
```

Output

```
╭───┬──────┬─────┬──────╮
│ # │ name │ sum │ mean │
├───┼──────┼─────┼──────┤
│ 0 │ two  │   5 │ 2.50 │
│ 1 │ one  │   2 │ 1.00 │
╰───┴──────┴─────┴──────╯
```

And we could join on a lazy dataframe that hasn't being collected. Let's join
the resulting group by to the original lazy frame

```nu
let lf_2 =  [[name value]; [one 1] [two 2] [one 1] [two 3]] | polars into-lazy
let group = $lf_2
    | polars group-by name
    | polars agg [
      (polars col value | polars sum | polars as sum)
      (polars col value | polars mean | polars as mean)
    ]

$lf_2 | polars join $group name name | polars collect
```

Output

```
╭───┬──────┬───────┬─────┬──────╮
│ # │ name │ value │ sum │ mean │
├───┼──────┼───────┼─────┼──────┤
│ 0 │ one  │     1 │   2 │ 1.00 │
│ 1 │ two  │     2 │   5 │ 2.50 │
│ 2 │ one  │     1 │   2 │ 1.00 │
│ 3 │ two  │     3 │   5 │ 2.50 │
╰───┴──────┴───────┴─────┴──────╯
```

As you can see lazy frames are a powerful construct that will let you query
data using a flexible syntax, resulting in blazing fast results.

## Dataframe Commands

So far we have seen quite a few operations that can be done using `DataFrame`s
commands. However, the commands we have used so far are not all the commands
available to work with data and be assured that there will be more as the
feature becomes more stable.

The next list shows the available dataframe commands with their descriptions, and
whenever possible, their analogous Nushell command.

::: warning
This list may be outdated. To get the up-to-date command list, see [Dataframe](/commands/categories/dataframe.md), [Lazyframe](/commands/categories/lazyframe.md), [Dataframe Or Lazyframe](/commands/categories/dataframe_or_lazyframe.md), [Expressions](/commands/categories/expression.html) command categories.
:::

<!-- This table was updated using the script from ../tools/dataframes_md-update.nu -->

| Command Name           | Applies To            | Description                                                                                      | Nushell Equivalent      |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------------ | ----------------------- |
| polars agg             | dataframe             | Performs a series of aggregations from a group-by.                                               | math                    |
| polars agg-groups      | expression            | Creates an agg_groups expression.                                                                |                         |
| polars all-false       | dataframe             | Returns true if all values are false.                                                            |                         |
| polars all-true        | dataframe             | Returns true if all values are true.                                                             | all                     |
| polars append          | dataframe             | Appends a new dataframe.                                                                         |                         |
| polars arg-max         | dataframe             | Return index for max value in series.                                                            |                         |
| polars arg-min         | dataframe             | Return index for min value in series.                                                            |                         |
| polars arg-sort        | dataframe             | Returns indexes for a sorted series.                                                             |                         |
| polars arg-true        | dataframe             | Returns indexes where values are true.                                                           |                         |
| polars arg-unique      | dataframe             | Returns indexes for unique values.                                                               |                         |
| polars arg-where       | any                   | Creates an expression that returns the arguments where expression is true.                       |                         |
| polars as              | expression            | Creates an alias expression.                                                                     |                         |
| polars as-date         | dataframe             | Converts string to date.                                                                         |                         |
| polars as-datetime     | dataframe             | Converts string to datetime.                                                                     |                         |
| polars cache           | dataframe             | Caches operations in a new LazyFrame.                                                            |                         |
| polars cast            | expression, dataframe | Cast a column to a different dtype.                                                              |                         |
| polars col             | any                   | Creates a named column expression.                                                               |                         |
| polars collect         | dataframe             | Collect lazy dataframe into eager dataframe.                                                     |                         |
| polars columns         | dataframe             | Show dataframe columns.                                                                          |                         |
| polars concat-str      | any                   | Creates a concat string expression.                                                              |                         |
| polars concatenate     | dataframe             | Concatenates strings with other array.                                                           |                         |
| polars contains        | dataframe             | Checks if a pattern is contained in a string.                                                    |                         |
| polars count           | expression            | Creates a count expression.                                                                      |                         |
| polars count-null      | dataframe             | Counts null values.                                                                              |                         |
| polars cumulative      | dataframe             | Cumulative calculation for a series.                                                             |                         |
| polars datepart        | expression            | Creates an expression for capturing the specified datepart in a column.                          |                         |
| polars drop            | dataframe             | Creates a new dataframe by dropping the selected columns.                                        | drop                    |
| polars drop-duplicates | dataframe             | Drops duplicate values in dataframe.                                                             |                         |
| polars drop-nulls      | dataframe             | Drops null values in dataframe.                                                                  |                         |
| polars dummies         | dataframe             | Creates a new dataframe with dummy variables.                                                    |                         |
| polars explode         | expression, dataframe | Explodes a dataframe or creates a explode expression.                                            |                         |
| polars expr-not        | expression            | Creates a not expression.                                                                        |                         |
| polars fetch           | dataframe             | Collects the lazyframe to the selected rows.                                                     |                         |
| polars fill-nan        | dataframe             | Replaces NaN values with the given expression.                                                   |                         |
| polars fill-null       | dataframe             | Replaces NULL values with the given expression.                                                  |                         |
| polars filter          | dataframe             | Filter dataframe based in expression.                                                            |                         |
| polars filter-with     | dataframe             | Filters dataframe using a mask or expression as reference.                                       |                         |
| polars first           | expression, dataframe | Show only the first number of rows or create a first expression                                  | first                   |
| polars flatten         | expression, dataframe | An alias for polars explode.                                                                     |                         |
| polars get             | dataframe             | Creates dataframe with the selected columns.                                                     | get                     |
| polars get-day         | dataframe             | Gets day from date.                                                                              |                         |
| polars get-hour        | dataframe             | Gets hour from date.                                                                             |                         |
| polars get-minute      | dataframe             | Gets minute from date.                                                                           |                         |
| polars get-month       | dataframe             | Gets month from date.                                                                            |                         |
| polars get-nanosecond  | dataframe             | Gets nanosecond from date.                                                                       |                         |
| polars get-ordinal     | dataframe             | Gets ordinal from date.                                                                          |                         |
| polars get-second      | dataframe             | Gets second from date.                                                                           |                         |
| polars get-week        | dataframe             | Gets week from date.                                                                             |                         |
| polars get-weekday     | dataframe             | Gets weekday from date.                                                                          |                         |
| polars get-year        | dataframe             | Gets year from date.                                                                             |                         |
| polars group-by        | dataframe             | Creates a group-by object that can be used for other aggregations.                               | group-by                |
| polars implode         | expression            | Aggregates a group to a Series.                                                                  |                         |
| polars into-df         | any                   | Converts a list, table or record into a dataframe.                                               |                         |
| polars into-lazy       | any                   | Converts a dataframe into a lazy dataframe.                                                      |                         |
| polars into-nu         | expression, dataframe | Converts a dataframe or an expression into into nushell value for access and exploration.        |                         |
| polars is-duplicated   | dataframe             | Creates mask indicating duplicated values.                                                       |                         |
| polars is-in           | expression, dataframe | Creates an is-in expression or checks to see if the elements are contained in the right series   | in                      |
| polars is-not-null     | expression, dataframe | Creates mask where value is not null.                                                            |                         |
| polars is-null         | expression, dataframe | Creates mask where value is null.                                                                | `<column_name> == null` |
| polars is-unique       | dataframe             | Creates mask indicating unique values.                                                           |                         |
| polars join            | dataframe             | Joins a lazy frame with other lazy frame.                                                        |                         |
| polars last            | expression, dataframe | Creates new dataframe with tail rows or creates a last expression.                               | last                    |
| polars lit             | any                   | Creates a literal expression.                                                                    |                         |
| polars lowercase       | dataframe             | Lowercase the strings in the column.                                                             |                         |
| polars max             | expression, dataframe | Creates a max expression or aggregates columns to their max value.                               |                         |
| polars mean            | expression, dataframe | Creates a mean expression for an aggregation or aggregates columns to their mean value.          |                         |
| polars median          | expression, dataframe | Median value from columns in a dataframe or creates expression for an aggregation                |                         |
| polars melt            | dataframe             | Unpivot a DataFrame from wide to long format.                                                    |                         |
| polars min             | expression, dataframe | Creates a min expression or aggregates columns to their min value.                               |                         |
| polars n-unique        | expression, dataframe | Counts unique values.                                                                            |                         |
| polars not             | dataframe             | Inverts boolean mask.                                                                            |                         |
| polars open            | any                   | Opens CSV, JSON, JSON lines, arrow, avro, or parquet file to create dataframe.                   | open                    |
| polars otherwise       | any                   | Completes a when expression.                                                                     |                         |
| polars quantile        | expression, dataframe | Aggregates the columns to the selected quantile.                                                 |                         |
| polars query           | dataframe             | Query dataframe using SQL. Note: The dataframe is always named 'df' in your query's from clause. |                         |
| polars rename          | dataframe             | Rename a dataframe column.                                                                       | rename                  |
| polars replace         | dataframe             | Replace the leftmost (sub)string by a regex pattern.                                             |                         |
| polars replace-all     | dataframe             | Replace all (sub)strings by a regex pattern.                                                     |                         |
| polars reverse         | dataframe             | Reverses the LazyFrame                                                                           |                         |
| polars rolling         | dataframe             | Rolling calculation for a series.                                                                |                         |
| polars sample          | dataframe             | Create sample dataframe.                                                                         |                         |
| polars save            | dataframe             | Saves a dataframe to disk. For lazy dataframes a sink operation will be used if the file type supports it (parquet, ipc/arrow, csv, and ndjson).|                         |
| polars schema          | dataframe             | Show schema for a dataframe.                                                                     |                         |
| polars select          | dataframe             | Selects columns from lazyframe.                                                                  | select                  |
| polars set             | dataframe             | Sets value where given mask is true.                                                             |                         |
| polars set-with-idx    | dataframe             | Sets value in the given index.                                                                   |                         |
| polars shape           | dataframe             | Shows column and row size for a dataframe.                                                       |                         |
| polars shift           | dataframe             | Shifts the values by a given period.                                                             |                         |
| polars slice           | dataframe             | Creates new dataframe from a slice of rows.                                                      |                         |
| polars sort-by         | dataframe             | Sorts a lazy dataframe based on expression(s).                                                   | sort                    |
| polars std             | expression, dataframe | Creates a std expression for an aggregation of std value from columns in a dataframe.            |                         |
| polars store-get       | any, any              | Gets a Dataframe or other object from the plugin cache.                                          |                         |
| polars store-ls        |                       | Lists stored dataframes.                                                                         |                         |
| polars store-rm        | any                   | Removes a stored Dataframe or other object from the plugin cache.                                |                         |
| polars str-lengths     | dataframe             | Get lengths of all strings.                                                                      |                         |
| polars str-slice       | dataframe             | Slices the string from the start position until the selected length.                             |                         |
| polars strftime        | dataframe             | Formats date based on string rule.                                                               |                         |
| polars sum             | expression, dataframe | Creates a sum expression for an aggregation or aggregates columns to their sum value.            |                         |
| polars summary         | dataframe             | For a dataframe, produces descriptive statistics (summary statistics) for its numeric columns.   |                         |
| polars take            | dataframe             | Creates new dataframe using the given indices.                                                   |                         |
| polars unique          | dataframe             | Returns unique values from a dataframe.                                                          | uniq                    |
| polars uppercase       | dataframe             | Uppercase the strings in the column.                                                             |                         |
| polars value-counts    | dataframe             | Returns a dataframe with the counts for unique values in series.                                 |                         |
| polars var             | expression, dataframe | Create a var expression for an aggregation.                                                      |                         |
| polars when            | expression            | Creates and modifies a when expression.                                                          |                         |
| polars with-column     | dataframe             | Adds a series to the dataframe.                                                                  | `insert <column_name> <value> \| upsert <column_name> { <new_value> }` |

## Future of Dataframes

We hope that by the end of this page you have a solid grasp of how to use the
dataframe commands. As you can see they offer powerful operations that can
help you process data faster and natively.

However, the future of these dataframes is still very experimental. New
commands and tools that take advantage of these commands will be added as they
mature.

Check this chapter, as well as our [Blog](/blog/), regularly to learn about new
dataframes features and how they can help you process data faster and efficiently.
# Default Shell

## Setting Nu as default shell on your terminal

|     Terminal     | Platform     |                                                                                                                 Instructions                                                                                                                 |
| :--------------: | ------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  GNOME Terminal  | Linux & BSDs |                                  Open `Edit > Preferences`. In the right-hand panel, select the `Command` tab, tick `Run a custom command instead of my shell`, and set `Custom command` to the path to Nu.                                  |
|  GNOME Console   | Linux & BSDs |  Type the command `gsettings set org.gnome.Console shell "['/usr/bin/nu']"` (replace `/usr/bin/nu` with the path to Nu). Equivalently, use [dconf Editor](https://apps.gnome.org/DconfEditor/) to edit the `/org/gnome/Console/shell` key.   |
|      Kitty       | Linux & BSDs |                                                     Press `Ctrl`+`Shift`+`F2` to open `kitty.conf`. Go to `shell` variable, uncomment the line and replace the `.` with the path to Nu.                                                      |
|     Konsole      | Linux & BSDs |                                                                                   Open `Settings > Edit Current Profile`. Set `Command` to the path to Nu.                                                                                   |
|  XFCE Terminal   | Linux & BSDs |                                                           Open `Edit > Preferences`. Check `Run a custom command instead of my shell`, and set `Custom command` to the path to Nu.                                                           |
|   Terminal.app   | macOS        | Open `Terminal > Preferences`. Ensure you are on the `Profiles` tab, which should be the default tab. In the right-hand panel, select the `Shell` tab. Tick `Run command`, put the path to Nu in the textbox, and untick `Run inside shell`. |
|      iTerm2      | macOS        |                       Open `iTerm > Preferences`. Select the `Profiles` tab. In the right-hand panel under `Command`, change the dropdown from `Login Shell` to `Custom Shell`, and put the path to Nu in the textbox.                       |
| Windows Terminal | Windows      |    Press `Ctrl`+`,` to open `Settings`. Go to `Add a new profile > New empty profile`. Fill in the 'Name' and enter path to Nu in the 'Command line' textbox. Go to `Startup` option and select Nu as the 'Default profile'. Hit `Save`.     |

## Setting Nu as login shell (Linux, BSD & macOS)

::: warning
Nu is not intended to be POSIX compliant.
Be aware that some programs on your system (or their documentation) might assume that your login shell is [POSIX](https://en.wikipedia.org/wiki/POSIX) compatible.
Breaking that assumption can lead to unexpected issues. See [Configuration - Login Shell](./configuration.md#configuring-nu-as-a-login-shell) for more details.
:::

To set the login shell you can use the [`chsh`](https://linux.die.net/man/1/chsh) command.
Some Linux distributions have a list of valid shells located in `/etc/shells` and will disallow changing the shell until Nu is in the whitelist.
You may see an error similar to the one below if you haven't updated the `shells` file:

@[code](@snippets/installation/chsh_invalid_shell_error.sh)

You can add Nu to the list of allowed shells by appending your Nu binary to the `shells` file.
The path to add can be found with the command `which nu`, usually it is `$HOME/.cargo/bin/nu`.
# Design Notes

This chapter intends to give more in-depth overview of certain aspects of Nushell's design. The topics are not necessary for a basic usage, but reading them will help you understand how Nushell works and why.

We intend to expand this chapter in the future. If there is some topic that you find confusing and hard to understand, let us know. It might be a good candidate for a page here.

[How Nushell Code Gets Run](how_nushell_code_gets_run.md) explains what happens when you run Nushell source code. It explains how Nushell is in many ways closer to classic compiled languages, like C or Rust, than to other shells and dynamic languages and hopefully clears some confusion that stems from that.
# Directory Stack

Like some other shells, Nushell provides a Directory Stack feature for easily switching between multiple directories. In Nushell, this feature is part of the [Standard Library](./standard_library.md) and can be accessed in several ways.

::: note
In Nushell, the "stack" is represented as a `list`, but the overall functionality is similar to that of other shells.
:::

[[toc]]

## `dirs` Module and Commands

To use the `dirs` command and its subcommands, first import the module using:

```nu
use std/dirs
```

::: tip
To make the feature available whenever you start Nushell, add the above command to your [startup configuration](./configuration.md).
:::

This makes several new commands available:

| Command     | Description                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dirs`      | Lists the directories on the stack                                                                                                                                  |
| `dirs add`  | Adds one or more directories to the list. The first directory listed becomes the new active directory. Similar to the `pushd` command in some other shells.         |
| `dirs drop` | Drops the current directory from the list. The previous directory in the list becomes the new active directory. Similar to the `popd` command in some other shells. |
| `dirs goto` | Jumps to directory by using its index in the list                                                                                                                   |
| `dirs next` | Makes the next directory on the list the active directory. If the current active directory is the last in the list, then cycle to the start of the list.            |
| `dirs prev` | Makes the previous directory on the list the active directory. If the current active directory is the first in the list, then cycle to the end of the list.         |

When we start using `dirs`, there is only one directory in the list, the active one. You can, as always, change this directory using the `cd` command.

```nu
cd ~
use std/dirs
dirs
# => ╭───┬────────┬─────────────────────────────────╮
# => │ # │ active │              path               │
# => ├───┼────────┼─────────────────────────────────┤
# => │ 0 │ true   │ /home/myuser                    │
# => ╰───┴────────┴─────────────────────────────────╯

cd ~/src/repo/nushell
dirs
# => ╭───┬────────┬─────────────────────────────────╮
# => │ # │ active │              path               │
# => ├───┼────────┼─────────────────────────────────┤
# => │ 0 │ true   │ /home/myuser/repo/nushell       │
# => ╰───┴────────┴─────────────────────────────────╯

```

Notice that `cd` only changes the Active directory.

To _add_ the current directory to the list, change to a new active directory using the `dirs add` command:

```nu
dirs add ../reedline
dirs
# => ╭───┬────────┬──────────────────────────────────╮
# => │ # │ active │               path               │
# => ├───┼────────┼──────────────────────────────────┤
# => │ 0 │ false  │ /home/myuser/src/repo/nushell    │
# => │ 1 │ true   │ /home/myuser/src/repo/reedline   │
# => ╰───┴────────┴──────────────────────────────────╯
```

Let's go ahead and add a few more commonly used directories to the list:

```nu
dirs add ../nu_scripts
dirs add ~
dirs
# => ╭───┬────────┬────────────────────────────────────╮
# => │ # │ active │                path                │
# => ├───┼────────┼────────────────────────────────────┤
# => │ 0 │ false  │ /home/myuser/src/repo/nushell      │
# => │ 1 │ false  │ /home/myuser/src/repo/reedline     │
# => │ 2 │ false  │ /home/myuser/src/repo/nu_scripts   │
# => │ 3 │ true   │ /home/myuser                       │
# => ╰───┴────────┴────────────────────────────────────╯
```

We can now switch between them easily using `dirs next`, `dirs prev` or `dirs goto`:

```nu
dirs next
# Active was 3, is now 0
pwd
# => /home/myuser/src/repo/nushell
dirs goto 2
# => /home/myuser/src/repo/nu_scripts
```

When you have finished your work in a directory, you can drop it from the list using:

```nu
dirs drop
dirs
# => ╭───┬────────┬──────────────────────────────────╮
# => │ # │ active │               path               │
# => ├───┼────────┼──────────────────────────────────┤
# => │ 0 │ false  │ /home/myuser/src/repo/nushell    │
# => │ 1 │ true   │ /home/myuser/src/repo/reedline   │
# => │ 2 │ false  │ /home/myuser                     │
# => ╰───┴────────┴──────────────────────────────────╯
```

When we drop `nu_scripts` from the list, the previous directory (`reedline`) becomes active.

## `shells` Aliases

Some users may prefer to think of this feature as multiple "shells within shells", where each has its own directory.

The Standard Library provides a set of aliases that can be used in place of the `dirs` commands above.

Import them using:

```nu
use std/dirs shells-aliases *
```

The built-in aliases are:

| Alias    | Description                                              |
| -------- | -------------------------------------------------------- |
| `shells` | in place of `dirs` to list current "shells"/directories. |
| `enter`  | in place of `dirs add` to enter a new "shell"/dir.       |
| `dexit`  | in place of `dirs drop` to exit a "shell"/dir.           |
| `g`      | as an alias for `dirs goto`.                             |
| `n`      | for `dirs next`                                          |
| `p`      | for `dirs prev`                                          |

Of course, you can also define your own aliases if desired.
# Environment

A common task in a shell is to control the environment that external applications will use. This is often done automatically, as the environment is packaged up and given to the external application as it launches. Sometimes, though, we want to have more precise control over what environment variables an application sees.

You can see the current environment variables in the $env variable:

```nu
$env | table -e
# => ╭──────────────────────────────────┬───────────────────────────────────────────╮
# => │                                  │ ╭──────┬────────────────────────────────╮ │
# => │ ENV_CONVERSIONS                  │ │      │ ╭─────────────┬──────────────╮ │ │
# => │                                  │ │ PATH │ │ from_string │ <Closure 32> │ │ │
# => │                                  │ │      │ │ to_string   │ <Closure 34> │ │ │
# => │                                  │ │      │ ╰─────────────┴──────────────╯ │ │
# => │                                  │ │      │ ╭─────────────┬──────────────╮ │ │
# => │                                  │ │ Path │ │ from_string │ <Closure 36> │ │ │
# => │                                  │ │      │ │ to_string   │ <Closure 38> │ │ │
# => │                                  │ │      │ ╰─────────────┴──────────────╯ │ │
# => │                                  │ ╰──────┴────────────────────────────────╯ │
# => │ HOME                             │ /Users/jelle                              │
# => │ LSCOLORS                         │ GxFxCxDxBxegedabagaced                    │
# => | ...                              | ...                                       |
# => ╰──────────────────────────────────┴───────────────────────────────────────────╯
```

In Nushell, environment variables can be any value and have any type. You can see the type of an env variable with the describe command, for example: `$env.PROMPT_COMMAND | describe`.

To send environment variables to external applications, the values will need to be converted to strings. See [Environment variable conversions](#environment-variable-conversions) on how this works.

The environment is initially created from the Nu [configuration files](configuration.md) and from the environment that Nu is run inside of.

## Setting Environment Variables

There are several ways to set an environment variable:

### $env.VAR assignment

Using the `$env.VAR = "val"` is the most straightforward method

```nu
$env.FOO = 'BAR'
```

So, if you want to extend the Windows `Path` variable, for example, you could do that as follows.

```nu
$env.Path = ($env.Path | prepend 'C:\path\you\want\to\add')
```

Here we've prepended our folder to the existing folders in the Path, so it will have the highest priority.
If you want to give it the lowest priority instead, you can use the [`append`](/commands/docs/append.md) command.

### [`load-env`](/commands/docs/load-env.md)

If you have more than one environment variable you'd like to set, you can use [`load-env`](/commands/docs/load-env.md) to create a table of name/value pairs and load multiple variables at the same time:

```nu
load-env { "BOB": "FOO", "JAY": "BAR" }
```

### One-shot Environment Variables

These are defined to be active only temporarily for a duration of executing a code block.
See [Single-use environment variables](environment.md#single-use-environment-variables) for details.

### Calling a Command Defined with [`def --env`](/commands/docs/def.md)

See [Defining environment from custom commands](custom_commands.md#changing-the-environment-in-a-custom-command) for details.

### Using Module's Exports

See [Modules](modules.md) for details.

## Reading Environment Variables

Individual environment variables are fields of a record that is stored in the `$env` variable and can be read with `$env.VARIABLE`:

```nu
$env.FOO
# => BAR
```

Sometimes, you may want to access an environmental variable which might be unset. Consider using the [question mark operator](types_of_data.md#optional-cell-paths) to avoid an error:

```nu
$env.FOO | describe
# => Error: nu::shell::column_not_found
# => 
# =>   × Cannot find column
# =>    ╭─[entry #1:1:1]
# =>  1 │ $env.FOO
# =>    · ──┬─ ─┬─
# =>    ·   │   ╰── cannot find column 'FOO'
# =>    ·   ╰── value originates here
# =>    ╰────

$env.FOO? | describe
# => nothing

$env.FOO? | default "BAR"
# => BAR
```

Alternatively, you can check for the presence of an environmental variable with `in`:

```
$env.FOO
# => BAR

if "FOO" in $env {
    echo $env.FOO
}
# => BAR
```

### Case sensitivity

Nushell's `$env` is case-insensitive, regardless of the OS. Although `$env` behaves mostly like a record, it is special in that it ignores the case when reading or updating. This means, for example, you can use any of `$env.PATH`, `$env.Path`, or `$env.path`, and they all work the same on any OS.

If you want to read `$env` in a case-sensitive manner, use `$env | get --sensitive`.

## Scoping

When you set an environment variable, it will be available only in the current scope (the block you're in and any block inside of it).

Here is a small example to demonstrate the environment scoping:

```nu
$env.FOO = "BAR"
do {
    $env.FOO = "BAZ"
    $env.FOO == "BAZ"
}
# => true
$env.FOO == "BAR"
# => true
```

See also: [Changing the Environment in a Custom Command](./custom_commands.html#changing-the-environment-in-a-custom-command).

## Changing the Directory

A common task in a shell is to change the directory using the [`cd`](/commands/docs/cd.md) command. In Nushell, calling [`cd`](/commands/docs/cd.md) is equivalent to setting the `PWD` environment variable. Therefore, it follows the same rules as other environment variables (for example, scoping).

## Single-use Environment Variables

A common shorthand to set an environment variable once is available, inspired by Bash and others:

```nu
FOO=BAR $env.FOO
# => BAR
```

You can also use [`with-env`](/commands/docs/with-env.md) to do the same thing more explicitly:

```nu
with-env { FOO: BAR } { $env.FOO }
# => BAR
```

The [`with-env`](/commands/docs/with-env.md) command will temporarily set the environment variable to the value given (here: the variable "FOO" is given the value "BAR"). Once this is done, the [block](types_of_data.md#blocks) will run with this new environment variable set.

## Permanent Environment Variables

You can also set environment variables at startup so they are available for the duration of Nushell running. To do this, set an environment variable inside [the Nu configuration file](configuration.md).

For example:

```nu
# In config.nu
$env.FOO = 'BAR'
```

## Environment Variable Conversions

You can set the `ENV_CONVERSIONS` environment variable to convert other environment variables between a string and a value.
For example, the [default environment config](https://github.com/nushell/nushell/blob/main/crates/nu-utils/src/sample_config/default_env.nu) includes conversion of PATH (and Path used on Windows) environment variables from a string to a list.
After both `env.nu` and `config.nu` are loaded, any existing environment variable specified inside `ENV_CONVERSIONS` will be translated according to its `from_string` field into a value of any type.
External tools require environment variables to be strings, therefore, any non-string environment variable needs to be converted first.
The conversion of value -> string is set by the `to_string` field of `ENV_CONVERSIONS` and is done every time an external command is run.

Let's illustrate the conversions with an example.
Put the following in your config.nu:

```nu
$env.ENV_CONVERSIONS = {
    # ... you might have Path and PATH already there, add:
    FOO : {
        from_string: { |s| $s | split row '-' }
        to_string: { |v| $v | str join '-' }
    }
}
```

Now, within a Nushell instance:

```nu
with-env { FOO : 'a-b-c' } { nu }  # runs Nushell with FOO env. var. set to 'a-b-c'

$env.FOO
# =>   0   a
# =>   1   b
# =>   2   c
```

You can see the `$env.FOO` is now a list in a new Nushell instance with the updated config.
You can also test the conversion manually by

```nu
do $env.ENV_CONVERSIONS.FOO.from_string 'a-b-c'
```

Now, to test the conversion list -> string, run:

```nu
nu -c '$env.FOO'
# => a-b-c
```

Because `nu` is an external program, Nushell translated the `[ a b c ]` list according to `ENV_CONVERSIONS.FOO.to_string` and passed it to the `nu` process.
Running commands with `nu -c` does not load the config file, therefore the env conversion for `FOO` is missing and it is displayed as a plain string -- this way we can verify the translation was successful.
You can also run this step manually by `do $env.ENV_CONVERSIONS.FOO.to_string [a b c]`

_(Important! The environment conversion string -> value happens **after** the env.nu and config.nu are evaluated. All environment variables in env.nu and config.nu are still strings unless you set them manually to some other values.)_

## Removing Environment Variables

You can remove an environment variable only if it was set in the current scope via [`hide-env`](/commands/docs/hide-env.md):

```nu
$env.FOO = 'BAR'
# => ...
hide-env FOO
```

The hiding is also scoped which both allows you to remove an environment variable temporarily and prevents you from modifying a parent environment from within a child scope:

```nu
$env.FOO = 'BAR'
do {
  hide-env FOO
  # $env.FOO does not exist
}
$env.FOO
# => BAR
```
# `explore`

Explore is a table pager, just like `less` but for table structured data.

## Signature

`> explore --head --index --reverse --peek`

### Parameters

 -  `--head {bool}`: Show or hide column headers (default true)
 -  `--index, -i`: Show row indexes when viewing a list
 -  `--tail, -t`: Start with the viewport scrolled to the bottom
 -  `--peek, -p`: When quitting, output the value of the cell the cursor was on

## Get Started

```nu
ls | explore -i
```

![explore-ls-png](https://user-images.githubusercontent.com/20165848/207849604-421312e3-537f-4b2e-b83e-f1f83f2a79d5.png)

So the main point of [`explore`](/commands/docs/explore.md) is `:table` (Which you see on the above screenshot).

You can interact with it via `<Left>`, `<Right>`, `<Up>`, `<Down>` _arrow keys_. It also supports the `Vim` keybindings `<h>`, `<j>`, `<k>`, and `<l>`, `<Ctrl-f>` and `<Ctrl-b>`, and it supports the `Emacs` keybindings `<Ctrl-v>`, `<Alt-v>`, `<Ctrl-p>`, and `<Ctrl-n>`.

You can inspect a underlying values by entering into cursor mode. You can press either `<i>` or `<Enter>` to do so.
Then using _arrow keys_ you can choose a necessary cell.
And you'll be able to see it's underlying structure.

You can obtain more information about the various aspects of it by `:help`.

## Commands

[`explore`](/commands/docs/explore.md) has a list of built in commands you can use. Commands are run through pressing `<:>` and then a command name.

To find out the comprehensive list of commands you can type `:help`.

## Config

You can configure many things (including styles and colors), via config.
You can find an example configuration in [`default-config.nu`](https://github.com/nushell/nushell/blob/main/crates/nu-utils/src/sample_config/default_config.nu).

## Examples

### Peeking a Value

```nu
$nu | explore --peek
```

![explore-peek-gif](https://user-images.githubusercontent.com/20165848/207854897-35cb7b1d-7f7d-4ae2-9ec8-df19ac04ac99.gif)

### `:try` Command

There's an interactive environment which you can use to navigate through data using `nu`.

![explore-try-gif](https://user-images.githubusercontent.com/20165848/208159049-0954c327-9cdf-4cb3-a6e9-e3ba86fde55c.gif)

#### Keeping the chosen value by `$nu`

Remember you can combine it with `--peek`.

![explore-try-nu-gif](https://user-images.githubusercontent.com/20165848/208161203-96b51209-726d-449a-959a-48b205c6f55a.gif)
# Externs

Using external commands (a.k.a. binaries or applications) is a fundamental feature of any shell. Nushell allows custom commands to take advantage of many of its features, such as:

- Parse-time type checking
- Completions
- Syntax highlighting

Support for these features is provided using the `extern` keyword, which allows a full signature to be defined for external commands.

Here's a short example for the `ssh` command:

```nu
module "ssh extern" {
  def complete_none [] { [] }

  def complete_ssh_identity [] {
    ls ~/.ssh/id_*
    | where {|f|
        ($f.name | path parse | get extension) != "pub"
      }
    | get name
  }

  export extern ssh [
    destination?: string@complete_none  # Destination Host
    -p: int                             # Destination Port
    -i: string@complete_ssh_identity    # Identity File
  ]
}
use "ssh extern" ssh
```

Notice that the syntax here is similar to that of the `def` keyword when defining a custom command. You can describe flags, positional parameters, types, completers, and more.

This implementation:

- Will provide `-p` and `-i` (with descriptions) as possible completions for `ssh -`.
- Will perform parse-time type checking. Attempting to use a non-`int` for the port number will result in an error (and error-condition syntax highlighting).
- Will offer parse-time syntax highlighting based on the shapes of the arguments.
- Will offer any private key files in `~/.ssh` as completion values for the `-i` (identity) option
- Will not offer completions for the destination host. Without a completer that returns an empty list, Nushell would attempt to use the default "File" completer.

  See the [Nu_scripts Repository](https://github.com/nushell/nu_scripts/blob/main/custom-completions/ssh/ssh-completions.nu) for an implementation that retrieves hosts from the SSH config files.

::: tip Note
A Nushell comment that continues on the same line for argument documentation purposes requires a space before the ` #` pound sign.
:::

## Format Specifiers

Positional parameters can be made optional with a `?` (as seen above). The remaining (`rest`) parameters can be matched with `...` before the parameter name. For example:

```nu
export extern "git add" [
  ...pathspecs: path
  # …
]
```

## Limitations

There are a few limitations to the current `extern` syntax. In Nushell, flags and positional arguments are very flexible—flags can precede positional arguments, flags can be mixed into positional arguments, and flags can follow positional arguments. Many external commands are not this flexible. There is not yet a way to require a particular ordering of flags and positional arguments to the style required by the external.

The second limitation is that some externals require flags to be passed using `=` to separate the flag and the value. In Nushell, the `=` is a convenient optional syntax and there's currently no way to require its use.

In addition, externals called via the caret sigil (e.g., `^ssh`) are not recognized by `extern`.

Finally, some external commands support `-long` arguments using a single leading hyphen. Nushell `extern` syntax can not yet represent these arguments.
# Getting Started

Let's get started! :elephant:

The next sections will give you a [short tour of Nushell by example](quick_tour.md) (including how to get help from within Nushell) and show you how to [move around your file system](moving_around.md).

Then, because Nushell takes some design decisions that are quite different from typical shells or dynamic scripting languages, make sure to check out [Thinking in Nu](thinking_in_nu.md) where we explain some of these concepts.
# Hooks

Hooks allow you to run a code snippet at some predefined situations.
They are only available in the interactive mode ([REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)), they do not work if you run a Nushell with a script (`nu script.nu`) or commands (`nu -c "print foo"`) arguments.

Currently, we support these types of hooks:

- `pre_prompt` : Triggered before the prompt is drawn
- `pre_execution` : Triggered before the line input starts executing
- `env_change` : Triggered when an environment variable changes
- `display_output` : A block that the output is passed to
- `command_not_found` : Triggered when a command is not found

To make it clearer, we can break down Nushell's execution cycle.
The steps to evaluate one line in the REPL mode are as follows:

1. Check for `pre_prompt` hooks and run them
1. Check for `env_change` hooks and run them
1. Display prompt and wait for user input
1. After user typed something and pressed "Enter": Check for `pre_execution` hooks and run them
1. Parse and evaluate user input
1. If a command is not found: Run the `command_not_found` hook. If it returns a string, show it.
1. If `display_output` is defined, use it to print command output
1. Return to 1.

## Basic Hooks

To enable hooks, define them in your [config](configuration.md):

```nu
$env.config = {
    # ...other config...

    hooks: {
        pre_prompt: { print "pre prompt hook" }
        pre_execution: { print "pre exec hook" }
        env_change: {
            PWD: {|before, after| print $"changing directory from ($before) to ($after)" }
        }
    }
}
```

Try putting the above to your config, running Nushell and moving around your filesystem.
When you change a directory, the `PWD` environment variable changes and the change triggers the hook with the previous and the current values stored in `before` and `after` variables, respectively.

Instead of defining just a single hook per trigger, it is possible to define a **list of hooks** which will run in sequence:

```nu
$env.config = {
    ...other config...

    hooks: {
        pre_prompt: [
            { print "pre prompt hook" }
            { print "pre prompt hook2" }
        ]
        pre_execution: [
            { print "pre exec hook" }
            { print "pre exec hook2" }
        ]
        env_change: {
            PWD: [
                {|before, after| print $"changing directory from ($before) to ($after)" }
                {|before, after| print $"changing directory from ($before) to ($after) 2" }
            ]
        }
    }
}
```

Also, it might be more practical to update the existing config with new hooks, instead of defining the whole config from scratch:

```nu
$env.config = ($env.config | upsert hooks {
    pre_prompt: ...
    pre_execution: ...
    env_change: {
        PWD: ...
    }
})
```

## Changing Environment

One feature of the hooks is that they preserve the environment.
Environment variables defined inside the hook **block** will be preserved in a similar way as [`def --env`](environment.md#defining-environment-from-custom-commands).
You can test it with the following example:

```nu
$env.config = ($env.config | upsert hooks {
    pre_prompt: { $env.SPAM = "eggs" }
})

$env.SPAM
# => eggs
```

The hook blocks otherwise follow the general scoping rules, i.e., commands, aliases, etc. defined within the block will be thrown away once the block ends.

## Conditional Hooks

One thing you might be tempted to do is to activate an environment whenever you enter a directory:

```nu
$env.config = ($env.config | upsert hooks {
    env_change: {
        PWD: [
            {|before, after|
                if $after == /some/path/to/directory {
                    load-env { SPAM: eggs }
                }
            }
        ]
    }
})
```

This won't work because the environment will be active only within the [`if`](/commands/docs/if.md) block.
In this case, you could easily rewrite it as `load-env (if $after == ... { ... } else { {} })` but this pattern is fairly common and later we'll see that not all cases can be rewritten like this.

To deal with the above problem, we introduce another way to define a hook -- **a record**:

```nu
$env.config = ($env.config | upsert hooks {
    env_change: {
        PWD: [
            {
                condition: {|before, after| $after == /some/path/to/directory }
                code: {|before, after| load-env { SPAM: eggs } }
            }
        ]
    }
})
```

When the hook triggers, it evaluates the `condition` block.
If it returns `true`, the `code` block will be evaluated.
If it returns `false`, nothing will happen.
If it returns something else, an error will be thrown.
The `condition` field can also be omitted altogether in which case the hook will always evaluate.

The `pre_prompt` and `pre_execution` hook types also support the conditional hooks but they don't accept the `before` and `after` parameters.

## Hooks as Strings

So far a hook was defined as a block that preserves only the environment, but nothing else.
To be able to define commands or aliases, it is possible to define the `code` field as **a string**.
You can think of it as if you typed the string into the REPL and hit Enter.
So, the hook from the previous section can be also written as

```nu
$env.config = ($env.config | upsert hooks {
    pre_prompt: '$env.SPAM = "eggs"'
})

$env.SPAM
# => eggs
```

This feature can be used, for example, to conditionally bring in definitions based on the current directory:

```nu
$env.config = ($env.config | upsert hooks {
    env_change: {
        PWD: [
            {
                condition: {|_, after| $after == /some/path/to/directory }
                code: 'def foo [] { print "foo" }'
            }
            {
                condition: {|before, _| $before == /some/path/to/directory }
                code: 'hide foo'
            }
        ]
    }
})
```

When defining a hook as a string, the `$before` and `$after` variables are set to the previous and current environment variable value, respectively, similarly to the previous examples:

```nu
$env.config = ($env.config | upsert hooks {
    env_change: {
        PWD: {
            code: 'print $"changing directory from ($before) to ($after)"'
        }
    }
}
```

## Examples

### Adding a Single Hook to Existing Config

An example for PWD env change hook:

```nu
$env.config = ($env.config | upsert hooks.env_change.PWD {|config|
    let val = ($config | get -i hooks.env_change.PWD)

    if $val == null {
        $val | append {|before, after| print $"changing directory from ($before) to ($after)" }
    } else {
        [
            {|before, after| print $"changing directory from ($before) to ($after)" }
        ]
    }
})
```

### Automatically Activating an Environment when Entering a Directory

This one looks for `test-env.nu` in a directory

```nu
$env.config = ($env.config | upsert hooks.env_change.PWD {
    [
        {
            condition: {|_, after|
                ($after == '/path/to/target/dir'
                    and ($after | path join test-env.nu | path exists))
            }
            code: "overlay use test-env.nu"
        }
        {
            condition: {|before, after|
                ('/path/to/target/dir' not-in $after
                    and '/path/to/target/dir' in ($before | default "")
                    and 'test-env' in (overlay list))
            }
            code: "overlay hide test-env --keep-env [ PWD ]"
        }
    ]
})
```

### Filtering or Diverting Command Output

You can use the `display_output` hook to redirect the output of commands.
You should define a block that works on all value types.
The output of external commands is not filtered through `display_output`.

This hook can display the output in a separate window,
perhaps as rich HTML text. Here is the basic idea of how to do that:

```nu
$env.config = ($env.config | upsert hooks {
    display_output: { to html --partial --no-color | save --raw /tmp/nu-output.html }
})
```

You can view the result by opening `file:///tmp/nu-output.html` in
a web browser.
Of course this isn't very convenient unless you use
a browser that automatically reloads when the file changes.
Instead of the [`save`](/commands/docs/save.md) command, you would normally customize this
to send the HTML output to a desired window.

### Changing how Output is Displayed

You can change to default behavior of how output is displayed by using the `display_output` hook.
Here is an example that changes the default display behavior to show a table 1 layer deep if the terminal is wide enough, or collapse otherwise:

```nu
$env.config = ($env.config | upsert hooks {
    display_output: {if (term size).columns >= 100 { table -ed 1 } else { table }}
})
```

### `command_not_found` Hook in _Arch Linux_

The following hook uses the `pkgfile` command, to find which packages commands belong to in _Arch Linux_.

```nu
$env.config = {
    ...other config...

    hooks: {
        ...other hooks...

        command_not_found: {
            |cmd_name| (
                try {
                    let pkgs = (pkgfile --binaries --verbose $cmd_name)
                    if ($pkgs | is-empty) {
                        return null
                    }
                    (
                        $"(ansi $env.config.color_config.shape_external)($cmd_name)(ansi reset) " +
                        $"may be found in the following packages:\n($pkgs)"
                    )
                }
            )
        }
    }
}
```

### `command_not_found` Hook in _NixOS_

NixOS comes with the command `command-not-found`. We only need to plug it in the nushell hook:

```nu
$env.config.hooks.command_not_found = {
  |command_name|
  print (command-not-found $command_name | str trim)
}
```


### `command_not_found` Hook in _Windows_

The following hook uses the `ftype` command, to find program paths in _Windows_ that might be relevant to the user for `alias`-ing.

```nu
$env.config = {
    ...other config...

    hooks: {
        ...other hooks...

        command_not_found: {
            |cmd_name| (
                try {
                    let attrs = (
                        ftype | find $cmd_name | to text | lines | reduce -f [] { |line, acc|
                            $line | parse "{type}={path}" | append $acc
                        } | group-by path | transpose key value | each { |row|
                            { path: $row.key, types: ($row.value | get type | str join ", ") }
                        }
                    )
                    let len = ($attrs | length)

                    if $len == 0 {
                        return null
                    } else {
                        return ($attrs | table --collapse)
                    }
                }
            )
        }
    }
}
```
# How Nushell Code Gets Run

In [Thinking in Nu](./thinking_in_nu.md#think-of-nushell-as-a-compiled-language), we encouraged you to _"Think of Nushell as a compiled language"_ due to the way in which Nushell code is processed. We also covered several code examples that won't work in Nushell due that process.

The underlying reason for this is a strict separation of the **_parsing and evaluation_** stages that **_disallows `eval`-like functionality_**. In this section, we'll explain in detail what this means, why we're doing it, and what the implications are. The explanation aims to be as simple as possible, but it might help if you've programmed in another language before.

[[toc]]

## Interpreted vs. Compiled Languages

### Interpreted Languages

Nushell, Python, and Bash (and many others) are _"interpreted"_ languages.

Let's start with a simple "Hello, World!" Nushell program:

```nu
# hello.nu

print "Hello, World!"
```

Of course, this runs as expected using `nu hello.nu`. A similar program written in Python or Bash would look (and behave) nearly the same.

In _"interpreted languages"_ code usually gets handled something like this:

```
Source Code → Interpreter → Result
```

Nushell follows this pattern, and its "Interpreter" is split into two parts:

1. `Source Code → Parser → Intermediate Representation (IR)`
2. `IR → Evaluation Engine → Result`

First, the source code is analyzed by the Parser and converted into an intermediate representation (IR), which in Nushell's case is just a collection of data structures. Then, these data structures are passed to the Engine for evaluation and output of the results.

This, as well, is common in interpreted languages. For example, Python's source code is typically [converted into bytecode](https://github.com/python/cpython/blob/main/InternalDocs/interpreter.md) before evaluation.

### Compiled Languages

On the other side are languages that are typically "compiled", such as C, C++, or Rust. For example, here's a simple _"Hello, World!"_ in Rust:

```rust
// main.rs

fn main() {
    println!("Hello, World!");
}
```

To "run" this code, it must be:

1. Compiled into [machine code instructions](https://en.wikipedia.org/wiki/Machine_code)
2. The compilation results stored as a binary file one the disk

The first two steps are handled with `rustc main.rs`.

3. Then, to produce a result, you need to run the binary (`./main`), which passes the instructions to the CPU

So:

1. `Source Code ⇒ Compiler ⇒ Machine Code`
2. `Machine Code ⇒ CPU ⇒ Result`

::: important
You can see that the compile-run sequence is not much different from the parse-evaluate sequence of an interpreter. You begin with source code, parse (or compile) it into some state (e.g., bytecode, IR, machine code), then evaluate (or run) the IR to get a result. You could think of machine code as just another type of IR and the CPU as its interpreter.

One big difference, however, between interpreted and compiled languages is that interpreted languages typically implement an _`eval` function_ while compiled languages do not. What does this mean?
:::

## Dynamic vs. Static Languages

::: tip Terminology
In general, the difference between a dynamic and static language is how much of the source code is resolved during Compilation (or Parsing) vs. Evaluation/Runtime:

- _"Static"_ languages perform more code analysis (e.g., type-checking, [data ownership](https://doc.rust-lang.org/stable/book/ch04-00-understanding-ownership.html)) during Compilation/Parsing.

- _"Dynamic"_ languages perform more code analysis, including `eval` of additional code, during Evaluation/Runtime.

For the purposes of this discussion, the primary difference between a static and dynamic language is whether or not it has an `eval` function.

:::

### Eval Function

Most dynamic, interpreted languages have an `eval` function. For example, [Python `eval`](https://docs.python.org/3/library/functions.html#eval) (also, [Python `exec`](https://docs.python.org/3/library/functions.html#exec)) or [Bash `eval`](https://linux.die.net/man/1/bash).

The argument to an `eval` is _"source code inside of source code"_, typically conditionally or dynamically computed. This means that, when an interpreted language encounters an `eval` in source code during Parse/Eval, it typically interrupts the normal Evaluation process to start a new Parse/Eval on the source code argument to the `eval`.

Here's a simple Python `eval` example to demonstrate this (potentially confusing!) concept:

```python:line-numbers
# hello_eval.py

print("Hello, World!")
eval("print('Hello, Eval!')")
```

When you run the file (`python hello_eval.py`), you'll see two messages: _"Hello, World!"_ and _"Hello, Eval!"_. Here is what happens:

1. The entire program is Parsed
2. (Line 3) `print("Hello, World!")` is Evaluated
3. (Line 4) In order to evaluate `eval("print('Hello, Eval!')")`:
   1. `print('Hello, Eval!')` is Parsed
   2. `print('Hello, Eval!')` is Evaluated

::: tip More fun
Consider `eval("eval(\"print('Hello, Eval!')\")")` and so on!
:::

Notice how the use of `eval` here adds a new "meta" step into the execution process. Instead of a single Parse/Eval, the `eval` creates additional, "recursive" Parse/Eval steps instead. This means that the bytecode produced by the Python interpreter can be further modified during the evaluation.

Nushell does not allow this.

As mentioned above, without an `eval` function to modify the bytecode during the interpretation process, there's very little difference (at a high level) between the Parse/Eval process of an interpreted language and that of the Compile/Run in compiled languages like C++ and Rust.

::: tip Takeaway
This is why we recommend that you _"think of Nushell as a compiled language"_. Despite being an interpreted language, its lack of `eval` gives it some of the characteristic benefits as well as limitations common in traditional static, compiled languages.
:::

We'll dig deeper into what it means in the next section.

## Implications

Consider this Python example:

```python:line-numbers
exec("def hello(): print('Hello eval!')")
hello()
```

::: note
We're using `exec` in this example instead of `eval` because it can execute any valid Python code rather than being limited to `eval` expressions. The principle is similar in both cases, though.
:::

During interpretation:

1. The entire program is Parsed
2. In order to Evaluate Line 1:
   1. `def hello(): print('Hello eval!')` is Parsed
   2. `def hello(): print('Hello eval!')` is Evaluated
3. (Line 2) `hello()` is evaluated.

Note, that until step 2.2, the interpreter has no idea that a function `hello` even exists! This makes [static analysis](https://en.wikipedia.org/wiki/Static_program_analysis) of dynamic languages challenging. In this example, the existence of the `hello` function cannot be checked just by parsing (compiling) the source code. The interpreter must evaluate (run) the code to discover it.

- In a static, compiled language, a missing function is guaranteed to be caught at compile-time.
- In a dynamic, interpreted language, however, it becomes a _possible_ runtime error. If the `eval`-defined function is conditionally called, the error may not be discovered until that condition is met in production.

::: important
In Nushell, there are **exactly two steps**:

1. Parse the entire source code
2. Evaluate the entire source code

This is the complete Parse/Eval sequence.
:::

::: tip Takeaway
By not allowing `eval`-like functionality, Nushell prevents these types of `eval`-related bugs. Calling a non-existent definition is guaranteed to be caught at parse-time in Nushell.

Furthermore, after parsing completes, we can be certain the bytecode (IR) won't change during evaluation. This gives us a deep insight into the resulting bytecode (IR), allowing for powerful and reliable static analysis and IDE integration which can be challenging to achieve with more dynamic languages.

In general, you have more peace of mind that errors will be caught earlier when scaling Nushell programs.
:::

## The Nushell REPL

As with most any shell, Nushell has a _"Read→Eval→Print Loop"_ ([REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)) that is started when you run `nu` without any file. This is often thought of, but isn't quite the same, as the _"commandline"_.

::: tip Note
In this section, the `> ` character at the beginning of a line in a code-block is used to represent the commandline **_prompt_**. For instance:

```nu
> some code...
```

Code after the prompt in the following examples is executed by pressing the <kbd>Enter</kbd> key. For example:

```nu
> print "Hello world!"
# => Hello world!

> ls
# => prints files and directories...
```

The above means:

- From inside Nushell (launched with `nu`):
  1. Type `print "Hello world!"`
  1. Press <kbd>Enter</kbd>
  1. Nushell will display the result
  1. Type `ls`
  1. Press <kbd>Enter</kbd>
  1. Nushell will display the result

:::

When you press <kbd>Enter</kbd> after typing a commandline, Nushell:

1. **_(Read):_** Reads the commandline input
1. **_(Evaluate):_** Parses the commandline input
1. **_(Evaluate):_** Evaluates the commandline input
1. **_(Evaluate):_** Merges the environment (such as the current working directory) to the internal Nushell state
1. **_(Print):_** Displays the results (if non-`null`)
1. **_(Loop):_** Waits for another input

In other words, each REPL invocation is its own separate parse-evaluation sequence. By merging the environment back to the Nushell's state, we maintain continuity between the REPL invocations.

Compare a simplified version of the [`cd` example](./thinking_in_nu.md#example-change-to-a-different-directory-cd-and-source-a-file) from _"Thinking in Nu"_:

```nu
cd spam
source-env foo.nu
```

There we saw that this cannot work (as a script or other single expression) because the directory will be changed _after_ the parse-time [`source-env` keyword](/commands/docs/source-env.md) attempts to read the file.

Running these commands as separate REPL entries, however, works:

```nu
> cd spam
> source-env foo.nu
# Yay, works!
```

To see why, let's break down what happens in the example:

1. Read the `cd spam` commandline.
2. Parse the `cd spam` commandline.
3. Evaluate the `cd spam` commandline.
4. Merge environment (including the current directory) into the Nushell state.
5. Read and Parse `source-env foo.nu`.
6. Evaluate `source-env foo.nu`.
7. Merge environment (including any changes from `foo.nu`) into the Nushell state.

When `source-env` tries to open `foo.nu` during the parsing in Step 5, it can do so because the directory change from Step 3 was merged into the Nushell state during Step 4. As a result, it's visible in the following Parse/Eval cycles.

### Multiline REPL Commandlines

Keep in mind that this only works for **_separate_** commandlines.

In Nushell, it's possible to group multiple commands into one commandline using:

- A semicolon:

  ```nu
  cd spam; source-env foo.nu
  ```

- A newline:

  ```
  > cd span
    source-env foo.nu
  ```

  Notice there is no "prompt" before the second line. This type of multiline commandline is usually created with a [keybinding](./line_editor.md#keybindings) to insert a Newline when <kbd>Alt</kbd>+<kbd>Enter</kbd> or <kbd>Shift</kbd>+ <kbd>Enter</kbd> is pressed.

These two examples behave exactly the same in the Nushell REPL. The entire commandline (both statements) are processed a single Read→Eval→Print Loop. As such, they will fail the same way that the earlier script-example did.

::: tip
Multiline commandlines are very useful in Nushell, but watch out for any out-of-order Parser-keywords.
:::

## Parse-time Constant Evaluation

While it is impossible to add parsing into the evaluation stage and yet still maintain our static-language benefits, we can safely add _a little bit_ of evaluation into parsing.

::: tip Terminology
In the text below, we use the term _"constant"_ to refer to:

- A `const` definition
- The result of any command that outputs a constant value when provide constant inputs.
  :::

By their nature, **_constants_** and constant values are known at Parse-time. This, of course, is in sharp contrast to _variable_ declarations and values.

As a result, we can utilize constants as safe, known arguments to parse-time keywords like [`source`](/commands/docs/source.md), [`use`](/commands/docs/use.md), and related commands.

Consider [this example](./thinking_in_nu.md#example-dynamically-creating-a-filename-to-be-sourced) from _"Thinking in Nu"_:

```nu
let my_path = "~/nushell-files"
source $"($my_path)/common.nu"
```

As noted there, we **_can_**, however, do the following instead:

```nu:line-numbers
const my_path = "~/nushell-files"
source $"($my_path)/common.nu"
```

Let's analyze the Parse/Eval process for this version:

1. The entire program is Parsed into IR.

   1. Line 1: The `const` definition is parsed. Because it is a constant assignment (and `const` is also a parser-keyword), that assignment can also be Evaluated at this stage. Its name and value are stored by the Parser.
   2. Line 2: The `source` command is parsed. Because `source` is also a parser-keyword, it is Evaluated at this stage. In this example, however, it can be **_successfully_** parsed since its argument is **_known_** and can be retrieved at this point.
   3. The source-code of `~/nushell-files/common.nu` is parsed. If it is invalid, then an error will be generated, otherwise the IR results will be included in evaluation in the next stage.

2. The entire IR is Evaluated:
   1. Line 1: The `const` definition is Evaluated. The variable is added to the runtime stack.
   2. Line 2: The IR result from parsing `~/nushell-files/common.nu` is Evaluated.

::: important

- An `eval` adds additional parsing during evaluation
- Parse-time constants do the opposite, adding additional evaluation to the parser.
  :::

Also keep in mind that the evaluation allowed during parsing is **_very restricted_**. It is limited to only a small subset of what is allowed during a regular evaluation.

For example, the following is not allowed:

```nu
const foo_contents = (open foo.nu)
```

Put differently, only a small subset of commands and expressions can generate a constant value. For a command to be allowed:

- It must be designed to output a constant value
- All of its inputs must also be constant values, literals, or composite types (e.g., records, lists, tables) of literals.

In general, the commands and resulting expressions will be fairly simple and **_without side effects_**. Otherwise, the parser could all-too-easily enter an unrecoverable state. Imagine, for instance, attempting to assign an infinite stream to a constant. The Parse stage would never complete!

::: tip
You can see which Nushell commands can return constant values using:

```nu
help commands | where is_const
```

:::

For example, the `path join` command can output a constant value. Nushell also defines several useful paths in the `$nu` constant record. These can be combined to create useful parse-time constant evaluations like:

```nu
const my_startup_modules =  $nu.default-config-dir | path join "my-mods"
use $"($my_startup_modules)/my-utils.nu"
```

::: note Additional Notes
Compiled ("static") languages also tend to have a way to convey some logic at compile time. For instance:

- C's preprocessor
- Rust macros
- [Zig's comptime](https://kristoff.it/blog/what-is-zig-comptime), which was an inspiration for Nushell's parse-time constant evaluation.

There are two reasons for this:

1. _Increasing Runtime Performance:_ Logic in the compilation stage doesn't need to be repeated during runtime.

   This isn't currently applicable to Nushell, since the parsed results (IR) are not stored beyond Evaluation. However, this has certainly been considered as a possible future feature.

2. As with Nushell's parse-time constant evaluations, these features help (safely) work around limitations caused by the absence of an `eval` function.
   :::

## Conclusion

Nushell operates in a scripting language space typically dominated by _"dynamic"_, _"interpreted"_ languages, such as Python, Bash, Zsh, Fish, and many others. Nushell is also _"interpreted"_ since code is run immediately (without a separate, manual compilation).

However, is not _"dynamic"_ in that it does not have an `eval` construct. In this respect, it shares more in common with _"static"_, compiled languages like Rust or Zig.

This lack of `eval` is often surprising to many new users and is why it can be helpful to think of Nushell as a compiled, and static, language.
# Installing Nu

There are lots of ways to get Nu up and running. You can download pre-built binaries from our [release page](https://github.com/nushell/nushell/releases), [use your favourite package manager](https://repology.org/project/nushell/versions), or build from source.

The main Nushell binary is named `nu` (or `nu.exe` on Windows). After installation, you can launch it by typing `nu`.

@[code](@snippets/installation/run_nu.sh)

[[toc]]

## Pre-built Binaries

Nu binaries are published for Linux, macOS, and Windows [with each GitHub release](https://github.com/nushell/nushell/releases). Just download, extract the binaries, then copy them to a location on your PATH.

## Package Managers

Nu is available via several package managers:

[![Packaging status](https://repology.org/badge/vertical-allrepos/nushell.svg)](https://repology.org/project/nushell/versions)

For macOS and Linux, [Homebrew](https://brew.sh/) is a popular choice (`brew install nushell`).

For Windows:

- [Winget](https://docs.microsoft.com/en-us/windows/package-manager/winget/) (`winget install nushell`)
- [Chocolatey](https://chocolatey.org/) (`choco install nushell`)
- [Scoop](https://scoop.sh/) (`scoop install nu`)

Cross Platform installation:

- [npm](https://www.npmjs.com/) (`npm install -g nushell` Note that nu plugins are not included if you install in this way)

## Docker Container Images

Docker images are available from the GitHub Container Registry. An image for the latest release is built regularly
for Alpine and Debian. You can run the image in interactive mode using:

```nu
docker run -it --rm ghcr.io/nushell/nushell:<version>-<distro>
```

Where `<version>` is the version of Nushell you want to run and `<distro>` is `alpine` or the latest supported Debian release, such as `bookworm`.

To run a specific command, use:

```nu
docker run --rm ghcr.io/nushell/nushell:latest-alpine -c "ls /usr/bin | where size > 10KiB"
```

To run a script from the current directory using Bash, use:

```nu
docker run --rm \
    -v $(pwd):/work \
    ghcr.io/nushell/nushell:latest-alpine \
    "/work/script.nu"
```

## Build from Source

You can also build Nu from source. First, you will need to set up the Rust toolchain and its dependencies.

### Installing a Compiler Suite

For Rust to work properly, you'll need to have a compatible compiler suite installed on your system. These are the recommended compiler suites:

- Linux: GCC or Clang
- macOS: Clang (install Xcode)
- Windows: MSVC (install [Visual Studio](https://visualstudio.microsoft.com/vs/community/) or the [Visual Studio Build Tools](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022))
  - Make sure to install the "Desktop development with C++" workload
  - Any Visual Studio edition will work (Community is free)

### Installing Rust

If you don't already have Rust on our system, the best way to install it is via [rustup](https://rustup.rs/). Rustup is a way of managing Rust installations, including managing using different Rust versions.

Nu currently requires the **latest stable (1.66.1 or later)** version of Rust. The best way is to let `rustup` find the correct version for you. When you first open `rustup` it will ask what version of Rust you wish to install:

@[code](@snippets/installation/rustup_choose_rust_version.sh)

Once you are ready, press 1 and then enter.

If you'd rather not install Rust via `rustup`, you can also install it via other methods (e.g. from a package in a Linux distro). Just be sure to install a version of Rust that is 1.66.1 or later.

### Dependencies

#### Debian/Ubuntu

You will need to install the "pkg-config", "build-essential" and "libssl-dev" packages:

@[code](@snippets/installation/install_pkg_config_libssl_dev.sh)

#### RHEL based distros

You will need to install "libxcb", "openssl-devel" and "libX11-devel":

@[code](@snippets/installation/install_rhel_dependencies.sh)

#### macOS

##### Homebrew

Using [Homebrew](https://brew.sh/), you will need to install "openssl" and "cmake" using:

@[code](@snippets/installation/macos_deps.sh)

##### Nix

If using [Nix](https://nixos.org/download/#nix-install-macos) for package management on macOS, the `openssl`, `cmake`, `pkg-config`, and `curl` packages are required. These can be installed:

- Globally, using `nix-env --install` (and others).
- Locally, using [Home Manager](https://github.com/nix-community/home-manager) in your `home.nix` config.
- Temporarily, using `nix-shell` (and others).

### Build from [crates.io](https://crates.io) using Cargo

Nushell releases are published as source to the popular Rust package registry [crates.io](https://crates.io/). This makes it easy to build and install the latest Nu release with `cargo`:

```nu
cargo install nu --locked
```

The `cargo` tool will do the work of downloading Nu and its source dependencies, building it, and installing it into the cargo bin path.

Note that the default plugins must be installed separately when using `cargo`. See the [Plugins Installation](./plugins.html#core-plugins) section of the Book for instructions.

### Building from the GitHub repository

You can also build Nu from the latest source on GitHub. This gives you immediate access to the latest features and bug fixes. First, clone the repo:

@[code](@snippets/installation/git_clone_nu.sh)

From there, we can build and run Nu with:

@[code](@snippets/installation/build_nu_from_source.sh)

You can also build and run Nu in release mode, which enables more optimizations:

@[code](@snippets/installation/build_nu_from_source_release.sh)

People familiar with Rust may wonder why we do both a "build" and a "run" step if "run" does a build by default. This is to get around a shortcoming of the new `default-run` option in Cargo, and ensure that all plugins are built, though this may not be required in the future.
# Reedline, Nu's Line Editor

Nushell's line-editor [Reedline](https://github.com/nushell/reedline) is
cross-platform and designed to be modular and flexible. The line-editor is
in charge of controlling the command history, validations, completions, hints,
screen paint, and more.

[[toc]]

## Multi-line Editing

Reedline allows Nushell commandlines to extend across multiple lines. This can be accomplished using several methods:

1. Pressing <kbd>Enter</kbd> when a bracketed expression is open.

   For example:

   ```nu
   def my-command [] {
   ```

   Pressing <kbd>Enter </kbd> after the open-bracket will insert a newline. This will also occur with opening (and valid) `(` and `[` expressions.

   This is commonly used to create blocks and closures (as above), but also list, record, and table literals:

   ```nu
   let file = {
     name: 'repos.sqlite'
     hash: 'b939a3fa4ca011ca1aa3548420e78cee'
     version: '1.4.2'
   }
   ```

   It can even be used to continue a single command across multiple lines:

   ::: details Example

   ```nu
   (
     tar
     -cvz
     -f archive.tgz
     --exclude='*.temp'
     --directory=../project/
     ./
   )
   ```

   :::

2. Pressing <kbd>Enter</kbd> at the end of a line with a trailing pipe-symbol (`|`).

   ```nu
   ls                     |
   where name =~ '^[0-9]' | # Comments after a trailing pipe are okay
   get name               |
   mv ...$in ./backups/
   ```

3. Manually insert a newline using <kbd>Alt</kbd>+<kbd>Enter</kbd> or <kbd>Shift</kbd>+<kbd>Enter</kbd>.

   This can be used to create a somewhat more readable version of the previous commandline:

   ```nu
   ls
   | where name =~ '^[0-9]'  # Files starting with a digit
   | get name
   | mv ...$in ./backups/
   ```

   ::: tip
   It's possible that one or both of these keybindings may be intercepted by the terminal application or window-manager. For instance, Windows Terminal (and most other terminal applications on Windows) assign <kbd>Alt</kbd>+<kbd>Enter</kbd> to expand the terminal to full-screen. If neither of the above keybindings work in your terminal, you can assign a different keybinding to:

   ```nu
   event: { edit: insertnewline }
   ```

   See [Keybindings](#keybindings) below for more details.

   :::

4. Pressing <kbd>Ctrl</kbd>+<kbd>O</kbd> opens the current commandline in your editor. Saving the resulting file and exiting the editor will update the commandline with the results.

## Setting the Editing Mode

Reedline allows you to edit text using two modes — Vi and Emacs. If not
specified, the default mode is Emacs. To change the mode, use the
`edit_mode` setting.

```nu
$env.config.edit_mode = 'vi'
```

This can be changed at the commandline or persisted in `config.nu`.

::: note
Vi is a "modal" editor with "normal" mode and an "insert" mode. We recommend
becoming familiar with these modes through the use of the Vim or Neovim editors
before using Vi mode in Nushell. Each has a built-in tutorial covering the basics
(and more) of modal editing.
:::

## Default Keybindings

Each edit mode comes with common keybindings for Vi and Emacs text editing.

### Emacs and Vi-insert Keybindings

These keybinding events apply to both Emacs and Vi-insert mode:

| Key                                        | Event                               |
| ------------------------------------------ | ----------------------------------- |
| <kbd>Shift</kbd>+<kbd>Enter</kbd>          | Insert newline                      |
| <kbd>Alt</kbd>+<kbd>Enter</kbd>            | Insert newline                      |
| <kbd>Backspace</kbd>                       | Backspace                           |
| <kbd>End</kbd>                             | Move to end of line                 |
| <kbd>End</kbd>                             | Complete history hint               |
| <kbd>Home</kbd>                            | Move to line start                  |
| <kbd>Ctrl</kbd>+<kbd>C</kbd>               | Cancel current line                 |
| <kbd>Ctrl</kbd>+<kbd>L</kbd>               | Clear screen                        |
| <kbd>Ctrl</kbd>+<kbd>R</kbd>               | Search history                      |
| <kbd>Ctrl</kbd>+<kbd>→</kbd> (Right Arrow) | Complete history word               |
| <kbd>Ctrl</kbd>+<kbd>→</kbd> (Right Arrow) | Move word right                     |
| <kbd>Ctrl</kbd>+<kbd>←</kbd> (Left Arrow)  | Move word left                      |
| <kbd>↑</kbd> (Up Arrow)                    | Move up                             |
| <kbd>↓</kbd> (Down Arrow)                  | Move down                           |
| <kbd>←</kbd> (Left Arrow)                  | Move left                           |
| <kbd>→</kbd> (Right Arrow)                 | Move right                          |
| <kbd>Ctrl</kbd>+<kbd>P</kbd>               | Move up                             |
| <kbd>Ctrl</kbd>+<kbd>N</kbd>               | Move down                           |
| <kbd>Ctrl</kbd>+<kbd>B</kbd>               | Move left                           |
| <kbd>Ctrl</kbd>+<kbd>F</kbd>               | Move right                          |
| <kbd>→</kbd> (Right Arrow)                 | History-hint complete               |
| <kbd>Ctrl</kbd>+<kbd>F</kbd>               | History-hint complete               |
| <kbd>Alt</kbd>+<kbd>F</kbd>                | History-hint complete one word      |
| <kbd>Alt</kbd>+<kbd>←</kbd> (Left Arrow)   | History-hint complete one word less |

### Vi-insert Keybindings

These keybinding events apply only to Vi-insert mode:

| Key            | Event                    |
| -------------- | ------------------------ |
| <kbd>Esc</kbd> | Switch to Vi-normal mode |

### Vi-normal Keybindings

These keybinding events apply only to Vi-normal mode:

| Key                                         | Event               |
| ------------------------------------------- | ------------------- |
| <kbd>Ctrl</kbd>+<kbd>C</kbd>                | Cancel current line |
| <kbd>Ctrl</kbd>+<kbd>L</kbd>                | Clear screen        |
| <kbd>↑</kbd> (Up Arrow)                     | Move up             |
| <kbd>↓</kbd> (Down Arrow)                   | Move down           |
| <kbd>←</kbd> (Left Arrow)                   | Move left           |
| <kbd>→</kbd> (Right Arrow)                  | Move right          |
| <kbd>Ctrl></kbd>+<kbd>→</kbd> (Right Arrow) | Move right one word |
| <kbd>Ctrl></kbd>+<kbd>←</kbd> (Left Arrow)  | Move left one word  |

### Vi-normal Motions

As with Vi, many motions and actions can be combined with an optional count in normal-mode. For example, <kbd>3</kbd><kbd>d</kbd><kbd>w</kbd> deletes the next three words.

| Key                                    | Motion                                        |
| -------------------------------------- | --------------------------------------------- |
| <kbd>w</kbd>                           | Move to beginning of next word                |
| <kbd>e</kbd>                           | Move to end of current or next word           |
| <kbd>b</kbd>                           | Move to beginning of current or previous word |
| <kbd>0</kbd>                           | Move to start of line                         |
| <kbd>$</kbd>                           | Move to end of line                           |
| <kbd>h</kbd>                           | Move left                                     |
| <kbd>l</kbd>                           | Move right                                    |
| <kbd>j</kbd>                           | Move down                                     |
| <kbd>k</kbd>                           | Move up                                       |
| <kbd>f</kbd>+\<char\>                  | Move right to \<char\>                        |
| <kbd>t</kbd>+\<char\>                  | Move right to before \<char\>                 |
| <kbd>Shift</kbd>+<kbd>F</kbd>+\<char\> | Move left to \<char\>                         |
| <kbd>Shift</kbd>+<kbd>T</kbd>+\<char\> | Move left to after \<char\>                   |

### Vi-normal Actions

These actions can be combined with many of the [motions above](#vi-normal-motions).

| Key                           | Action                                             |
| ----------------------------- | -------------------------------------------------- |
| <kbd>d</kbd>                  | Delete                                             |
| <kbd>Shift</kbd>+<kbd>D</kbd> | Delete to end of line                              |
| <kbd>p</kbd>                  | Paste after current character                      |
| <kbd>Shift</kbd>+<kbd>P</kbd> | Paste before current character                     |
| <kbd>i</kbd>                  | Enter Vi insert-mode (append) at current character |
| <kbd>Shift</kbd>+<kbd>I</kbd> | Enter insert-mode at beginning of line             |
| <kbd>a</kbd>                  | Append after current character                     |
| <kbd>Shift</kbd>+<kbd>A</kbd> | Append to end of line                              |
| <kbd>0</kbd>                  | Move to start of line                              |
| <kbd>^</kbd>                  | Move to start of line                              |
| <kbd>$</kbd>                  | Move to end of line                                |
| <kbd>c</kbd>                  | Change                                             |
| <kbd>r</kbd>                  | Replace                                            |
| <kbd>s</kbd>                  | Substitute character(s)                            |
| <kbd>x</kbd>                  | Delete character                                   |
| <kbd>u</kbd>                  | Undo                                               |

## Command History

As mentioned before, Reedline manages and stores all the commands that are
edited and sent to Nushell. To configure the max number of records that
Reedline should store you will need to adjust this value in your config file:

```nu
  $env.config = {
    ...
    history: {
      ...
      max_size: 1000
      ...
    }
    ...
  }
```

## Customizing the Prompt

The Reedline prompt is configured using a number of environment variables. See [Prompt Configuration](./configuration.md#prompt-configuration) for details.

## Keybindings

Reedline keybindings are powerful constructs that let you build chains of
events that can be triggered with a specific combination of keys.

For example, let's say that you would like to map the completion menu to the
`Ctrl + t` keybinding (default is `tab`). You can add the next entry to your
config file.

```nu
  $env.config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: { send: menu name: completion_menu }
      }
    ]

    ...
  }
```

After loading this new `config.nu`, your new keybinding (`Ctrl + t`) will open
the completion command.

Each keybinding requires the next elements:

- name: Unique name for your keybinding for easy reference in `$config.keybindings`
- modifier: A key modifier for the keybinding. The options are:
  - none
  - control
  - alt
  - shift
  - shift_alt
  - alt_shift
  - control_alt
  - alt_control
  - control_shift
  - shift_control
  - control_alt_shift
  - control_shift_alt
- keycode: This represent the key to be pressed
- mode: emacs, vi_insert, vi_normal (a single string or a list. e.g.
  [`vi_insert` `vi_normal`])
- event: The type of event that is going to be sent by the keybinding. The
  options are:
  - send
  - edit
  - until

::: tip
All of the available modifiers, keycodes and events can be found with
the command [`keybindings list`](/commands/docs/keybindings_list.md)
:::

::: tip
The keybindings added to `vi_insert` mode will be available when the
line editor is in insert mode (when you can write text), and the keybindings
marked with `vi_normal` mode will be available when in normal (when the cursor
moves using h, j, k or l)
:::

The event section of the keybinding entry is where the actions to be performed
are defined. In this field you can use either a record or a list of records.
Something like this

```nu
  ...
  event: { send: Enter }
  ...
```

or

```nu
  ...
  event: [
    { edit: Clear }
    { send: Enter }
  ]
  ...
```

The first keybinding example shown in this page follows the first case; a
single event is sent to the engine.

The next keybinding is an example of a series of events sent to the engine. It
first clears the prompt, inserts a string and then enters that value

```nu
  $env.config = {
    ...

    keybindings: [
    {
      name: change_dir_with_fzf
      modifier: CONTROL
      keycode: Char_t
      mode: emacs
      event:[
          { edit: Clear }
          { edit: InsertString,
            value: "cd (ls | where type == dir | each { |row| $row.name} | str join (char nl) | fzf | decode utf-8 | str trim)"

          }
          { send: Enter }
        ]
    }

    ...
  }
```

One disadvantage of the previous keybinding is the fact that the inserted text
will be processed by the validator and saved in the history, making the
keybinding a bit slow and populating the command history with the same command.
For that reason there is the `executehostcommand` type of event. The next
example does the same as the previous one in a simpler way, sending a single
event to the engine

```nu
  $env.config = {
    ...

    keybindings: [
    {
      name: change_dir_with_fzf
      modifier: CONTROL
      keycode: Char_y
      mode: emacs
      event: {
        send: executehostcommand,
        cmd: "cd (ls | where type == dir | each { |row| $row.name} | str join (char nl) | fzf | decode utf-8 | str trim)"
      }
    }
  ]

    ...
  }
```

Before we continue you must have noticed that the syntax changes for edits and
sends, and for that reason it is important to explain them a bit more. A `send`
is all the `Reedline` events that can be processed by the engine and an `edit`
are all the `EditCommands` that can be processed by the engine.

### Send Type

To find all the available options for `send` you can use

```nu
keybindings list | where type == events
```

And the syntax for `send` events is the next one

```nu
    ...
      event: { send: <NAME OF EVENT FROM LIST> }
    ...
```

::: tip
You can write the name of the events with capital letters. The
keybinding parser is case insensitive
:::

There are two exceptions to this rule: the `Menu` and `ExecuteHostCommand`.
Those two events require an extra field to be complete. The `Menu` needs the
name of the menu to be activated (completion_menu or history_menu)

```nu
    ...
      event: {
        send: menu
        name: completion_menu
      }
    ...
```

and the `ExecuteHostCommand` requires a valid command that will be sent to the
engine

```nu
    ...
      event: {
        send: executehostcommand
        cmd: "cd ~"
      }
    ...
```

It is worth mentioning that in the events list you will also see `Edit([])`,
`Multiple([])` and `UntilFound([])`. These options are not available for the
parser since they are constructed based on the keybinding definition. For
example, a `Multiple([])` event is built for you when defining a list of
records in the keybinding's event. An `Edit([])` event is the same as the
`edit` type that was mentioned. And the `UntilFound([])` event is the same as
the `until` type mentioned before.

### Edit Type

The `edit` type is the simplification of the `Edit([])` event. The `event` type
simplifies defining complex editing events for the keybindings. To list the
available options you can use the next command

```nu
keybindings list | where type == edits
```

The usual syntax for an `edit` is the next one

```nu
    ...
      event: { edit: <NAME OF EDIT FROM LIST> }
    ...
```

The syntax for the edits in the list that have a `()` changes a little bit.
Since those edits require an extra value to be fully defined. For example, if
we would like to insert a string where the prompt is located, then you will
have to use

```nu
    ...
      event: {
        edit: insertstring
        value: "MY NEW STRING"
      }
    ...
```

or say you want to move right until the first `S`

```nu
    ...
      event: {
        edit: moverightuntil
        value: "S"
      }
    ...
```

As you can see, these two types will allow you to construct any type of
keybinding that you require

### Until Type

To complete this keybinding tour we need to discuss the `until` type for event.
As you have seen so far, you can send a single event or a list of events. And
as we have seen, when a list of events is sent, each and every one of them is
processed.

However, there may be cases when you want to assign different events to the
same keybinding. This is especially useful with Nushell menus. For example, say
you still want to activate your completion menu with `Ctrl + t` but you also
want to move to the next element in the menu once it is activated using the
same keybinding.

For these cases, we have the `until` keyword. The events listed inside the
until event will be processed one by one with the difference that as soon as
one is successful, the event processing is stopped.

The next keybinding represents this case.

```nu
  $env.config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: {
          until: [
            { send: menu name: completion_menu }
            { send: menunext }
          ]
        }
      }
    ]

    ...
  }
```

The previous keybinding will first try to open a completion menu. If the menu
is not active, it will activate it and send a success signal. If the keybinding
is pressed again, since there is an active menu, then the next event it will
send is MenuNext, which means that it will move the selector to the next
element in the menu.

As you can see the `until` keyword allows us to define two events for the same
keybinding. At the moment of this writing, only the Menu events allow this type
of layering. The other non menu event types will always return a success value,
meaning that the `until` event will stop as soon as it reaches the command.

For example, the next keybinding will always send a `down` because that event
is always successful

```nu
  $env.config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: emacs
        event: {
          until: [
            { send: down }
            { send: menu name: completion_menu }
            { send: menunext }
          ]
        }
      }
    ]

    ...
  }
```

### Removing a Default Keybinding

If you want to remove a certain default keybinding without replacing it with a different action, you can set `event: null`.

e.g. to disable screen clearing with `Ctrl + l` for all edit modes

```nu
  $env.config = {
    ...

    keybindings: [
      {
        modifier: control
        keycode: char_l
        mode: [emacs, vi_normal, vi_insert]
        event: null
      }
    ]

    ...
  }

```

### Troubleshooting Keybinding Problems

Your terminal environment may not always propagate your key combinations on to Nushell the way you expect it to. You can use the command [`keybindings listen`](/commands/docs/keybindings_listen.md) to determine if certain keypresses are actually received by Nushell, and how.

## Menus

Thanks to Reedline, Nushell has menus that can help you with your day to day
shell scripting. Next we present the default menus that are always available
when using Nushell

### Menu Keybindings

When a menu is active, some keybindings change based on the keybinding [`until` specifier](#until-type) discussed above. Common keybindings for menus are:

| Key                             | Event                |
| ------------------------------- | -------------------- |
| <kbd>Tab</kbd>                  | Select next item     |
| <kbd>Shift</kbd>+<kbd>Tab</kbd> | Select previous item |
| <kbd>Enter</kbd>                | Accept selection     |
| <kbd>↑</kbd> (Up Arrow)         | Move menu up         |
| <kbd>↓</kbd> (Down Arrow)       | Move menu down       |
| <kbd>←</kbd> (Left Arrow)       | Move menu left       |
| <kbd>→</kbd> (Right Arrow)      | Move menu right      |
| <kbd>Ctrl</kbd>+<kbd>P</kbd>    | Move menu up         |
| <kbd>Ctrl</kbd>+<kbd>N</kbd>    | Move menu down       |
| <kbd>Ctrl</kbd>+<kbd>B</kbd>    | Move menu left       |
| <kbd>Ctrl</kbd>+<kbd>F</kbd>    | Move menu right      |

::: note
Menu direction behavior varies based on the menu type (see below). For example,
in a `description` menu, "Up" and "Down" apply to the "Extra" list, but in a
`list` menu the directions apply to the selection.
:::

### Help Menu

The help menu is there to ease your transition into Nushell. Say you are
putting together an amazing pipeline and then you forgot the internal command
that would reverse a string for you. Instead of deleting your pipe, you can
activate the help menu with `F1`. Once active just type keywords for the
command you are looking for and the menu will show you commands that match your
input. The matching is done on the name of the commands or the commands
description.

To navigate the menu you can select the next element by using `tab`, you can
scroll the description by pressing left or right and you can even paste into
the line the available command examples.

The help menu can be configured by modifying the next parameters

```nu
  $env.config = {
    ...

    menus = [
      ...
      {
        name: help_menu
        only_buffer_difference: true # Search is done on the text written after activating the menu
        marker: "? "                 # Indicator that appears with the menu is active
        type: {
            layout: description      # Type of menu
            columns: 4               # Number of columns where the options are displayed
            col_width: 20            # Optional value. If missing all the screen width is used to calculate column width
            col_padding: 2           # Padding between columns
            selection_rows: 4        # Number of rows allowed to display found options
            description_rows: 10     # Number of rows allowed to display command description
        }
        style: {
            text: green                   # Text style
            selected_text: green_reverse  # Text style for selected option
            description_text: yellow      # Text style for description
        }
      }
      ...
    ]
    ...
```

### Completion Menu

The completion menu is a context sensitive menu that will present suggestions
based on the status of the prompt. These suggestions can range from path
suggestions to command alternatives. While writing a command, you can activate
the menu to see available flags for an internal command. Also, if you have
defined your custom completions for external commands, these will appear in the
menu as well.

The completion menu by default is accessed by pressing `tab` and it can be configured by
modifying these values from the config object:

```nu
  $env.config = {
    ...

    menus: [
      ...
      {
        name: completion_menu
        only_buffer_difference: false # Search is done on the text written after activating the menu
        marker: "| "                  # Indicator that appears with the menu is active
        type: {
            layout: columnar          # Type of menu
            columns: 4                # Number of columns where the options are displayed
            col_width: 20             # Optional value. If missing all the screen width is used to calculate column width
            col_padding: 2            # Padding between columns
        }
        style: {
            text: green                   # Text style
            selected_text: green_reverse  # Text style for selected option
            description_text: yellow      # Text style for description
        }
      }
      ...
    ]
    ...
```

By modifying these parameters you can customize the layout of your menu to your
liking.

### History Menu

The history menu is a handy way to access the editor history. When activating
the menu (default `Ctrl+r`) the command history is presented in reverse
chronological order, making it extremely easy to select a previous command.

The history menu can be configured by modifying these values from the config object:

```nu
  $env.config = {
    ...

    menus = [
      ...
      {
        name: history_menu
        only_buffer_difference: true # Search is done on the text written after activating the menu
        marker: "? "                 # Indicator that appears with the menu is active
        type: {
            layout: list             # Type of menu
            page_size: 10            # Number of entries that will presented when activating the menu
        }
        style: {
            text: green                   # Text style
            selected_text: green_reverse  # Text style for selected option
            description_text: yellow      # Text style for description
        }
      }
      ...
    ]
    ...
```

When the history menu is activated, it pulls `page_size` records from the
history and presents them in the menu. If there is space in the terminal, when
you press `Ctrl+x` again the menu will pull the same number of records and
append them to the current page. If it isn't possible to present all the pulled
records, the menu will create a new page. The pages can be navigated by
pressing `Ctrl+z` to go to previous page or `Ctrl+x` to go to next page.

#### Searching the History

To search in your history you can start typing key words for the command you
are looking for. Once the menu is activated, anything that you type will be
replaced by the selected command from your history. for example, say that you
have already typed this

```nu
let a = ()
```

you can place the cursor inside the `()` and activate the menu. You can filter
the history by typing key words and as soon as you select an entry, the typed
words will be replaced

```nu
let a = (ls | where size > 10MiB)
```

#### Menu Quick Selection

Another nice feature of the menu is the ability to quick select something from
it. Say you have activated your menu and it looks like this

```nu
>
0: ls | where size > 10MiB
1: ls | where size > 20MiB
2: ls | where size > 30MiB
3: ls | where size > 40MiB
```

Instead of pressing down to select the fourth entry, you can type `!3` and
press enter. This will insert the selected text in the prompt position, saving
you time scrolling down the menu.

History search and quick selection can be used together. You can activate the
menu, do a quick search, and then quick select using the quick selection
character.

### User Defined Menus

In case you find that the default menus are not enough for you and you have
the need to create your own menu, Nushell can help you with that.

In order to add a new menu that fulfills your needs, you can use one of the default
layouts as a template. The templates available in nushell are columnar, list or
description.

The columnar menu will show you data in a columnar fashion adjusting the column
number based on the size of the text displayed in your columns.

The list type of menu will always display suggestions as a list, giving you the
option to select values using `!` plus number combination.

The description type will give you more space to display a description for some
values, together with extra information that could be inserted into the buffer.

Let's say we want to create a menu that displays all the variables created
during your session, we are going to call it `vars_menu`. This menu will use a
list layout (layout: list). To search for values, we want to use only the things
that are written after the menu has been activated (only_buffer_difference:
true).

With that in mind, the desired menu would look like this

```nu
  $env.config = {
    ...

    menus = [
      ...
      {
        name: vars_menu
        only_buffer_difference: true
        marker: "# "
        type: {
            layout: list
            page_size: 10
        }
        style: {
            text: green
            selected_text: green_reverse
            description_text: yellow
        }
        source: { |buffer, position|
            scope variables
            | where name =~ $buffer
            | sort-by name
            | each { |row| {value: $row.name description: $row.type} }
        }
      }
      ...
    ]
    ...
```

As you can see, the new menu is identical to the `history_menu` previously
described. The only huge difference is the new field called [`source`](/commands/docs/source.md). The
[`source`](/commands/docs/source.md) field is a nushell definition of the values you want to display in the
menu. For this menu we are extracting the data from `scope variables` and we
are using it to create records that will be used to populate the menu.

The required structure for the record is the next one

```nu
{
  value:       # The value that will be inserted in the buffer
  description: # Optional. Description that will be display with the selected value
  span: {      # Optional. Span indicating what section of the string will be replaced by the value
    start:
    end:
  }
  extra: [string] # Optional. A list of strings that will be displayed with the selected value. Only works with a description menu
}
```

For the menu to display something, at least the `value` field has to be present
in the resulting record.

In order to make the menu interactive, these two variables are available in
the block: `$buffer` and `$position`. The `$buffer` contains the value captured
by the menu, when the option `only_buffer_difference` is true, `$buffer` is the
text written after the menu was activated. If `only_buffer_difference` is
false, `$buffer` is all the string in line. The `$position` variable can be
used to create replacement spans based on the idea you had for your menu. The
value of `$position` changes based on whether `only_buffer_difference` is true
or false. When true, `$position` is the starting position in the string where
text was inserted after the menu was activated. When the value is false,
`$position` indicates the actual cursor position.

Using this information, you can design your menu to present the information you
require and to replace that value in the location you need it. The only thing
extra that you need to play with your menu is to define a keybinding that will
activate your brand new menu.

### Menu Keybindings

In case you want to change the default way both menus are activated, you can
change that by defining new keybindings. For example, the next two keybindings
assign the completion and history menu to `Ctrl+t` and `Ctrl+y` respectively

```nu
  $env.config = {
    ...

    keybindings: [
      {
        name: completion_menu
        modifier: control
        keycode: char_t
        mode: [vi_insert vi_normal]
        event: {
          until: [
            { send: menu name: completion_menu }
            { send: menupagenext }
          ]
        }
      }
      {
        name: history_menu
        modifier: control
        keycode: char_y
        mode: [vi_insert vi_normal]
        event: {
          until: [
            { send: menu name: history_menu }
            { send: menupagenext }
          ]
        }
      }
    ]

    ...
  }
```
# Loading Data

Earlier, we saw how you can use commands like [`ls`](/commands/docs/ls.md), [`ps`](/commands/docs/ps.md), [`date`](/commands/docs/date.md), and [`sys`](/commands/docs/sys.md) to load information about your files, processes, time of day, and the system itself. Each command gives us a table of information that we can explore. There are other ways we can load in a table of data to work with.

## Opening files

One of Nu's most powerful assets in working with data is the [`open`](/commands/docs/open.md) command. It is a multi-tool that can work with a number of different data formats. To see what this means, let's try opening a json file:

@[code](@snippets/loading_data/vscode.sh)

In a similar way to [`ls`](/commands/docs/ls.md), opening a file type that Nu understands will give us back something that is more than just text (or a stream of bytes). Here we open a "package.json" file from a JavaScript project. Nu can recognize the JSON text and parse it to a table of data.

If we wanted to check the version of the project we were looking at, we can use the [`get`](/commands/docs/get.md) command.

```nu
open editors/vscode/package.json | get version
# => 1.0.0
```

Nu currently supports the following formats for loading data directly into tables:

- csv
- eml
- ics
- ini
- json
- [nuon](#nuon)
- ods
- [SQLite databases](#sqlite)
- ssv
- toml
- tsv
- url
- vcf
- xlsx / xls
- xml
- yaml / yml

::: tip Did you know?
Under the hood `open` will look for a `from ...` subcommand in your scope which matches the extension of your file.
You can thus simply extend the set of supported file types of `open` by creating your own `from ...` subcommand.
:::

But what happens if you load a text file that isn't one of these? Let's try it:

```nu
open README.md
```

We're shown the contents of the file.

Below the surface, what Nu sees in these text files is one large string. Next, we'll talk about how to work with these strings to get the data we need out of them.

## NUON

Nushell Object Notation (NUON) aims to be for Nushell what JavaScript Object Notation (JSON) is for JavaScript.
That is, NUON code is a valid Nushell code that describes some data structure.
For example, this is a valid NUON (example from the [default configuration file](https://github.com/nushell/nushell/blob/main/crates/nu-utils/src/default_files/default_config.nu)):

```nu
{
  menus: [
    # Configuration for default nushell menus
    # Note the lack of source parameter
    {
      name: completion_menu
      only_buffer_difference: false
      marker: "| "
      type: {
          layout: columnar
          columns: 4
          col_width: 20   # Optional value. If missing all the screen width is used to calculate column width
          col_padding: 2
      }
      style: {
          text: green
          selected_text: green_reverse
          description_text: yellow
      }
    }
  ]
}
```

You might notice it is quite similar to JSON, and you're right.
**NUON is a superset of JSON!**
That is, any JSON code is a valid NUON code, therefore a valid Nushell code.
Compared to JSON, NUON is more "human-friendly".
For example, comments are allowed and commas are not required.

One limitation of NUON currently is that it cannot represent all of the Nushell [data types](types_of_data.md).
Most notably, NUON does not allow the serialization of blocks.

## Handling Strings

An important part of working with data coming from outside Nu is that it's not always in a format that Nu understands. Often this data is given to us as a string.

Let's imagine that we're given this data file:

```nu
open people.txt
# => Octavia | Butler | Writer
# => Bob | Ross | Painter
# => Antonio | Vivaldi | Composer
```

Each bit of data we want is separated by the pipe ('|') symbol, and each person is on a separate line. Nu doesn't have a pipe-delimited file format by default, so we'll have to parse this ourselves.

The first thing we want to do when bringing in the file is to work with it a line at a time:

```nu
open people.txt | lines
# => ───┬──────────────────────────────
# =>  0 │ Octavia | Butler | Writer
# =>  1 │ Bob | Ross | Painter
# =>  2 │ Antonio | Vivaldi | Composer
# => ───┴──────────────────────────────
```

We can see that we're working with the lines because we're back into a list. Our next step is to see if we can split up the rows into something a little more useful. For that, we'll use the [`split`](/commands/docs/split.md) command. [`split`](/commands/docs/split.md), as the name implies, gives us a way to split a delimited string. We will use [`split`](/commands/docs/split.md)'s `column` subcommand to split the contents across multiple columns. We tell it what the delimiter is, and it does the rest:

```nu
open people.txt | lines | split column "|"
# => ───┬──────────┬───────────┬───────────
# =>  # │ column1  │ column2   │ column3
# => ───┼──────────┼───────────┼───────────
# =>  0 │ Octavia  │  Butler   │  Writer
# =>  1 │ Bob      │  Ross     │  Painter
# =>  2 │ Antonio  │  Vivaldi  │  Composer
# => ───┴──────────┴───────────┴───────────
```

That _almost_ looks correct. It looks like there's an extra space there. Let's [`trim`](/commands/docs/str_trim.md) that extra space:

```nu
open people.txt | lines | split column "|" | str trim
# => ───┬─────────┬─────────┬──────────
# =>  # │ column1 │ column2 │ column3
# => ───┼─────────┼─────────┼──────────
# =>  0 │ Octavia │ Butler  │ Writer
# =>  1 │ Bob     │ Ross    │ Painter
# =>  2 │ Antonio │ Vivaldi │ Composer
# => ───┴─────────┴─────────┴──────────
```

Not bad. The [`split`](/commands/docs/split.md) command gives us data we can use. It also goes ahead and gives us default column names:

```nu
open people.txt | lines | split column "|" | str trim | get column1
# => ───┬─────────
# =>  0 │ Octavia
# =>  1 │ Bob
# =>  2 │ Antonio
# => ───┴─────────
```

We can also name our columns instead of using the default names:

```nu
open people.txt | lines | split column "|" first_name last_name job | str trim
# => ───┬────────────┬───────────┬──────────
# =>  # │ first_name │ last_name │ job
# => ───┼────────────┼───────────┼──────────
# =>  0 │ Octavia    │ Butler    │ Writer
# =>  1 │ Bob        │ Ross      │ Painter
# =>  2 │ Antonio    │ Vivaldi   │ Composer
# => ───┴────────────┴───────────┴──────────
```

Now that our data is in a table, we can use all the commands we've used on tables before:

```nu
open people.txt | lines | split column "|" first_name last_name job | str trim | sort-by first_name
# => ───┬────────────┬───────────┬──────────
# =>  # │ first_name │ last_name │ job
# => ───┼────────────┼───────────┼──────────
# =>  0 │ Antonio    │ Vivaldi   │ Composer
# =>  1 │ Bob        │ Ross      │ Painter
# =>  2 │ Octavia    │ Butler    │ Writer
# => ───┴────────────┴───────────┴──────────
```

There are other commands you can use to work with strings:

- [`str`](/commands/docs/str.md)
- [`lines`](/commands/docs/lines.md)

There is also a set of helper commands we can call if we know the data has a structure that Nu should be able to understand. For example, let's open a Rust lock file:

```nu
open Cargo.lock
# => # This file is automatically @generated by Cargo.
# => # It is not intended for manual editing.
# => [[package]]
# => name = "adhoc_derive"
# => version = "0.1.2"
```

The "Cargo.lock" file is actually a .toml file, but the file extension isn't .toml. That's okay, we can use the [`from`](/commands/docs/from.md) command using the `toml` subcommand:

@[code](@snippets/loading_data/cargo-toml.sh)

The [`from`](/commands/docs/from.md) command can be used for each of the structured data text formats that Nu can open and understand by passing it the supported format as a subcommand.

## Opening in raw mode

While it's helpful to be able to open a file and immediately work with a table of its data, this is not always what you want to do. To get to the underlying text, the [`open`](/commands/docs/open.md) command can take an optional `--raw` flag:

```nu
open Cargo.toml --raw
# => [package]                                                                                        name = "nu"
# => version = "0.1.3"
# => authors = ["Yehuda Katz <wycats@gmail.com>", "Sophia Turner <547158+sophiajt@users.noreply.github.com>"]
# => description = "A shell for the GitHub era"
# => license = "MIT"
```

## SQLite

SQLite databases are automatically detected by [`open`](/commands/docs/open.md), no matter what their file extension is. You can open a whole database:

```nu
open foo.db
```

Or [`get`](/commands/docs/get.md) a specific table:

```nu
open foo.db | get some_table
```

Or run any SQL query you like:

```nu
open foo.db | query db "select * from some_table"
```

(Note: some older versions of Nu use `into db | query` instead of `query db` )

## Fetching URLs

In addition to loading files from your filesystem, you can also load URLs by using the [`http get`](/commands/docs/http.md) command. This will fetch the contents of the URL from the internet and return it:

@[code](@snippets/loading_data/rust-lang-feed.sh)
# Metadata

In using Nu, you may have come across times where you felt like there was something extra going on behind the scenes. For example, let's say that you try to open a file that Nu supports only to forget and try to convert again:

```nu
open Cargo.toml | from toml
# => error: Expected a string from pipeline
# => - shell:1:18
# => 1 | open Cargo.toml | from toml
# =>   |                   ^^^^^^^^^ requires string input
# => - shell:1:5
# => 1 | open Cargo.toml | from toml
# =>   |      ---------- object originates from here
```

The error message tells us not only that what we gave [`from toml`](/commands/docs/from_toml.md) wasn't a string, but also where the value originally came from. How would it know that?

Values that flow through a pipeline in Nu often have a set of additional information, or metadata, attached to them. These are known as tags, like the tags on an item in a store. These tags don't affect the data, but they give Nu a way to improve the experience of working with that data.

Let's run the [`open`](/commands/docs/open.md) command again, but this time, we'll look at the tags it gives back:

```nu
metadata (open Cargo.toml)
# => ╭──────┬───────────────────╮
# => │ span │ {record 2 fields} │
# => ╰──────┴───────────────────╯
```

Currently, we track only the span of where values come from. Let's take a closer look at that:

```nu
metadata (open Cargo.toml) | get span
# => ╭───────┬────────╮
# => │ start │ 212970 │
# => │ end   │ 212987 │
# => ╰───────┴────────╯
```

The span "start" and "end" here refer to where the underline will be in the line. If you count over 5, and then count up to 15, you'll see it lines up with the "Cargo.toml" filename. This is how the error we saw earlier knew what to underline.
# Modules

Like many programming languages, Nushell supports writing and using _modules_ as a way to organize code more efficiently. Modules can be thought of a "containers" that hold various definitions, including:

- Additional custom commands
- Aliases
- Constants
- Externs
- Environment variables
- And even other modules (a.k.a., submodules)

Many users will start off using modules written by others, then progress to writing their own. This chapter covers those two topics:

- [Using Modules](./modules/using_modules.md)
- [Creating Modules](./modules/creating_modules.md)
# Moving Around the System

A defining characteristic of a shell is the ability to navigate and interact with the filesystem. Nushell is, of course, no exception. Here are some common commands you might use when interacting with the filesystem:

## Viewing Directory Contents

```nu
ls
```

As seen in the Quick Tour, the [`ls`](/commands/docs/ls.md) command returns the contents of a directory. Nushell's `ls` will return the contents as a [table](types_of_data.html#tables).

The [`ls`](/commands/docs/ls.md) command also takes an optional argument to change what you'd like to view. For example, we can list the files that end in ".md"

```nu
ls *.md
# => ╭───┬────────────────────┬──────┬──────────┬──────────────╮
# => │ # │        name        │ type │   size   │   modified   │
# => ├───┼────────────────────┼──────┼──────────┼──────────────┤
# => │ 0 │ CODE_OF_CONDUCT.md │ file │  3.4 KiB │ 9 months ago │
# => │ 1 │ CONTRIBUTING.md    │ file │ 11.0 KiB │ 5 months ago │
# => │ 2 │ README.md          │ file │ 12.0 KiB │ 6 days ago   │
# => │ 3 │ SECURITY.md        │ file │  2.6 KiB │ 2 months ago │
# => ╰───┴────────────────────┴──────┴──────────┴──────────────╯
```

## Glob Patterns (wildcards)

The asterisk (`*`) in the above optional argument `*.md` is sometimes called a wildcard or a glob. It lets us match anything. You can read this glob `*.md` as _"match any filename, so long as it ends with '.md'."_

The most general glob is `*`, which will match all paths. More often, you'll see this pattern used as part of another pattern, for example `*.bak` and `temp*`.

Nushell also supports a double `*` which will traverse paths that are nested inside of other directories. For example, `ls **/*` will list all the non-hidden paths nested under the current directory.

```nu
ls **/*.md
# => ╭───┬───────────────────────────────┬──────┬──────────┬──────────────╮
# => │ # │             name              │ type │   size   │   modified   │
# => ├───┼───────────────────────────────┼──────┼──────────┼──────────────┤
# => │ 0 │ CODE_OF_CONDUCT.md            │ file │  3.4 KiB │ 5 months ago │
# => │ 1 │ CONTRIBUTING.md               │ file │ 11.0 KiB │ a month ago  │
# => │ 2 │ README.md                     │ file │ 12.0 KiB │ a month ago  │
# => │ 3 │ SECURITY.md                   │ file │  2.6 KiB │ 5 hours ago  │
# => │ 4 │ benches/README.md             │ file │    249 B │ 2 months ago │
# => │ 5 │ crates/README.md              │ file │    795 B │ 5 months ago │
# => │ 6 │ crates/nu-cli/README.md       │ file │    388 B │ 5 hours ago  │
# => │ 7 │ crates/nu-cmd-base/README.md  │ file │    262 B │ 5 hours ago  │
# => │ 8 │ crates/nu-cmd-extra/README.md │ file │    669 B │ 2 months ago │
# => │ 9 │ crates/nu-cmd-lang/README.md  │ file │  1.5 KiB │ a month ago  │
# => ╰───┴───────────────────────────────┴──────┴──────────┴──────────────╯
```

Here, we're looking for any file that ends with ".md". The double-asterisks further specify _"in any directory starting from here."_

Nushell's globbing syntax not only supports `*`, but also matching [single characters with `?` and character groups with `[...]`](https://docs.rs/nu-glob/latest/nu_glob/struct.Pattern.html).

Escaping the `*`, `?`, and `[]` patterns works by enclosing them in a single-quoted, double-quoted, or
[raw string](working_with_strings.md#raw-strings). For example, to show the contents of a directory named
`[slug]`, use `ls "[slug]"` or `ls '[slug]'`.

However, _backtick_ quoted strings do not escape globs. For example, compare the following scenarios:

1. Unquoted: Glob pattern

   An unquoted [bare word string](working_with_strings.html#bare-word-strings) with glob characters is interpreted as a glob pattern, so the following will remove all files in the current directory that contain
   `myfile` as any part of the filename:

   ```nu
   rm *myfile*
   ```

2. Quoted: String literal with asterisks

   When quoting with single or double quotes, or using a [raw string](working_with_strings.html#raw-strings), a _string_ with the literal, escaped asterisks (or other glob characters) is passed to the command. The result is not a glob. The following command will only remove a file literally named `*myfile*` (including the asterisks). Other files with `myfile` in the name are not affected:

   ```nu
   rm "*myfile*"
   ```

3. Backtick-quoted: Glob pattern

   Asterisks (and other glob patterns) within a [backtick-quoted string](working_with_strings.html#backtick-quoted-strings) are interpreted as a glob pattern. Notice that this is the same behavior as that of the bare-word string example in #1 above.

   The following, as with that first example, removes all files in the current directory that contain `myfile` as part of the filename

   ```nu
   rm `*myfile*`
   ```

::: tip
Nushell also includes a dedicated [`glob` command](https://www.nushell.sh/commands/docs/glob.html) with support for more complex globbing scenarios.
:::

### Converting Strings to Globs

The quoting techniques above are useful when constructing glob-literals, but you may need to construct globs programmatically. There are several techniques available for this purpose:

1. `into glob`

   The [`into glob` command](/commands/docs/into_glob.html) can be used to convert a string (and other types) into a glob. For instance:

   ```nu
   # Find files whose name includes the current month in the form YYYY-mm
   let current_month = (date now | format date '%Y-%m')
   let glob_pattern = ($"*($current_month)*" | into glob)
   ls $glob_pattern
   ```

2. The spread operator combined with the [`glob` command](/commands/docs/glob.html):

   The [`glob` command](/commands/docs/glob.html) (note: not the same as `into glob`) produces a [`list`](types_of_data.html#lists) of filenames that match the glob pattern. This list can be expanded and passed to filesystem commands using the [spread operator](operators.html#spread-operator):

   ```nu
   # Find files whose name includes the current month in the form YYYY-mm
   let current_month = (date now | format date '%Y-%m')
   ls ...(glob $"*($current_month)*")
   ```

3. Force `glob` type via annotation:

   ```nu
   # Find files whose name includes the current month in the form YYYY-mm
   let current_month = (date now | format date '%Y-%m')
   let glob_pattern: glob = ($"*($current_month)*")
   ls $glob_pattern
   ```

## Creating a Directory

As with most other shells, the [`mkdir` command](/commands/docs/mkdir.md) is used to create new directories. One subtle difference is that Nushell's internal `mkdir` command operates like the Unix/Linux `mkdir -p` by default, in that it:

- Will create multiple directory levels automatically. For example:

  ```nu
  mkdir modules/my/new_module
  ```

  This will create all three directories even if none of them currently exists. On Linux/Unix, this requires `mkdir -p`.

- Will not error if the directory already exists. For example:

  ```nu
  mkdir modules/my/new_module
  mkdir modules/my/new_module
  # => No error
  ```

  ::: tip
  A common mistake when coming to Nushell is to attempt to use `mkdir -p <directory>` as in the native Linux/Unix version. However, this will generate an `Unknown Flag` error on Nushell.

  Just repeat the command without the `-p` to achieve the same effect.
  :::

## Changing the Current Directory

```nu
cd cookbook
```

To change from the current directory to a new one, use the [`cd`](/commands/docs/cd.md) command.

Changing the current working directory can also be done if [`cd`](/commands/docs/cd.md) is omitted and a path by itself is given:

```nu
cookbook/
```

Just as in other shells, you can use either the name of the directory, or if you want to go up a directory you can use the `..` shortcut.

You can also add additional dots to go up additional directory levels:

```nu
# Change to the parent directory
cd ..
# or
..
# Go up two levels (parent's parent)
cd ...
# or
...
# Go up three levels (parent of parent's parent)
cd ....
# Etc.
```

::: tip
Multi-dot shortcuts are available to both internal Nushell [filesystem commands](/commands/categories/filesystem.html) as well as to external commands. For example, running `^stat ....` on a Linux/Unix system will show that the path is expanded to `../../../..`
:::

You can combine relative directory levels with directory names as well:

```nu
cd ../sibling
```

::: tip IMPORTANT TIP
Changing the directory with [`cd`](/commands/docs/cd.md) changes the `PWD` environment variable. This means that a change of a directory is kept to the current scope (e.g. block or closure). Once you exit the block, you'll return to the previous directory. You can learn more about this in the [Environment](./environment.md) chapter.
:::

## Filesystem Commands

Nu also provides some basic [filesystem commands](/commands/categories/filesystem.html) that work cross-platform such as:

- [`mv`](/commands/docs/mv.md) to rename or move a file or directory to a new location
- [`cp`](/commands/docs/cp.md) to copy an item to a new location
- [`rm`](/commands/docs/rm.md) to remove items from the filesystem

::: tip NOTE
Under Bash and many other shells, most filesystem commands (other than `cd`) are actually separate binaries in the system. For instance, on a Linux system, `cp` is the `/usr/bin/cp` binary. In Nushell, these commands are built-in. This has several advantages:

- They work consistently on platforms where a binary version may not be available (e.g. Windows). This allows the creation of cross-platform scripts, modules, and custom commands.
- They are more tightly integrated with Nushell, allowing them to understand Nushell types and other constructs
- As mentioned in the [Quick Tour](quick_tour.html), they are documented in the Nushell help system. Running `help <command>` or `<command> --help` will display the Nushell documentation for the command.

While the use of the Nushell built-in versions is typically recommended, it is possible to access the Linux binaries. See [Running System Commands](./running_externals.md) for details.
# Navigating and Accessing Structured Data

Given Nushell's strong support for structured data, some of the more common tasks involve navigating and accessing that data.

## Index to this Section

- [Background and Definitions](#background)
- [Cell-paths](#cell-paths)
  - [With Records](#records)
  - [With Lists](#lists)
  - [With Tables](#tables)
    - Sample Data
    - Example - Access a Table Row
    - Example - Access a Table Column
  - [With Nested Data](#nested-data)
- [Using `get` and `select`](#using-get-and-select)
  - Example - `get` vs. `select` with a Table Row
  - Example - `select` with multiple rows and columns
- [Handling missing data using the optional operator](#the-optional-operator)
- [Key/Column names with spaces](#keycolumn-names-with-spaces)
- [Other commands for navigating structured data](#other-commands-for-accessing-structured-data)

## Background

For the examples and descriptions below, keep in mind several definitions regarding structured data:

- **_List:_** Lists contain a series of zero or more values of any type. A list with zero values is known as an "empty list."
- **_Record:_** Records contain zero or more pairs of named keys and their corresponding value. The data in a record's value can also be of any type. A record with zero key-value pairs is known as an "empty record."
- **_Nested Data:_** The values contained in a list, record, or table can be either of a basic type or structured data themselves. This means that data can be nested multiple levels and in multiple forms:
  - List values can contain tables, records, and even other lists
    - **_Table:_** Tables are a list of records
  - Record values can contain tables, lists, and other records
    - This means that the records of a table can also contain nested tables, lists, and other records

::: tip
Because a table is a list of records, any command or syntax that works on a list will also work on a table. The converse is not necessarily the case; there are some commands and syntax that work on tables but not lists.
:::

## Cell-paths

A cell-path is the primary way to access values inside structured data. This path is based on a concept similar to that of a spreadsheet, where columns have names and rows have numbers. Cell-path names and indices are separated by dots.

### Records

For a record, the cell-path specifies the name of a key, which is a `string`.

#### Example - Access a Record Value:

```nu
let my_record = {
    a: 5
    b: 42
  }
$my_record.b + 5
# => 47
```

### Lists

For a list, the cell-path specifies the position (index) of the value in the list. This is an `int`:

#### Example - Access a List Value:

Remember, list indices are 0-based.

```nu
let scoobies_list = [ Velma Fred Daphne Shaggy Scooby ]
$scoobies_list.2
# => Daphne
```

### Tables

- To access a column, a cell-path uses the name of the column, which is a `string`
- To access a row, it uses the index number of the row, which is an `int`
- To access a single cell, it uses a combination of the column name with the row index.

The next few examples will use the following table:

```nu
let data = [
    [date                        temps                                   condition      ];
    [2022-02-01T14:30:00+05:00,  [38.24, 38.50, 37.99, 37.98, 39.10],   'sunny'       ],
    [2022-02-02T14:30:00+05:00,  [35.24, 35.94, 34.91, 35.24, 36.65],   'sunny'       ],
    [2022-02-03T14:30:00+05:00,  [35.17, 36.67, 34.42, 35.76, 36.52],   'cloudy'      ],
    [2022-02-04T14:30:00+05:00,  [39.24, 40.94, 39.21, 38.99, 38.80],   'rain'        ]
]
```

::: details Expand for a visual representation of this data

```nu
╭───┬─────────────┬───────────────┬───────────╮
│ # │    date     │     temps     │ condition │
├───┼─────────────┼───────────────┼───────────┤
│ 0 │ 2 years ago │ ╭───┬───────╮ │ sunny     │
│   │             │ │ 0 │ 38.24 │ │           │
│   │             │ │ 1 │ 38.50 │ │           │
│   │             │ │ 2 │ 37.99 │ │           │
│   │             │ │ 3 │ 37.98 │ │           │
│   │             │ │ 4 │ 39.10 │ │           │
│   │             │ ╰───┴───────╯ │           │
│ 1 │ 2 years ago │ ╭───┬───────╮ │ sunny     │
│   │             │ │ 0 │ 35.24 │ │           │
│   │             │ │ 1 │ 35.94 │ │           │
│   │             │ │ 2 │ 34.91 │ │           │
│   │             │ │ 3 │ 35.24 │ │           │
│   │             │ │ 4 │ 36.65 │ │           │
│   │             │ ╰───┴───────╯ │           │
│ 2 │ 2 years ago │ ╭───┬───────╮ │ cloudy    │
│   │             │ │ 0 │ 35.17 │ │           │
│   │             │ │ 1 │ 36.67 │ │           │
│   │             │ │ 2 │ 34.42 │ │           │
│   │             │ │ 3 │ 35.76 │ │           │
│   │             │ │ 4 │ 36.52 │ │           │
│   │             │ ╰───┴───────╯ │           │
│ 3 │ 2 years ago │ ╭───┬───────╮ │ rain      │
│   │             │ │ 0 │ 39.24 │ │           │
│   │             │ │ 1 │ 40.94 │ │           │
│   │             │ │ 2 │ 39.21 │ │           │
│   │             │ │ 3 │ 38.99 │ │           │
│   │             │ │ 4 │ 38.80 │ │           │
│   │             │ ╰───┴───────╯ │           │
╰───┴─────────────┴───────────────┴───────────╯
```

:::

This represents weather data in the form of a table with three columns:

1. **_date_**: A Nushell `date` for each day
2. **_temps_**: A Nushell `list` of 5 `float` values representing temperature readings at different weather stations in the area
3. **_conditions_**: A Nushell `string` for each day's weather condition for the area

#### Example - Access a Table Row (Record)

Access the second day's data as a record:

```nu
$data.1
# => ╭───────────┬───────────────╮
# => │ date      │ 2 years ago   │
# => │           │ ╭───┬───────╮ │
# => │ temps     │ │ 0 │ 35.24 │ │
# => │           │ │ 1 │ 35.94 │ │
# => │           │ │ 2 │ 34.91 │ │
# => │           │ │ 3 │ 35.24 │ │
# => │           │ │ 4 │ 36.65 │ │
# => │           │ ╰───┴───────╯ │
# => │ condition │ sunny         │
# => ╰───────────┴───────────────╯
```

#### Example - Access a Table Column (List)

```nu
$data.condition
# => ╭───┬────────╮
# => │ 0 │ sunny  │
# => │ 1 │ sunny  │
# => │ 2 │ cloudy │
# => │ 3 │ rain   │
# => ╰───┴────────╯
```

#### Example - Access a Table Cell (Value)

The condition for the fourth day:

```nu
$data.condition.3
# => rain
```

### Nested Data

Since data can be nested, a cell-path can contain references to multiple names or indices.

#### Example - Accessing Nested Table Data

To obtain the temperature at the second weather station on the third day:

```nu
$data.temps.2.1
# => 36.67
```

The first index `2` accesses the third day, then the next index `1` accesses the second weather station's temperature reading.

## Using `get` and `select`

In addition to the cell-path literal syntax used above, Nushell also provides several commands that utilize cell-paths. The most important of these are:

- `get` is equivalent to using a cell-path literal but with support for variable names and expressions. `get`, like the cell-path examples above, returns the **value** indicated by the cell-path.
- `select` is subtly, but critically, different. It returns the specified **data structure** itself, rather than just its value.
  - Using `select` on a table will return a table of equal or lesser size
  - Using `select` on a list will return a list of equal or lesser size
  - using `select` on a record will return a record of equal or lesser size

Continuing with the sample table above:

### Example - `get` vs. `select` a table row

```nu
$data | get 1
# => ╭───────────┬───────────────╮
# => │ date      │ 2 years ago   │
# => │           │ ╭───┬───────╮ │
# => │ temps     │ │ 0 │ 35.24 │ │
# => │           │ │ 1 │ 35.94 │ │
# => │           │ │ 2 │ 34.91 │ │
# => │           │ │ 3 │ 35.24 │ │
# => │           │ │ 4 │ 36.65 │ │
# => │           │ ╰───┴───────╯ │
# => │ condition │ sunny         │
# => ╰───────────┴───────────────╯

$data | select 1
# => ╭───┬─────────────┬───────────────┬───────────╮
# => │ # │    date     │     temps     │ condition │
# => ├───┼─────────────┼───────────────┼───────────┤
# => │ 0 │ 2 years ago │ ╭───┬───────╮ │ sunny     │
# => │   │             │ │ 0 │ 35.24 │ │           │
# => │   │             │ │ 1 │ 35.94 │ │           │
# => │   │             │ │ 2 │ 34.91 │ │           │
# => │   │             │ │ 3 │ 35.24 │ │           │
# => │   │             │ │ 4 │ 36.65 │ │           │
# => │   │             │ ╰───┴───────╯ │           │
# => ╰───┴─────────────┴───────────────┴───────────╯
```

Notice that:

- [`get`](/commands/docs/get.md) returns the same record as the `$data.1` example above
- [`select`](/commands/docs/select.md) returns a new, single-row table, including column names and row indices

::: tip
The row indices of the table resulting from `select` are not the same as that of the original. The new table has its own, 0-based index.

To obtain the original index, you can using the [`enumerate`](/commands/docs/enumerate.md) command. For example:

```nu
$data | enumerate | select 1
```

:::

### Example - `select` with multiple rows and columns

Because `select` results in a new table, it's possible to specify multiple column names, row indices, or even both. This example creates a new table containing the date and condition columns of the first and second rows:

```nu
$data | select date condition 0 1
# => ╭───┬─────────────┬───────────╮
# => │ # │    date     │ condition │
# => ├───┼─────────────┼───────────┤
# => │ 0 │ 2 years ago │ sunny     │
# => │ 1 │ 2 years ago │ sunny     │
# => ╰───┴─────────────┴───────────╯
```

## Key/Column names with spaces

If a key name or column name contains spaces or other characters that prevent it from being accessible as a bare-word string, then the key name may be quoted.

Example:

```nu
let record_example = {
    "key x":12
    "key y":4
  }
$record_example."key x"
# => 12

# or
$record_example | get "key x"
# => 12
```

Quotes are also required when a key name may be confused for a numeric value.

Example:

```nu
let record_example = {
  "1": foo
  "2": baz
  "3": far
}

$record_example."1"
# =>   foo
```

Do not confuse the key name with a row index in this case. Here, the first item is _assigned_ the key name `1` (a string). If converted to a table using the `transpose` command, key `1` (`string`) would be at row-index `0` (an integer).

## Handling Missing Data

### The Optional Operator

By default, cell path access will fail if it can't access the requested row or column. To suppress these errors, you can add a `?` to a cell path member to mark it as optional:

#### Example - The Optional Operator

Using the temp data from above:

```nu
let cp: cell-path = $.temps?.1 # only get the 2nd location from the temps column

# Ooops, we've removed the temps column
$data | reject temps | get $cp
```

By default missing cells will be replaced by `null` when accessed via the optional operator.

### Assigning a `default` for missing or `null` data

The [`default` command](/commands/docs/default.html) can be used to apply a default value to missing or null column result.

```nu
let missing_value = [{a:1 b:2} {b:1}]
$missing_value
# => ╭───┬────┬───╮
# => │ # │ a  │ b │
# => ├───┼────┼───┤
# => │ 0 │  1 │ 2 │
# => │ 1 │ ❎ │ 1 │
# => ╰───┴────┴───╯

let with_default_value = ($missing_value | default 'n/a' a)
$with_default_value
# => ╭───┬─────┬───╮
# => │ # │  a  │ b │
# => ├───┼─────┼───┤
# => │ 0 │   1 │ 2 │
# => │ 1 │ n/a │ 1 │
# => ╰───┴─────┴───╯

$with_default_value.1.a
# => n/a
```

## Other commands for accessing structured data

- [`reject`](/commands/docs/reject.md) is the opposite of `select`, removing the specified rows and columns
- [`range`](/commands/docs/range.md) specifies the rows of a list or table to select using a [`range`](./types_of_data.md#ranges) type
# Nu as a Shell

The [Nu Fundamentals](nu_fundamentals.md) and [Programming in Nu](programming_in_nu.md) chapter focused mostly on the language aspects of Nushell.
This chapter sheds the light on the parts of Nushell that are related to the Nushell interpreter (the Nushell [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)).
Some of the concepts are directly a part of the Nushell programming language (such as environment variables) while others are implemented purely to enhance the interactive experience (such as hooks) and thus are not present when, for example, running a script.

Many parameters of Nushell can be [configured](configuration.md).
The config itself is stored as an environment variable.
Furthermore, Nushell has several different configuration files that are run on startup where you can put custom commands, aliases, etc.

A big feature of any shell are [environment variables](environment.md).
In Nushell, environment variables are scoped and can have any type supported by Nushell.
This brings in some additional design considerations so please refer to the linked section for more details.

The other sections explain how to work with [stdout, stderr and exit codes](stdout_stderr_exit_codes.md), how to [run an external command when there is a built-in with the same name](./running_externals.md), and how to [configure 3rd party prompts](3rdpartyprompts.md) to work with Nushell.

An interesting feature of Nushell is the [Directory Stack](directory_stack.md) which let you work in multiple directories simultaneously.

Nushell also has its own line editor [Reedline](line_editor.md).
With Nushell's config, it is possible to configure some of the Reedline's features, such as the prompt, keybindings, history, or menus.

It is also possible to define [custom signatures for external commands](externs.md) which lets you define [custom completions](custom_completions.md) for them (the custom completions also work for Nushell custom commands).

[Coloring and Theming in Nu](coloring_and_theming.md) goes into more detail about how to configure Nushell's appearance.

If you want to schedule some commands to run in the background, [Background task in Nu](background_task.md) provides simple guidelines for you to follow.

Finally, [hooks](hooks.md) allow you to insert fragments of Nushell code to run at certain events.
# Nu Fundamentals

This chapter explains some of the fundamentals of the Nushell programming language.
After going through it, you should have an idea how to write simple Nushell programs.

Nushell has a rich type system.
You will find typical data types such as strings or integers and less typical data types, such as cell paths.
Furthermore, one of the defining features of Nushell is the notion of _structured data_ which means that you can organize types into collections: lists, records, or tables.
Contrary to the traditional Unix approach where commands communicate via plain text, Nushell commands communicate via these data types.
All of the above is explained in [Types of Data](types_of_data.md).

[Loading Data](loading_data.md) explains how to read common data formats, such as JSON, into _structured data_. This includes our own "NUON" data format.

Just like Unix shells, Nushell commands can be composed into [pipelines](pipelines.md) to pass and modify a stream of data.

Some data types have interesting features that deserve their own sections: [strings](working_with_strings.md), [lists](working_with_lists.md), and [tables](working_with_tables.md).
Apart from explaining the features, these sections also show how to do some common operations, such as composing strings or updating values in a list.

Finally, [Command Reference](/commands/) lists all the built-in commands with brief descriptions.
Note that you can also access this info from within Nushell using the [`help`](/commands/docs/help.md) command.
# Nu map from other shells and domain specific languages

The idea behind this table is to help you understand how Nu builtins and plugins relate to other known shells and domain specific languages. We've tried to produce a map of relevant Nu commands and what their equivalents are in other languages. Contributions are welcome.

| Nushell                                                    | SQL                           | .Net LINQ (C#)                                       | PowerShell (without external modules)      | Bash                                            |
| ---------------------------------------------------------- | ----------------------------- | ---------------------------------------------------- | ------------------------------------------ | ----------------------------------------------- |
| [`alias`](/commands/docs/alias.md)                         |                               |                                                      | `alias`                                    | `alias`                                         |
| [`append`](/commands/docs/append.md)                       |                               | `Append`                                             | `-Append`                                  |                                                 |
| [`math avg`](/commands/docs/math_avg.md)                   | `avg`                         | `Average`                                            | `Measure-Object`, `measure`                |                                                 |
| [Operators](operators.md) and [`math`](/commands/docs/math.md) | Math operators            | `Aggregate`, `Average`, `Count`, `Max`, `Min`, `Sum` |                                            | `bc`                                            |
| [`cd`](/commands/docs/cd.md)                               |                               |                                                      | `Set-Location`, `cd`                       | `cd`                                            |
| [`clear`](/commands/docs/clear.md)<br /><kbd>Ctrl/⌘</kbd>+<kbd>L</kbd> |                  |                                                      | `Clear-Host`<br /><kbd>Ctrl/⌘</kbd>+<kbd>L</kbd>  | `clear`<br /><kbd>Ctrl/⌘</kbd>+<kbd>L</kbd> |
| [`config`](/commands/docs/config.md)<br />`$nu.default-config-dir`  |                      |                                                      | `$Profile`                                 | `~/.bashrc`, `~/.profile`                       |
| [`cp`](/commands/docs/cp.md)                               |                               |                                                      | `Copy-Item`, `cp`, `copy`                  | `cp`                                            |
| [`date`](/commands/docs/date.md)                           | `NOW()`, `getdate()`          | `DateTime` class                                     | `Get-Date`                                 | `date`                                          |
| [`du`](/commands/docs/du.md)<br />[`ls --du`](/commands/docs/ls.md) |                      |                                                      |                                            | `du`                                            |
| [`each`](/commands/docs/each.md)                           | Cursors                       |                                                      | `ForEach-Object`, `foreach`, `for`         | `for`                                           |
| [`exit`](/commands/docs/exit.md)<br /><kbd>Ctrl/⌘</kbd>+<kbd>D</kbd> |                    |                                                      | `exit`<br /><kbd>Ctrl/⌘</kbd>+<kbd>D</kbd> | `exit`<br /><kbd>Ctrl/⌘</kbd>+<kbd>D</kbd>    | 
| [`http`](/commands/docs/http.md)                           |                               | `HttpClient`, `WebClient`, `HttpWebRequest/Response` | `Invoke-WebRequest`                        | `wget`, `curl`                                  |
| [`first`](/commands/docs/first.md)                         | `top`, `limit`                | `First`, `FirstOrDefault`                            | `Select-Object -First`                     | `head`                                          |
| [`format`](/commands/docs/format.md), [`str`](/commands/docs/str.md) |                     | `String.Format`                                      | `String.Format`                            | `printf`                                        |
| [`from`](/commands/docs/from.md)                           | `import flatfile,` `openjson`, `cast(variable as xml) `|                             | `Import/ConvertFrom-{Csv,Xml,Html,Json}`   |                                                 |
| [`get`](/commands/docs/get.md)                             |                               | `Select`                                             | `(cmd).column`                             |                                                 |
| [`group-by`](/commands/docs/group-by.md)                   | `group by`                    | `GroupBy`, `group`                                   | `Group-Object`, `group`                    |                                                 |
| [`help`](/commands/docs/help.md)                           | `sp_help`                     |                                                      | `Get-Help`, `help`, `man`                  | `man`                                           |
| [`history`](/commands/docs/history.md)                     |                               |                                                      | `Get-History`, `history`                   | `history`                                       |
| [`is-empty`](/commands/docs/is-empty.md)                   | `is null`                     | `String.IsNullOrEmpty`                               | `String.IsNullOrEmpty`                     |                                                 |
| [`kill`](/commands/docs/kill.md)                           |                               |                                                      | `Stop-Process`, `kill`                     | `kill`                                          |
| [`last`](/commands/docs/last.md)                           |                               | `Last`, `LastOrDefault`                              | `Select-Object -Last`                      | `tail`                                          |
| [`str stats`](/commands/docs/str_stats.md)<br />[`length`](/commands/docs/length.md)<br />[`str length`](/commands/docs/str_length.md) | `count`  | `Count`                                        | `Measure-Object`, `measure`                | `wc`                                            |
| [`lines`](/commands/docs/lines.md)                         |                               |                                                      | `File.ReadAllLines`                        |                                                 |
| [`ls`](/commands/docs/ls.md)                               |                               |                                                      | `Get-ChildItem`, `dir`, `ls`               | `ls`                                            |
| [`mkdir`](/commands/docs/mkdir.md)                         |                               |                                                      | `mkdir`, `md`, `New-Item -ItemType Directory` | `mkdir`                                      |
| [`mv`](/commands/docs/mv.md)                               |                               |                                                      | `Move-Item`, `mv`, `move`, `mi`            | `mv`                                            |
| [`open`](/commands/docs/open.md)                           |                               |                                                      | `Get-Content`, `gc`, `cat`, `type`         | `cat`                                           |
| [`print`](/commands/docs/print.md)                         | `print`, `union all`          |                                                      | `Write-Output`, `write`                    | `echo`, `print`                                 |
| [`transpose`](/commands/docs/transpose.md)                 | `pivot`                       |                                                      |                                            |                                                 |
| [`ps`](/commands/docs/ps.md)                               |                               |                                                      | `Get-Process`, `ps`, `gps`                 | `ps`                                            |
| [`pwd`](/commands/docs/pwd.md)                             |                               |                                                      | `Get-Location`, `pwd`                      | `pwd`                                           |
| [`range` (command)](/commands/docs/range.md)               | `limit x offset y`, `rownumber` | `ElementAt`                                        | `[x]`, indexing operator, `ElementAt`      |                                                 |
| [`range` (type)](types_of_data.html#ranges)                |                               | `Range`                                              | `1..10`, `'a'..'f'`                        |                                                 |
| [`reduce`](/commands/docs/reduce.md)                       |                               | `Aggregate`                                          |                                            |                                                 |
| [`rename`](/commands/docs/rename.md)                       |                               |                                                      | `Rename-Item`, `ren`, `rni`                | `mv`                                            |
| [`reverse`](/commands/docs/reverse.md)                     |                               | `Reverse`                                            | `[Array]::Reverse($var)`                   |                                                 |
| [`rm`](/commands/docs/rm.md)                               |                               |                                                      | `Remove-Item`, `del`, `erase`, `rd`, `ri`, `rm`, `rmdir` | `rm`                              |
| [`save`](/commands/docs/save.md)                           |                               |                                                      | `Write-Output`, `Out-File`                 | `> foo.txt` redirection                         |
| [`select`](/commands/docs/select.md)                       | `select`                      | `Select`                                             | `Select-Object`, `select`                  |                                                 |
| [`shuffle`](/commands/docs/shuffle.md)                     |                               | `Random`                                             | `Sort-Object {Get-Random}`                 |                                                 |
| [`skip`](/commands/docs/skip.md)                           | `where row_number()`          | `Skip`                                               | `Select-Object -Skip`                      |                                                 |
| [`skip until`](/commands/docs/skip_until.md)               |                               | `SkipWhile`                                          |                                            |                                                 |
| [`skip while`](/commands/docs/skip_while.md)               |                               | `SkipWhile`                                          |                                            |                                                 |
| [`sort-by`](/commands/docs/sort-by.md)                     | `order by`                    | `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending` | `Sort-Object`, `sort`              | `sort`                                          |
| [`str`](/commands/docs/str.md)                             | String functions              | `String` class                                       | `String` class                             |                                                 |
| [`str join`](/commands/docs/str_join.md)                   | `concat_ws`                   | `Join`                                               | `Join-String`                              |                                                 |
| [`str trim`](/commands/docs/str_trim.md)                   | `rtrim`, `ltrim`              | `Trim`, `TrimStart`, `TrimEnd`                       | `Trim`                                     |                                                 |
| [`math sum`](/commands/docs/math_sum.md)                   | ``sum`                        | `Sum`                                                | `Measure-Object`, `measure`                |                                                 |
| [`uname`](/commands/docs/uname.md)<br />[`sys host`](/commands/docs/sys_host.md)                   |                               |                                                      | `Get-ComputerInfo`                         | `uname`                                         |
| [`sys disks`](/commands/docs/sys_disks.md)                 |                               |                                                      | `Get-ComputerInfo`                         | `lsblk`                                         |
| [`sys mem`](/commands/docs/sys_mem.md)                     |                               |                                                      | `Get-ComputerInfo`                         | `free`                                          |
| [`table`](/commands/docs/table.md)                         |                               |                                                      | `Format-Table`, `ft`, `Format-List`, `fl`  |                                                 |
| [`take`](/commands/docs/take.md)                           | `top`, `limit`                | `Take`                                               | `Select-Object -First`                     | `head`                                          |
| [`take until`](/commands/docs/take_until.md)               |                               | `TakeWhile`                                          |                                            |                                                 |
| [`take while`](/commands/docs/take_while.md)               |                               | `TakeWhile`                                          |                                            |                                                 |
| [`timeit`](/commands/docs/timeit.md)                       |                               |                                                      | `Measure-Command`                          | `time`                                          |
| [`to`](/commands/docs/to.md)                               |                               |                                                      | `Export`/`ConvertTo-{Csv,Xml,Html,Json}`   |                                                 |
| [`touch`](/commands/docs/touch.md)                         |                               |                                                      | `Set-Content`                              | `touch`                                         |
| [`uniq`](/commands/docs/uniq.md)                           | `distinct`                    | `Distinct`                                           | `Get-Unique`, `gu`                         | `uniq`                                          |
| [`update`](/commands/docs/update.md)                       |                               |                                                      | `ForEach-Object`                           |                                                 |
| [`upsert`](/commands/docs/upsert.md)                       | `As`                          |                                                      | `ForEach-Object`                           |                                                 |
| [`version`](/commands/docs/version.md)                     | `select @@version`            |                                                      | `$PSVersionTable`                          |                                                 |
| `$env.FOO = "bar"`<br />[`with-env`](/commands/docs/with-env.md) |                         |                                                      | `$env:FOO = 'bar'`                         | `export FOO "bar"`                              |
| [`where`](/commands/docs/where.md)                         | `where`                       | `Where`                                              | `Where-Object`, `where`, `?` operator      |                                                 |
| [`which`](/commands/docs/which.md)                         |                               |                                                      | `Get-Command`                              | `which`                                         |
# Nu Map from Functional Languages

The idea behind this table is to help you understand how Nu builtins and plugins relate to functional languages. We've tried to produce a map of relevant Nu commands and what their equivalents are in other languages. Contributions are welcome.

Note: this table assumes Nu 0.43 or later.

| Nushell      | Clojure                      | Tablecloth (Ocaml / Elm)        | Haskell                  |
| ------------ | ---------------------------- | ------------------------------- | ------------------------ |
| append       | conj, into, concat           | append, (++), concat, concatMap | (++)                     |
| into binary  | Integer/toHexString          |                                 | showHex                  |
| count        | count                        | length, size                    | length, size             |
| date         | java.time.LocalDate/now      |                                 |                          |
| each         | map, mapv, iterate           | map, forEach                    | map, mapM                |
| exit         | System/exit                  |                                 |                          |
| first        | first                        | head                            | head                     |
| format       | format                       |                                 | Text.Printf.printf       |
| group-by     | group-by                     |                                 | group, groupBy           |
| help         | doc                          |                                 |                          |
| is-empty     | empty?                       | isEmpty                         |                          |
| last         | last, peek, take-last        | last                            | last                     |
| lines        |                              |                                 | lines, words, split-with |
| match        |                              | match (Ocaml), case (Elm)       | case                     |
| nth          | nth                          | Array.get                       | lookup                   |
| open         | with-open                    |                                 |                          |
| transpose    | (apply mapv vector matrix)   |                                 | transpose                |
| prepend      | cons                         | cons, ::                        | ::                       |
| print        | println                      |                                 | putStrLn, print          |
| range, 1..10 | range                        | range                           | 1..10, 'a'..'f'          |
| reduce       | reduce, reduce-kv            | foldr                           | foldr                    |
| reverse      | reverse, rseq                | reverse, reverseInPlace         | reverse                  |
| select       | select-keys                  |                                 |                          |
| shuffle      | shuffle                      |                                 |                          |
| size         | count                        |                                 | size, length             |
| skip         | rest                         | tail                            | tail                     |
| skip until   | drop-while                   |                                 |                          |
| skip while   | drop-while                   | dropWhile                       | dropWhile, dropWhileEnd  |
| sort-by      | sort, sort-by, sorted-set-by | sort, sortBy, sortWith          | sort, sortBy             |
| split row    | split, split-{at,with,lines} | split, words, lines             | split, words, lines      |
| str          | clojure.string functions     | String functions                |                          |
| str join     | join                         | concat                          | intercalate              |
| str trim     | trim, triml, trimr           | trim, trimLeft, trimRight       | strip                    |
| sum          | apply +                      | sum                             | sum                      |
| take         | take, drop-last, pop         | take, init                      | take, init               |
| take until   | take-while                   | takeWhile                       | takeWhile                |
| take while   | take-while                   | takeWhile                       | takeWhile                |
| uniq         | set                          | Set.empty                       | Data.Set                 |
| where        | filter, filterv, select      | filter, filterMap               | filter                   |
# Nu Map from Imperative Languages

The idea behind this table is to help you understand how Nu built-ins and plugins relate to imperative languages. We've tried to produce a map of programming-relevant Nu commands and what their equivalents are in other languages. Contributions are welcome.

Note: this table assumes Nu 0.94 or later.

| Nushell                                                                                                                                | Python                               | Kotlin (Java)                                               | C++                         | Rust                                                 |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ----------------------------------------------------------- | --------------------------- | ---------------------------------------------------- |
| [`append`](/commands/docs/append.md)                                                                                                   | `list.append`, `set.add`             | `add`                                                       | `push_back`, `emplace_back` | `push`, `push_back`                                  |
| [`math avg`](/commands/docs/math_avg.md)                                                                                               | `statistics.mean`                    |                                                             |                             |                                                      |
| [`math`](/commands/docs/math.md), [Math Operators](nushell_operator_map.md)                                                            | Math operators                       | Math operators                                              | Math operators              | Math operators                                       |
| [`cp`](/commands/docs/cp.md)                                                                                                           | `shutil.copy`                        |                                                             |                             | `fs::copy`                                           |
| [`date`](/commands/docs/date.md)                                                                                                       | `datetime.date.today`                | `java.time.LocalDate.now`                                   |                             |                                                      |
| [`drop`](/commands/docs/drop.md)                                                                                                       | `list[:-3]`                          |                                                             |                             |                                                      |
| [`du`](/commands/docs/du.md), [`ls --du`](/commands/docs/ls.md)                                                                        | `shutil.disk_usage`                  |                                                             |                             |                                                      |
| [`each`](/commands/docs/each.md)<br />[`for`](/commands/docs/for.md)                                                                   | `for`                                | `for`                                                       | `for`                       | `for`                                                |
| [`exit`](/commands/docs/exit.md)                                                                                                       | `exit()`                             | `System.exit`, `kotlin.system.exitProcess`                  | `exit`                      | `exit`                                               |
| [`http get`](/commands/docs/http_get.md)                                                                                               | `urllib.request.urlopen`             |                                                             |                             |                                                      |
| [`first`](/commands/docs/first.md)                                                                                                     | `list[:x]`                           | `List[0]`, `peek`                                           | `vector[0]`, `top`          | `Vec[0]`                                             |
| [`format`](/commands/docs/format.md)                                                                                                   | `format`                             | `format`                                                    | `format`                    | `format!`                                            |
| [`from`](/commands/docs/from.md)                                                                                                       | `csv`, `json`, `sqlite3`             |                                                             |                             |                                                      |
| [`get`](/commands/docs/get.md)                                                                                                         | `dict[\"key\"]`                      | `Map[\"key\"]`                                              | `map[\"key\"]`              | `HashMap["key"]`, `get`, `entry`                     |
| [`group-by`](/commands/docs/group-by.md)                                                                                               | `itertools.groupby`                  | `groupBy`                                                   |                             | `group_by`                                           |
| [`headers`](/commands/docs/headers.md)                                                                                                 | `keys`                               |                                                             |                             |                                                      |
| [`help`](/commands/docs/help.md)                                                                                                       | `help()`                             |                                                             |                             |                                                      |
| [`insert`](/commands/docs/insert.md)                                                                                                   | `dict[\"key\"] = val`                |                                                             | `map.insert({ 20, 130 })`   | `map.insert(\"key\", val)`                           |
| [`is-empty`](/commands/docs/is-empty.md)                                                                                               | `is None`, `is []`                   | `isEmpty`                                                   | `empty`                     | `is_empty`                                           |
| [`take`](/commands/docs/take.md)                                                                                                       | `list[:x]`                           |                                                             |                             | `&Vec[..x]`                                          |
| [`take until`](/commands/docs/take_until.md)                                                                                           | `itertools.takewhile`                |                                                             |                             |                                                      |
| [`take while`](/commands/docs/take_while.md)                                                                                           | `itertools.takewhile`                |                                                             |                             |                                                      |
| [`kill`](/commands/docs/kill.md)                                                                                                       | `os.kill`                            |                                                             |                             |                                                      |
| [`last`](/commands/docs/last.md)                                                                                                       | `list[-x:]`                          |                                                             |                             | `&Vec[Vec.len()-1]`                                  |
| [`length`](/commands/docs/length.md)                                                                                                   | `len`                                | `size`, `length`                                            | `length`                    | `len`                                                |
| [`lines`](/commands/docs/lines.md)                                                                                                     | `split`, `splitlines`                | `split`                                                     | `views::split`              | `split`, `split_whitespace`, `rsplit`, `lines`       |
| [`ls`](/commands/docs/ls.md)                                                                                                           | `os.listdir`                         |                                                             |                             | `fs::read_dir`                                       |
| [`match`](/commands/docs/match.md)                                                                                                     | `match`                              | `when`                                                      |                             | `match`                                              |
| [`merge`](/commands/docs/merge.md)                                                                                                     | `dict.append`                        |                                                             |                             | `map.extend`                                         |
| [`mkdir`](/commands/docs/mkdir.md)                                                                                                     | `os.mkdir`                           |                                                             |                             | `fs::create_dir`                                     |
| [`mv`](/commands/docs/mv.md)                                                                                                           | `shutil.move`                        |                                                             |                             | `fs::rename`                                         |
| [`get`](/commands/docs/get.md)                                                                                                         | `list[x]`                            | `List[x]`                                                   | `vector[x]`                 | `Vec[x]`                                             |
| [`open`](/commands/docs/open.md)                                                                                                       | `open`                               |                                                             |                             |                                                      |
| [`transpose`](/commands/docs/transpose.md)                                                                                             | `zip(\*matrix)`                      |                                                             |                             |                                                      |
| [`http post`](/commands/docs/http_post.md)                                                                                             | `urllib.request.urlopen`             |                                                             |                             |                                                      |
| [`prepend`](/commands/docs/prepend.md)                                                                                                 | `deque.appendleft`                   |                                                             |                             |                                                      |
| [`print`](/commands/docs/print.md)                                                                                                     | `print`                              | `println`                                                   | `printf`                    | `println!`                                           |
| [`ps`](/commands/docs/ps.md)                                                                                                           | `os.listdir('/proc')`                |                                                             |                             |                                                      |
| [`pwd`](/commands/docs/pwd.md)                                                                                                         | `os.getcwd`                          |                                                             |                             | `env::current_dir`                                   |
| [`range`](types_of_data.html#ranges) type                                                                                              | `range`                              | `..`, `until`, `downTo`, `step`                             | `iota`                      | `..`                                                 |
| [`reduce`](/commands/docs/reduce.md)                                                                                                   | `functools.reduce`                   | `reduce`                                                    | `reduce`                    | `fold`, `rfold`, `scan`                              |
| [`reject`](/commands/docs/reject.md)                                                                                                   | `del`                                |                                                             |                             |                                                      |
| [`rename`](/commands/docs/rename.md)                                                                                                   | `dict[\"key2\"] = dict.pop(\"key\")` |                                                             |                             | `map.insert(\"key2\", map.remove(\"key\").unwrap())` |
| [`reverse`](/commands/docs/reverse.md)                                                                                                 | `reversed`, `list.reverse`           | `reverse`, `reversed`, `asReversed`                         | `reverse`                   | `rev`                                                |
| [`rm`](/commands/docs/rm.md)                                                                                                           | `os.remove`                          |                                                             |                             |                                                      |
| [`save`](/commands/docs/save.md)                                                                                                       | `io.TextIOWrapper.write`             |                                                             |                             |                                                      |
| [`select`](/commands/docs/select.md)                                                                                                   | `{k:dict[k] for k in keys}`          |                                                             |                             |                                                      |
| [`shuffle`](/commands/docs/shuffle.md)                                                                                                 | `random.shuffle`                     |                                                             |                             |                                                      |
| [`str stats`](/commands/docs/str_stats.md)<br />[`str length`](/commands/docs/str_length.md)<br />[`length`](/commands/docs/length.md) | `len`                                |                                                             |                             | `len`                                                |
| [`skip`](/commands/docs/skip.md)                                                                                                       | `list[x:]`                           |                                                             |                             | `&Vec[x..]`, `skip`                                  |
| [`skip until`](/commands/docs/skip_until.md)                                                                                           | `itertools.dropwhile`                |                                                             |                             |                                                      |
| [`skip while`](/commands/docs/skip_while.md)                                                                                           | `itertools.dropwhile`                |                                                             |                             | `skip_while`                                         |
| [`sort-by`](/commands/docs/sort-by.md)                                                                                                 | `sorted`, `list.sort`                | `sortedBy`, `sortedWith`, `Arrays.sort`, `Collections.sort` | `sort`                      | `sort`                                               |
| [`split row`](/commands/docs/split_row.md)                                                                                             | `str.split{,lines}`, `re.split`      | `split`                                                     | `views::split`              | `split`                                              |
| [`str`](/commands/docs/str.md)                                                                                                         | `str` functions                      | String functions                                            | String functions            | `&str`, String functions                             |
| [`str join`](/commands/docs/str_join.md)                                                                                               | `str.join`                           | `joinToString`                                              |                             | `join`                                               |
| [`str trim`](/commands/docs/str_trim.md)                                                                                               | `strip`, `rstrip`, `lstrip`          | `trim`, `trimStart`, `trimEnd`                              | Regex                       | `trim`, `trim*{start,end}`, `strip*{suffix,prefix}`  |
| [`math sum`](/commands/docs/math_sum.md)                                                                                               | `sum`                                | `sum`                                                       | `reduce`                    | `sum`                                                |
| [`to`](/commands/docs/to.md)                                                                                                           | `import csv`, `json`, `sqlite3`      |                                                             |                             |                                                      |
| [`touch`](/commands/docs/touch.md)                                                                                                     | `open(path, 'a').close()`            |                                                             |                             |                                                      |
| [`uniq`](/commands/docs/uniq.md)                                                                                                       | `set`                                | Set                                                         | `set`                       | `HashSet`                                            |
| [`upsert`](/commands/docs/upsert.md)                                                                                                   | `dict[\"key\"] = val`                |                                                             |                             |                                                      |
| [`version`](/commands/docs/version.md)                                                                                                 | `sys.version`, `sys.version_info`    |                                                             |                             |                                                      |
| [`with-env`](/commands/docs/with-env.md)<br />`$env.FOO = "bar"`                                                                       | `os.environ`                         |                                                             |                             |                                                      |
| [`where`](/commands/docs/where.md)                                                                                                     | `filter`                             | `filter`                                                    | `filter`                    | `filter`                                             |
| [`which`](/commands/docs/which.md)                                                                                                     | `shutil.which`                       |                                                             |                             |                                                      |
| [`wrap`](/commands/docs/wrap.md)                                                                                                       | `{ "key" : val }`                    |                                                             |                             |                                                      |
# Nushell operator map

The idea behind this table is to help you understand how Nu operators relate to other language operators. We've tried to produce a map of all the nushell operators and what their equivalents are in other languages. Contributions are welcome.

Note: this table assumes Nu 0.14.1 or later.

| Nushell | SQL      | Python             | .NET LINQ (C#)       | PowerShell             | Bash               |
| ------- | -------- | ------------------ | -------------------- | ---------------------- | ------------------ |
| ==      | =        | ==                 | ==                   | -eq, -is               | -eq                |
| !=      | !=, <>   | !=                 | !=                   | -ne, -isnot            | -ne                |
| <       | <        | <                  | <                    | -lt                    | -lt                |
| <=      | <=       | <=                 | <=                   | -le                    | -le                |
| >       | >        | >                  | >                    | -gt                    | -gt                |
| >=      | >=       | >=                 | >=                   | -ge                    | -ge                |
| =~      | like     | re, in, startswith | Contains, StartsWith | -like, -contains       | =~                 |
| !~      | not like | not in             | Except               | -notlike, -notcontains | ! "str1" =~ "str2" |
| +       | +        | +                  | +                    | +                      | +                  |
| -       | -        | -                  | -                    | -                      | -                  |
| \*      | \*       | \*                 | \*                   | \*                     | \*                 |
| /       | /        | /                  | /                    | /                      | /                  |
| \*\*    | pow      | \*\*               | Power                | Pow                    | \*\*               |
| in      | in       | re, in, startswith | Contains, StartsWith | -In                    | case in            |
| not-in  | not in   | not in             | Except               | -NotIn                 |                    |
| and     | and      | and                | &&                   | -And, &&               | -a, &&             |
| or      | or       | or                 | \|\|                 | -Or, \|\|              | -o, \|\|           |
# Operators

Nushell supports the following operators for common math, logic, and string operations:

| Operator      | Description                                             |
| ------------- | ------------------------------------------------------- |
| `+`           | add                                                     |
| `-`           | subtract                                                |
| `*`           | multiply                                                |
| `/`           | divide                                                  |
| `//`          | floor division                                          |
| `mod`         | modulo                                                  |
| `**`          | exponentiation (power)                                  |
| `==`          | equal                                                   |
| `!=`          | not equal                                               |
| `<`           | less than                                               |
| `<=`          | less than or equal                                      |
| `>`           | greater than                                            |
| `>=`          | greater than or equal                                   |
| `=~`          | regex match / string contains another                   |
| `!~`          | inverse regex match / string does *not* contain another |
| `in`          | value in list                                           |
| `not-in`      | value not in list                                       |
| `not`         | logical not                                             |
| `and`         | and two Boolean expressions (short-circuits)            |
| `or`          | or two Boolean expressions (short-circuits)             |
| `xor`         | exclusive or two boolean expressions                    |
| `bit-or`      | bitwise or                                              |
| `bit-xor`     | bitwise xor                                             |
| `bit-and`     | bitwise and                                             |
| `bit-shl`     | bitwise shift left                                      |
| `bit-shr`     | bitwise shift right                                     |
| `starts-with` | string starts with                                      |
| `ends-with`   | string ends with                                        |
| `++`          | append lists                                            |


Parentheses can be used for grouping to specify evaluation order or for calling commands and using the results in an expression.

## Order of Operations

To understand the precedence of operations, you can run the command: `help operators | sort-by precedence -r`.

Presented in descending order of precedence, the article details the operations as follows:

- Parentheses (`()`)
- Exponentiation/Power (`**`)
- Multiply (`*`), Divide (`/`), Integer/Floor Division (`//`), and Modulo (`mod`)
- Add (`+`) and Subtract (`-`)
- Bit shifting (`bit-shl`, `bit-shr`)
- Comparison operations (`==`, `!=`, `<`, `>`, `<=`, `>=`), membership tests (`in`, `not-in`, `starts-with`, `ends-with`), regex matching (`=~`, `!~`), and list appending (`++`)
- Bitwise and (`bit-and`)
- Bitwise xor (`bit-xor`)
- Bitwise or (`bit-or`)
- Logical and (`and`)
- Logical xor (`xor`)
- Logical or (`or`)
- Assignment operations
- Logical not (`not`)

```
3 * (1 + 2)
# => 9
```

## Types

Not all operations make sense for all data types.
If you attempt to perform an operation on non-compatible data types, you will be met with an error message that should explain what went wrong:
```nu
"spam" - 1
# => Error: nu::parser::unsupported_operation (link)
# => 
# =>   × Types mismatched for operation.
# =>    ╭─[entry #49:1:1]
# =>  1 │ "spam" - 1
# =>    · ───┬── ┬ ┬
# =>    ·    │   │ ╰── int
# =>    ·    │   ╰── doesn't support these values.
# =>    ·    ╰── string
# =>    ╰────
# =>   help: Change string or int to be the right types and try again.
```

The rules might sometimes feel a bit strict, but on the other hand there should be less unexpected side effects.

## Regular Expression / string-contains Operators

The `=~` and `!~` operators provide a convenient way to evaluate [regular expressions](https://cheatography.com/davechild/cheat-sheets/regular-expressions/). You don't need to know regular expressions to use them - they're also an easy way to check whether 1 string contains another.

- `string =~ pattern` returns **true** if `string` contains a match for `pattern`, and **false** otherwise.
- `string !~ pattern` returns **false** if `string` contains a match for `pattern`, and **true** otherwise.

For example:

```nu
foobarbaz =~ bar # returns true
foobarbaz !~ bar # returns false
ls | where name =~ ^nu # returns all files whose names start with "nu"
```

Both operators use [the Rust regex crate's `is_match()` function](https://docs.rs/regex/latest/regex/struct.Regex.html#method.is_match).

## Case Sensitivity

Operators are usually case-sensitive when operating on strings. There are a few ways to do case-insensitive work instead:

1. In the regular expression operators, specify the `(?i)` case-insensitive mode modifier:

```nu
"FOO" =~ "foo" # returns false
"FOO" =~ "(?i)foo" # returns true
```

2. Use the [`str contains`](/commands/docs/str_contains.md) command's `--ignore-case` flag:

```nu
"FOO" | str contains --ignore-case "foo"
```

3. Convert strings to lowercase with [`str downcase`](/commands/docs/str_downcase.md) before comparing:

```nu
("FOO" | str downcase) == ("Foo" | str downcase)
```

## Spread operator

Nushell has a spread operator (`...`) for unpacking lists and records. You may be familiar with it
if you've used JavaScript before. Some languages use `*` for their spread/splat operator. It can
expand lists or records in places where multiple values or key-value pairs are expected.

There are three places you can use the spread operator:

- [In list literals](#in-list-literals)
- [In record literals](#in-record-literals)
- [In command calls](#in-command-calls)

### In List literals

Suppose you have multiple lists you want to concatenate together, but you also want to intersperse
some individual values. This can be done with `append` and `prepend`, but the spread
operator can let you do it more easily.

```nushell
let dogs = [Spot, Teddy, Tommy]
let cats = ["Mr. Humphrey Montgomery", Kitten]
[
  ...$dogs
  Polly
  ...($cats | each { |elt| $"($elt) \(cat\)" })
  ...[Porky Bessie]
  ...Nemo
]
# => ╭───┬───────────────────────────────╮
# => │ 0 │ Spot                          │
# => │ 1 │ Teddy                         │
# => │ 2 │ Tommy                         │
# => │ 3 │ Polly                         │
# => │ 4 │ Mr. Humphrey Montgomery (cat) │
# => │ 5 │ Kitten (cat)                  │
# => │ 6 │ Porky                         │
# => │ 7 │ Bessie                        │
# => │ 8 │ ...Nemo                       │
# => ╰───┴───────────────────────────────╯
```

The below code is an equivalent version using `append`:
```nushell
$dogs |
  append Polly |
  append ($cats | each { |elt| $"($elt) \(cat\)" }) |
  append [Porky Bessie] |
  append ...Nemo
```

Note that each call to `append` results in the creation of a new list, meaning that in this second
example, 3 unnecessary intermediate lists are created. This is not the case with the spread operator,
so there may be (very minor) performance benefits to using `...` if you're joining lots of large
lists together, over and over.

You may have noticed that the last item of the resulting list above is `"...Nemo"`. This is because
inside list literals, it can only be used to spread lists, not strings. As such, inside list literals, it can
only be used before variables (`...$foo`), subexpressions (`...(foo)`), and list literals (`...[foo]`).

The `...` also won't be recognized as the spread operator if there's any whitespace between it and
the next expression:

```nushell
[ ... [] ]
# => ╭───┬────────────────╮
# => │ 0 │ ...            │
# => │ 1 │ [list 0 items] │
# => ╰───┴────────────────╯
```

This is mainly so that `...` won't be confused for the spread operator in commands such as `mv ... $dir`.

### In Record literals

Let's say you have a record with some configuration information and you want to add more fields to
this record:

```nushell
let config = { path: /tmp, limit: 5 }
```

You can make a new record with all the fields of `$config` and some new additions using the spread
operator. You can use the spread multiple records inside a single record literal.

```nushell
{
  ...$config,
  users: [alice bob],
  ...{ url: example.com },
  ...(sys mem)
}
# => ╭────────────┬───────────────╮
# => │ path       │ /tmp          │
# => │ limit      │ 5             │
# => │            │ ╭───┬───────╮ │
# => │ users      │ │ 0 │ alice │ │
# => │            │ │ 1 │ bob   │ │
# => │            │ ╰───┴───────╯ │
# => │ url        │ example.com   │
# => │ total      │ 8.3 GB        │
# => │ free       │ 2.6 GB        │
# => │ used       │ 5.7 GB        │
# => │ available  │ 2.6 GB        │
# => │ swap total │ 2.1 GB        │
# => │ swap free  │ 18.0 MB       │
# => │ swap used  │ 2.1 GB        │
# => ╰────────────┴───────────────╯
```

Similarly to lists, inside record literals, the spread operator can only be used before variables (`...$foo`),
subexpressions (`...(foo)`), and record literals (`...{foo:bar}`). Here too, there needs to be no
whitespace between the `...` and the next expression for it to be recognized as the spread operator.

### In Command calls

You can also spread arguments to a command, provided that it either has a rest parameter or is an
external command.

Here is an example custom command that has a rest parameter:

```nushell
def foo [ --flag req opt? ...args ] { [$flag, $req, $opt, $args] | to nuon }
```

It has one flag (`--flag`), one required positional parameter (`req`), one optional positional parameter
(`opt?`), and rest parameter (`args`).

If you have a list of arguments to pass to `args`, you can spread it the same way you'd spread a list
[inside a list literal](#in-list-literals). The same rules apply: the spread operator is only
recognized before variables, subexpressions, and list literals, and no whitespace is allowed in between.

```nushell
foo "bar" "baz" ...[1 2 3] # With ..., the numbers are treated as separate arguments
# => [false, bar, baz, [1, 2, 3]]
foo "bar" "baz" [1 2 3] # Without ..., [1 2 3] is treated as a single argument
# => [false, bar, baz, [[1, 2, 3]]]
```

A more useful way to use the spread operator is if you have another command with a rest parameter
and you want it to forward its arguments to `foo`:

```nushell
def bar [ ...args ] { foo --flag "bar" "baz" ...$args }
bar 1 2 3
# => [true, bar, baz, [1, 2, 3]]
```

You can spread multiple lists in a single call, and also intersperse individual arguments:

```nushell
foo "bar" "baz" 1 ...[2 3] 4 5 ...(6..9 | take 2) last
# => [false, bar, baz, [1, 2, 3, 4, 5, 6, 7, last]]
```

Flags/named arguments can go after a spread argument, just like they can go after regular rest arguments:

```nushell
foo "bar" "baz" 1 ...[2 3] --flag 4
# => [true, bar, baz, [1, 2, 3, 4]]
```

If a spread argument comes before an optional positional parameter, that optional parameter is treated
as being omitted:

```nushell
foo "bar" ...[1 2] "not opt" # The null means no argument was given for opt
# => [false, bar, null, [1, 2, "not opt"]]
```
# Overlays

Overlays act as "layers" of definitions (custom commands, aliases, environment variables) that can be activated and deactivated on demand.
They resemble virtual environments found in some languages, such as Python.

_Note: To understand overlays, make sure to check [Modules](modules.md) first as overlays build on top of modules._

## Basics

First, Nushell comes with one default overlay called `zero`.
You can inspect which overlays are active with the [`overlay list`](/commands/docs/overlay_list.md) command.
You should see the default overlay listed there.

To create a new overlay, you first need a module:

```nu
module spam {
    export def foo [] {
        "foo"
    }

    export alias bar = echo "bar"

    export-env {
        load-env { BAZ: "baz" }
    }
}
```

We'll use this module throughout the chapter, so whenever you see `overlay use spam`, assume `spam` is referring to this module.

::: tip
The module can be created by any of the three methods described in [Modules](modules.md):

- "inline" modules (used in this example)
- file
- directory
:::

To create the overlay, call [`overlay use`](/commands/docs/overlay_use.md):

```nu
overlay use spam

foo
# => foo

bar
# => bar

$env.BAZ
# => baz

overlay list
# => ───┬──────
# =>  0 │ zero
# =>  1 │ spam
# => ───┴──────
```

It brought the module's definitions into the current scope and evaluated the [`export-env`](/commands/docs/export-env.md) block the same way as [`use`](/commands/docs/use.md) command would (see [Modules](modules.md#environment-variables) chapter).

::: tip
In the following sections, the `>` prompt will be preceded by the name of the last active overlay.
`(spam)> some-command` means the `spam` overlay is the last active overlay when the command was typed.
:::

## Removing an Overlay

If you don't need the overlay definitions anymore, call [`overlay hide`](/commands/docs/overlay_hide.md):

```nu
(spam)> overlay hide spam

(zero)> foo
Error: Can't run executable...

(zero)> overlay list
───┬──────
 0 │ zero
───┴──────
```

The overlays are also scoped.
Any added overlays are removed at the end of the scope:

```nu
(zero)> do { overlay use spam; foo }  # overlay is active only inside the block
foo

(zero)> overlay list
───┬──────
 0 │ zero
───┴──────
```

The last way to remove an overlay is to call [`overlay hide`](/commands/docs/overlay_hide.md) without an argument which will remove the last active overlay.

## Overlays Are Recordable

Any new definition (command, alias, environment variable) is recorded into the last active overlay:

```nu
(zero)> overlay use spam

(spam)> def eggs [] { "eggs" }
```

Now, the `eggs` command belongs to the `spam` overlay.
If we remove the overlay, we can't call it anymore:

```nu
(spam)> overlay hide spam

(zero)> eggs
Error: Can't run executable...
```

But we can bring it back!

```nu
(zero)> overlay use spam

(spam)> eggs
eggs
```

Overlays remember what you add to them and store that information even if you remove them.
This can let you repeatedly swap between different contexts.

::: tip
Sometimes, after adding an overlay, you might not want custom definitions to be added into it.
The solution can be to create a new empty overlay that would be used just for recording the custom changes:

```nu
(zero)> overlay use spam

(spam)> module scratchpad { }

(spam)> overlay use scratchpad

(scratchpad)> def eggs [] { "eggs" }
```

The `eggs` command is added into `scratchpad` while keeping `spam` intact.

To make it less verbose, you can use the [`overlay new`](/commands/docs/overlay_new.md) command:

```nu
(zero)> overlay use spam

(spam)> overlay new scratchpad

(scratchpad)> def eggs [] { "eggs" }
```

:::

## Prefixed Overlays

The [`overlay use`](/commands/docs/overlay_use.md) command would take all commands and aliases from the module and put them directly into the current namespace.
However, you might want to keep them as subcommands behind the module's name.
That's what `--prefix` is for:

```nu
(zero)> module spam {
    export def foo [] { "foo" }
}

(zero)> overlay use --prefix spam

(spam)> spam foo
foo
```

Note that this does not apply for environment variables.

## Rename an Overlay

You can change the name of the added overlay with the `as` keyword:

```nu
(zero)> module spam { export def foo [] { "foo" } }

(zero)> overlay use spam as eggs

(eggs)> foo
foo

(eggs)> overlay hide eggs

(zero)>
```

This can be useful if you have a generic script name, such as virtualenv's `activate.nu` but you want a more descriptive name for your overlay.

## Preserving Definitions

Sometimes, you might want to remove an overlay, but keep all the custom definitions you added without having to redefine them in the next active overlay:

```nu
(zero)> overlay use spam

(spam)> def eggs [] { "eggs" }

(spam)> overlay hide --keep-custom spam

(zero)> eggs
eggs
```

The `--keep-custom` flag does exactly that.

One can also keep a list of environment variables that were defined inside an overlay, but remove the rest, using the `--keep-env` flag:

```nu
(zero)> module spam {
    export def foo [] { "foo" }
    export-env { $env.FOO = "foo" }
}

(zero)> overlay use spam

(spam)> overlay hide spam --keep-env [ FOO ]

(zero)> foo
Error: Can't run executable...

(zero)> $env.FOO
foo
```

## Ordering Overlays

The overlays are arranged as a stack.
If multiple overlays contain the same definition, say `foo`, the one from the last active one would take precedence.
To bring an overlay to the top of the stack, you can call [`overlay use`](/commands/docs/overlay_use.md) again:

```nu
(zero)> def foo [] { "foo-in-zero" }

(zero)> overlay use spam

(spam)> foo
foo

(spam)> overlay use zero

(zero)> foo
foo-in-zero

(zero)> overlay list
───┬──────
 0 │ spam
 1 │ zero
───┴──────
```

Now, the `zero` overlay takes precedence.
# Parallelism

Nushell now has early support for running code in parallel. This allows you to process elements of a stream using more hardware resources of your computer.

You will notice these commands with their characteristic `par-` naming. Each corresponds to a non-parallel version, allowing you to easily write code in a serial style first, and then go back and easily convert serial scripts into parallel scripts with a few extra characters.

## par-each

The most common parallel command is [`par-each`](/commands/docs/par-each.md), a companion to the [`each`](/commands/docs/each.md) command.

Like [`each`](/commands/docs/each.md), [`par-each`](/commands/docs/par-each.md) works on each element in the pipeline as it comes in, running a block on each. Unlike [`each`](/commands/docs/each.md), [`par-each`](/commands/docs/par-each.md) will do these operations in parallel.

Let's say you wanted to count the number of files in each sub-directory of the current directory. Using [`each`](/commands/docs/each.md), you could write this as:

```nu
ls | where type == dir | each { |row|
    { name: $row.name, len: (ls $row.name | length) }
}
```

We create a record for each entry, and fill it with the name of the directory and the count of entries in that sub-directory.

On your machine, the times may vary. For this machine, it took 21 milliseconds for the current directory.

Now, since this operation can be run in parallel, let's convert the above to parallel by changing [`each`](/commands/docs/each.md) to [`par-each`](/commands/docs/par-each.md):

```nu
ls | where type == dir | par-each { |row|
    { name: $row.name, len: (ls $row.name | length) }
}
```

On this machine, it now runs in 6ms. That's quite a difference!

As a side note: Because [environment variables are scoped](environment.md#scoping), you can use [`par-each`](/commands/docs/par-each.md) to work in multiple directories in parallel (notice the [`cd`](/commands/docs/cd.md) command):

```nu
ls | where type == dir | par-each { |row|
    { name: $row.name, len: (cd $row.name; ls | length) }
}
```

You'll notice, if you look at the results, that they come back in different orders each run (depending on the number of hardware threads on your system). As tasks finish, and we get the correct result, we may need to add additional steps if we want our results in a particular order. For example, for the above, we may want to sort the results by the "name" field. This allows both [`each`](/commands/docs/each.md) and [`par-each`](/commands/docs/par-each.md) versions of our script to give the same result.
# Pipelines

One of the core designs of Nu is the pipeline, a design idea that traces its roots back decades to some of the original philosophy behind Unix. Just as Nu extends from the single string data type of Unix, Nu also extends the idea of the pipeline to include more than just text.

## Basics

A pipeline is composed of three parts: the input, the filter, and the output.

```nu
open Cargo.toml | update workspace.dependencies.base64 0.24.2 | save Cargo_new.toml
```

The first command, `open Cargo.toml`, is an input (sometimes also called a "source" or "producer"). This creates or loads data and feeds it into a pipeline. It's from input that pipelines have values to work with. Commands like [`ls`](/commands/docs/ls.md) are also inputs, as they take data from the filesystem and send it through the pipelines so that it can be used.

The second command, `update workspace.dependencies.base64 0.24.2`, is a filter. Filters take the data they are given and often do something with it. They may change it (as with the [`update`](/commands/docs/update.md) command in our example), or they may perform other operations, like logging, as the values pass through.

The last command, `save Cargo_new.toml`, is an output (sometimes called a "sink"). An output takes input from the pipeline and does some final operation on it. In our example, we save what comes through the pipeline to a file as the final step. Other types of output commands may take the values and view them for the user.

The `$in` variable will collect the pipeline into a value for you, allowing you to access the whole stream as a parameter:

```nu
[1 2 3] | $in.1 * $in.2
# => 6
```

## Multi-line pipelines

If a pipeline is getting a bit long for one line, you can enclose it within parentheses `()`:

```nu
let year = (
    "01/22/2021" |
    parse "{month}/{day}/{year}" |
    get year
)
```

## Semicolons

Take this example:

```nu
line1; line2 | line3
```

Here, semicolons are used in conjunction with pipelines. When a semicolon is used, no output data is produced to be piped. As such, the `$in` variable will not work when used immediately after the semicolon.

- As there is a semicolon after `line1`, the command will run to completion and get displayed on the screen.
- `line2` | `line3` is a normal pipeline. It runs, and its contents are displayed after `line1`'s contents.

## Pipeline Input and the Special `$in` Variable

Much of Nu's composability comes from the special `$in` variable, which holds the current pipeline input.

`$in` is particular useful when used in:

- Command or external parameters
- Filters
- Custom command definitions or scripts that accept pipeline input

### `$in` as a Command Argument or as Part of an Expression

Compare the following two command-lines that create a directory with tomorrow's date as part of the name. The following are equivalent:

Using subexpressions:

```nushell
mkdir $'((date now) + 1day | format date '%F') Report'
```

or using pipelines:

```nushell
date now                    # 1: today
| $in + 1day                # 2: tomorrow
| format date '%F'          # 3: Format as YYYY-MM-DD
| $'($in) Report'           # 4: Format the directory name
| mkdir $in                 # 5: Create the directory
```

While the second form may be overly verbose for this contrived example, you'll notice several advantages:

- It can be composed step-by-step with a simple <kbd>↑</kbd> (up arrow) to repeat the previous command and add the next stage of the pipeline.
- It's arguably more readable.
- Each step can, if needed, be commented.
- Each step in the pipeline can be [`inspect`ed for debugging](/commands/docs/inspect.html).

Let's examine the contents of `$in` on each line of the above example:

- On line 2, `$in` refers to the results of line 1's `date now` (a datetime value).
- On line 4, `$in` refers to tomorrow's formatted date from line 3 and is used in an interpolated string
- On line 5, `$in` refers to the results of line 4's interpolated string, e.g. '2024-05-14 Report'

### Pipeline Input in Filter Closures

Certain [filter commands](/commands/categories/filters.html) may modify the pipeline input to their closure in order to provide more convenient access to the expected context. For example:

```nushell
1..10 | each {$in * 2}
```

Rather than referring to the entire range of 10 digits, the `each` filter modifies `$in` to refer to the value of the _current iteration_.

In most filters, the pipeline input and its resulting `$in` will be the same as the closure parameter. For the `each` filter, the following example is equivalent to the one above:

```nushell
1..10 | each {|value| $value * 2}
```

However, some filters will assign an even more convenient value to their closures' input. The `update` filter is one example. The pipeline input to the `update` command's closure (as well as `$in`) refers to the _column_ being updated, while the closure parameter refers to the entire record. As a result, the following two examples are also equivalent:

```nushell
ls | update name {|file| $file.name | str upcase}
ls | update name {str upcase}
```

With most filters, the second version would refer to the entire `file` record (with `name`, `type`, `size`, and `modified` columns). However, with `update`, it refers specifically to the contents of the _column_ being updated, in this case `name`.

### Pipeline Input in Custom Command Definitions and Scripts

See: [Custom Commands -> Pipeline Input](custom_commands.html#pipeline-input)

### When Does `$in` Change (and when can it be reused)?

- **_Rule 1:_** When used in the first position of a pipeline in a closure or block, `$in` refers to the pipeline (or filter) input to the closure/block.

  Example:

  ```nushell
  def echo_me [] {
    print $in
  }
  true | echo_me
  # => true
  ```

- **_Rule 1.5:_** This is true throughout the current scope. Even on subsequent lines in a closure or block, `$in` is the same value when used in the first position of _any pipeline_ inside that scope.

  Example:

  ```nu
  [ a b c ] | each {
    print $in
    print $in
    $in
  }
  ```

  All three of the `$in` values are the same on each iteration, so this outputs:

  ```nu
  a
  a
  b
  b
  c
  c
  ╭───┬───╮
  │ 0 │ a │
  │ 1 │ b │
  │ 2 │ c │
  ╰───┴───╯
  ```

- **_Rule 2:_** When used anywhere else in a pipeline (other than the first position), `$in` refers to the previous expression's result:

  Example:

  ```nushell
  4               # Pipeline input
  | $in * $in     # $in is 4 in this expression
  | $in / 2       # $in is now 16 in this expression
  | $in           # $in is now 8
  # =>   8
  ```

- **_Rule 2.5:_** Inside a closure or block, Rule 2 usage occurs inside a new scope (a sub-expression) where that "new" `$in` value is valid. This means that Rule 1 and Rule 2 usage can coexist in the same closure or block.

  Example:

  ```nushell
  4 | do {
    print $in            # closure-scope $in is 4

    let p = (            # explicit sub-expression, but one will be created regardless
      $in * $in          # initial-pipeline position $in is still 4 here
      | $in / 2          # $in is now 16
    )                    # $p is the result, 8 - Sub-expression scope ends

    print $in            # At the closure-scope, the "original" $in is still 4
    print $p
  }
  ```

  So the output from the 3 `print` statements is:

  ```nu
  4
  4
  8
  ```

  Again, this would hold true even if the command above used the more compact, implicit sub-expression form:

  Example:

  ```nushell
  4 | do {
    print $in                       # closure-scope $in is 4
    let p = $in * $in | $in / 2     # Implicit let sub-expression
    print $in                       # At the closure-scope, $in is still 4
    print $p
  }

  4
  4
  8
  ```

- **_Rule 3:_** When used with no input, `$in` is null.

  Example:

  ```nushell
  # Input
  1 | do { $in | describe }
  # =>   int
  "Hello, Nushell" | do { $in | describe }
  # =>   string
  {||} | do { $in | describe }
  # =>   closure

  # No input
  do { $in | describe }
  # =>   nothing
  ```

* **_Rule 4:_** In a multi-statement line separated by semicolons, `$in` cannot be used to capture the results of the previous _statement_.

  This is the same as having no-input:

  ```nushell
  ls / | get name; $in | describe
  # => nothing
  ```

  Instead, simply continue the pipeline:

  ```nushell
  ls / | get name | $in | describe
  # => list<string>
  ```

### Best practice for `$in` in Multiline Code

While `$in` can be reused as demonstrated above, assigning its value to another variable in the first line of your closure/block will often aid in readability and debugging.

Example:

```nushell
def "date info" [] {
  let day = $in
  print ($day | format date '%v')
  print $'... was a ($day | format date '%A')'
  print $'... was day ($day | format date '%j') of the year'
}

'2000-01-01' | date info
# =>  1-Jan-2000
# => ... was a Saturday
# => ... was day 001 of the year
```

### Collectability of `$in`

Currently, the use of `$in` on a stream in a pipeline results in a "collected" value, meaning the pipeline "waits" on the stream to complete before handling `$in` with the full results. However, this behavior is not guaranteed in future releases. To ensure that a stream is collected into a single variable, use the [`collect` command](/commands/docs/collect.html).

Likewise, avoid using `$in` when normal pipeline input will suffice, as internally `$in` forces a conversion from `PipelineData` to `Value` and _may_ result in decreased performance and/or increased memory usage.

## Working with External Commands

Nu commands communicate with each other using the Nu data types (see [types of data](types_of_data.md)), but what about commands outside of Nu? Let's look at some examples of working with external commands:

`internal_command | external_command`

Data will flow from the internal_command to the external_command. This data will get converted to a string, so that they can be sent to the `stdin` of the external_command.

`external_command | internal_command`

Data coming from an external command into Nu will come in as bytes that Nushell will try to automatically convert to UTF-8 text. If successful, a stream of text data will be sent to internal_command. If unsuccessful, a stream of binary data will be sent to internal command. Commands like [`lines`](/commands/docs/lines.md) help make it easier to bring in data from external commands, as it gives discrete lines of data to work with.

`external_command_1 | external_command_2`

Nu works with data piped between two external commands in the same way as other shells, like Bash would. The `stdout` of external_command_1 is connected to the `stdin` of external_command_2. This lets data flow naturally between the two commands.

### Command Input and Output Types

The Basics section above describes how commands can be combined in pipelines as input, filters, or output.
How you can use commands depends on what they offer in terms of input/output handling.

You can check what a command supports with [`help <command name>`](/commands/docs/help.md), which shows the relevant *Input/output types*.

For example, through `help first` we can see that the [`first` command](/commands/docs/first.md) supports multiple input and output types:

```nu
help first
# => […]
# => Input/output types:
# =>   ╭───┬───────────┬────────╮
# =>   │ # │   input   │ output │
# =>   ├───┼───────────┼────────┤
# =>   │ 0 │ list<any> │ any    │
# =>   │ 1 │ binary    │ binary │
# =>   │ 2 │ range     │ any    │
# =>   ╰───┴───────────┴────────╯

[a b c] | first                                                                                                                                   took 1ms
# => a

1..4 | first                                                                                                                                     took 21ms
# => 1
```

As another example, the [`ls` command](/commands/docs/ls.md) supports output but not input:

```nu
help ls
# => […]
# => Input/output types:
# =>   ╭───┬─────────┬────────╮
# =>   │ # │  input  │ output │
# =>   ├───┼─────────┼────────┤
# =>   │ 0 │ nothing │ table  │
# =>   ╰───┴─────────┴────────╯
```

This means, for example, that attempting to pipe into `ls` (`echo .. | ls`) leads to unintended results.
The input stream is ignored, and `ls` defaults to listing the current directory.

To integrate a command like `ls` into a pipeline, you have to explicitly reference the input and pass it as a parameter:

```nu
echo .. | ls $in
```

Note that this only works if `$in` matches the argument type. For example, `[dir1 dir2] | ls $in` will fail with the error `can't convert list<string> to string`.

Other commands without default behavior may fail in different ways, and with explicit errors.

For example, `help sleep` tells us that [`sleep`](/commands/docs/sleep.md) supports no input and no output types:

```nu
help sleep
# => […]
# => Input/output types:
# =>   ╭───┬─────────┬─────────╮
# =>   │ # │  input  │ output  │
# =>   ├───┼─────────┼─────────┤
# =>   │ 0 │ nothing │ nothing │
# =>   ╰───┴─────────┴─────────╯
```

When we erroneously pipe into it, instead of unintended behavior like in the `ls` example above, we receive an error:

```nu
echo 1sec | sleep
# => Error: nu::parser::missing_positional
# => 
# =>   × Missing required positional argument.
# =>    ╭─[entry #53:1:18]
# =>  1 │ echo 1sec | sleep
# =>    ╰────
# =>   help: Usage: sleep <duration> ...(rest) . Use `--help` for more information.
```

While there is no steadfast rule, Nu generally tries to copy established conventions in command behavior,
or do what 'feels right'.
The `sleep` behavior of not supporting an input stream matches Bash `sleep` behavior for example.

Many commands do have piped input/output however, and if it's ever unclear, check their `help` documentation as described above.

## Behind the Scenes

You may have wondered how we see a table if [`ls`](/commands/docs/ls.md) is an input and not an output. Nu adds this output for us automatically using another command called [`table`](/commands/docs/table.md). The [`table`](/commands/docs/table.md) command is appended to any pipeline that doesn't have an output. This allows us to see the result.

In effect, the command:

```nu
ls
```

And the pipeline:

```nu
ls | table
```

Are one and the same.

::: tip Note
The phrase _"are one and the same"_ above only applies to the graphical output in the shell, it does not mean the two data structures are the same:

```nushell
(ls) == (ls | table)
# => false
```

`ls | table` is not even structured data!
:::

## Output Result to External Commands

Sometimes you want to output Nushell structured data to an external command for further processing. However, Nushell's default formatting options for structured data may not be what you want.
For example, you want to find a file named "tutor" under "/usr/share/vim/runtime" and check its ownership

```nu
ls /usr/share/nvim/runtime/
# => ╭────┬───────────────────────────────────────┬──────┬─────────┬───────────────╮
# => │  # │                 name                  │ type │  size   │   modified    │
# => ├────┼───────────────────────────────────────┼──────┼─────────┼───────────────┤
# => │  0 │ /usr/share/nvim/runtime/autoload      │ dir  │  4.1 KB │ 2 days ago    │
# => ..........
# => ..........
# => ..........
# => 
# => │ 31 │ /usr/share/nvim/runtime/tools         │ dir  │  4.1 KB │ 2 days ago    │
# => │ 32 │ /usr/share/nvim/runtime/tutor         │ dir  │  4.1 KB │ 2 days ago    │
# => ├────┼───────────────────────────────────────┼──────┼─────────┼───────────────┤
# => │  # │                 name                  │ type │  size   │   modified    │
# => ╰────┴───────────────────────────────────────┴──────┴─────────┴───────────────╯
```

You decided to use `grep` and [pipe](https://www.nushell.sh/book/pipelines.html) the result to external `^ls`

```nu
ls /usr/share/nvim/runtime/ | get name | ^grep tutor | ^ls -la $in
# => ls: cannot access ''$'\342\224\202'' 32 '$'\342\224\202'' /usr/share/nvim/runtime/tutor        '$'\342\224\202\n': No such file or directory
```

What's wrong? Nushell renders lists and tables (by adding a border with characters like `╭`,`─`,`┬`,`╮`) before piping them as text to external commands. If that's not the behavior you want, you must explicitly convert the data to a string before piping it to an external. For example, you can do so with [`to text`](/commands/docs/to_text.md):

```nu
ls /usr/share/nvim/runtime/ | get name | to text | ^grep tutor | tr -d '\n' | ^ls -la $in
# => total 24
# => drwxr-xr-x@  5 pengs  admin   160 14 Nov 13:12 .
# => drwxr-xr-x@  4 pengs  admin   128 14 Nov 13:42 en
# => -rw-r--r--@  1 pengs  admin  5514 14 Nov 13:42 tutor.tutor
# => -rw-r--r--@  1 pengs  admin  1191 14 Nov 13:42 tutor.tutor.json
```

(Actually, for this simple usage you can just use [`find`](/commands/docs/find.md))

```nu
ls /usr/share/nvim/runtime/ | get name | find tutor | ansi strip | ^ls -al ...$in
```

## Command Output in Nushell

Unlike external commands, Nushell commands are akin to functions. Most Nushell commands do not print anything to `stdout` and instead just return data.

```nu
do { ls; ls; ls; "What?!" }
```

This means that the above code will not display the files under the current directory three times.
In fact, running this in the shell will only display `"What?!"` because that is the value returned by the `do` command in this example. However, using the system `^ls` command instead of `ls` would indeed print the directory thrice because `^ls` does print its result once it runs.

Knowing when data is displayed is important when using configuration variables that affect the display output of commands such as `table`.

```nu
do { $env.config.table.mode = "none"; ls }
```

For instance, the above example sets the `$env.config.table.mode` configuration variable to `none`, which causes the `table` command to render data without additional borders. However, as it was shown earlier, the command is effectively equivalent to

```nu
do { $env.config.table.mode = "none"; ls } | table
```

Because Nushell `$env` variables are [scoped](https://www.nushell.sh/book/environment.html#scoping), this means that the `table` command in the example is not affected by the
environment modification inside the `do` block and the data will not be shown with the applied configuration.

When displaying data early is desired, it is possible to explicitly apply `| table` inside the scope, or use the `print` command.

```nu
do { $env.config.table.mode = "none"; ls | table }
do { $env.config.table.mode = "none"; print (ls) }
```
# Plugins

Nu can be extended using plugins. Plugins behave much like Nu's built-in commands, with the added benefit that they can be added separately from Nu itself.

::: warning Important
Plugins communicate with Nushell using the `nu-plugin` protocol. This protocol is versioned, and plugins must use the same `nu-plugin` version provided by Nushell.

When updating Nushell, please make sure to also update any plugins that you have registered.
:::

[[toc]]

## Installing Plugins

### Core Plugins

Nushell ships with a set of officially maintained plugins which includes:

- `polars`: Extremely fast columnar operations using DataFrames via the [Polars Library](https://github.com/pola-rs/polars). See the [DataFrames Chapter](dataframes.html) for more details.
- `formats`: Support for several additional data formats - EML, ICS, INI, plist, and VCF.
- `gstat`: Returns information on the status of a Git repository as Nushell structured data.
- `query`: Support for querying SQL, XML, JSON, HTML (via selector), and WebPage Metadata
- `inc`: Increment a value or version (e.g., semver). This plugin acts as both an end-user plugin as well as a simple developer example of how to create a plugin.

Nushell also ships with several plugins that serve as examples or tools for plugin developers. These include `nu_plugin_example`, `nu_plugin_custom_values`, and `nu_plugin_stress_internals`.

Core plugins are typically distributed with the Nushell release and should already be installed in the same directory as the Nushell executable. If this is the case on your system, core plugins should be using correct `nu-plugin` protocol version. If your package management system installs them separately, please make sure to update the core plugins whenever Nushell itself is updated.

::: tip Installing using Cargo
For example, when installing or upgrading Nushell directly from crates.io using `cargo install nu --locked`, the corresponding core plugins for that version may also be installed or updated using `cargo install nu_plugin_<plugin_name> --locked`.

To install all of the default plugins, from within Nushell run:

```nu
[ nu_plugin_inc
  nu_plugin_polars
  nu_plugin_gstat
  nu_plugin_formats
  nu_plugin_query
] | each { cargo install $in --locked } | ignore
```

:::

### Third-party Plugins

You can find third-party plugins on crates.io, online Git repositories, [`awesome-nu`](https://github.com/nushell/awesome-nu/blob/main/plugin_details.md), and other sources. As with any third-party code you run on your system, please make sure you trust its source.

To install a third-party plugin on your system, you first need to make sure the plugin uses the same version of Nu as your system:

- Confirm your Nushell version with the `version` command
- Confirm the version the plugin requires by checking its `Cargo.toml` file

To install a plugin by name from crates.io, run:

```nu
cargo install nu_plugin_<plugin_name> --locked
```

When installing from a repository (e.g., GitHub), run the following from inside the cloned repository:

```nu
cargo install --path . --locked
```

This will create a binary file that can be used to add the plugin.

::: tip Cargo installation location
By default, binaries installed with `cargo install` are placed in your home directory under `.cargo/bin`.
However, this can change depending on how your system is configured.
:::

## Registering Plugins

To add a plugin to the plugin registry file, call the [`plugin add`](/commands/docs/plugin_add.md) command to tell Nu where to find it.

::: tip Note
The plugin file name must start with `nu_plugin_`, Nu uses this filename prefix to identify plugins.
:::

- Linux and macOS:

  ```nu
  plugin add ./my_plugins/nu_plugin_cool
  ```

- Windows:

  ```nu
  plugin add .\my_plugins\nu_plugin_cool.exe
  ```

When [`plugin add`](/commands/docs/plugin_add.md) is called, Nu:

- Runs the plugin binary
- Communicates via the [plugin protocol](/contributor-book/plugin_protocol_reference.md) in order to ensure compatibility and to get a list of all of the commands it supports
- This plugin information is then saved to the plugin registry file (`$nu.plugin-path`), which acts as a cache

Once added to the registry, the next time `nu` is started, the plugin will be available in that session.

You can also immediately reload a plugin in the current session by calling `plugin use`. In this case, the name of the plugin (rather than the filename) is used without the `nu_plugin` prefix:

```nu
plugin use cool
```

It is not necessary to add `plugin use` statements to your config file. All previously added plugins are automatically loaded at startup.

::: tip Note
`plugin use` is a parser keyword, so when evaluating a script, it will be evaluated first. This means that while you can execute `plugin add` and then `plugin use` at the REPL on separate lines, you can't do this in a single script. If you need to run `nu` with a specific plugin or set of plugins without preparing a cache file, you can pass the `--plugins` option to `nu` with a list of plugin executable files:

```nu
nu --plugins '[./my_plugins/nu_plugin_cool]'
```

:::

### Updating Plugins

When updating a plugin, it is important to run `plugin add` again just as above to load the new signatures from the plugin and allow Nu to rewrite them to the plugin file (`$nu.plugin-path`). You can then `plugin use` to get the updated signatures within the current session.

## Managing Plugins

Installed plugins are displayed using [`plugin list`](/commands/docs/plugin_list.md):

```nu
plugin list
# =>
╭───┬─────────┬────────────┬─────────┬───────────────────────┬───────┬───────────────────────────────╮
│ # │  name   │ is_running │   pid   │       filename        │ shell │           commands            │
├───┼─────────┼────────────┼─────────┼───────────────────────┼───────┼───────────────────────────────┤
│ 0 │ gstat   │ true       │ 1389890 │ .../nu_plugin_gstat   │       │ ╭───┬───────╮                 │
│   │         │            │         │                       │       │ │ 0 │ gstat │                 │
│   │         │            │         │                       │       │ ╰───┴───────╯                 │
│ 1 │ inc     │ false      │         │ .../nu_plugin_inc     │       │ ╭───┬─────╮                   │
│   │         │            │         │                       │       │ │ 0 │ inc │                   │
│   │         │            │         │                       │       │ ╰───┴─────╯                   │
│ 2 │ example │ false      │         │ .../nu_plugin_example │       │ ╭───┬───────────────────────╮ │
│   │         │            │         │                       │       │ │ 0 │ nu-example-1          │ │
│   │         │            │         │                       │       │ │ 1 │ nu-example-2          │ │
│   │         │            │         │                       │       │ │ 2 │ nu-example-3          │ │
│   │         │            │         │                       │       │ │ 3 │ nu-example-config     │ │
│   │         │            │         │                       │       │ │ 4 │ nu-example-disable-gc │ │
│   │         │            │         │                       │       │ ╰───┴───────────────────────╯ │
╰───┴─────────┴────────────┴─────────┴───────────────────────┴───────┴───────────────────────────────╯
```

All of the commands from installed plugins are available in the current scope:

```nu
scope commands | where type == "plugin"
```

### Plugin Lifecycle

Plugins stay running while they are in use, and are automatically stopped by default after a period of time of inactivity. This behavior is managed by the [plugin garbage collector](#plugin-garbage-collector). To manually stop a plugin, call `plugin stop` with its name:

For example, run the `gstat` command from the corresponding plugin, then check its `is_running` status:

```nu
gstat
# => gstat output
plugin list | where name == gstat | select name is_running
# =>
╭───┬───────┬────────────╮
│ # │ name  │ is_running │
├───┼───────┼────────────┤
│ 0 │ gstat │ true       │
╰───┴───────┴────────────╯
```

Now stop the plugin manually, and we can see that it is no longer running:

```nu
plugin stop gstat
plugin list | where name == gstat | select name is_running
# =>
╭───┬───────┬────────────╮
│ # │ name  │ is_running │
├───┼───────┼────────────┤
│ 0 │ gstat │ false      │
╰───┴───────┴────────────╯
```

### Plugin Garbage Collector

As mentioned above, Nu comes with a plugin garbage collector which automatically stops plugins that are not actively in use after a period of time (by default, 10 seconds). This behavior is fully configurable:

```nu
$env.config.plugin_gc = {
    # Settings for plugins not otherwise specified:
    default: {
        enabled: true # set to false to never automatically stop plugins
        stop_after: 10sec # how long to wait after the plugin is inactive before stopping it
    }
    # Settings for specific plugins, by plugin name
    # (i.e. what you see in `plugin list`):
    plugins: {
        gstat: {
            stop_after: 1min
        }
        inc: {
            stop_after: 0sec # stop as soon as possible
        }
        example: {
            enabled: false # never stop automatically
        }
    }
}
```

For information on when a plugin is considered to be active, see [the relevant section in the contributor book](/contributor-book/plugins.html#plugin-garbage-collection).

## Removing Plugins

To remove a plugin, call `plugin rm <plugin_name>`. Note that this is the plugin name, rather than the filename. For example, if you previously added the plugin `~/.cargo/bin/nu_plugin_gstat`, its name would be `gstat`. To remove it:

```nu
plugin rm gstat
```

You can confirm the name of a plugin by running `plugin list`.

Running `plugin rm` removes the plugin from the registry so that it will not be loaded the next time Nushell starts. However, any commands created by the plugin remain in scope until the current Nushell session ends.

## For Plugin Developers

Nu plugins are executables; Nu launches them as needed and communicates with them over [stdin and stdout](https://en.wikipedia.org/wiki/Standard_streams) or [local sockets](https://en.wikipedia.org/wiki/Inter-process_communication). Nu plugins can use either [JSON](https://www.json.org/) or [MessagePack](https://msgpack.org/) as their communication encoding.

### Examples

Nu's main repo contains example plugins that are useful for learning how the plugin protocol works:

- [Rust](https://github.com/nushell/nushell/tree/main/crates/nu_plugin_example)
- [Python](https://github.com/nushell/nushell/blob/main/crates/nu_plugin_python)

### Debugging

The simplest way to debug a plugin is to print to stderr; plugins' standard error streams are redirected through Nu and displayed to the user.

#### Tracing

The Nu plugin protocol message stream may be captured for diagnostic purposes using [trace_nu_plugin](https://crates.io/crates/trace_nu_plugin/).

::: warning
Trace output will accumulate for as long as the plugin is installed with the trace wrapper. Large files are possible. Be sure to remove the plugin with `plugin rm` when finished tracing, and reinstall without the trace wrapper.\*\*
:::

### Developer Help

Nu's plugin documentation is a work in progress. If you're unsure about something, the #plugins channel on [the Nu Discord](https://discord.gg/NtAbbGn) is a great place to ask questions!

### More details

The [plugin chapter in the contributor book](/contributor-book/plugins.md) offers more details on the intricacies of how plugins work from a software developer point of view.
# Programming in Nu

This chapter goes into more detail of Nushell as a programming language.
Each major language feature has its own section.

Just like most programming languages allow you to define functions, Nushell uses [custom commands](custom_commands.md) for this purpose.

From other shells you might be used to [aliases](aliases.md).
Nushell's aliases work in a similar way and are a part of the programming language, not just a shell feature.

Common operations, such as addition or regex search, can be done with [operators](operators.md).
Not all operations are supported for all data types, and Nushell will make sure to let you know when there is a mismatch.

You can store intermediate results to [variables](variables.md).
Variables can be immutable, mutable, or a parse-time constant.

The last three sections are aimed at organizing your code:

[Scripts](scripts.md) are the simplest form of code organization: You just put the code into a file and source it.
However, you can also run scripts as standalone programs with command line signatures using the "special" `main` command.

With [modules](modules.md), just like in many other programming languages, it is possible to compose your code from smaller pieces.
Modules let you define a public interface vs. private commands and you can import custom commands, aliases, and environment variables from them.

[Overlays](overlays.md) build on top of modules.
By defining an overlay, you bring in module's definitions into its own swappable "layer" that gets applied on top of other overlays.
This enables features like activating virtual environments or overriding sets of default commands with custom variants.

The standard library also has a [testing framework](testing.md) if you want to prove your reusable code works perfectly.
# Quick Tour

[[toc]]

## Nushell Commands Output _Data_

The easiest way to see what Nu can do is to start with some examples, so let's dive in.

The first thing you'll notice when you run a command like [`ls`](/commands/docs/ls.md) is that instead of a block of text coming back, you get a structured table.

```nu:no-line-numbers
ls
# => ╭────┬─────────────────────┬──────┬───────────┬──────────────╮
# => │  # │        name         │ type │   size    │   modified   │
# => ├────┼─────────────────────┼──────┼───────────┼──────────────┤
# => │  0 │ CITATION.cff        │ file │     812 B │ 2 months ago │
# => │  1 │ CODE_OF_CONDUCT.md  │ file │   3.4 KiB │ 9 months ago │
# => │  2 │ CONTRIBUTING.md     │ file │  11.0 KiB │ 5 months ago │
# => │  3 │ Cargo.lock          │ file │ 194.9 KiB │ 15 hours ago │
# => │  4 │ Cargo.toml          │ file │   9.2 KiB │ 15 hours ago │
# => │  5 │ Cross.toml          │ file │     666 B │ 6 months ago │
# => │  6 │ LICENSE             │ file │   1.1 KiB │ 9 months ago │
# => │  7 │ README.md           │ file │  12.0 KiB │ 15 hours ago │
# => ...
```

This table does more than just format the output nicely. Like a spreadsheet, it allows us to work with the data _interactively_.

## Acting on Data

Next, let's sort this table by each file's size. To do this, we'll take the output from [`ls`](/commands/docs/ls.md) and feed it into a command that can sort tables based on the _values_ in a column.

```nu:no-line-numbers
ls | sort-by size | reverse
# => ╭───┬─────────────────┬──────┬───────────┬──────────────╮
# => │ # │      name       │ type │   size    │   modified   │
# => ├───┼─────────────────┼──────┼───────────┼──────────────┤
# => │ 0 │ Cargo.lock      │ file │ 194.9 KiB │ 15 hours ago │
# => │ 1 │ toolkit.nu      │ file │  20.0 KiB │ 15 hours ago │
# => │ 2 │ README.md       │ file │  12.0 KiB │ 15 hours ago │
# => │ 3 │ CONTRIBUTING.md │ file │  11.0 KiB │ 5 months ago │
# => │ 4 │ ...             │ ...  │ ...       │ ...          │
# => │ 5 │ LICENSE         │ file │   1.1 KiB │ 9 months ago │
# => │ 6 │ CITATION.cff    │ file │     812 B │ 2 months ago │
# => │ 7 │ Cross.toml      │ file │     666 B │ 6 months ago │
# => │ 8 │ typos.toml      │ file │     513 B │ 2 months ago │
# => ╰───┴─────────────────┴──────┴───────────┴──────────────╯
```

Notice that we didn't pass commandline arguments or switches to [`ls`](/commands/docs/ls.md). Instead, we used Nushell's built-in [`sort-by`](/commands/docs/sort-by.md) command to sort the _output_ of the `ls` command. Then, to see the largest files on top, we used [`reverse`](/commands/docs/reverse.md) on the _output_ of `sort-by`.

::: tip Cool!
If you compare the sort order closely, you might realize that the data isn't sorted alphabetically. It's not even sorted by the _numerical_ values. Instead, since the `size` column is a [`filesize` type](./types_of_data.md#file-sizes), Nushell knows that `1.1 KiB` (kibibytes) is larger than `812 B` (bytes).
:::

# Finding Data Using the `where` Command

Nu provides many commands that can operate on the structured output of the previous command. These are usually categorized as "Filters" in Nushell.

For example, we can use [`where`](/commands/docs/where.md) to filter the contents of the table so that it only shows files over 10 kilobytes:

```nu
ls | where size > 10kb
# => ╭───┬─────────────────┬──────┬───────────┬───────────────╮
# => │ # │      name       │ type │   size    │   modified    │
# => ├───┼─────────────────┼──────┼───────────┼───────────────┤
# => │ 0 │ CONTRIBUTING.md │ file │  11.0 KiB │ 5 months ago  │
# => │ 1 │ Cargo.lock      │ file │ 194.6 KiB │ 2 minutes ago │
# => │ 2 │ README.md       │ file │  12.0 KiB │ 16 hours ago  │
# => │ 3 │ toolkit.nu      │ file │  20.0 KiB │ 16 hours ago  │
# => ╰───┴─────────────────┴──────┴───────────┴───────────────╯
```

## More Than Just Directories

Of course, this isn't limited to the `ls` command. Nushell follows the Unix philosophy where each command does one thing well and you can typically expect the output of one command to become the input of another. This allows us to mix-and-match commands in many different combinations.

Let's look at a different command:

```nu:no-line-numbers
ps
# => ╭───┬──────┬──────┬───────────────┬──────────┬──────┬───────────┬─────────╮
# => │ # │ pid  │ ppid │     name      │  status  │ cpu  │    mem    │ virtual │
# => ├───┼──────┼──────┼───────────────┼──────────┼──────┼───────────┼─────────┤
# => │ 0 │    1 │    0 │ init(void)    │ Sleeping │ 0.00 │   1.2 MiB │ 2.2 MiB │
# => │ 1 │    8 │    1 │ init          │ Sleeping │ 0.00 │ 124.0 KiB │ 2.3 MiB │
# => │ 2 │ 6565 │    1 │ SessionLeader │ Sleeping │ 0.00 │ 108.0 KiB │ 2.2 MiB │
# => │ 3 │ 6566 │ 6565 │ Relay(6567)   │ Sleeping │ 0.00 │ 116.0 KiB │ 2.2 MiB │
# => │ 4 │ 6567 │ 6566 │ nu            │ Running  │ 0.00 │  28.4 MiB │ 1.1 GiB │
# => ╰───┴──────┴──────┴───────────────┴──────────┴──────┴───────────┴─────────╯
```

You may be familiar with the Linux/Unix `ps` command. It provides a list of all of the current processes running in the system along with their current status. As with `ls`, Nushell provides a cross-platform, built-in [`ps` command](/commands/docs/ps.md) that returns its results as structured data.

::: note
The traditional Unix `ps` only shows the current process and its parents by default. Nushell's implementation shows all of the processes on the system by default.

Normally, running `ps` in Nushell uses its **_internal_**, cross-platform command. However, it is still possible to run the **_external_**, system-dependent version on Unix/Linux platforms by prefacing it with the _caret sigil_. For example:

```nu
^ps aux  # run the Unix ps command with all processes in user-oriented form
```

See [Running External System Commands](./running_externals.md) for more details.
:::

What if we wanted to just show the processes that are actively running? As with `ls` above, we can also work with the table that `ps` _outputs_:

```nu
ps | where status == Running
# => ╭───┬──────┬──────┬──────┬─────────┬──────┬──────────┬─────────╮
# => │ # │ pid  │ ppid │ name │ status  │ cpu  │   mem    │ virtual │
# => ├───┼──────┼──────┼──────┼─────────┼──────┼──────────┼─────────┤
# => │ 0 │ 6585 │ 6584 │ nu   │ Running │ 0.00 │ 31.9 MiB │ 1.2 GiB │
# => ╰───┴──────┴──────┴──────┴─────────┴──────┴──────────┴─────────╯
```

::: tip
Remember above, where the `size` column from the `ls` command was a `filesize`? Here, `status` is really just a string, and you can use all the normal string operations and commands with it, including (as above) the `==` comparison.

You can examine the types for the table's columns using:

```nu
ps | describe
# => table<pid: int, ppid: int, name: string, status: string, cpu: float, mem: filesize, virtual: filesize> (stream)
```

The [`describe` command](/commands/docs/describe.md) can be used to display the output type of any command or expression.

:::

## Command Arguments in a Pipeline

Sometimes, a command takes an _argument_ instead of pipeline _input_. For this scenario, Nushell provides the [`$in` variable](./pipelines.md#pipeline-input-and-the-special-in-variable) that let's you use the previous command's output in variable-form. For example:

```nu:line-numbers
ls
| sort-by size
| reverse
| first
| get name
| cp $in ~
```

::: tip Nushell Design Note
Whenever possible, Nushell commands are designed to act on pipeline _input_. However, some commands, like `cp` in this example, have two (or more)
arguments with different meanings. In this case, `cp` needs to know both the path to _copy_ as well as the _target_ path. As a result, this command is
more ergonomic with two _positional parameters_.
:::

::: tip
Nushell commands can extend across multiple lines for readability. The above is the same as:

```nu
ls | sort-by size | reverse | first | get name | cp $in ~
```

See Also: [Multi-line Editing](./line_editor.md#multi-line-editing)
:::

The first three lines are the same commands we used in the second example above, so let's examine the last three:

4. The [`first` command](/commands/docs/first.md) simply returns the first value from the table. In this case, that means the file with the largest size. That's the `Cargo.lock` file if using the directory listing from the second example above. This "file" is a [`record`](./types_of_data.md#records) from the table which still contains its `name`, `type`, `size`, and `modified` columns/fields.
5. `get name` returns the _value_ of the `name` field from the previous command, so `"Cargo.lock"` (a string). This is also a simple example of a [`cell-path`](./types_of_data.md#cell-paths) that can be used to navigate and isolate structured data.
6. The last line uses the `$in` variable to reference the output of line 5. The result is a command that says _"Copy 'Cargo.lock' to the home directory"_

::: tip
[`get`](/commands/docs/get.md) and its counterpart [`select`](/commands/docs/select.md) are two of the most used filters in Nushell, but it might not be easy to
spot the difference between them at first glance. When you're ready to start using them more extensively, see [Using `get` and `select`](./navigating_structured_data.md#using-get-and-select) for a guide.
:::

## Getting Help

Nushell provides an extensive, in-shell Help system. For example

```nu
# help <command>
help ls
# Or
ls --help
# Also
help operators
help escapes
```

::: tip Cool!
Press the <kbd>F1</kbd> key to access the Help [menu](./line_editor.md#menus). Search for the `ps` command here, but _don't press <kbd>Enter</kbd> just yet_!

Instead, press the <kbd>Down Arrow</kbd> key, and notice that you are scrolling through the Examples section. Highlight an example, _then_ press <kbd>Enter</kbd> and
the example will be entered at the commandline, ready to run!

This can be a great way to explore and learn about the extensive set of Nushell commands.
:::

The Help system also has a "search" feature:

```nu
help --find filesize
# or
help -f filesize
```

It may not surprise you by now that the Help system itself is based on structured data! Notice that the output of `help -f filesize` is a table.

The Help for each command is stored as a record with the:

- Name
- Category
- Type (built-in, plugin, custom)
- Parameters it accepts
- Signatures showing what types of data it can accept as well as output
- And more

You can view _all_ commands (other than externals) as a single large table using:

```nu
help commands
```

::: tip
Notice that the `params` and `input_output` columns of the output above are _nested_ tables. Nushell allows [arbitrarily nested data structures](./navigating_structured_data.md#background).
:::

## `explore`'ing from Here

That `help commands` output is quite long. You could send it to a pager like `less` or `bat`, but Nushell includes a built-in `explore` command that lets you not only scroll, but also telescope-in to nested data. Try:

```nu
help commands | explore
```

Then press the <kbd>Enter</kbd> key to access the data itself. Use the arrow keys to scroll to the `cp` command, and over to the `params` column. Hit <kbd>Enter</kbd> again to
telescope in to the complete list of parameters available to the `cp` command.

::: note
Pressing <kbd>Esc</kbd> one time returns from Scroll-mode to the View; Pressing it a second time returns to the previous view (or exits, if already at the top view level).
:::

::: tip
You can, of course, use the `explore` command on _any_ structured data in Nushell. This might include JSON data coming from a Web API, a spreadsheet or CSV file, YAML, or anything that can be represented as structured data in Nushell.

Try `$env.config | explore` for fun!
:::
# Regular Expressions

Regular expressions in Nushell's commands are handled by the `rust-lang/regex` crate. If you want to know more, check the crate documentation: "[regex](https://github.com/rust-lang/regex)".
# Running System (External) Commands

Nu provides a set of commands that you can use across different operating systems ("internal" commands) and having this consistency is helpful when creating cross-platform code. Sometimes, though, you want to run an external command that has the same name as an internal Nu command. To run the external [`ls`](/commands/docs/ls.md) or [`date`](/commands/docs/date.md) command, for example, preface it with the caret (^) sigil. Prefacing with the caret calls the external command found in the user's `PATH` (e.g. `/bin/ls`) instead of Nu's internal [`ls`](/commands/docs/ls.md) command).

Nu internal command:

```nu
ls
```

External command (typically `/usr/bin/ls`):

```nu
^ls
```

::: note
On Windows, `ls` is a PowerShell _alias_ by default, so `^ls` will not find a matching system _command_.
:::

## Additional Windows Notes

When running an external command on Windows,
Nushell forwards some `CMD.EXE` internal commands to cmd instead of attempting to run external commands.
[Coming from CMD.EXE](coming_from_cmd.md) contains a list of these commands and describes the behavior in more detail.
# Scripts

In Nushell, you can write and run scripts in the Nushell language. To run a script, you can pass it as an argument to the `nu` commandline application:

```nu
nu myscript.nu
```

This will run the script to completion in a new instance of Nu. You can also run scripts inside the _current_ instance of Nu using [`source`](/commands/docs/source.md):

```nu
source myscript.nu
```

Let's look at an example script file:

```nu
# myscript.nu
def greet [name] {
  ["hello" $name]
}

greet "world"
```

A script file defines the definitions for custom commands as well as the main script itself, which will run after the custom commands are defined.

In the above example, first `greet` is defined by the Nushell interpreter. This allows us to later call this definition. We could have written the above as:

```nu
greet "world"

def greet [name] {
  ["hello" $name]
}
```

There is no requirement that definitions have to come before the parts of the script that call the definitions, allowing you to put them where you feel comfortable.

## How Scripts are Processed

In a script, definitions run first. This allows us to call the definitions using the calls in the script.

After the definitions run, we start at the top of the script file and run each group of commands one after another.

## Script Lines

To better understand how Nushell sees lines of code, let's take a look at an example script:

```nu
a
b; c | d
```

When this script is run, Nushell will first run the `a` command to completion and view its results. Next, Nushell will run `b; c | d` following the rules in the ["Semicolons" section](pipelines.html#semicolons).

## Parameterizing Scripts

Script files can optionally contain a special "main" command. `main` will be run after any other Nu code, and is primarily used to add parameters to scripts. You can pass arguments to scripts after the script name (`nu <script name> <script args>`).

For example:

```nu
# myscript.nu

def main [x: int] {
  $x + 10
}
```

```nu
nu myscript.nu 100
# => 110
```

## Argument Type Interpretation

By default, arguments provided to a script are interpreted with the type `Type::Any`, implying that they are not constrained to a specific data type and can be dynamically interpreted as fitting any of the available data types during script execution.

In the previous example, `main [x: int]` denotes that the argument x should possess an integer data type. However, if arguments are not explicitly typed, they will be parsed according to their apparent data type.

For example:

```nu
# implicit_type.nu
def main [x] {
  $"Hello ($x | describe) ($x)"
}

# explicit_type.nu
def main [x: string] {
  $"Hello ($x | describe) ($x)"
}
```

```nu
nu implicit_type.nu +1
# => Hello int 1

nu explicit_type.nu +1
# => Hello string +1
```

## Subcommands

A script can have multiple [subcommands](custom_commands.html#subcommands), like `run` or `build` for example:

```nu
# myscript.nu
def "main run" [] {
    print "running"
}

def "main build" [] {
    print "building"
}

def main [] {
    print "hello from myscript!"
}
```

You can then execute the script's subcommands when calling it:

```nu
nu myscript.nu
# => hello from myscript!
nu myscript.nu build
# => building
nu myscript.nu run
# => running
```

[Unlike modules](modules.html#main), `main` does _not_ need to be exported in order to be visible. In the above example, our `main` command is not `export def`, however it was still executed when running `nu myscript.nu`. If we had used myscript as a module by running `use myscript.nu`, rather than running `myscript.nu` as a script, trying to execute the `myscript` command would not work since `myscript` is not exported.

It is important to note that you must define a `main` command in order for subcommands of `main` to be correctly exposed. For example, if we had just defined the `run` and `build` subcommands, they wouldn't be accessible when running the script:

```nu
# myscript.nu
def "main run" [] {
    print "running"
}

def "main build" [] {
    print "building"
}
```

```nu
nu myscript.nu build
nu myscript.nu run
```

This is a limitation of the way scripts are currently processed. If your script only has subcommands, you can add an empty `main` to expose the subcommands, like so:

```nu
def main [] {}
```

## Shebangs (`#!`)

On Linux and macOS you can optionally use a [shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) to tell the OS that a file should be interpreted by Nu. For example, with the following in a file named `myscript`:

```nu
#!/usr/bin/env nu
"Hello World!"
```

```nu
./myscript
# => Hello World!
```

For script to have access to standard input, `nu` should be invoked with `--stdin` flag:

```nu
#!/usr/bin/env -S nu --stdin
def main [] {
  echo $"stdin: ($in)"
}
```

```nu
echo "Hello World!" | ./myscript
# => stdin: Hello World!
```
# Sorting

Nushell offers many ways of sorting data, and which method you reach for will depend on the problem and what kind of data you're working with. Let's take a look at some of the ways you might wish to sort data.

## Basic sorting

### Lists

Sorting a basic list works exactly how you might expect:

```nu
[9 3 8 1 4 6] | sort
# => ╭───┬───╮
# => │ 0 │ 1 │
# => │ 1 │ 3 │
# => │ 2 │ 4 │
# => │ 3 │ 6 │
# => │ 4 │ 8 │
# => │ 5 │ 9 │
# => ╰───┴───╯
```

However, things get a bit more complex when you start combining types. For example, let's see what happens when we have a list with numbers _and_ strings:

```nu
["hello" 4 9 2 1 "foobar" 8 6] | sort
# => ╭───┬────────╮
# => │ 0 │      1 │
# => │ 1 │      2 │
# => │ 2 │      4 │
# => │ 3 │      6 │
# => │ 4 │      8 │
# => │ 5 │      9 │
# => │ 6 │ foobar │
# => │ 7 │ hello  │
# => ╰───┴────────╯
```

We can see that the numbers are sorted in order, and the strings are sorted to the end of the list, also in order. If you are coming from other programming languages, this may not be quite what you expect. In Nushell, as a general rule, **data can always be sorted without erroring**.

::: tip
If you _do_ want a sort containing differing types to error, see [strict sort](#strict-sort).
:::

Nushell's sort is also **stable**, meaning equal values will retain their original ordering relative to each other. This is illustrated here using the [case insensitive](#case-insensitive-sort) sort option:

```nu
["foo" "FOO" "BAR" "bar"] | sort -i
# => ╭───┬─────╮
# => │ 0 │ BAR │
# => │ 1 │ bar │
# => │ 2 │ foo │
# => │ 3 │ FOO │
# => ╰───┴─────╯
```

Since this sort is case insensitive, `foo` and `FOO` are considered equal to each other, and the same is true for `bar` and `BAR`. In the result, the uppercase `BAR` precedes the lowercase `bar`, since the uppercase `BAR` also precedes the lowercase `bar` in the input. Similarly, the lowercase `foo` precedes the uppercase `FOO` in both the input and the result.

### Records

Records can be sorted two ways: by key, and by value. By default, passing a record to `sort` will sort in order of its keys:

```nu
{x: 123, a: hello!, foo: bar} | sort
# => ╭─────┬────────╮
# => │ a   │ hello! │
# => │ foo │ bar    │
# => │ x   │ 123    │
# => ╰─────┴────────╯
```

To instead sort in order of values, use the `-v` flag:

```nu
{x: 123, a: hello! foo: bar} | sort -v
# => ╭─────┬────────╮
# => │ x   │ 123    │
# => │ foo │ bar    │
# => │ a   │ hello! │
# => ╰─────┴────────╯
```

### Tables

Table rows are sorted by comparing rows by the columns in order. If two rows have equal values in their first column, they are sorted by their second column. This repeats until the rows are sorted different or all columns are equal.

```nu
let items = [
    {id: 100, quantity: 10, price: 5 }
    {id: 100, quantity: 5,  price: 8 }
    {id: 100, quantity: 5,  price: 1 }
]
$items | sort
# => ╭───┬─────┬──────────┬───────╮
# => │ # │ id  │ quantity │ price │
# => ├───┼─────┼──────────┼───────┤
# => │ 0 │ 100 │        5 │     1 │
# => │ 1 │ 100 │        5 │     8 │
# => │ 2 │ 100 │       10 │     5 │
# => ╰───┴─────┴──────────┴───────╯
```

In this example, the `id` column for all items is equal. Then, the two items with price `5` are sorted before the item with price `10`. Finally, the `item` with quantity `1` is sorted before the item with quantity `8`.

## Sorting structured data

### Cell path

In order to sort more complex types, such as tables, you can use the `sort-by` command. `sort-by` can order its input by a [cell path](navigating_structured_data.html#cell-paths).

Here's an example directory, sorted by filesize:

```nu
ls | sort-by size
# => ╭───┬─────────────────────┬──────┬──────────┬────────────────╮
# => │ # │        name         │ type │   size   │    modified    │
# => ├───┼─────────────────────┼──────┼──────────┼────────────────┤
# => │ 0 │ my-secret-plans.txt │ file │    100 B │ 10 minutes ago │
# => │ 1 │ shopping_list.txt   │ file │    100 B │ 2 months ago   │
# => │ 2 │ myscript.nu         │ file │  1.1 KiB │ 2 weeks ago    │
# => │ 3 │ bigfile.img         │ file │ 10.0 MiB │ 3 weeks ago    │
# => ╰───┴─────────────────────┴──────┴──────────┴────────────────╯
```

We can also provide multiple cell paths to `sort-by`, which will sort by each cell path in order of priority. You can think of providing multiple cell paths as a "tiebreaker" for elements which have equal values. Let's sort first by size, then by modification time:

```nu
ls | sort-by size modified
# => ╭───┬─────────────────────┬──────┬──────────┬────────────────╮
# => │ # │        name         │ type │   size   │    modified    │
# => ├───┼─────────────────────┼──────┼──────────┼────────────────┤
# => │ 0 │ shopping_list.txt   │ file │    100 B │ 2 months ago   │
# => │ 1 │ my-secret-plans.txt │ file │    100 B │ 10 minutes ago │
# => │ 2 │ myscript.nu         │ file │  1.1 KiB │ 2 weeks ago    │
# => │ 3 │ bigfile.img         │ file │ 10.0 MiB │ 3 weeks ago    │
# => ╰───┴─────────────────────┴──────┴──────────┴────────────────╯
```

This time, `shopping_list.txt` comes before `my-secret-plans.txt`, since it has an earlier modification time, but two larger files remain sorted after the `.txt` files.

Furthermore, we can use more complex cell paths to sort nested data:

```nu
let cities = [
    {name: 'New York', info: { established: 1624, population: 18_819_000 } }
    {name: 'Kyoto', info: { established: 794, population: 37_468_000 } }
    {name: 'São Paulo', info: { established: 1554, population: 21_650_000 } }
]
$cities | sort-by info.established
# => ╭───┬───────────┬────────────────────────────╮
# => │ # │   name    │            info            │
# => ├───┼───────────┼────────────────────────────┤
# => │ 0 │ Kyoto     │ ╭─────────────┬──────────╮ │
# => │   │           │ │ established │ 794      │ │
# => │   │           │ │ population  │ 37468000 │ │
# => │   │           │ ╰─────────────┴──────────╯ │
# => │ 1 │ São Paulo │ ╭─────────────┬──────────╮ │
# => │   │           │ │ established │ 1554     │ │
# => │   │           │ │ population  │ 21650000 │ │
# => │   │           │ ╰─────────────┴──────────╯ │
# => │ 2 │ New York  │ ╭─────────────┬──────────╮ │
# => │   │           │ │ established │ 1624     │ │
# => │   │           │ │ population  │ 18819000 │ │
# => │   │           │ ╰─────────────┴──────────╯ │
# => ╰───┴───────────┴────────────────────────────╯
```

### Sort by key closure

Sometimes, it's useful to sort data in a more complicated manner than "increasing" or "decreasing". Instead of using `sort-by` with a cell path, you can supply a [closure](types_of_data.html#closures), which will transform each value into a [sorting key](https://en.wikipedia.org/wiki/Collation#Sort_keys) _without changing the underlying data_. Here's an example of a key closure, where we want to sort a list of assignments by their average grade:

```nu
let assignments = [
    {name: 'Homework 1', grades: [97 89 86 92 89] }
    {name: 'Homework 2', grades: [91 100 60 82 91] }
    {name: 'Exam 1', grades: [78 88 78 53 90] }
    {name: 'Project', grades: [92 81 82 84 83] }
]
$assignments | sort-by { get grades | math avg }
# => ╭───┬────────────┬───────────────────────╮
# => │ # │    name    │        grades         │
# => ├───┼────────────┼───────────────────────┤
# => │ 0 │ Exam 1     │ [78, 88, 78, 53, 90]  │
# => │ 1 │ Project    │ [92, 81, 82, 84, 83]  │
# => │ 2 │ Homework 2 │ [91, 100, 60, 82, 91] │
# => │ 3 │ Homework 1 │ [97, 89, 86, 92, 89]  │
# => ╰───┴────────────┴───────────────────────╯
```

The value is passed into the pipeline input of the key closure, however, you can also use it as a parameter:

```nu
let weight = {alpha: 10, beta: 5, gamma: 3}
[alpha gamma beta gamma alpha] | sort-by {|val| $weight | get $val }
# => ╭───┬───────╮
# => │ 0 │ gamma │
# => │ 1 │ gamma │
# => │ 2 │ beta  │
# => │ 3 │ alpha │
# => │ 4 │ alpha │
# => ╰───┴───────╯
```

### Custom sort order

In addition to [key closures](#sort-by-key-closure), `sort-by` also supports closures which specify a custom sort order. The `--custom`, or `-c`, flag will tell `sort-by` to interpret closures as custom sort closures. A custom sort closure has two parameters, and returns a boolean. The closure should return `true` if the first parameter comes _before_ the second parameter in the sort order.

For a simple example, we could rewrite a cell path sort as a custom sort. This can be read as "If $a.size is less than $b.size, a should appear before b in the sort order":

```nu
ls | sort-by -c {|a, b| $a.size < $b.size }
# => ╭───┬─────────────────────┬──────┬──────────┬────────────────╮
# => │ # │        name         │ type │   size   │    modified    │
# => ├───┼─────────────────────┼──────┼──────────┼────────────────┤
# => │ 0 │ my-secret-plans.txt │ file │    100 B │ 10 minutes ago │
# => │ 1 │ shopping_list.txt   │ file │    100 B │ 2 months ago   │
# => │ 2 │ myscript.nu         │ file │  1.1 KiB │ 2 weeks ago    │
# => │ 3 │ bigfile.img         │ file │ 10.0 MiB │ 3 weeks ago    │
# => ╰───┴─────────────────────┴──────┴──────────┴────────────────╯
```

::: tip
The parameters are also passed to the custom closure as a two element list, so the following are equivalent:

- `{|a, b| $a < $b }`
- `{ $in.0 < $in.1 }`
  :::

Here's an example of a custom sort which couldn't be trivially written as a key sort. In this example, we have a queue of tasks with some amount of work time and a priority. We want to sort by priority (highest first). If a task has had zero work time, we want to schedule it immediately; otherwise, we ignore the work time.

```nu
let queue = [
    {task: 139, work_time: 0,   priority: 1 }
    {task: 52,  work_time: 355, priority: 8 }
    {task: 948, work_time: 72,  priority: 2 }
    {task: 583, work_time: 0,   priority: 5 }
]
let my_sort = {|a, b|
    match [$a.work_time, $b.work_time] {
        [0, 0] => ($a.priority > $b.priority) # fall back to priority if equal work time
        [0, _] => true, # only a has 0 work time, so a comes before b in the sort order
        [_, 0] => false, # only b has 0 work time, so a comes after b in the sort order
        _ => ($a.priority > $b.priority) # both have non-zero work time, sort by priority
    }
}
$queue | sort-by -c $my_sort
```

## Special sorts

### Case insensitive sort

When using case insensitive sort, strings (and globs) which are the same except for different casing will be considered equal for sorting, while other types remain unaffected:

```nu
let data = [
    Nushell,
    foobar,
    10,
    nushell,
    FoOBaR,
    9
]
$data | sort -i
# => ╭───┬─────────╮
# => │ 0 │       9 │
# => │ 1 │      10 │
# => │ 2 │ foobar  │
# => │ 3 │ FoOBaR  │
# => │ 4 │ Nushell │
# => │ 5 │ nushell │
# => ╰───┴─────────╯
```

### Natural sort

The [natural sort option](https://en.wikipedia.org/wiki/Natural_sort_order) allows strings which contain numbers to be sorted in the same way that numbers are normally sorted. This works both for strings which consist solely of numbers, and strings which have numbers and letters:

```nu
let data = ["10", "9", "foo123", "foo20", "bar123", "bar20"]
$data | sort
# => ╭───┬────────╮
# => │ 0 │ 10     │
# => │ 1 │ 9      │
# => │ 2 │ bar123 │
# => │ 3 │ bar20  │
# => │ 4 │ foo123 │
# => │ 5 │ foo20  │
# => ╰───┴────────╯
# "1" is sorted before "9", so "10" is sorted before "9"
$data | sort -n
# => ╭───┬────────╮
# => │ 0 │ 9      │
# => │ 1 │ 10     │
# => │ 2 │ bar20  │
# => │ 3 │ bar123 │
# => │ 4 │ foo20  │
# => │ 5 │ foo123 │
# => ╰───┴────────╯
```

Furthermore, natural sort allows you to sort numbers together with numeric strings:

```nu
let data = [4, "6.2", 1, "10", 2, 8.1, "3", 5.5, "9", 7]
$data | sort -n
# => ╭───┬──────╮
# => │ 0 │    1 │
# => │ 1 │    2 │
# => │ 2 │ 3    │
# => │ 3 │    4 │
# => │ 4 │ 5.50 │
# => │ 5 │ 6.2  │
# => │ 6 │    7 │
# => │ 7 │ 8.10 │
# => │ 8 │ 9    │
# => │ 9 │ 10   │
# => ╰───┴──────╯
```

### Sorting with mixed types

Under some circumstances, you might need to sort data containing mixed types. There are a couple things to be aware of when sorting mixed types:

- Generally, values of the same type will appear next to each other in the sort order. For example, sorted numbers come first, then sorted strings, then sorted lists.
- Some types will be intermixed in the sort order. These are:
  - Integers and floats. For example, `[2.2, 1, 3]` will be sorted as `[1, 2.2, 3]`.
  - Strings and globs. For example, `[("b" | into glob) a c]` will be sorted as `[a b c]` (where b is still a glob).
  - If using [natural sort](#natural-sort), integers, floats, and strings will be intermixed as described in that section.
- The ordering between non-intermixed types is not guaranteed, **except** for `null` values, which will always be sorted to the end of a list.
  - Within the same Nushell version the ordering should always be the same, but this should not be relied upon. If you have code which is sensitive to the ordering across types, consider using a [custom sort](#custom-sort-order) which better expresses your requirements.

If you need to sort data which may contain mixed types, consider one of the following strategies:

- [Strict sort](#strict-sort) to disallow sorting of incompatible types
- [Natural sort](#natural-sort) to sort intermixed numbers and numeric strings
- A [key sort](#sort-by-key-closure) using [`to text`](/commands/docs/to_text.html), [`to nuon`](/commands/docs/to_nuon.html), or [`to json`](/commands/docs/to_json.html), as appropriate
- A [custom sort](#custom-sort-order) using [`describe`](/commands/docs/describe.html) to explicitly check types

### Strict sort

Custom sort closures also provide a simple way to sort data while ensuring only types with well-defined comparisons are sorted together. This takes advantage of [operators requiring compatible data types](operators.html#types):

```nu
let compatible = [8 3.2 null 58 2]
let incompatible = ["hello" 4 9 2 1 "meow" 8 6]
$compatible | sort-by -c {|a, b| $a < $b | default ($a != null) }
# => ╭───┬──────╮
# => │ 0 │    2 │
# => │ 1 │ 3.20 │
# => │ 2 │    8 │
# => │ 3 │   58 │
# => │ 4 │      │
# => ╰───┴──────╯
$incompatible | sort-by -c {|a, b| $a < $b | default ($a != null) }
# => Error: nu::shell::type_mismatch
# => 
# =>   × Type mismatch during operation.
# =>    ╭─[entry #26:1:36]
# =>  1 │ $incompatible | sort-by -c {|a, b| $a < $b | default ($a != null) }
# =>    ·                                    ─┬ ┬ ─┬
# =>    ·                                     │ │  ╰── string
# =>    ·                                     │ ╰── type mismatch for operator
# =>    ·                                     ╰── int
# =>    ╰────
```

Special handling is required for `null` values, since comparison between any value and `null` returns `null`. To instead reject `null` values, try the following:

```nu
let baddata = [8 3.2 null 58 2]
let strict = {|a, b|
    match [$a, $b] {
        [null, _] => (error make {msg: "Attempt to sort null"}),
        [_, null] => (error make {msg: "Attempt to sort null"}),
        _ => ($a < $b)
    }
}
$baddata | sort-by -c $strict
# => Error:   × Attempt to sort null
# =>    ╭─[entry #3:4:21]
# =>  3 │   match [$a, $b] {
# =>  4 │       [null, _] => (error make {msg: "Attempt to sort null"}),
# =>    ·                     ─────┬────
# =>    ·                          ╰── originates from here
# =>  5 │       [_, null] => (error make {msg: "Attempt to sort null"}),
# =>    ╰────
```
# Standard Library (Preview)

Nushell ships with a standard library of useful commands written in native Nu. By default, the standard library is loaded into memory (but not automatically imported) when Nushell starts.

[[toc]]

## Overview

The standard library currently includes:

- Assertions
- An alternative `help` system with support for completions.
- Additional JSON variant formats
- XML Access
- Logging
- And more

To see a complete list of the commands available in the standard library, run the following:

```nu
nu -c "
  use std
  scope commands
  | where name =~ '^std '
  | select name description extra_description
  | wrap 'Standard Library Commands'
  | table -e
"
```

::: note
The `use std` command above loads the entire standard library so that you can see all of the commands at once. This is typically not how it will be used (more info below). It is also run in a separate Nu subshell simply so that it is not loaded into scope in the shell you are using.
:::

## Importing the Standard Library

The Standard Library modules and submodules are imported with the [`use`](/commands/docs/use.md) command, just as any other module. See [Using Modules](./modules/using_modules.md) for more information.

While working at the commandline, it can be convenient to load the entire standard library using:

```nu
use std *
```

However, this form should be avoided in custom commands and scripts since it has the longest load time.

::: important Optimal Startup when Using the Standard Library
See the [notes below](#optimal-startup) on how to ensure that your configuration isn't loading the entire Standard Library.
:::

### Importing Submodules

Each submodule of the standard library can be loaded separately. Again, _for best performance, load only the submodule(s) that you need in your code._

See [Importing Modules](./modules/using_modules.md#importing-modules) for general information on using modules. The recommended import for each of the Standard Library submodules is listed below:

#### 1. Submodules with `<command> <subcommand>` form

These submodules are normally imported with `use std/<submodule>` (without a glob/`*`):

- `use std/assert`: `assert` and its subcommands
- `use std/bench`: The benchmarking command `bench`
- `use std/dirs`: The directory stack command `dirs` and its subcommands
- `use std/input`: The `input display` command
- `use std/help`: An alternative version of the `help` command and its subcommands which supports completion and other features
- `use std/iters`: Additional `iters`-prefixed iteration commands.
- `use std/log`: The `log <subcommands>` such as `log warning <msg>`
- `use std/math`: Mathematical constants such as `$math.E`. These can also be imported as definitions as in Form #2 below.

#### 2. Import the _definitions_ (contents) of the module directly

Some submodules are easier to use when their definitions (commands, aliases, constants, etc.) are loaded into the current scope. For instance:

```nu
use std/formats *
ls | to jsonl
```

Submodules that are normally imported with `use std/<submodule> *` (**with** a glob/`*`):

- `use std/dt *`: Additional commands for working with `date` values
- `use std/formats *`: Additional `to` and `from` format conversions
- `use std/math *`: The math constants without a prefix, such as `$E`. Note that the prefixed form #1 above is likely more understandable when reading and maintaining code.
- `use std/xml *`: Additional commands for working with XML data

#### 3. `use std <submodule>`

It is _possible_ to import Standard Library submodules using a space-separated form:

```nu
use std log
use std formats *
```

::: important
As mentioned in [Using Modules](./modules/using_modules.md#module-definitions), this form (like `use std *`) first loads the _entire_ Standard Library into scope and _then_ imports the submodules. In contrast, the slash-separated versions in #1 and #2 above _only_ import the submodule and will be much faster as a result.
:::

## The Standard Library Candidate Module

(Also known as `std-rfc`)

`stdlib-candidate`, found in the [nu_scripts Repository](https://github.com/nushell/nu_scripts/tree/main/stdlib-candidate/std-rfc), serves as a staging ground for possible Standard Library additions.

If you are interested in adding to the Standard Library, please submit your code via PR to the Candidate module in that repository. We also encourage you to install this module and provide feedback on upcoming candidate commands.

::: details More details

Candidate commands for the Standard Library should, in general:

- Have broad appeal - Be useful to a large number of users or use cases
- Be well-written and clearly commented for future maintainers
- Implement help comments with example usage
- Have a description that explains why you feel the command should be a part of the standard library. Think of this as an "advertisement" of sorts to convince people to try the command and provide feedback so that it can be promoted in the future.

In order for a command to be graduated from RFC to the Standard Library, it must have:

- Positive feedback
- Few (or no) outstanding issues and, of course, no significant issues
- A PR author for the `std` submission. This does not necessarily have to be the original author of the command.
- Test cases as part of the `std` submission PR

Ultimately a member of the core team will decide when and if to merge the command into `std` based on these criteria.

Of course, if a candidate command in `std-rfc` no longer works or has too many issues, it may be removed from or disabled in `std-rfc`.

:::

## Disabling the Standard Library

To disable the standard library, you can start Nushell using:

```nu
nu --no-std-lib
```

This can be especially useful to minimize overhead when running a command in a subshell using `nu -c`. For example:

```nu
nu --no-std-lib -n -c "$nu.startup-time"
# => 1ms 125µs 10ns

nu -n -c "$nu.startup-time"
# => 4ms 889µs 576ns
```

You will not be able to import the library, any of its submodules, nor use any of its commands, when it is disabled in this way.

## Using `std/log` in Modules

::: warning Important!
`std/log` exports environment variables. To use the `std/log` module in your own module, please see [this caveat](./modules/creating_modules.md#export-env-runs-only-when-the-use-call-is-evaluated) in the "Creating Modules" Chapter.

:::

## Optimal Startup

If Nushell's startup time is important to your workflow, review your [startup configuration]([./configuration.md]) in `config.nu`, `env.nu`, and potentially others for inefficient use of the standard library. The following command should identify any problem areas:

```nu
view files
| enumerate | flatten
| where filename !~ '^std'
| where filename !~ '^entry'
| where {|file|
    (view span $file.start $file.end) =~ 'use\W+std[^\/]'
  }
```

Edit those files to use the recommended syntax in the [Importing Submodules](#importing-submodules) section above.

::: note
If a Nushell library (e.g., from [the `nu_scripts` repository](https://github.com/nushell/nu_scripts)), example, or doc is using this syntax, please report it via an issue or PR. These will be updated over time after Nushell 0.99.0 is released.

If a third-party module is using this syntax, please report it to the author/maintainers to update.
:::
# Stdout, Stderr, and Exit Codes

An important piece of interop between Nushell and external commands is working with the standard streams of data coming from the external.

The first of these important streams is stdout.

## Stdout

Stdout is the way that most external apps will send data into the pipeline or to the screen. Data sent by an external app to its stdout is received by Nushell by default if it's part of a pipeline:

```nu
external | str join
```

The above would call the external named `external` and would redirect the stdout output stream into the pipeline. With this redirection, Nushell can then pass the data to the next command in the pipeline, here [`str join`](/commands/docs/str_join.md).

Without the pipeline, Nushell will not do any redirection, allowing it to print directly to the screen.

## Stderr

Another common stream that external applications often use to print error messages is stderr. By default, Nushell does not do any redirection of stderr, which means that by default it will print to the screen.

## Exit Code

Finally, external commands have an "exit code". These codes help give a hint to the caller whether the command ran successfully.

Nushell tracks the last exit code of the recently completed external in one of two ways. The first way is with the `LAST_EXIT_CODE` environment variable.

```nu
do { external }
$env.LAST_EXIT_CODE
```

The second way is to use the [`complete`](/commands/docs/complete.md) command.

## Using the [`complete`](/commands/docs/complete.md) command

The [`complete`](/commands/docs/complete.md) command allows you to run an external to completion, and gather the stdout, stderr, and exit code together in one record.

If we try to run the external `cat` on a file that doesn't exist, we can see what [`complete`](/commands/docs/complete.md) does with the streams, including the redirected stderr:

```nu
cat unknown.txt | complete
# => ╭───────────┬─────────────────────────────────────────────╮
# => │ stdout    │                                             │
# => │ stderr    │ cat: unknown.txt: No such file or directory │
# => │ exit_code │ 1                                           │
# => ╰───────────┴─────────────────────────────────────────────╯
```

## `echo`, `print`, and `log` commands

The [`echo`](/commands/docs/echo.md) command is mainly for _pipes_. It returns its arguments, ignoring the piped-in value. There is usually little reason to use this over just writing the values as-is.

In contrast, the [`print`](/commands/docs/print.md) command prints the given values to stdout as plain text. It can be used to write to standard error output, as well. Unlike [`echo`](/commands/docs/echo.md), this command does not return any value (`print | describe` will return "nothing"). Since this command has no output, there is no point in piping it with other commands.

The [standard library](/book/standard_library.md) has commands to write out messages in different logging levels. For example:

@[code](@snippets/book/std_log.nu)

![Log message examples](../assets/images/0_79_std_log.png)

The log level for output can be set with the `NU_LOG_LEVEL` environment variable:

```nu
NU_LOG_LEVEL=DEBUG nu std_log.nu
```

## File Redirections

If you want to redirect stdout of an external command to a file, you can use `out>` followed by a file path. Similarly, you can use `err>` to redirect stderr:

```nu
cat unknown.txt out> out.log err> err.log
```

If you want to redirect both stdout and stderr to the same file, you can use `out+err>`:

```nu
cat unknown.txt out+err> log.log
```

Note that `out` can be shortened to just `o`, and `err` can be shortened to just `e`. So, the following examples are equivalent to the previous ones above:
```nu
cat unknown.txt o> out.log e> err.log

cat unknown.txt o+e> log.log
```

Also, any expression can be used for the file path, as long as it is a string value:
```nu
use std
cat unknown.txt o+e> (std null-device)
```

Note that file redirections are scoped to an expression and apply to all external commands in the expression. In the example below, `out.txt` will contain `hello\nworld`:
```nu
let text = "hello\nworld"
($text | head -n 1; $text | tail -n 1) o> out.txt
```
Pipes and additional file redirections inside the expression will override any file redirections applied from the outside.

## Pipe Redirections

If a regular pipe `|` comes after an external command, it redirects the stdout of the external command as input to the next command. To instead redirect the stderr of the external command, you can use the stderr pipe, `err>|` or `e>|`:

```nu
cat unknown.txt e>| str upcase
```

Of course, there is a corresponding pipe for combined stdout and stderr, `out+err>|` or `o+e>|`:

```nu
nu -c 'print output; print -e error' o+e>| str upcase
```

Unlike file redirections, pipe redirections do not apply to all commands inside an expression. Rather, only the last command in the expression is affected. For example, only `cmd2` in the snippet below will have its stdout and stderr redirected by the pipe.
```nu
(cmd1; cmd2) o+e>| cmd3
```

## Raw Streams

Both stdout and stderr are represented as "raw streams" inside of Nushell. These are streams of bytes rather than the structured data used by internal Nushell commands.

Because streams of bytes can be difficult to work with, especially given how common it is to use output as if it was text data, Nushell attempts to convert raw streams into text data. This allows other commands to pull on the output of external commands and receive strings they can further process.

Nushell attempts to convert to text using UTF-8. If at any time the conversion fails, the rest of the stream is assumed to always be bytes.

If you want more control over the decoding of the byte stream, you can use the [`decode`](/commands/docs/decode.md) command. The [`decode`](/commands/docs/decode.md) command can be inserted into the pipeline after the external, or other raw stream-creating command, and will handle decoding the bytes based on the argument you give decode. For example, you could decode shift-jis text this way:

```nu
0x[8a 4c] | decode shift-jis
# => 貝
```
# Best Practices

This page is a working document collecting syntax guidelines and best practices we have discovered so far.
The goal of this document is to eventually land on a canonical Nushell code style, but as for now it is still work in
progress and subject to change. We welcome discussion and contributions.

Keep in mind that these guidelines are not required to be used in external repositories (not ours), you can change them in the
way you want, but please be consistent and follow your rules.

All escape sequences should not be interpreted literally, unless directed to do so. In other words,
treat something like `\n` like the new line character and not a literal slash followed by `n`.

## Formatting

### Defaults

**It's recommended to** assume that by default no spaces or tabs allowed, but the following rules define where they are allowed.

### Basic

- **It's recommended to** put one space before and after pipe `|` symbol, commands, subcommands, their options and arguments.
- **It's recommended to** never put several consecutive spaces unless they are part of string.
- **It's recommended to** omit commas between list items.

Correct:

```nu
'Hello, Nushell! This is a gradient.' | ansi gradient --fgstart '0x40c9ff' --fgend '0xe81cff'
```

Incorrect:

```nu
# - too many spaces after "|": 2 instead of 1
'Hello, Nushell! This is a gradient.' |  ansi gradient --fgstart '0x40c9ff' --fgend '0xe81cff'
```

#### One-line Format

One-line format is a format for writing all commands in one line.

**It's recommended to** default to this format:

1. unless you are writing scripts
2. in scripts for lists and records unless they either:
   1. more than 80 characters long
   2. contain nested lists or records
3. for pipelines less than 80 characters long not containing items should be formatted with
   a long format

Rules:

1. parameters:
   1. **It's recommended to** put one space after comma `,` after block or closure parameter.
   2. **It's recommended to** put one space after pipe `|` symbol denoting block or closure parameter list end.
2. block and closure bodies:
   1. **It's recommended to** put one space after opening block or closure curly brace `{` if no explicit parameters defined.
   2. **It's recommended to** put one space before closing block or closure curly brace `}`.
3. records:
   1. **It's recommended to** put one space after `:` after record key.
   2. **It's recommended to** put one space after comma `,` after key value.
4. lists:
   1. **It's recommended to** put one space after comma `,` after list value.
5. surrounding constructs:
   1. **It's recommended to** put one space before opening square `[`, curly brace `{`, or parenthesis `(` if preceding symbol is not the same.
   2. **It's recommended to** put one space after closing square `]`, curly brace `}`, or parenthesis `)` if following symbol is not the same.

Correct:

```nu
[[status]; [UP] [UP]] | all {|el| $el.status == UP }
[1 2 3 4] | reduce {|elt, acc| $elt + $acc }
[1 2 3 4] | reduce {|elt acc| $elt + $acc }
{x: 1, y: 2}
{x: 1 y: 2}
[1 2] | zip [3 4]
[]
(1 + 2) * 3
```

Incorrect:

```nu
# too many spaces before "|el|": no space is allowed
[[status]; [UP] [UP]] | all { |el| $el.status == UP }

# too many spaces before ",": no space is allowed
[1 2 3 4] | reduce {|elt , acc| $elt + $acc }

# too many spaces before "x": no space is allowed
{ x: 1, y: 2}

# too many spaces before "[3": one space is required
[1 2] | zip  [3 4]

# too many spaces before "]": no space is allowed
[ ]

# too many spaces before ")": no space is allowed
(1 + 2 ) * 3
```

#### Multi-line Format

Multi-line format is a format for writing all commands in several lines. It inherits all rules from one-line format
and modifies them slightly.

**It's recommended to** default to this format:

1. while you are writing scripts
2. in scripts for lists and records while they either:
   1. more than 80 characters long
   2. contain nested lists or records
3. for pipelines more 80 characters long

Rules:

1. general:
   1. **It's required to omit** trailing spaces.
2. block and closure bodies:
   1. **It's recommended to** put each body pipeline on a separate line.
3. records:
   1. **It's recommended to** put each record key-value pair on separate line.
4. lists:
   1. **It's recommended to** put each list item on separate line.
5. surrounding constructs:
   1. **It's recommended to** put one `\n` before opening square `[`, curly brace `{`, or parenthesis `(` if preceding symbol is not the and applying this rule produce line with a singular parenthesis.
   2. **It's recommended to** put one `\n` after closing square `]`, curly brace `}`, or parenthesis `)` if following symbol is not the same and applying this rule produce line with a singular parenthesis.

Correct:

```nu
[[status]; [UP] [UP]] | all {|el|
    $el.status == UP
}

[1 2 3 4] | reduce {|elt, acc|
    $elt + $acc
}

{x: 1, y: 2}

[
  {name: "Teresa", age: 24},
  {name: "Thomas", age: 26}
]

let selectedProfile = (for it in ($credentials | transpose name credentials) {
    echo $it.name
})
```

Incorrect:

```nu
# too many spaces before "|el|": no space is allowed (like in one-line format)
[[status]; [UP] [UP]] | all { |el|
    # too few "\n" before "}": one "\n" is required
    $el.status == UP}

# too many spaces before "2": one space is required (like in one-line format)
[1  2 3 4] | reduce {|elt, acc|
    $elt + $acc
}

{
   # too many "\n" before "x": one-line format required as no nested lists or record exist
   x: 1,
   y: 2
}

# too few "\n" before "{": multi-line format required as there are two nested records
[{name: "Teresa", age: 24},
  {name: "Thomas", age: 26}]

let selectedProfile = (
    # too many "\n" before "foo": no "\n" is allowed
    for it in ($credentials | transpose name credentials) {
        echo $it.name
})
```

## Naming Convention

### Abbreviations and Acronyms

**It's recommended** to use full concise words over abbreviations and acronyms, unless they are well-known and/or
commonly used.

Correct:

```nu
query-user --id 123

$user.name | str downcase
```

Incorrect:

```nu
qry-usr --id 123

$user.name | string downcase
```

### Case

#### Commands

**It's recommended to** use kebab-case for command names with multiple words.

Correct:

```nu
fetch-user --id 123
```

Incorrect:

```nu
fetch_user --id 123
fetchUser --id 123
```

See also [Naming Commands](custom_commands.md#naming-commands).

#### Sub-Commands

Sub commands are commands that are logically grouped under a parent command and separated by a space.
**It's recommended to** use kebab-case for the sub-command name.

Correct:

```nu
date now

date list-timezone

def "login basic-auth" [username: string password: string] {
    # ...
}
```

See also [Naming Sub-Commands](custom_commands.md#subcommands).

#### Flags

**It's recommended to** use kebab-case for flag names.

Correct:

```nu
def greet [name: string, --all-caps] {
    # ...
}
```

Incorrect:

```nu
def greet [name: string, --all_caps] {
    # ...
}
```

::: tip
Notice that the name used to access the flag is accessed by replacing the dash with an underscore in the resulting
variable name.

See [Flags](custom_commands.md#flags).
:::

#### Variables and Command Parameters

**It's recommended to** use snake_case for variable names, including command parameters.

Correct:

```nu
let user_id = 123

def fetch-user [user_id: int] {
  # ...
}
```

Incorrect:

```nu
let user-id = 123
let userId = 123

def fetch-user [user-id: int] {
  # ...
}
```

#### Environment Variables

**It's recommended to** use SCREAMING_SNAKE_CASE for environment variable names.

Correct:

```nu
$env.ENVIRONMENT_CODE = "prod"

$env.APP_VERSION = "1.0.0"
```

Incorrect:

```nu
$env.ENVIRONMENT-CODE = "prod"

$env.app_version = "1.0.0"
```

## Options and Parameters of Custom Commands

- **It's recommended to** keep count of all positional parameters less than or equal to 2, for remaining inputs use options. Assume that command can expect source and destination parameter, like `mv`: source and target file or directory.
- **It's recommended to** use positional parameters unless they can't be used due to rules listed here or technical restrictions.
  For instance, when there are several kinds of optional parameters (but at least one parameter should be provided)
  use options. Great example of this is `ansi gradient` command where at least foreground or background must be passed.
- **It's recommended to** provide both long and short options.

## Documentation

- **It's recommended to** provide documentation for all exported entities (like custom commands) and their
  inputs (like custom command parameters and options).
# Table of Contents

- [Installation](installation.md) - Installing Nushell
- [Introduction](README.md) - Getting started
- [Thinking in Nu](thinking_in_nu.md) - Thinking in Nushell
- [Moving around](moving_around.md) - Moving around in Nushell
- [Types of data](types_of_data.md) - Types of data in Nushell
- [Loading data](loading_data.md) - Loading data and using it
- [Strings](working_with_strings.md) - Strings, escape characters, and string interpolation
- [Working with lists](working_with_lists.md) - Working with Nu lists
- [Working with tables](working_with_tables.md) - Working with Nu tables
- [Pipelines](pipelines.md) - How the pipeline works
- [Configuration](configuration.md) - How to configure Nushell
- [3rd Party Prompts](3rdpartyprompts.md) - How to configure 3rd party prompts
- [Custom commands](custom_commands.md) - Creating your own commands
- [Aliases](aliases.md) - How to alias commands
- [Operators](operators.md) - Operators supported by Nushell
- [Variables](variables.md) - Working with variables
- [Control flow](control_flow.md) - Working with the control flow commands
- [Environment](environment.md) - Working with environment variables
- [Stdout, stderr, and exit codes](stdout_stderr_exit_codes.md) - Working with stdout, stderr, and exit codes
- [Modules](modules.md) - Creating and using your own modules
- [Hooks](hooks.md) - Adding code snippets to be run automatically
- [Scripts](scripts.md) - Creating your own scripts
- [Metadata](metadata.md) - An explanation of Nu's metadata system
- [Creating your own errors](creating_errors.md) - Creating your own error messages
- [Directory Stack](directory_stack.md) - Working with multiple locations
- [Running External (System) Commands](./running_externals.md) - Running external commands with a naming conflict
- [Plugins](plugins.md) - Enhancing Nushell with more features using plugins
- [Parallelism](parallelism.md) - Running your code in parallel
- [Line editor](line_editor.md) - Nushell's line editor
- [Dataframes](dataframes.md) - Working with dataframes in Nushell
- [Explore](explore.md) - Using the Nushell TUI
- [Coloring and Theming](coloring_and_theming.md) - How to change the colors and themes in Nushell
- [Regular Expressions](regular_expressions.md) - Guide to use regex
- [Coming from Bash](coming_from_bash.md) - Guide for those coming to Nushell from Bash
- [Nushell map from shells/DSL](nushell_map.md) - Guide to show how Nushell compares with SQL, LINQ, PowerShell, and Bash
- [Nushell map from imperative languages](nushell_map_imperative.md) - Guide to show how Nushell compares with Python, Kotlin, C++, C#, and Rust
- [Nushell map from functional languages](nushell_map_functional.md) - Guide to show how Nushell compares with Clojure, Tablecloth (OCaml / Elm) and Haskell
- [Nushell operator map](nushell_operator_map.md) - Guide to show how Nushell operators compare with those in general purpose programming languages
- [Command Reference](/commands/) - List of all Nushell's commands
# Testing your Nushell Code

## Assert Commands

Nushell provides a set of "assertion" commands in the standard library.
One could use built-in equality / order tests such as `==` or `<=` or more complex commands and throw errors manually when an expected condition fails, but using what the standard library has to offer is arguably easier!

In the following, it will be assumed that the `std assert` module has been imported inside the current scope
```nushell
use std assert
```

The foundation for every assertion is the `std assert` command. If the condition is not true, it makes an error.

```nu
assert (1 == 2)
```
```
Error:
  × Assertion failed.
   ╭─[entry #13:1:1]
 1 │ assert (1 == 2)
   ·         ───┬──
   ·            ╰── It is not true.
   ╰────
```

Optionally, a message can be set to show the intention of the assert command, what went wrong or what was expected:

```nu
let a = 0
assert ($a == 19) $"The lockout code is wrong, received: ($a)"
```
```
Error:
  × The lockout code is wrong, received: 13
   ╭─[entry #25:1:1]
 1 │ let a = 0
 2 │ assert ($a == 19) $"The lockout code is wrong, received: ($a)"
   ·         ────┬───
   ·             ╰── It is not true.
   ╰────
```

There are many assert commands, which behave exactly as the base one with the proper operator. The additional value for them is the ability for better error messages.

For example this is not so helpful without additional message:

```nu
let a = "foo"
let b = "bar"
assert ($b | str contains $a)
```
```
Error:   × Assertion failed.
   ╭─[entry #5:3:8]
 2 │ let b = "bar"
 3 │ assert ($b | str contains $a)
   ·        ───────────┬──────────
   ·                   ╰── It is not true.
   ╰────
```

While with using `assert str contains`:

```nu
let a = "a needle"
let b = "haystack"
assert str contains $b $a
```
```
Error:   × Assertion failed.
   ╭─[entry #7:3:21]
 2 │ let b = "bar"
 3 │ assert str contains $b $a
   ·                     ──┬──
   ·                       ╰─┤ This does not contain 'a needle'.
   ·                         │         value: "haystack"
   ╰────
```

In general for base `assert` command it is encouraged to always provide the additional message to show what went wrong. If you cannot use any built-in assert command, you can create a custom one with passing the label for [`error make`](/commands/docs/error_make.md) for the `assert` command:

```nu
def "assert even" [number: int] {
    assert ($number mod 2 == 0) --error-label {
        text: $"($number) is not an even number",
        span: (metadata $number).span,
    }
}
```

Then you'll have your detailed custom error message:

```nu
let $a = 13
assert even $a
```
```
Error:
  × Assertion failed.
   ╭─[entry #37:1:1]
 1 │ assert even $a
   ·             ─┬
   ·              ╰── 13 is not an even number
   ╰────
```

## Running the Tests

Now that we are able to write tests by calling commands from `std assert`, it would be great to be able to run them and see our tests fail when there is an issue and pass when everything is correct :)


### Nupm Package

In this first case, we will assume that the code you are trying to test is part of a [Nupm] package.

In that case, it is as easy as following the following steps
- create a `tests/` directory next to the `nupm.nuon` package file of your package
- make the `tests/` directory a valid module by adding a `mod.nu` file into it
- write commands inside `tests/`
- call `nupm test`

The convention is that any command fully exported from the `tests` module will be run as a test, e.g.
- `export def some-test` in `tests/mod.nu` will run
- `def just-an-internal-cmd` in `tests/mod.nu` will NOT run
- `export def another-test` in `tests/spam.nu` will run if and only if there is something like `export use spam.nu *` in `tests/mod.nu`


### Standalone Tests

If your Nushell script or module is not part of a [Nupm] package, the simplest way is to write tests in standalone scripts and then call them, either from a `Makefile` or in a CI:

Let's say we have a simple `math.nu` module which contains a simple Fibonacci command:
```nushell
# `fib n` is the n-th Fibonacci number
export def fib [n: int] [ nothing -> int ] {
    if $n == 0 {
        return 0
    } else if $n == 1 {
        return 1
    }

    (fib ($n - 1)) + (fib ($n - 2))
}
```
then a test script called `tests.nu` could look like
```nushell
use math.nu fib
use std assert

for t in [
    [input, expected];
    [0, 0],
    [1, 1],
    [2, 1],
    [3, 2],
    [4, 3],
    [5, 5],
    [6, 8],
    [7, 13],
] {
    assert equal (fib $t.input) $t.expected
}
```
and be invoked as `nu tests.nu`

### Basic Test Framework

It is also possible to define tests in Nushell as functions with descriptive names and discover
them dynamically without requiring a [Nupm] package. The following uses `scope commands` and a
second instance of Nushell to run the generated list of tests.

```nushell
use std assert

source fib.nu

def main [] {
    print "Running tests..."

    let test_commands = (
        scope commands
            | where ($it.type == "custom")
                and ($it.name | str starts-with "test ")
                and not ($it.description | str starts-with "ignore")
            | get name
            | each { |test| [$"print 'Running test: ($test)'", $test] } | flatten
            | str join "; "
    )

    nu --commands $"source ($env.CURRENT_FILE); ($test_commands)"
    print "Tests completed successfully"
}

def "test fib" [] {
    for t in [
        [input, expected];
        [0, 0],
        [1, 1],
        [2, 1],
        [3, 2],
        [4, 3],
        [5, 5],
        [6, 8],
        [7, 13]
    ] {
        assert equal (fib $t.input) $t.expected
    }
}

# ignore
def "test show-ignored-test" [] {
    print "This test will not be executed"
}
```

This is a simple example but could be extended to include many of the things you might expect from
a testing framework, including setup and tear down functions and test discovery across files.

[Nupm]: https://github.com/nushell/nupm
# Thinking in Nu

Nushell is different! It's common (and expected!) for new users to have some existing "habits" or mental models coming from other shells or languages.

The most common questions from new users typically fall into one of the following topics:

[[toc]]

## Nushell isn't Bash

### It can sometimes look like Bash

Nushell is both a programming language and a shell. Because of this, it has its own way of working with files, directories, websites, and more. You'll find that some features in Nushell work similar to those you're familiar with in other shells. For instance, pipelines work by combining two (or more) commands together, just like in other shells.

For example, the following commandline works the same in both Bash and Nushell on Unix/Linux platforms:

```nu
curl -s https://api.github.com/repos/nushell/nushell/contributors | jq '.[].login'
# => returns contributors to Nushell, ordered by number of contributions
```

Nushell has many other similarities with Bash (and other shells) and many commands in common.

::: tip
Bash is primarily a command interpreter which runs external commands. Nushell provides many of these as cross-platform, built-in commands.

While the above commandline works in both shells, in Nushell there's just no need to use the `curl` and `jq` commands. Instead, Nushell has a built-in [`http get` command](/commands/docs/http_get.md) and handles JSON data natively. For example:

```nu
http get https://api.github.com/repos/nushell/nushell/contributors | select login contributions
```

:::

::: warning Thinking in Nushell
Nushell borrows concepts from many shells and languages. You'll likely find many of Nushell's features familiar.
:::

### But it's not Bash

Because of this, however, it's sometimes easy to forget that some Bash (and POSIX in general) style constructs just won't work in Nushell. For instance, in Bash, it would be normal to write:

```sh
# Redirect using >
echo "hello" > output.txt
# But compare (greater-than) using the test command
test 4 -gt 7
echo $?
# => 1
```

In Nushell, however, the `>` is used as the greater-than operator for comparisons. This is more in line with modern programming expectations.

```nu
4 > 10
# => false
```

Since `>` is an operator, redirection to a file in Nushell is handled through a pipeline command that is dedicated to saving content - [`save`](/commands/docs/save.md):

```nu
"hello" | save output.txt
```

::: warning Thinking in Nushell
We've put together a list of common Bash'isms and how to accomplish those tasks in Nushell in the [Coming from Bash](./coming_from_bash.md) Chapter.
:::

## Implicit Return

Users coming from other shells will likely be very familiar with the `echo` command. Nushell's
[`echo`](/commands/docs/echo.md) might appear the same at first, but it is _very_ different.

First, notice how the following output _looks_ the same in both Bash and Nushell (and even PowerShell and Fish):

```nu
echo "Hello, World"
# => Hello, World
```

But while the other shells are sending `Hello, World` straight to _standard output_, Nushell's `echo` is
simply _returning a value_. Nushell then _renders_ the return value of a command, or more technically, an _expression_.

More importantly, Nushell _implicitly returns_ the value of an expression. This is similar to PowerShell or Rust in many respects.

::: tip
An expression can be more than just a pipeline. Even custom commands (similar to functions in many languages, but we'll cover them more in depth in a [later chapter](./custom_commands.md)) automatically, implicitly _return_ the last value. There's no need for an `echo` or even a [`return` command](/commands/docs/return.md) to return a value - It just _happens_.
:::

In other words, the string _"Hello, World"_ and the output value from `echo "Hello, World"` are equivalent:

```nu
"Hello, World" == (echo "Hello, World")
# => true
```

Here's another example with a custom command definition:

```nu
def latest-file [] {
    ls | sort-by modified | last
}
```

The _output_ of that pipeline (its _"value"_) becomes the _return value_ of the `latest-file` custom command.

::: warning Thinking in Nushell
Most anywhere you might write `echo <something>`, in Nushell, you can just write `<something>` instead.
:::

## Single Return Value per Expression

It's important to understand that an expression can only return a single value. If there are multiple subexpressions inside an expression, only the **_last_** value is returned.

A common mistake is to write a custom command definition like this:

```nu:line-numbers
def latest-file [] {
    echo "Returning the last file"
    ls | sort-by modified | last
}

latest-file
```

New users might expect:

- Line 2 to output _"Returning the last file"_
- Line 3 to return/output the file

However, remember that `echo` **_returns a value_**. Since only the last value is returned, the Line 2 _value_ is discarded. Only the file will be returned by line 3.

To make sure the first line is _displayed_, use the [`print` command](/commands/docs/print.md):

```nu
def latest-file [] {
    print "Returning last file"
    ls | sort-by modified | last
}
```

Also compare:

```nu
40; 50; 60
```

::: tip
A semicolon is the same as a newline in a Nushell expression. The above is the same as a file or multi-line command:

```nu
40
50
60
```

or

```nu
echo 40
echo 50
echo 60
```

See Also: [Multi-line Editing](./line_editor.md#multi-line-editing)
:::

In all of the above:

- The first value is evaluated as the integer 40 but is not returned
- The second value is evaluated as the integer 50 but is not returned
- The third value is evaluated as the integer 60, and since it is the last
  value, it is is returned and displayed (rendered).

::: warning Thinking in Nushell
When debugging unexpected results, be on the lookout for:

- Subexpressions (e.g., commands or pipelines) that ...
- ... output a (non-`null`) value ...
- ... where that value isn't returned from the parent expression.

These can be likely sources of issues in your code.
:::

## Every Command Returns a Value

Some languages have the concept of "statements" which don't return values. Nushell does not.

In Nushell, **_every command returns a value_**, even if that value is `null` (the `nothing` type). Consider the following multiline expression:

```nu:line-numbers
let p = 7
print $p
$p * 6
```

1. Line 1: The integer 7 is assigned to `$p`, but the return value of the
   [`let` command](/commands/docs/let.md) itself is `null`. However, because it is not the last
   value in the expression, it is not displayed.
2. Line 2: The return value of the `print` command itself is `null`, but the `print` command
   forces its argument (`$p`, which is 7) to be _displayed_. As with Line 1, the `null` return value
   is discarded since this isn't the last value in the expression.
3. Line 3: Evaluates to the integer value 42. As the last value in the expression, this is the return
   result, and is also displayed (rendered).

::: warning Thinking in Nushell
Becoming familiar with the output types of common commands will help you understand how
to combine simple commands together to achieve complex results.

`help <command>` will show the signature, including the output type(s), for each command in Nushell.
:::

## Think of Nushell as a Compiled Language

In Nushell, there are exactly two, separate, high-level stages when running code:

1. _Stage 1 (Parser):_ Parse the **_entire_** source code
2. _Stage 2 (Engine):_ Evaluate the **_entire_** source code

It can be useful to think of Nushell's parsing stage as _compilation_ in [static](./how_nushell_code_gets_run.md#dynamic-vs-static-languages) languages like Rust or C++. By this, we mean that all of the code that will be evaluated in Stage 2 must be **_known and available_** during the parsing stage.

::: important
However, this also means that Nushell cannot currently support an `eval` construct as with _dynamic_ languages such as Bash or Python.
:::

### Features Built on Static Parsing

On the other hand, the **_static_** results of Parsing are key to many features of Nushell its REPL, such as:

- Accurate and expressive error messages
- Semantic analysis for earlier and robust detection of error conditions
- IDE integration
- The type system
- The module system
- Completions
- Custom command argument parsing
- Syntax highlighting
- Real-time error highlighting
- Profiling and debugging commands
- (Future) Formatting
- (Future) Saving IR (Intermediate Representation) "compiled" results for faster execution

### Limitations

The static nature of Nushell often leads to confusion for users coming to Nushell from languages where an `eval` is available.

Consider a simple two-line file:

```text
<line1 code>
<line2 code>
```

1. Parsing:
   1. Line 1 is parsed
   2. Line 2 is parsed
2. If parsing was successful, then Evaluation:
   1. Line 1 is evaluated
   2. Line 2 is evaluated

This helps demonstrate why the following examples cannot run as a single expression (e.g., a script) in Nushell:

::: note
The following examples use the [`source` command](/commands/docs/source.md), but similar conclusions apply to other commands that parse Nushell source code, such as [`use`](/commands/docs/use.md), [`overlay use`](/commands/docs/overlay_use.md), [`hide`](/commands/docs/hide.md) or [`source-env`](/commands/docs/source-env.md).

:::

#### Example: Dynamically Generating Source

Consider this scenario:

```nu
"print Hello" | save output.nu
source output.nu
# => Error: nu::parser::sourced_file_not_found
# =>
# =>   × File not found
# =>    ╭─[entry #5:2:8]
# =>  1 │ "print Hello" | save output.nu
# =>  2 │ source output.nu
# =>    ·        ────┬────
# =>    ·            ╰── File not found: output.nu
# =>    ╰────
# =>   help: sourced files need to be available before your script is run
```

This is problematic because:

1. Line 1 is parsed but not evaluated. In other words, `output.nu` is not created during the parsing stage, but only during evaluation.
2. Line 2 is parsed. Because `source` is a parser-keyword, resolution of the sourced file is attempted during Parsing (Stage 1). But `output.nu` doesn't even exist yet! If it _does_ exist, then it's probably not even the correct file! This results in the error.

::: note
Typing these as two _separate_ lines in the **_REPL_** will work since the first line will be parsed and evaluated, then the second line will be parsed and evaluated.

The limitation only occurs when both are parsed _together_ as a single expression, which could be part of a script, block, closure, or other expression.

See the [REPL](./how_nushell_code_gets_run.md#the-nushell-repl) section in _"How Nushell Code Gets Run"_ for more explanation.
:::

#### Example: Dynamically Creating a Filename to be Sourced

Another common scenario when coming from another shell might be attempting to dynamically create a filename that will be sourced:

```nu
let my_path = "~/nushell-files"
source $"($my_path)/common.nu"
# => Error:
# =>   × Error: nu::shell::not_a_constant
# =>   │
# =>   │   × Not a constant.
# =>   │    ╭─[entry #6:2:11]
# =>   │  1 │ let my_path = "~/nushell-files"
# =>   │  2 │ source $"($my_path)/common.nu"
# =>   │    ·           ────┬───
# =>   │    ·               ╰── Value is not a parse-time constant
# =>   │    ╰────
# =>   │   help: Only a subset of expressions are allowed constants during parsing. Try using the 'const' command or typing the value literally.
# =>   │
# =>    ╭─[entry #6:2:8]
# =>  1 │ let my_path = "~/nushell-files"
# =>  2 │ source $"($my_path)/common.nu"
# =>    ·        ───────────┬───────────
# =>    ·                   ╰── Encountered error during parse-time evaluation
# =>    ╰────
```

Because the `let` assignment is not resolved until evaluation, the parser-keyword `source` will fail during parsing if passed a variable.

::: details Comparing Rust and C++
Imagine that the code above was written in a typical compiled language such as C++:

```cpp
#include <string>

std::string my_path("foo");
#include <my_path + "/common.h">
```

or Rust

```rust
let my_path = "foo";
use format!("{}::common", my_path);
```

If you've ever written a simple program in any of these languages, you can see these examples aren't valid in those languages. Like Nushell, compiled languages require that all of the source code files are ready and available to the compiler beforehand.

:::

::: tip See Also
As noted in the error message, however, this can work if `my_path` can be defined as a [constant](/book/variables#constant-variables) since constants can be (and are) resolved during parsing.

```nu
const my_path = "~/nushell-files"
source $"($my_path)/common.nu"
```

See [Parse-time Constant Evaluation](./how_nushell_code_gets_run.md#parse-time-constant-evaluation) for more details.
:::

#### Example: Change to a different directory (`cd`) and `source` a file

Here's one more — Change to a different directory and then attempt to `source` a file in that directory.

```nu:line-numbers
if ('spam/foo.nu' | path exists) {
    cd spam
    source-env foo.nu
}
```

Based on what we've covered about Nushell's Parse/Eval stages, see if you can spot the problem with that example.

::: details Solution

In line 3, during Parsing, the `source-env` attempts to parse `foo.nu`. However, `cd` doesn't occur until Evaluation. This results in a parse-time error, since the file is not found in the _current_ directory.

To resolve this, of course, simply use the full-path to the file to be sourced.

```nu
    source-env spam/foo.nu
```

:::

### Summary

::: important
For a more in-depth explanation of this section, see [How Nushell Code Gets Run](how_nushell_code_gets_run.md).
:::

::: warning Thinking in Nushell
Nushell is designed to use a single Parsing stage for each expression or file. This Parsing stage occurs before and is separate from Evaluation. While this enables many of Nushell's features, it also means that users need to understand the limitations it creates.
:::

## Variables are Immutable by Default

Another common surprise when coming from other languages is that Nushell variables are immutable by default. While Nushell has optional mutable variables, many of Nushell's commands are based on a functional-style of programming which requires immutability.

Immutable variables are also key to Nushell's [`par-each` command](/commands/docs/par-each.md), which allows you to operate on multiple values in parallel using threads.

See [Immutable Variables](variables.html#immutable-variables) and [Choosing between mutable and immutable variables](variables.html#choosing-between-mutable-and-immutable-variables) for more information.

::: warning Thinking in Nushell
If you're used to relying on mutable variables, it may take some time to relearn how to code in a more functional style. Nushell has many functional features and commands that operate on and with immutable variables. Learning them will help you write code in a more Nushell-idiomatic style.

A nice bonus is the performance increase you can realize by running parts of your code in parallel with `par-each`.
:::

## Nushell's Environment is Scoped

Nushell takes multiple design cues from compiled languages. One such cue is that languages should avoid global mutable state. Shells have commonly used global mutation to update the environment, but Nushell attempts to steer clear of this approach.

In Nushell, blocks control their own environment. Changes to the environment are scoped to the block where they occur.

In practice, this lets you write (as just one example) more concise code for working with subdirectories. Here's an example that builds each sub-project in the current directory:

```nu
ls | each { |row|
  cd $row.name
  make
}
```

The [`cd`](/commands/docs/cd.md) command changes the `PWD` environment variables, but this variable change does not survive past the end of the block. This allows each iteration to start from the current directory and then enter the next subdirectory.

Having a scoped environment makes commands more predictable, easier to read, and when the time comes, easier to debug. It's also another feature that is key to the `par-each` command we discussed above.

Nushell also provides helper commands like [`load-env`](/commands/docs/load-env.md) as a convenient way of loading multiple updates to the environment at once.

::: tip See Also
[Environment - Scoping](./environment.md#scoping)
:::

::: note
[`def --env`](/commands/docs/def.md) is an exception to this rule. It allows you to create a command that changes the parent's environment.
:::

::: warning Thinking in Nushell
Use scoped-environment to write more concise scripts and prevent unnecessary or unwanted global environment mutation.
:::
# Types of Data

Traditional Unix shell commands communicate with each other using strings of text -- One command writes text to standard output (often abbreviated `stdout`) and the other reads text from standard input (or `stdin`). This allows multiple commands to be combined together to communicate through what is called a "pipeline".

Nushell embraces this approach and expands it to include other types of data in addition to strings.

Like many programming languages, Nu models data using a set of simple, structured data types. Simple data types include integers, floats, strings, and booleans. There are also special types for dates, file sizes, and time durations.

The [`describe`](/commands/docs/describe.md) command returns the type of a data value:

```nu
42 | describe
# => int
```

## Types at a Glance

| Type                                  | Example                                                               |
| ------------------------------------- | --------------------------------------------------------------------- |
| [Integers](#integers)                 | `-65535`                                                              |
| [Floats (decimals)](#floats-decimals) | `9.9999`, `Infinity`                                                  |
| [Strings](#text-strings)              | <code>"hole 18", 'hole 18', \`hole 18\`, hole18, r#'hole18'#</code>   |
| [Booleans](#booleans)                 | `true`                                                                |
| [Dates](#dates)                       | `2000-01-01`                                                          |
| [Durations](#durations)               | `2min + 12sec`                                                        |
| [File-sizes](#file-sizes)             | `64mb`                                                                |
| [Ranges](#ranges)                     | `0..4`, `0..<5`, `0..`, `..4`                                         |
| [Binary](#binary-data)                | `0x[FE FF]`                                                           |
| [Lists](#lists)                       | `[0 1 'two' 3]`                                                       |
| [Records](#records)                   | `{name:"Nushell", lang: "Rust"}`                                      |
| [Tables](#tables)                     | `[{x:12, y:15}, {x:8, y:9}]`, `[[x, y]; [12, 15], [8, 9]]`            |
| [Closures](#closures)                 | `{\|e\| $e + 1 \| into string }`, `{ $in.name.0 \| path exists }`     |
| [Cell-paths](#cell-paths)             | `$.name.0`                                                            |
| [Blocks](#blocks)                     | `if true { print "hello!" }`, `loop { print "press ctrl-c to exit" }` |
| [Null (Nothing)](#nothing-null)       | `null`                                                                |
| [Any](#any)                           | `let p: any = 5`                                                      |

## Basic Data Types

### Integers

|                       |                                                                                                                                                           |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **_Description:_**    | Numbers without a fractional component (positive, negative, and 0)                                                                                        |
| **_Annotation:_**     | `int`                                                                                                                                                     |
| **_Literal Syntax:_** | A decimal, hex, octal, or binary numeric value without a decimal place. E.g., `-100`, `0`, `50`, `+50`, `0xff` (hex), `0o234` (octal), `0b10101` (binary) |
| **_See also:_**       | [Language Reference - Integer](/lang-guide/chapters/types/basic_types/int.md)                                                                             |

Simple Example:

```nu
10 / 2
# => 5
5 | describe
# => int
```

### Floats/Decimals

|                       |                                                                                  |
| --------------------- | -------------------------------------------------------------------------------- |
| **_Description:_**    | Numbers with some fractional component                                           |
| **_Annotation:_**     | `float`                                                                          |
| **_Literal Syntax:_** | A decimal numeric value including a decimal place. E.g., `1.5`, `2.0`, `-15.333` |
| **_See also:_**       | [Language Reference - Float](/lang-guide/chapters/types/basic_types/float.md)    |

Simple Example:

```nu
2.5 / 5.0
# => 0.5
```

::: tip
As in most programming languages, decimal values in Nushell are approximate.

```nu
10.2 * 5.1
# => 52.01999999999999
```

:::

### Text/Strings

|                       |                                                                                 |
| --------------------- | ------------------------------------------------------------------------------- |
| **_Description:_**    | A series of characters that represents text                                     |
| **_Annotation:_**     | `string`                                                                        |
| **_Literal Syntax:_** | See [Working with strings](working_with_strings.md)                             |
| **_See also:_**       | [Handling Strings](/book/loading_data.html#handling-strings)                    |
|                       | [Language Reference - String](/lang-guide/chapters/types/basic_types/string.md) |

As with many languages, Nushell provides multiple ways to specify String values and numerous commands for working with strings.

Simple (obligatory) example:

```nu
let audience: string = "World"
$"Hello, ($audience)"
# => Hello, World
```

### Booleans

|                       |                                                                                |
| --------------------- | ------------------------------------------------------------------------------ |
| **_Description:_**    | True or False value                                                            |
| **_Annotation:_**     | `bool`                                                                         |
| **_Literal Syntax:_** | Either a literal `true` or `false`                                             |
| **_See also:_**       | [Language Reference - Boolean](/lang-guide/chapters/types/basic_types/bool.md) |

Booleans are commonly the result of a comparison. For example:

```nu
let mybool: bool = (2 > 1)
$mybool
# => true
let mybool: bool = ($env.HOME | path exists)
$mybool
# => true
```

A boolean result is commonly used to control the flow of execution:

```nu
let num = -2
if $num < 0 { print "It's negative" }
# => It's negative
```

### Dates

|                       |                                                                                        |
| --------------------- | -------------------------------------------------------------------------------------- |
| **_Description:_**    | Represents a specific point in time using international standard date-time descriptors |
| **_Annotation:_**     | `datetime`                                                                             |
| **_Literal Syntax:_** | See [Language Guide - Date](/lang-guide/chapters/types/basic_types/datetime.md)        |

Simple example:

```nu
date now
# => Mon, 12 Aug 2024 13:59:22 -0400 (now)
# Format as Unix epoch
date now | format date '%s'
# => 1723485562
```

### Durations

|                       |                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **_Description:_**    | Represent a unit of a passage of time                                                     |
| **_Annotation:_**     | `duration`                                                                                |
| **_Literal Syntax:_** | See [Language Reference - Duration](/lang-guide/chapters/types/basic_types/duration.html) |

Durations support fractional values as well as calculations.

Simple example:

```nu
3.14day
# => 3day 3hr 21min
30day / 1sec  # How many seconds in 30 days?
# => 2592000
```

### File sizes

|                       |                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------- |
| **_Description:_**    | Specialized numeric type to represent the size of files or a number of bytes              |
| **_Annotation:_**     | `filesize`                                                                                |
| **_Literal Syntax:_** | See [Language Reference - Filesize](/lang-guide/chapters/types/basic_types/filesize.html) |

Nushell also has a special type for file sizes.

As with durations, Nushell supports fractional file sizes and calculations:

```nu
0.5kB
# => 500 B
1GiB / 1B
# => 1073741824
(1GiB / 1B) == 2 ** 30
# => true
```

See the [Language Reference](/lang-guide/chapters/types/basic_types/filesize.html) for a complete list of units and more detail.

### Ranges

|                       |                                                                                                |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| **_Description:_**    | Describes a range of values from a starting value to an ending value, with an optional stride. |
| **_Annotation:_**     | `range`                                                                                        |
| **_Literal Syntax:_** | `<start_value>..<end_value>`. E.g., `1..10`.                                                   |
|                       | `<start_value>..<second_value>..<end_value>`. E.g., `2..4..20`                                 |
| **_See also:_**       | [Language Guide - Range](/lang-guide/chapters/types/basic_types/range.md)                      |

Simple example:

```nu
1..5
# => ╭───┬───╮
# => │ 0 │ 1 │
# => │ 1 │ 2 │
# => │ 2 │ 3 │
# => │ 3 │ 4 │
# => │ 4 │ 5 │
# => ╰───┴───╯
```

::: tip
You can also easily create lists of characters with a form similar to ranges with the command [`seq char`](/commands/docs/seq_char.html) as well as with dates using the [`seq date`](/commands/docs/seq_date.html) command.
:::

### Cell Paths

|                       |                                                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------- |
| **_Description:_**    | An expression that is used to navigated to an inner value in a structured value.                                |
| **_Annotation:_**     | `cell-path`                                                                                                     |
| **_Literal syntax:_** | A dot-separated list of row (int) and column (string) IDs. E.g., `name.4.5`.                                    |
|                       | Optionally, use a leading `$.` when needed for disambiguation, such as when assigning a cell-path to a variable |
| **_See also:_**       | [Language Reference - Cell-path](/lang-guide/chapters/types/basic_types/cellpath.md)                            |
|                       | [Navigating and Accessing Structured Data](/book/navigating_structured_data.md) chapter.                        |

Simple example:

```nu
let cp = $.2
# Return list item at index 2
[ foo bar goo glue ] | get $cp
# => goo
```

### Closures

|                       |                                                                                                                                                 |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **_Description:_**    | An anonymous function, often called a lambda function, which accepts parameters and _closes over_ (i.e., uses) variables from outside its scope |
| **_Annotation:_**     | `closure`                                                                                                                                       |
| **_Literal Syntax:_** | `{\|args\| expressions }`                                                                                                                       |
| **_See also:_**       | [Language Reference - Closure](/lang-guide/chapters/types/basic_types/closure.md)                                                               |

Simple example:

This closure returns a boolean result of the comparison and then uses it in a `filter` command to return all values greater than 5.

```nu
let compare_closure = {|a| $a > 5 }
let original_list = [ 40 -4 0 8 12 16 -16 ]
$original_list | filter $compare_closure
# => ╭───┬────╮
# => │ 0 │ 40 │
# => │ 1 │  8 │
# => │ 2 │ 12 │
# => │ 3 │ 16 │
# => ╰───┴────╯
```

Closures are a useful way to represent code that can be executed on each row of data via [filters](/lang-guide/chapters/filters/00_filters_overview.md)

### Binary data

|                       |                                                                             |
| --------------------- | --------------------------------------------------------------------------- |
| **_Description:_**    | Represents binary data                                                      |
| **_Annotation:_**     | `binary`                                                                    |
| **_Literal Syntax:_** | `0x[ffffffff]` - hex-based binary representation                            |
|                       | `0o[1234567]` - octal-based binary representation                           |
|                       | `0b[10101010101]` - binary-based binary representation                      |
| **_See also:_**       | [Language Guide - Binary](/lang-guide/chapters/types/basic_types/binary.md) |

Binary data, like the data from an image file, is a group of raw bytes.

Simple example - Confirm that a JPEG file starts with the proper identifier:

```nu
open nushell_logo.jpg
| into binary
| first 2
| $in == 0x[ff d8]
# => true
```

## Structured Data Types

Nushell includes a collection of structured data types that can contain the primitive types above. For example, instead of a single `float`, structured data gives us a way to represent multiple `float` values, such as a `list` of temperature readings, in the same value. Nushell supports the following structured data types:

### Lists

|                       |                                                                                 |
| --------------------- | ------------------------------------------------------------------------------- |
| **_Description:_**    | Ordered sequence of zero or more values of any type                             |
| **_Annotation:_**     | `list`                                                                          |
| **_Literal Syntax:_** | See [Language Guide - List](/lang-guide/chapters/types/basic_types/list.md)     |
| **_See Also:_**       | [Working with Lists](./working_with_lists.md)                                   |
|                       | [Navigating and Accessing Structured Data](/book/navigating_structured_data.md) |

Simple example:

```nu
[Sam Fred George]
# => ╭───┬────────╮
# => │ 0 │ Sam    │
# => │ 1 │ Fred   │
# => │ 2 │ George │
# => ╰───┴────────╯
```

### Records

|                       |                                                                                 |
| --------------------- | ------------------------------------------------------------------------------- |
| **_Description:_**    | Holds key-value pairs which associate string keys with various data values.     |
| **_Annotation:_**     | `record`                                                                        |
| **_Literal Syntax:_** | See [Language Guide - Record](/lang-guide/chapters/types/basic_types/record.md) |
| **_See Also:_**       | [Working with Records](./working_with_records.md)                               |
|                       | [Navigating and Accessing Structured Data](/book/navigating_structured_data.md) |

Simple example:

```nu
let my_record = {
  name: "Kylian"
  rank: 99
}
$my_record
# => ╭───────┬────────────╮
# => │ name  │ Kylian     │
# => │ rank  │ 99         │
# => ╰───────┴────────────╯

$my_record | get name
# =>  Kylian
```

### Tables

|                    |                                                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| **_Description:_** | A two-dimensional container with both columns and rows where each cell can hold any basic or structured data type |
| **_Annotation:_**  | `table`                                                                                                           |
| **_See Also:_**    | [Working with Tables](./working_with_tables.md)                                                                   |
|                    | [Navigating and Accessing Structured Data](/book/navigating_structured_data.md)                                   |
|                    | [Language Guide - Table](/lang-guide/chapters/types/basic_types/table.md)                                         |

The table is a core data structure in Nushell. As you run commands, you'll see that many of them return tables as output. A table has both rows and columns.

:::tip
Internally, tables are simply **lists of records**. This means that any command which extracts or isolates a specific row of a table will produce a record. For example, `get 0`, when used on a list, extracts the first value. But when used on a table (a list of records), it extracts a record:

```nu
[{x:12, y:5}, {x:3, y:6}] | get 0
# => ╭───┬────╮
# => │ x │ 12 │
# => │ y │ 5  │
# => ╰───┴────╯
```

:::

## Other Data Types

### Any

|                       |                                                                                                             |
| --------------------- | ----------------------------------------------------------------------------------------------------------- |
| **_Description:_**    | When used in a type annotation or signature, matches any type. In other words, a "superset" of other types. |
| **_Annotation:_**     | `any`                                                                                                       |
| **_Literal syntax:_** | N/A - Any literal value can be assigned to an `any` type                                                    |
| **_See also:_**       | [Language Reference - Any](/lang-guide/chapters/types/basic_types/any.md)                                   |

### Blocks

|                       |                                                                               |
| --------------------- | ----------------------------------------------------------------------------- |
| **_Description:_**    | A syntactic form used by some Nushell keywords (e.g., `if` and `for`)         |
| **_Annotation:_**     | N/A                                                                           |
| **_Literal Syntax:_** | N/A                                                                           |
| **_See also:_**       | [Language Reference - Block](/lang-guide/chapters/types/other_types/block.md) |

Simple example:

```nu
if true { print "It's true" }
```

The `{ print "It's true" }` portion above is a block.

### Nothing (Null)

|                       |                                                                                   |
| --------------------- | --------------------------------------------------------------------------------- |
| **_Description:_**    | The `nothing` type is to be used to represent the absence of another value.       |
| **_Annotation:_**     | `nothing`                                                                         |
| **_Literal Syntax:_** | `null`                                                                            |
| **_See also:_**       | [Language Reference - Nothing](/lang-guide/chapters/types/basic_types/nothing.md) |

#### Simple Example

Using the optional operator `?` returns `null` if the requested cell-path doesn't exist:

```nu
let simple_record = { a: 5, b: 10 }
$simple_record.a?
# => 5
$simple_record.c?
# => Nothing is output
$simple_record.c? | describe
# => nothing
$simple_record.c? == null
# => true
```
# Variables

Nushell values can be assigned to named variables using the `let`, `const`, or `mut` keywords.
After creating a variable, we can refer to it using `$` followed by its name.

## Types of Variables

### Immutable Variables

An immutable variable cannot change its value after declaration. They are declared using the `let` keyword,

```nu
let val = 42
$val
# => 42
$val = 100
# => Error: nu::shell::assignment_requires_mutable_variable
# => 
# =>   × Assignment to an immutable variable.
# =>    ╭─[entry #10:1:1]
# =>  1 │ $val = 100
# =>    · ──┬─
# =>    ·   ╰── needs to be a mutable variable
# =>    ╰────
```

However, immutable variables can be 'shadowed'. Shadowing means that they are redeclared and their initial value cannot be used anymore within the same scope.

```nu
let val = 42                   # declare a variable
do { let val = 101;  $val }    # in an inner scope, shadow the variable
# => 101
$val                           # in the outer scope the variable remains unchanged
# => 42
let val = $val + 1             # now, in the outer scope, shadow the original variable
$val                           # in the outer scope, the variable is now shadowed, and
# => 43                               # its original value is no longer available.
```

### Mutable Variables

A mutable variable is allowed to change its value by assignment. These are declared using the `mut` keyword.

```nu
mut val = 42
$val += 27
$val
# => 69
```

There are a couple of assignment operators used with mutable variables

| Operator | Description                                                                |
| -------- | -------------------------------------------------------------------------- |
| `=`      | Assigns a new value to the variable                                        |
| `+=`     | Adds a value to the variable and makes the sum its new value               |
| `-=`     | Subtracts a value from the variable and makes the difference its new value |
| `*=`     | Multiplies the variable by a value and makes the product its new value     |
| `/=`     | Divides the variable by a value and makes the quotient its new value       |
| `++=`    | Appends a list or a value to a variable                                    |

::: tip Note

1. `+=`, `-=`, `*=` and `/=` are only valid in the contexts where their root operations are expected to work. For example, `+=` uses addition, so it can not be used for contexts where addition would normally fail
2. `++=` requires that either the variable **or** the argument is a list.
   :::

#### More on Mutability

Closures and nested `def`s cannot capture mutable variables from their environment. For example

```nu
# naive method to count number of elements in a list
mut x = 0

[1 2 3] | each { $x += 1 }   # error: $x is captured in a closure
```

To use mutable variables for such behaviour, you are encouraged to use the loops

### Constant Variables

A constant variable is an immutable variable that can be fully evaluated at parse-time. These are useful with commands that need to know the value of an argument at parse time, like [`source`](/commands/docs/source.md), [`use`](/commands/docs/use.md) and [`plugin use`](/commands/docs/plugin_use.md). See [how nushell code gets run](how_nushell_code_gets_run.md) for a deeper explanation. They are declared using the `const` keyword

```nu
const script_file = 'path/to/script.nu'
source $script_file
```

## Choosing between mutable and immutable variables

Try to use immutable variables for most use-cases.

You might wonder why Nushell uses immutable variables by default. For the first few years of Nushell's development, mutable variables were not a language feature. Early on in Nushell's development, we decided to see how long we could go using a more data-focused, functional style in the language. This experiment showed its value when Nushell introduced parallelism. By switching from [`each`](/commands/docs/each.md) to [`par-each`](/commands/docs/par-each.md) in any Nushell script, you're able to run the corresponding block of code in parallel over the input. This is possible because Nushell's design leans heavily on immutability, composition, and pipelining.

Many, if not most, use-cases for mutable variables in Nushell have a functional solution that:

- Only uses immutable variables, and as a result ...
- Has better performance
- Supports streaming
- Can support additional features such as `par-each` as mentioned above

For instance, loop counters are a common pattern for mutable variables and are built into most iterating commands. For example, you can get both each item and the index of each item using [`each`](/commands/docs/each.md) with [`enumerate`](/commands/docs/enumerate.md):

```nu
ls | enumerate | each { |elt| $"Item #($elt.index) is size ($elt.item.size)" }
# => ╭───┬───────────────────────────╮
# => │ 0 │ Item #0 is size 812 B     │
# => │ 1 │ Item #1 is size 3.4 KiB   │
# => │ 2 │ Item #2 is size 11.0 KiB  │
# => │ 3 │ ...                       │
# => │ 4 │ Item #18 is size 17.8 KiB │
# => │ 5 │ Item #19 is size 482 B    │
# => │ 6 │ Item #20 is size 4.0 KiB  │
# => ╰───┴───────────────────────────╯
```

You can also use the [`reduce`](/commands/docs/reduce.md) command to work in the same way you might mutate a variable in a loop. For example, if you wanted to find the largest string in a list of strings, you might do:

```nu
[one, two, three, four, five, six] | reduce {|current_item, max|
  if ($current_item | str length) > ($max | str length) {
      $current_item
  } else {
      $max
  }
}

three
```

While `reduce` processes lists, the [`generate`](/commands/docs/generate.md) command can be used with arbitrary sources such as external REST APIs, also without requiring mutable variables. Here's an example that retrieves local weather data every hour and generates a continuous list from that data. The `each` command can be used to consume each new list item as it becomes available.

```nu
generate khot {|weather_station|
  let res = try {
    http get -ef $'https://api.weather.gov/stations/($weather_station)/observations/latest'
  } catch {
    null
  }
  sleep 1hr
  match $res {
    null => {
      next: $weather_station
    }
    _ => {
      out: ($res.body? | default '' | from json)
      next: $weather_station
    }
  }
}
| each {|weather_report|
    {
      time: ($weather_report.properties.timestamp | into datetime)
      temp: $weather_report.properties.temperature.value
    }
}
```

### Performance Considerations

Using [filter commands](/commands/categories/filters.html) with immutable variables is often far more performant than mutable variables with traditional flow-control statements such as `for` and `while`. For example:

- Using a `for` statement to create a list of 50,000 random numbers:

  ```nu
  timeit {
    mut randoms = []
    for _ in 1..50_000 {
      $randoms = ($randoms | append (random int))
    }
  }
  ```

  Result: 1min 4sec 191ms 135µs 90ns

- Using `each` to do the same:

  ```nu
  timeit {
    let randoms = (1..50_000 | each {random int})
  }
  ```

  Result: 19ms 314µs 205ns

- Using `each` with 10,000,000 iterations:

  ```nu
  timeit {
    let randoms = (1..10_000_000 | each {random int})
  }
  ```

  Result: 4sec 233ms 865µs 238ns

  As with many filters, the `each` statement also streams its results, meaning the next stage of the pipeline can continue processing without waiting for the results to be collected into a variable.

  For tasks which can be optimized by parallelization, as mentioned above, `par-each` can have even more drastic performance gains.

## Variable Names

Variable names in Nushell come with a few restrictions as to what characters they can contain. In particular, they cannot contain these characters:

```
.  [  (  {  +  -  *  ^  /  =  !  <  >  &  |
```

It is common for some scripts to declare variables that start with `$`. This is allowed, and it is equivalent to the `$` not being there at all.

```nu
let $var = 42
# identical to `let var = 42`
```
# Working with Lists

:::tip
Lists are equivalent to the individual columns of tables. You can think of a list as essentially being a "one-column table" (with no column name). Thus, any command which operates on a column _also_ operates on a list. For instance, [`where`](/commands/docs/where.md) can be used with lists:

```nu
[bell book candle] | where ($it =~ 'b')
# => ╭───┬──────╮
# => │ 0 │ bell │
# => │ 1 │ book │
# => ╰───┴──────╯
```

:::

## Creating lists

A list is an ordered collection of values.
A list is created using square brackets surrounding values separated by spaces, linebreaks, and/or commas.
For example, `[foo bar baz]` or `[foo, bar, baz]`.

::: tip
Nushell lists are similar to JSON arrays. The same `[ "Item1", "Item2", "Item3" ]` that represents a JSON array can also be used to create a Nushell list.
:::

## Updating lists

We can [`insert`](/commands/docs/insert.md) values into lists as they flow through the pipeline, for example let's insert the value `10` into the middle of a list:

```nu
[1, 2, 3, 4] | insert 2 10
# => [1, 2, 10, 3, 4]
```

We can also use [`update`](/commands/docs/update.md) to replace the 2nd element with the value `10`.

```nu
[1, 2, 3, 4] | update 1 10
# => [1, 10, 3, 4]
```

## Removing or Adding Items from List

In addition to [`insert`](/commands/docs/insert.md) and [`update`](/commands/docs/update.md), we also have [`prepend`](/commands/docs/prepend.md) and [`append`](/commands/docs/append.md). These let you insert to the beginning of a list or at the end of the list, respectively.

For example:

```nu
let colors = [yellow green]
let colors = ($colors | prepend red)
let colors = ($colors | append purple)
let colors = ($colors ++ ["blue"])
let colors = (["black"] ++ $colors)
$colors
# => [black red yellow green purple blue]
```

In case you want to remove items from list, there are many ways. [`skip`](/commands/docs/skip.md) allows you skip first rows from input, while [`drop`](/commands/docs/drop.md) allows you to skip specific numbered rows from end of list.

```nu
let colors = [red yellow green purple]
let colors = ($colors | skip 1)
let colors = ($colors | drop 2)
$colors
# => [yellow]
```

We also have [`last`](/commands/docs/last.md) and [`first`](/commands/docs/first.md) which allow you to [`take`](/commands/docs/take.md) from the end or beginning of the list, respectively.

```nu
let colors = [red yellow green purple black magenta]
let colors = ($colors | last 3)
$colors
# => [purple black magenta]
```

And from the beginning of a list,

```nu
let colors = [yellow green purple]
let colors = ($colors | first 2)
$colors
# => [yellow green]
```

### Using the Spread Operator

To append one or more lists together, optionally with values interspersed in between, you can also use the
[spread operator](/book/operators#spread-operator) (`...`):

```nu
let x = [1 2]
[
  ...$x
  3
  ...(4..7 | take 2)
]
# => ╭───┬───╮
# => │ 0 │ 1 │
# => │ 1 │ 2 │
# => │ 2 │ 3 │
# => │ 3 │ 4 │
# => │ 4 │ 5 │
# => ╰───┴───╯
```

## Iterating over Lists

To iterate over the items in a list, use the [`each`](/commands/docs/each.md) command with a [block](types_of_data.html#blocks)
of Nu code that specifies what to do to each item. The block parameter (e.g. `|elt|` in `{ |elt| print $elt }`) is the current list
item, but the [`enumerate`](/commands/docs/enumerate.md) filter can be used to provide `index` and `item` values if needed. For example:

```nu
let names = [Mark Tami Amanda Jeremy]
$names | each { |elt| $"Hello, ($elt)!" }
# Outputs "Hello, Mark!" and three more similar lines.

$names | enumerate | each { |elt| $"($elt.index + 1) - ($elt.item)" }
# Outputs "1 - Mark", "2 - Tami", etc.
```

The [`where`](/commands/docs/where.md) command can be used to create a subset of a list, effectively filtering the list based on a condition.

The following example gets all the colors whose names end in "e".

```nu
let colors = [red orange yellow green blue purple]
$colors | where ($it | str ends-with 'e')
# The block passed to `where` must evaluate to a boolean.
# This outputs the list [orange blue purple].
```

In this example, we keep only values higher than `7`.

```nu
let scores = [7 10 8 6 7]
$scores | where $it > 7 # [10 8]
```

The [`reduce`](/commands/docs/reduce.md) command computes a single value from a list.
It uses a block which takes 2 parameters: the current item (conventionally named `elt`) and an accumulator
(conventionally named `acc`). To specify an initial value for the accumulator, use the `--fold` (`-f`) flag.
To change `elt` to have `index` and `item` values, use the [`enumerate`](/commands/docs/enumerate.md) filter.
For example:

```nu
let scores = [3 8 4]
$"total = ($scores | reduce { |elt, acc| $acc + $elt })" # total = 15

$"total = ($scores | math sum)" # easier approach, same result

$"product = ($scores | reduce --fold 1 { |elt, acc| $acc * $elt })" # product = 96

$scores | enumerate | reduce --fold 0 { |elt, acc| $acc + $elt.index * $elt.item } # 0*3 + 1*8 + 2*4 = 16
```

## Accessing the List

::: tip Note
The following is a basic overview. For a more in-depth discussion of this topic, see the chapter, [Navigating and Accessing Structured Data](/book/navigating_structured_data.md).
:::

To access a list item at a given index, use the `$name.index` form where `$name` is a variable that holds a list.

For example, the second element in the list below can be accessed with `$names.1`.

```nu
let names = [Mark Tami Amanda Jeremy]
$names.1 # gives Tami
```

If the index is in some variable `$index` we can use the `get` command to extract the item from the list.

```nu
let names = [Mark Tami Amanda Jeremy]
let index = 1
$names | get $index # gives Tami
```

The [`length`](/commands/docs/length.md) command returns the number of items in a list.
For example, `[red green blue] | length` outputs `3`.

The [`is-empty`](/commands/docs/is-empty.md) command determines whether a string, list, or table is empty.
It can be used with lists as follows:

```nu
let colors = [red green blue]
$colors | is-empty # false

let colors = []
$colors | is-empty # true
```

The `in` and `not-in` operators are used to test whether a value is in a list. For example:

```nu
let colors = [red green blue]
'blue' in $colors # true
'yellow' in $colors # false
'gold' not-in $colors # true
```

The [`any`](/commands/docs/any.md) command determines if any item in a list
matches a given condition.
For example:

```nu
let colors = [red green blue]
# Do any color names end with "e"?
$colors | any {|elt| $elt | str ends-with "e" } # true

# Is the length of any color name less than 3?
$colors | any {|elt| ($elt | str length) < 3 } # false

let scores = [3 8 4]
# Are any scores greater than 7?
$scores | any {|elt| $elt > 7 } # true

# Are any scores odd?
$scores | any {|elt| $elt mod 2 == 1 } # true
```

The [`all`](/commands/docs/all.md) command determines if every item in a list
matches a given condition.
For example:

```nu
let colors = [red green blue]
# Do all color names end with "e"?
$colors | all {|elt| $elt | str ends-with "e" } # false

# Is the length of all color names greater than or equal to 3?
$colors | all {|elt| ($elt | str length) >= 3 } # true

let scores = [3 8 4]
# Are all scores greater than 7?
$scores | all {|elt| $elt > 7 } # false

# Are all scores even?
$scores | all {|elt| $elt mod 2 == 0 } # false
```

## Converting the List

The [`flatten`](/commands/docs/flatten.md) command creates a new list from an existing list
by adding items in nested lists to the top-level list.
This can be called multiple times to flatten lists nested at any depth.
For example:

```nu
[1 [2 3] 4 [5 6]] | flatten # [1 2 3 4 5 6]

[[1 2] [3 [4 5 [6 7 8]]]] | flatten | flatten | flatten # [1 2 3 4 5 6 7 8]
```

The [`wrap`](/commands/docs/wrap.md) command converts a list to a table. Each list value will
be converted to a separate row with a single column:

```nu
let zones = [UTC CET Europe/Moscow Asia/Yekaterinburg]

# Show world clock for selected time zones
$zones | wrap 'Zone' | upsert Time {|row| (date now | date to-timezone $row.Zone | format date '%Y.%m.%d %H:%M')}
```
# Working with Records

:::tip
Records are roughly equivalent to the individual rows of a table. You can think of a record as essentially being a "one-row table". Thus, most commands which operate on a table row _also_ operates on a record. For instance, [`update`](/commands/docs/update.md) can be used with records:

```nu
let my_record = {
 name: "Sam"
 age: 30
 }
$my_record | update age { $in + 1 }
# => ╭──────┬─────╮
# => │ name │ Sam │
# => │ age  │ 31  │
# => ╰──────┴─────╯
```

:::

## Creating records

A record is a collection of zero or more key-value pair mappings. It is similar to a JSON object, and can be created using the same syntax:

```nu
# Nushell
{ "apples": 543, "bananas": 411, "oranges": 0 }
# => ╭─────────┬─────╮
# => │ apples  │ 543 │
# => │ bananas │ 411 │
# => │ oranges │ 0   │
# => ╰─────────┴─────╯
# JSON
'{ "apples": 543, "bananas": 411, "oranges": 0 }' | from json
# => ╭─────────┬─────╮
# => │ apples  │ 543 │
# => │ bananas │ 411 │
# => │ oranges │ 0   │
# => ╰─────────┴─────╯
```

In Nushell, the key-value pairs of a record can also be separated using spaces or line-breaks.

::: tip
As records can have many fields, they are, by default, displayed vertically rather than left-to-right. To display a record left-to-right, convert it to a nuon. For example:

```nu
  {
    name: "Sam"
    rank: 10
  } | to nuon
  # =>   {name: Sam, rank: 10}
```

:::

## Updating Records

As with lists, you can [`insert`](/commands/docs/insert.md) values in records. For example, let's add some pears:

```nu
{ "apples": 543, "bananas": 411, "oranges": 0 }
| insert pears { 21 }
# => ╭─────────┬─────╮
# => │ apples  │ 543 │
# => │ bananas │ 411 │
# => │ oranges │ 0   │
# => │ pears   │ 21  │
# => ╰─────────┴─────╯
```

You can also [`update`](/commands/docs/update.md) values:

```nu
{ "apples": 543, "bananas": 411, "oranges": 0 }
| update oranges { 100 }
# => ╭─────────┬─────╮
# => │ apples  │ 543 │
# => │ bananas │ 411 │
# => │ oranges │ 100 │
# => ╰─────────┴─────╯
```

To make a copy of a record with new fields, you can either:

- Use the [`merge`](/commands/docs/merge.md) command:

  ```nu
  let first_record = { name: "Sam", rank: 10 }
  $first_record | merge { title: "Mayor" }
  # =>   ╭───────┬───────╮
  # =>   │ name  │ Sam   │
  # =>   │ rank  │ 10    │
  # =>   │ title │ Mayor │
  # =>   ╰───────┴───────╯
  ```

- Use the [spread operator](/book/operators#spread-operator) (`...`) to expand the first record inside a new record:

  ```nu
  let first_record = { name: "Sam", rank: 10 }
  {
    ...$first_record
    title: "Mayor"
  }
  # =>   ╭───────┬───────╮
  # =>   │ name  │ Sam   │
  # =>   │ rank  │ 10    │
  # =>   │ title │ Mayor │
  # =>   ╰───────┴───────╯
  ```

## Iterating over a Record

You can iterate over the key-value pairs of a record by first transposing it into a table:

```nu
{ "apples": 543, "bananas": 411, "oranges": 0 }
| transpose fruit count
| each {|f| $"We have ($f.count) ($f.fruit)" }
# => ╭───┬─────────────────────╮
# => │ 0 │ We have 543 apples  │
# => │ 1 │ We have 411 bananas │
# => │ 2 │ We have 0 oranges   │
# => ╰───┴─────────────────────╯
```

## Accessing Record Values

See [Navigating and Accessing Structured Data](/book/navigating_structured_data.md) for an in-depth explanation of how to access record values (and other structured data).

## Other Record Commands

See [Working with Tables](./working_with_tables.md) - Remember, commands that operate on table rows will usually operate the same way on records.
# Working with Strings

As with most languages, strings are a collection of 0 or more characters that represent text. This can include file names, file paths, names of columns,
and much more. Strings are so common that Nushell offers multiple string formats to match your use-case:

## String Formats at a Glance

| Format of string                                     | Example                 | Escapes                   | Notes                                                                  |
| ---------------------------------------------------- | ----------------------- | ------------------------- | ---------------------------------------------------------------------- |
| [Single-quoted string](#single-quoted-strings)       | `'[^\n]+'`              | None                      | Cannot contain single quotes within the string                         |
| [Double-quoted string](#double-quoted-strings)       | `"The\nEnd"`            | C-style backslash escapes | All literal backslashes must be escaped                                |
| [Raw strings](#raw-strings)                          | `r#'Raw string'#`       | None                      | May include single quotes                                              |
| [Bare word string](#bare-word-strings)               | `ozymandias`            | None                      | Can only contain "word" characters; Cannot be used in command position |
| [Backtick string](#backtick-quoted-strings)          | <code>\`[^\n]+\`</code> | None                      | Bare string that can include whitespace. Cannot contain any backticks  |
| [Single-quoted interpolation](#string-interpolation) | `$'Captain ($name)'`    | None                      | Cannot contain any `'` or unmatched `()`                               |
| [Double-quoted interpolation](#string-interpolation) | `$"Captain ($name)"`    | C-style backslash escapes | All literal backslashes and `()` must be escaped                       |

## Single-quoted Strings

The simplest string in Nushell is the single-quoted string. This string uses the `'` character to surround some text. Here's the text for hello world as a single-quoted string:

```nu
'hello world'
# => hello world
'The
end'
# => The
# => end
```

Single-quoted strings don't do anything to the text they're given, making them ideal for holding a wide range of text data.

## Double-quoted Strings

For more complex strings, Nushell also offers double-quoted strings. These strings use the `"` character to surround text. They also support the ability escape characters inside the text using the `\` character.

For example, we could write the text hello followed by a new line and then world, using escape characters and a double-quoted string:

```nu
"hello\nworld"
# => hello
# => world
```

Escape characters let you quickly add in a character that would otherwise be hard to type.

Nushell currently supports the following escape characters:

- `\"` - double-quote character
- `\'` - single-quote character
- `\\` - backslash
- `\/` - forward slash
- `\b` - backspace
- `\f` - formfeed
- `\r` - carriage return
- `\n` - newline (line feed)
- `\t` - tab
- `\u{X...}` - a single unicode character, where X... is 1-6 hex digits (0-9, A-F)

## Raw Strings

Raw strings behave the same as a single quoted strings, except that raw strings
may also contain single quotes. This is possible because raw strings are enclosed
by a starting `r#'` and a closing `'#`. This syntax should look familiar to users
of Rust.

```nu
r#'Raw strings can contain 'quoted' text.'#
# => Raw strings can contain 'quoted' text.
```

Additional `#` symbols can be added to the start and end of the raw string to enclose
one less than the same number of `#` symbols next to a `'` symbol in the string. This can
be used to nest raw strings:

```nu
r###'r##'This is an example of a raw string.'##'###
# => r##'This is an example of a raw string.'##
```

## Bare Word Strings

Like other shell languages (but unlike most other programming languages) strings consisting of a single 'word' can also be written without any quotes:

```nu
print hello
# => hello
[hello] | describe
# => list<string>
```

But be careful - if you use a bare word plainly on the command line (that is, not inside a data structure or used as a command parameter) or inside round brackets `(` `)`, it will be interpreted as an external command:

```nu
hello
# => Error: nu::shell::external_command
# => 
# =>   × External command failed
# =>    ╭─[entry #5:1:1]
# =>  1 │ hello
# =>    · ──┬──
# =>    ·   ╰── executable was not found
# =>    ╰────
# =>   help: program not found
```

Also, many bare words have special meaning in nu, and so will not be interpreted as a string:

```nu
true | describe
# => bool
[true] | describe
# => list<bool>
[trueX] | describe
# => list<string>
trueX | describe
# => Error: nu::shell::external_command
# => 
# =>   × External command failed
# =>    ╭─[entry #5:1:1]
# =>  1 │ trueX | describe
# =>    · ──┬──
# =>    ·   ╰── executable was not found
# =>    ╰────
# =>   help: program not found
```

So, while bare strings are useful for informal command line usage, when programming more formally in nu, you should generally use quotes.

## Backtick-quoted Strings

Bare word strings, by their nature, cannot include spaces or quotes. As an alternative, Nushell also includes backtick-quoted
strings using the <code>`</code> character. In most cases, these should operate the same as a bare word string.

For instance, as with a bare word, a backtick-quoted string in the first position of an expression will be interpreted as a _command_ or _path_.
For example:

```nu
# Run the external ls binary found on the path
`ls`

# Move up one directory
`..`

# Change to the "my dir" subdirectory, if it exists
`./my dir`
```

Backtick-quoted strings can be useful for combining globs with files or directories which include spaces:

```nu
ls `./my dir/*`
```

Backtick-quoted strings cannot contain _unmatched_ backticks in the string itself. For example:

`````nu
echo ````
``

echo ```
# Unterminated string which will start a new line in the CLI
`````

## Strings as external commands

You can place the `^` sigil in front of any string (including a variable) to have Nushell execute the string as if it was an external command:

```nu
^'C:\Program Files\exiftool.exe'

let foo = 'C:\Program Files\exiftool.exe'
^$foo
```

You can also use the [`run-external`](/commands/docs/run-external.md) command for this purpose, which provides additional flags and options.

## Appending and Prepending to strings

There are various ways to pre, or append strings. If you want to add something to the beginning of each string closures are a good option:

```nu
['foo', 'bar'] | each {|s| '~/' ++ $s} # ~/foo, ~/bar
['foo', 'bar'] | each {|s| '~/' + $s} # ~/foo, ~/bar
```

You can also use a regex to replace the beginning or end of a string:

```nu
['foo', 'bar'] | str replace -r '^' '~/'# ~/foo, ~/bar
['foo', 'bar'] | str replace -r '$' '~/'# foo~/, bar~/
```

If you want to get one string out of the end then `str join` is your friend:

```nu
"hello" | append "world!" | str join " " # hello world!
```

You can also use reduce:

```nu
1..10 | reduce -f "" {|elt, acc| $acc + ($elt | into string) + " + "} # 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 +
```

Though in the cases of strings, especially if you don't have to operate on the strings, it's usually easier and more correct (notice the extra + at the end in the example above) to use `str join`.

Finally you could also use string interpolation, but that is complex enough that it is covered in its own subsection below.

## String interpolation

More complex string use cases also need a new form of string: string interpolation. This is a way of building text from both raw text and the result of running expressions. String interpolation combines the results together, giving you a new string.

String interpolation uses `$" "` and `$' '` as ways to wrap interpolated text.

For example, let's say we have a variable called `$name` and we want to greet the name of the person contained in this variable:

```nu
let name = "Alice"
$"greetings, ($name)"
# => greetings, Alice
```

By wrapping expressions in `()`, we can run them to completion and use the results to help build the string.

String interpolation has both a single-quoted, `$' '`, and a double-quoted, `$" "`, form. These correspond to the single-quoted and double-quoted strings: single-quoted string interpolation doesn't support escape characters while double-quoted string interpolation does.

As of version 0.61, interpolated strings support escaping parentheses, so that the `(` and `)` characters may be used in a string without Nushell trying to evaluate what appears between them:

```nu
$"2 + 2 is (2 + 2) \(you guessed it!)"
# => 2 + 2 is 4 (you guessed it!)
```

Interpolated strings can be evaluated at parse time, but if they include values whose formatting depends
on your configuration and your `config.nu` hasn't been loaded yet, they will use the default configuration.
So if you have something like this in your `config.nu`, `x` will be `"2.0 KB"` even if your config says to use
`MB` for all file sizes (datetimes will similarly use the default config).

```nu
const x = $"(2kb)"
```

## Splitting Strings

The [`split row`](/commands/docs/split_row.md) command creates a list from a string based on a delimiter.

```nu
"red,green,blue" | split row ","
# => ╭───┬───────╮
# => │ 0 │ red   │
# => │ 1 │ green │
# => │ 2 │ blue  │
# => ╰───┴───────╯
```

The [`split column`](/commands/docs/split_column.md) command will create a table from a string based on a delimiter. This applies generic column names to the table.

```nu
"red,green,blue" | split column ","
# => ╭───┬─────────┬─────────┬─────────╮
# => │ # │ column1 │ column2 │ column3 │
# => ├───┼─────────┼─────────┼─────────┤
# => │ 0 │ red     │ green   │ blue    │
# => ╰───┴─────────┴─────────┴─────────╯
```

Finally, the [`split chars`](/commands/docs/split_chars.md) command will split a string into a list of characters.

```nu
'aeiou' | split chars
# => ╭───┬───╮
# => │ 0 │ a │
# => │ 1 │ e │
# => │ 2 │ i │
# => │ 3 │ o │
# => │ 4 │ u │
# => ╰───┴───╯
```

## The [`str`](/commands/docs/str.md) command

Many string functions are subcommands of the [`str`](/commands/docs/str.md) command. You can get a full list using `help str`.

For example, you can look if a string contains a particular substring using [`str contains`](/commands/docs/str_contains.md):

```nu
"hello world" | str contains "o wo"
# => true
```

(You might also prefer, for brevity, the `=~` operator (described below).)

### Trimming Strings

You can trim the sides of a string with the [`str trim`](/commands/docs/str_trim.md) command. By default, the [`str trim`](/commands/docs/str_trim.md) commands trims whitespace from both sides of the string. For example:

```nu
'       My   string   ' | str trim
# => My   string
```

You can specify on which side the trimming occurs with the `--right` and `--left` options. (`-r` and `-l` being the short-form options respectively)

To trim a specific character, use `--char <Character>` or `-c <Character>` to specify the character to trim.

Here's an example of all the options in action:

```nu
'=== Nu shell ===' | str trim -r -c '='
# => === Nu shell
```

### Substrings

Substrings are slices of a string. They have a startpoint and an endpoint. Here's an example of using a substring:

```nu
'Hello World!' | str index-of 'o'
# => 4
'Hello World!' | str index-of 'r'
# => 8
'Hello World!' | str substring 4..8
# => o Wo
```

### String Padding

With the [`fill`](/commands/docs/fill.md) command you can add padding to a string. Padding adds characters to string until it's a certain length. For example:

```nu
'1234' | fill -a right -c '0' -w 10
# => 0000001234
'1234' | fill -a left -c '0' -w 10 | str length
# => 10
```

### Reversing Strings

This can be done easily with the [`str reverse`](/commands/docs/str_reverse.md) command.

```nu
'Nushell' | str reverse
# => llehsuN
['Nushell' 'is' 'cool'] | str reverse
# => ╭───┬─────────╮
# => │ 0 │ llehsuN │
# => │ 1 │ si      │
# => │ 2 │ looc    │
# => ╰───┴─────────╯
```

## String Parsing

With the [`parse`](/commands/docs/parse.md) command you can parse a string into columns. For example:

```nu
'Nushell 0.80' | parse '{shell} {version}'
# => ╭───┬─────────┬─────────╮
# => │ # │  shell  │ version │
# => ├───┼─────────┼─────────┤
# => │ 0 │ Nushell │ 0.80    │
# => ╰───┴─────────┴─────────╯
'where all data is structured!' | parse --regex '(?P<subject>\w*\s?\w+) is (?P<adjective>\w+)'
# => ╭───┬──────────┬────────────╮
# => │ # │ subject  │ adjective  │
# => ├───┼──────────┼────────────┤
# => │ 0 │ all data │ structured │
# => ╰───┴──────────┴────────────╯
```

If a string is known to contain comma-separated, tab-separated or multi-space-separated data, you can use [`from csv`](/commands/docs/from_csv.md), [`from tsv`](/commands/docs/from_tsv.md) or [`from ssv`](/commands/docs/from_ssv.md):

```nu
"acronym,long\nAPL,A Programming Language" | from csv
# => ╭───┬─────────┬────────────────────────╮
# => │ # │ acronym │          long          │
# => ├───┼─────────┼────────────────────────┤
# => │ 0 │ APL     │ A Programming Language │
# => ╰───┴─────────┴────────────────────────╯
"name  duration\nonestop.mid  4:06" | from ssv
# => ╭───┬─────────────┬──────────╮
# => │ # │    name     │ duration │
# => ├───┼─────────────┼──────────┤
# => │ 0 │ onestop.mid │ 4:06     │
# => ╰───┴─────────────┴──────────╯
"rank\tsuit\nJack\tSpades\nAce\tClubs" | from tsv
# => ╭───┬──────┬────────╮
# => │ # │ rank │  suit  │
# => ├───┼──────┼────────┤
# => │ 0 │ Jack │ Spades │
# => │ 1 │ Ace  │ Clubs  │
# => ╰───┴──────┴────────╯
```

## String Comparison

In addition to the standard `==` and `!=` operators, a few operators exist for specifically comparing strings to one another.

Those familiar with Bash and Perl will recognise the regex comparison operators:

```nu
'APL' =~ '^\w{0,3}$'
# => true
'FORTRAN' !~ '^\w{0,3}$'
# => true
```

Two other operators exist for simpler comparisons:

```nu
'JavaScript' starts-with 'Java'
# => true
'OCaml' ends-with 'Caml'
# => true
```

## Converting Strings

There are multiple ways to convert strings to and from other types.

### To string

1. Using [`into string`](/commands/docs/into_string.md). e.g. `123 | into string`
2. Using string interpolation. e.g. `$'(123)'`

### From string

1. Using [`into <type>`](/commands/docs/into.md). e.g. `'123' | into int`

## Coloring Strings

You can color strings with the [`ansi`](/commands/docs/ansi.md) command. For example:

```nu
$'(ansi purple_bold)This text is a bold purple!(ansi reset)'
```

`ansi purple_bold` makes the text a bold purple
`ansi reset` resets the coloring to the default.

::: tip
You should always end colored strings with `ansi reset`
:::
# Working with Tables

[[toc]]

## Overview

One of the common ways of seeing data in Nu is through a table. Nu comes with a number of commands for working with tables to make it convenient to find what you're looking for, and for narrowing down the data to just what you need.

To start off, let's get a table that we can use:

```nu
ls
# => ───┬───────────────┬──────┬─────────┬────────────
# =>  # │ name          │ type │ size    │ modified
# => ───┼───────────────┼──────┼─────────┼────────────
# =>  0 │ files.rs      │ File │  4.6 KB │ 5 days ago
# =>  1 │ lib.rs        │ File │   330 B │ 5 days ago
# =>  2 │ lite_parse.rs │ File │  6.3 KB │ 5 days ago
# =>  3 │ parse.rs      │ File │ 49.8 KB │ 1 day ago
# =>  4 │ path.rs       │ File │  2.1 KB │ 5 days ago
# =>  5 │ shapes.rs     │ File │  4.7 KB │ 5 days ago
# =>  6 │ signature.rs  │ File │  1.2 KB │ 5 days ago
# => ───┴───────────────┴──────┴─────────┴────────────
```

::: tip Changing how tables are displayed
Nu will try to expands all table's structure by default. You can change this behavior by changing the `display_output` hook.
See [hooks](/book/hooks.md#changing-how-output-is-displayed) for more information.
:::

## Sorting the Data

We can sort a table by calling the [`sort-by`](/commands/docs/sort-by.md) command and telling it which columns we want to use in the sort. Let's say we wanted to sort our table by the size of the file:

```nu
ls | sort-by size
# => ───┬───────────────┬──────┬─────────┬────────────
# =>  # │ name          │ type │ size    │ modified
# => ───┼───────────────┼──────┼─────────┼────────────
# =>  0 │ lib.rs        │ File │   330 B │ 5 days ago
# =>  1 │ signature.rs  │ File │  1.2 KB │ 5 days ago
# =>  2 │ path.rs       │ File │  2.1 KB │ 5 days ago
# =>  3 │ files.rs      │ File │  4.6 KB │ 5 days ago
# =>  4 │ shapes.rs     │ File │  4.7 KB │ 5 days ago
# =>  5 │ lite_parse.rs │ File │  6.3 KB │ 5 days ago
# =>  6 │ parse.rs      │ File │ 49.8 KB │ 1 day ago
# => ───┴───────────────┴──────┴─────────┴────────────
```

We can sort a table by any column that can be compared. For example, we could also have sorted the above using the "name", "accessed", or "modified" columns.

For more info on sorting, see [Sorting](/book/sorting.md).

## Selecting the Data you Want

::: tip Note
The following is a basic overview. For a more in-depth discussion of this topic, see the chapter, [Navigating and Accessing Structured Data](/book/navigating_structured_data.md).
:::

We can select data from a table by choosing to select specific columns or specific rows. Let's [`select`](/commands/docs/select.md) a few columns from our table to use:

```nu
ls | select name size
# => ───┬───────────────┬─────────
# =>  # │ name          │ size
# => ───┼───────────────┼─────────
# =>  0 │ files.rs      │  4.6 KB
# =>  1 │ lib.rs        │   330 B
# =>  2 │ lite_parse.rs │  6.3 KB
# =>  3 │ parse.rs      │ 49.8 KB
# =>  4 │ path.rs       │  2.1 KB
# =>  5 │ shapes.rs     │  4.7 KB
# =>  6 │ signature.rs  │  1.2 KB
# => ───┴───────────────┴─────────
```

This helps to create a table that's more focused on what we need. Next, let's say we want to only look at the 5 smallest files in this directory:

```nu
ls | sort-by size | first 5
# => ───┬──────────────┬──────┬────────┬────────────
# =>  # │ name         │ type │ size   │ modified
# => ───┼──────────────┼──────┼────────┼────────────
# =>  0 │ lib.rs       │ File │  330 B │ 5 days ago
# =>  1 │ signature.rs │ File │ 1.2 KB │ 5 days ago
# =>  2 │ path.rs      │ File │ 2.1 KB │ 5 days ago
# =>  3 │ files.rs     │ File │ 4.6 KB │ 5 days ago
# =>  4 │ shapes.rs    │ File │ 4.7 KB │ 5 days ago
# => ───┴──────────────┴──────┴────────┴────────────
```

You'll notice we first sort the table by size to get to the smallest file, and then we use the `first 5` to return the first 5 rows of the table.

You can also [`skip`](/commands/docs/skip.md) rows that you don't want. Let's skip the first two of the 5 rows we returned above:

```nu
ls | sort-by size | first 5 | skip 2
# => ───┬───────────┬──────┬────────┬────────────
# =>  # │ name      │ type │ size   │ modified
# => ───┼───────────┼──────┼────────┼────────────
# =>  0 │ path.rs   │ File │ 2.1 KB │ 5 days ago
# =>  1 │ files.rs  │ File │ 4.6 KB │ 5 days ago
# =>  2 │ shapes.rs │ File │ 4.7 KB │ 5 days ago
# => ───┴───────────┴──────┴────────┴────────────
```

We've narrowed it to three rows we care about.

Let's look at a few other commands for selecting data. You may have wondered why the rows of the table are numbers. This acts as a handy way to get to a single row. Let's sort our table by the file name and then pick one of the rows with the [`select`](/commands/docs/select.md) command using its row number:

```nu
ls | sort-by name
# => ───┬───────────────┬──────┬─────────┬────────────
# =>  # │ name          │ type │ size    │ modified
# => ───┼───────────────┼──────┼─────────┼────────────
# =>  0 │ files.rs      │ File │  4.6 KB │ 5 days ago
# =>  1 │ lib.rs        │ File │   330 B │ 5 days ago
# =>  2 │ lite_parse.rs │ File │  6.3 KB │ 5 days ago
# =>  3 │ parse.rs      │ File │ 49.8 KB │ 1 day ago
# =>  4 │ path.rs       │ File │  2.1 KB │ 5 days ago
# =>  5 │ shapes.rs     │ File │  4.7 KB │ 5 days ago
# =>  6 │ signature.rs  │ File │  1.2 KB │ 5 days ago
# => ───┴───────────────┴──────┴─────────┴────────────

ls | sort-by name | select 5
# => ───┬───────────────┬──────┬─────────┬────────────
# =>  # │ name          │ type │ size    │ modified
# => ───┼───────────────┼──────┼─────────┼────────────
# =>  0 │ shapes.rs     │ File │  4.7 KB │ 5 days ago
# => ───┴───────────────┴──────┴─────────┴────────────
```

## Getting Data out of a Table

So far, we've worked with tables by trimming the table down to only what we need. Sometimes we may want to go a step further and only look at the values in the cells themselves rather than taking a whole column. Let's say, for example, we wanted to only get a list of the names of the files. For this, we use the [`get`](/commands/docs/get.md) command:

```nu
ls | get name
# => ───┬───────────────
# =>  0 │ files.rs
# =>  1 │ lib.rs
# =>  2 │ lite_parse.rs
# =>  3 │ parse.rs
# =>  4 │ path.rs
# =>  5 │ shapes.rs
# =>  6 │ signature.rs
# => ───┴───────────────
```

We now have the values for each of the filenames.

This might look like the [`select`](/commands/docs/select.md) command we saw earlier, so let's put that here as well to compare the two:

```nu
ls | select name
# => ───┬───────────────
# =>  # │ name
# => ───┼───────────────
# =>  0 │ files.rs
# =>  1 │ lib.rs
# =>  2 │ lite_parse.rs
# =>  3 │ parse.rs
# =>  4 │ path.rs
# =>  5 │ shapes.rs
# =>  6 │ signature.rs
# => ───┴───────────────
```

These look very similar! Let's see if we can spell out the difference between these two commands to make it clear:

- [`select`](/commands/docs/select.md) - creates a new table which includes only the columns specified
- [`get`](/commands/docs/get.md) - returns the values inside the column specified as a list

:::tip
The arguments provided to `select` and `get` are [cell-paths](/book/types_of_data.html#cell-paths), a fundamental part of Nu's query language. For a more in-depth discussion of cell-paths and other navigation topics, see the next chapter, [Navigating and Accessing Structured Data](/book/navigating_structured_data.md).
:::

## Changing Data in a Table

In addition to selecting data from a table, we can also update what the table has. We may want to combine tables, add new columns, or edit the contents of a cell. In Nu, rather than editing in place, each of the commands in the section will return a new table in the pipeline.

### Concatenating Tables

We can concatenate tables using [`append`](/commands/docs/append.md):

```nu
let first = [[a b]; [1 2]]
let second = [[a b]; [3 4]]
$first | append $second
# => ───┬───┬───
# =>  # │ a │ b
# => ───┼───┼───
# =>  0 │ 1 │ 2
# =>  1 │ 3 │ 4
# => ───┴───┴───
```

If the column names are not identical then additionally columns and values will be created as necessary:

```nu
let first = [[a b]; [1 2]]
let second = [[a b]; [3 4]]
let third = [[a c]; [3 4]]
$first | append $second | append $third
# => ───┬───┬────┬────
# =>  # │ a │ b  │ c
# => ───┼───┼────┼────
# =>  0 │ 1 │  2 │ ❎
# =>  1 │ 3 │  4 │ ❎
# =>  2 │ 3 │ ❎ │  4
# => ───┴───┴────┴────
```

You can also use the `++` operator as an inline replacement for `append`:

```nu
$first ++ $second ++ $third
# => ───┬───┬────┬────
# =>  # │ a │ b  │ c
# => ───┼───┼────┼────
# =>  0 │ 1 │  2 │ ❎
# =>  1 │ 3 │  4 │ ❎
# =>  2 │ 3 │ ❎ │  4
# => ───┴───┴────┴───
```

### Merging Tables

We can use the [`merge`](/commands/docs/merge.md) command to merge two (or more) tables together

```nu
let first = [[a b]; [1 2]]
let second = [[c d]; [3 4]]
$first | merge $second
# => ───┬───┬───┬───┬───
# =>  # │ a │ b │ c │ d
# => ───┼───┼───┼───┼───
# =>  0 │ 1 │ 2 │ 3 │ 4
# => ───┴───┴───┴───┴───
```

Let's add a third table:

```nu
let third = [[e f]; [5 6]]
```

We could join all three tables together like this:

```nu
$first | merge $second  | merge $third
# => ───┬───┬───┬───┬───┬───┬───
# =>  # │ a │ b │ c │ d │ e │ f
# => ───┼───┼───┼───┼───┼───┼───
# =>  0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6
# => ───┴───┴───┴───┴───┴───┴───
```

Or we could use the [`reduce`](/commands/docs/reduce.md) command to dynamically merge all tables:

```nu
[$first $second $third] | reduce {|elt, acc| $acc | merge $elt }
# => ───┬───┬───┬───┬───┬───┬───
# =>  # │ a │ b │ c │ d │ e │ f
# => ───┼───┼───┼───┼───┼───┼───
# =>  0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6
# => ───┴───┴───┴───┴───┴───┴───
```

### Adding a new Column

We can use the [`insert`](/commands/docs/insert.md) command to add a new column to the table. Let's look at an example:

```nu
open rustfmt.toml
# => ─────────┬──────
# =>  edition │ 2018
# => ─────────┴──────
```

Let's add a column called "next_edition" with the value 2021:

```nu
open rustfmt.toml | insert next_edition 2021
# => ──────────────┬──────
# =>  edition      │ 2018
# =>  next_edition │ 2021
# => ──────────────┴──────
```

This visual may be slightly confusing, because it looks like what we've just done is add a row. In this case, remember: rows have numbers, columns have names. If it still is confusing, note that appending one more row will make the table render as expected:

```nu
open rustfmt.toml | insert next_edition 2021 | append {edition: 2021 next_edition: 2024}
# => ───┬─────────┬──────────────
# =>  # │ edition │ next_edition
# => ───┼─────────┼──────────────
# =>  0 │    2018 │         2021
# =>  1 │    2021 │         2024
# => ───┴─────────┴──────────────

```

Notice that if we open the original file, the contents have stayed the same:

```nu
open rustfmt.toml
# => ─────────┬──────
# =>  edition │ 2018
# => ─────────┴──────
```

Changes in Nu are functional changes, meaning that they work on values themselves rather than trying to cause a permanent change. This lets us do many different types of work in our pipeline until we're ready to write out the result with any changes we'd like if we choose to. Here we could write out the result using the [`save`](/commands/docs/save.md) command:

```nu
open rustfmt.toml | insert next_edition 2021 | save rustfmt2.toml
open rustfmt2.toml
# => ──────────────┬──────
# =>  edition      │ 2018
# =>  next_edition │ 2021
# => ──────────────┴──────
```

### Updating a Column

In a similar way to the [`insert`](/commands/docs/insert.md) command, we can also use the [`update`](/commands/docs/update.md) command to change the contents of a column to a new value. To see it in action let's open the same file:

```nu
open rustfmt.toml
# => ─────────┬──────
# =>  edition │ 2018
# => ─────────┴──────
```

And now, let's update the edition to point at the next edition we hope to support:

```nu
open rustfmt.toml | update edition 2021
# => ─────────┬──────
# =>  edition │ 2021
# => ─────────┴──────
```

You can also use the [`upsert`](/commands/docs/upsert.md) command to insert or update depending on whether the column already exists.

### Moving Columns

You can use [`move`](/commands/docs/move.md) to move columns in the table. For example, if we wanted to move the "name" column from [`ls`](/commands/docs/ls.md) after the "size" column, we could do:

```nu
ls | move name --after size
# => ╭────┬──────┬─────────┬───────────────────┬──────────────╮
# => │ #  │ type │  size   │       name        │   modified   │
# => ├────┼──────┼─────────┼───────────────────┼──────────────┤
# => │  0 │ dir  │   256 B │ Applications      │ 3 days ago   │
# => │  1 │ dir  │   256 B │ Data              │ 2 weeks ago  │
# => │  2 │ dir  │   448 B │ Desktop           │ 2 hours ago  │
# => │  3 │ dir  │   192 B │ Disks             │ a week ago   │
# => │  4 │ dir  │   416 B │ Documents         │ 4 days ago   │
# => ...
```

### Renaming Columns

You can also [`rename`](/commands/docs/rename.md) columns in a table by passing it through the rename command. If we wanted to run [`ls`](/commands/docs/ls.md) and rename the columns, we can use this example:

```nu
ls | rename filename filetype filesize date
# => ╭────┬───────────────────┬──────────┬──────────┬──────────────╮
# => │ #  │     filename      │ filetype │ filesize │     date     │
# => ├────┼───────────────────┼──────────┼──────────┼──────────────┤
# => │  0 │ Applications      │ dir      │    256 B │ 3 days ago   │
# => │  1 │ Data              │ dir      │    256 B │ 2 weeks ago  │
# => │  2 │ Desktop           │ dir      │    448 B │ 2 hours ago  │
# => │  3 │ Disks             │ dir      │    192 B │ a week ago   │
# => │  4 │ Documents         │ dir      │    416 B │ 4 days ago   │
# => ...
```

### Rejecting/Deleting Columns

You can also [`reject`](/commands/docs/reject.md) columns in a table by passing it through the reject command. If we wanted to run [`ls`](/commands/docs/ls.md) and delete the columns, we can use this example:

```nu
ls -l / | reject readonly num_links inode created accessed modified
# => ╭────┬────────┬─────────┬─────────┬───────────┬──────┬───────┬────────╮
# => │  # │  name  │  type   │ target  │   mode    │ uid  │ group │  size  │
# => ├────┼────────┼─────────┼─────────┼───────────┼──────┼───────┼────────┤
# => │  0 │ /bin   │ symlink │ usr/bin │ rwxrwxrwx │ root │ root  │    7 B │
# => │  1 │ /boot  │ dir     │         │ rwxr-xr-x │ root │ root  │ 1.0 KB │
# => │  2 │ /dev   │ dir     │         │ rwxr-xr-x │ root │ root  │ 4.1 KB │
# => │  3 │ /etc   │ dir     │         │ rwxr-xr-x │ root │ root  │ 3.6 KB │
# => │  4 │ /home  │ dir     │         │ rwxr-xr-x │ root │ root  │   12 B │
# => │  5 │ /lib   │ symlink │ usr/lib │ rwxrwxrwx │ root │ root  │    7 B │
# => │  6 │ /lib64 │ symlink │ usr/lib │ rwxrwxrwx │ root │ root  │    7 B │
# => │  7 │ /mnt   │ dir     │         │ rwxr-xr-x │ root │ root  │    0 B │
# => ...
```

### The # Index Column

You've noticed that every table, by default, starts with a column with the heading `#`. This column can operate in one of two modes:

1. Internal #

   - The default mode
   - Nushell provides a 0-based, consecutive index
   - Always corresponds to the cell-path row-number, where `select 0` will return the first item in the list, and `select <n-1>` returns the nth item
   - Is a display of an internal representation only. In other words, it is not accessible by column name. For example, `get index` will not work, nor `get #`

1. "Index"-Renamed #

   - When a column named "index" is created, either directly or as a side-effect of another command, then this `index` column takes the place of the `#` column in the table display. In the table output, the column header is still `#`, but the _name_ of the column is now `index`.

     Example:

     ```nu
     ls | each { insert index { 1000 }} | first 5
     # => ╭──────┬─────────────────┬──────┬─────────┬──────────────╮
     # => │    # │      name       │ type │  size   │   modified   │
     # => ├──────┼─────────────────┼──────┼─────────┼──────────────┤
     # => │ 1000 │ CNAME           │ file │    15 B │ 9 months ago │
     # => │ 1000 │ CONTRIBUTING.md │ file │ 4.3 KiB │ 9 hours ago  │
     # => │ 1000 │ LICENSE         │ file │ 1.0 KiB │ 9 months ago │
     # => │ 1000 │ README.md       │ file │ 2.2 KiB │ 3 weeks ago  │
     # => │ 1000 │ assets          │ dir  │ 4.0 KiB │ 9 months ago │
     # => ╰──────┴─────────────────┴──────┴─────────┴──────────────╯
     ```

     - If an `index` key is added to each row in the table, then it can be accessed via `select` and `get`:

     ```nu
     ls | each { insert index { 1000 }} | first 5 | select index name
     # => ╭──────┬─────────────────╮
     # => │    # │      name       │
     # => ├──────┼─────────────────┤
     # => │ 1000 │ CNAME           │
     # => │ 1000 │ CONTRIBUTING.md │
     # => │ 1000 │ LICENSE         │
     # => │ 1000 │ README.md       │
     # => │ 1000 │ assets          │
     # => ╰──────┴─────────────────╯
     ```

     - On the other hand, if some rows have an `index` key and others don't, the result is no longer a table; it is a `list<any>` due to the different record types:

       ```nu
       ls | upsert 3.index { "--->" } | first 5
       # => ╭──────┬─────────────────┬──────┬─────────┬──────────────╮
       # => │    # │      name       │ type │  size   │   modified   │
       # => ├──────┼─────────────────┼──────┼─────────┼──────────────┤
       # => │    0 │ CNAME           │ file │    15 B │ 9 months ago │
       # => │    1 │ CONTRIBUTING.md │ file │ 4.3 KiB │ 9 hours ago  │
       # => │    2 │ LICENSE         │ file │ 1.0 KiB │ 9 months ago │
       # => │ ---> │ README.md       │ file │ 2.2 KiB │ 3 weeks ago  │
       # => │    4 │ assets          │ dir  │ 4.0 KiB │ 9 months ago │
       # => ╰──────┴─────────────────┴──────┴─────────┴──────────────╯

       ls | upsert 3.index { "--->" } | first 5 | describe
       # => list<any> (stream)

       ls | upsert 3.index { "--->" } | select index name
       # Error: cannot find column 'index'

       ls | upsert 3.index { "--->" } | select index? name | first 5
       # => ╭──────┬─────────────────╮
       # => │    # │      name       │
       # => ├──────┼─────────────────┤
       # => │      │ CNAME           │
       # => │      │ CONTRIBUTING.md │
       # => │      │ LICENSE         │
       # => │ ---> │ README.md       │
       # => │      │ assets          │
       # => ╰──────┴─────────────────╯
       ```

   - As demonstrated in the example above, any rows (records) in the table without an `index` key will continue to display the internal representation.

#### Additional Index Examples

##### Convert # to Index

A useful pattern for converting an internal `#` into an index for all rows, while maintaining the original numbering, is:

```nu
ls | enumerate | flatten
```

While the results _look_ the same, the `index` is now decoupled from the internal cell-path. For example:

```nu
ls | enumerate | flatten | sort-by modified | first 5
# => ╭────┬──────────────┬──────┬─────────┬──────────────╮
# => │  # │     name     │ type │  size   │   modified   │
# => ├────┼──────────────┼──────┼─────────┼──────────────┤
# => │  0 │ CNAME        │ file │    15 B │ 9 months ago │
# => │  2 │ LICENSE      │ file │ 1.0 KiB │ 9 months ago │
# => │  4 │ assets       │ dir  │ 4.0 KiB │ 9 months ago │
# => │ 17 │ lefthook.yml │ file │ 1.1 KiB │ 9 months ago │
# => │ 24 │ snippets     │ dir  │ 4.0 KiB │ 9 months ago │
# => ╰────┴──────────────┴──────┴─────────┴──────────────╯

ls | enumerate | flatten | sort-by modified | select 4
# => ╭────┬──────────┬──────┬─────────┬──────────────╮
# => │  # │   name   │ type │  size   │   modified   │
# => ├────┼──────────┼──────┼─────────┼──────────────┤
# => │ 24 │ snippets │ dir  │ 4.0 KiB │ 9 months ago │
# => ╰────┴──────────┴──────┴─────────┴──────────────╯
```

The `sort-by modified` now _also_ sorts the `index` along with the rest of the columns.

##### Adding a Row Header

```nu
let table = [
[additions   deletions   delta ];
[       10          20     -10 ]
[       15           5      10 ]
[        8           6       2 ]]

let totals_row = ($table | math sum | insert index {"Totals"})
$table | append $totals_row
# => ╭────────┬───────────┬───────────┬───────╮
# => │      # │ additions │ deletions │ delta │
# => ├────────┼───────────┼───────────┼───────┤
# => │      0 │        10 │        20 │   -10 │
# => │      1 │        15 │         5 │    10 │
# => │      2 │         8 │         6 │     2 │
# => │ Totals │        33 │        31 │     2 │
# => ╰────────┴───────────┴───────────┴───────╯
```

### The `table` command

The [`table`](/commands/docs/table.md) command is used to _render_ structured data. This includes:

- Tables
- Lists
- Records
- Ranges

Perhaps contrary to initial assumptions, the result of rendering these types is a `string`. For example:

```nu
[ Nagasaki Ghent Cambridge Izmir Graz Lubango ] | table | describe
# => string (stream)
```

Other data types are passed through the `table` command unchanged.

With no arguments, the output rendered from the `table` command will often _display_ the same as unrendered form. For example:

```nu
[ Nagasaki Ghent Cambridge Izmir Graz Lubango ]
# => ╭───┬───────────╮
# => │ 0 │ Nagasaki  │
# => │ 1 │ Ghent     │
# => │ 2 │ Cambridge │
# => │ 3 │ Izmir     │
# => │ 4 │ Graz      │
# => │ 5 │ Lubango   │
# => ╰───┴───────────╯
[ Nagasaki Ghent Cambridge Izmir Graz Lubango ] | table
# => ╭───┬───────────╮
# => │ 0 │ Nagasaki  │
# => │ 1 │ Ghent     │
# => │ 2 │ Cambridge │
# => │ 3 │ Izmir     │
# => │ 4 │ Graz      │
# => │ 5 │ Lubango   │
# => ╰───┴───────────╯
```

This can be useful when you need to store the rendered view of structured data as a string. For example, to remove all ANSI formatting (colors) from a table:

```
ls | table | ansi strip
```

The `table` command also has multiple options for _changing_ the rendering of a table, such as:

- `-e` to expand data that would normally be collapsed when rendering. Compare `scope modules | table` to `scope modules | table -e`.
- `-i false` to hide the `index`/`#` column
- `-a 5` to abbreviate the table to just the first and last 5 entries
- And more
