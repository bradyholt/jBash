# jBash

Helpers for Bash like shell scripting in JavaScript

[![Build Status](https://travis-ci.org/bradyholt/jBash.svg?branch=master)](https://travis-ci.org/bradyholt/jBash) [![NPM Package](https://img.shields.io/npm/v/jbash.svg)](https://www.npmjs.com/package/jbash)


Write your shell scripts using jBash and you get the best of both worlds: JavaScript and Bash.  jBash is a small JavaScript library that provides helper aliases and functions that are similar to Bash syntax, allowing you to write shell scripts in JavaScript that are simple and familiar.

## Quick Start

Create a file called script.js:

```
#!/usr/bin/env npx jbash

echo("Hello jBash")
```

Make the file executable, run it, and you should see "Hello jBash" printed:

```
chmod +x script.js
./script.js

> Hello jBash
```

## Helpers

| Bash                      | jBash                            | notes                                   |
|---------------------------|--------------------------------------|-----------------------------------------|
| ``echo "Hello"``              | ``echo("Hello")``                | print text to console |
| ``$1, $2, ...``               | ``$1; $2; ...``                  | $1, $2, etc. variables contain args passed in |
|                               | ``args[0], args[1]``             | arguments passed in are also in args array |
| ``$0``                        | ``$0``                           | file path of current script |
| ``$HOME``                     | ``$HOME``                        | environment variables |
|                               | ``env.HOME``                     | all env variables are also mapped on env var  |
| ``set -x``                    | ``set("-x")``                    | echos all commands  |
|                               | ``options.xtrace=true``          | alternative to set("-x")  |
| ``set -e``                    | ``set("-e")``                    | throw when a command exits with non-zero status |
|                               | ``options.errexit=true``         | alternative to set("-x")  |
| ``cd "/usr/bin"``             | ``cd("/usr/bin")``               | change current working directory |
| ``exit 1``                    | ``exit(1)``                      | exit with code |
| ``result=$(cmd.sh)``          | ``result=$(`cmd.sh`)``           | $(...) buffers output as return value |
| ``eval ping g.cn``            | ``eval(`ping g.cn`)``            | eval() streams output to console (no return value) |
|                               | ``exec(`ping g.cn`)``            | exec() is an alias for eval() |
| ``config=$(cat cnf.txt)``     | ``config=$(`cat cnf.txt`)``      | read text from file |
|                               | ``config=readFile(`cnf.txt`)``   | readFile reads file to text |
| ``echo $cnf > cnf.txt``       | ``$(`echo ${cnf} > cnf.txt`)``   | save text to file |
|                               | ``writeFile(`cnf.txt`, cnf)``    | writeFile saves text to file |

## Command Execution

To run commands in jBash you can use either the **$()** or **eval()** helper, depending upon whether you need to capture the output of the commands.  Both of these commands are synchronous and will block execution until completion.

### $()

When you want to run a command and buffer the output (stdout) of that command as a return value, you'll want to use **$()**.  As the command is running, stdout will _not_ be printed to the console but will instead be captured and returned as the result.  This helper is intended for short running commands that do not produce a large amount of output.  Example: To grab the output of `git status --porcelain` and store in variable named `result`:

```
// Will wait for `git status` to complete and assign
// output to result variable.  Nothing will be
// printed to the console.

let result=$(`git status --porcelain`);
```

### eval()

**eval()** should be used for running commands where the output does not need to be captured, but only printed to the console.  This helper is intended for long running commands or those where output does not need to be captured.  Example: To run `npm install`:

```
// Will print `npm install` output immediately as it happens
// eval() will return null

eval(`npm install`)
```

### Reading and Writing Files

In Bash, reading files is usually done with the `cat` command (i.e. `config=$(cat cnf.txt)`) and writing to files is usally done with `cat` or `echo` and piping (i.e. `echo $cnf > cnf.txt`).  The same approaches can be used in jBash, using `$()` (reading: ``config=$(`cat cnf.txt`)``; writing: ``$(`echo ${cnf} > cnf.txt`)``).  But jBash also provides a `readFile` and `writeFile` helper for convenience.

Example

<pre>
config=readFile(`cnf.txt`)
writeFile(`cnf.txt`, cnf)
</pre>


### Error Handling

If a command exits with a non-zero status, the stderr will be echoed on console.  If the command was run with `$()`, the stderr will also be returned.  To throw an error and therefore halt the script unless the error is handled with `try / catch`, you can enable errexit option by calling `set("-e")` which behaves just like `set -e` in Bash.  The error will have properties `{ message, status, stderr }` that contain detail of the error.

Example:

```
// This command will error but will not throw because errexit is not enabled
// The string "cat: invalid.txt: No such file or directory" will be returned and
// assigned to `content`.
let content=$(`cat invalid.txt`)

// Turn on errexit
set("-e")

try {
  // This command will throw because errexit is enabled
  eval(`cat invalid.txt`)
} catch (err) {
  console.log(err.status) // 1
  console.log(err.stderr) // "cat: invalid.txt: No such file or directory"
}
```

## Installation

### npx

By far the easiest way to use jBash is with a [npx](https://github.com/zkat/npx) [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) which will allow your script to be executed directly without an npm install needed.

```
#!/usr/bin/env npx jbash

echo("Hello jBash")
```


### npm global install

If you don't want to use npx, you can install jBash globally with npm:

```
npm install -g jbash
```

Once it is installed globally, you can write your script with a jBash [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) which will allow your script to be executed directly, with the globally installed jBash loaded at runtime.

```
#!/usr/local/bin/jbash

echo(`Hello jBash`)
```

### require

Rather than installing jBash globally, you can simply download it to a local folder and reference it directly from your script using a `require` statement.  This is a good option for scripts running on a remote system where you may not have the ability to use npx or be able to install npm packages globally.  Node.js will still need to be available, though.

First, download jBash:

```
wget -O jbash.js https://raw.githubusercontent.com/bradyholt/jbash/master/index.js
```

Then, in your script:

```
#!/usr/bin/env node
require('./jbash.js)

echo(`Hello jBash`)
```

## It's Still JavaScript

When you write your shell scripts in jBash, you get to use a simple Bash like syntax but remember, it's still JavaScript!  This means you can install npm packages and use them to your ❤️'s content.

Example:

```
npm install uuid
```

```
#!/usr/local/bin/jbash
require('uuid/v4')

echo(uuidv4()) // -> '110ec58a-a0f2-4ac4-8393-c866d813b8d1'
```

## Examples

Looking at real-world scripts is a good way to get a feel for the library.

- [jBash release script](https://github.com/bradyholt/jBash/blob/master/scripts/release.js)
- [Cron Expression Descriptor release script](https://github.com/bradyholt/cron-expression-descriptor/blob/master/scripts/release.js)
