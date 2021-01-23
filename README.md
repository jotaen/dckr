# dckr 🐳

*dckr* is a bash utility that makes it quick and convenient to start up docker containers based on predefined configurations. It’s meant for you if you regularly use containers as part of your local workflow, for example to spawn up disposable runtime environments or to execute utilities in an isolated container. If you frequently find yourself typing `docker run ...` with the same list of arguments all over again (like `-it`, `--rm`, … and, hang on, what was that image called again?), then *dckr* is for you.

Under the hood, *dckr* is a thin wrapper around `docker run`. You can store run configurations in a dot-file and refer to them via short aliases. (It’s similar to bash aliases, but more flexible.) Upon invocation the respective parameters get read from the config file and applied to `docker run` along with some sensible defaults. There is no further magic to it: *dckr* is just a convenience for lazy people to save some key strokes – nothing more, nothing less.

## Get started

1. [Download the dckr bash script](https://raw.githubusercontent.com/jotaen/dckr/main/dckr) (via right-click, save as)
2. Make it executable: `chmod +x dckr`
3. Put it in your path, e.g.: `mv dckr /usr/local/bin`
4. Create a config file at `~/.dckr`

## Usage

The configurations for *dckr* live in a dot-file called `.dckr` in your home directory. Let’s say your config file looks like this:

```ini
; Python development environments
[python:2]
    image = python:2.7-buster
    args = --entrypoint "/bin/bash"
[python:3]
    image = python:3.9-buster
    args = --entrypoint "/bin/bash"

; Local file server in current directory
[serve]
    image = nginx:1.18.0-alpine
    args = -p 8080:80 -v $(pwd):/usr/share/nginx/html:ro

; Pretty print a json file
[json]
    image = node:14.15.1-buster
    cmd = -e 'console.log(JSON.stringify(require("./"+process.argv[1]),undefined,2))'
```

This would allow you to run the following commands:

- `dckr python:2` (or `dckr python:3` respectively) opens a bash inside a python development environment
- `dckr serve` starts an nginx container and exposes the files in your working directory under localhost:8080
- `dckr json -- mydata.json` pretty prints a JSON file onto the console

The first two examples would be typical when you utilise docker as a way to start up and “jump into” disposable and encapsulated development environments. Advantages of employing such a workflow are a) that these environments are isolated from host system (apart from the current working directory), and b) that you don’t need to maintain an armada of different versions and runtime dependencies on your computer.

The last two examples demonstrate how you can define configurations for when you want to use docker containers in the same way as command line tools, except that you a) don’t need to install any binaries on your host machine, and b) the utility doesn’t get full access to your computer.

That being said, the aim of *dckr* is to facilitate such use-cases and make them more accessible.

## Guide

The configuration reside in `~/.dckr` and is “ini” format. For every *dckr* alias you create a section (e.g. `[myalias]`) with the following properties: “image” (mandatory), “args” (optional), and “cmd” (optional). All indentation is optional. Lines can be commented out by putting a `;` at the beginning of the line.

Upon invocation, *dckr* looks into the respective section of your config file and runs a container for you …

- … based off the image that you have configured via `image = …`
- … with the following predefined arguments (always provided by *dckr*):
    - in interactive mode: `--interactive`
    - with TTY attached: `--tty`
    - in transient mode: `--rm` (container gets cleaned up after exit)
    - with the current working directory mounted into the container at `/dckr`
    - with the container workdir set to `/dckr`
- … (if specified) with the additional arguments configured via `args = …`
- … (if specified) with the command configured via `cmd = …`
- … (if passed) with any given flags as additional arguments (e.g. `dckr python:3 -e PYTHONPATH=/dckr/libs`)
- … (if passed) with any given input appearing after a ` -- ` separator as additional command (e.g. `dckr python:3 -- python main.py`)

If you want to review the exact command that gets executed you can invoke *dckr* with the `-v` flag. With the sample configuration above for example, `dckr -v python:3 --memory 2g` would output (and execute):

```shell
docker run --rm --tty --interactive --workdir=/dckr --volume=/home/foo/bar:/dckr --entrypoint /bin/bash --memory 2g python:3.9-buster
```

You can also run `dckr --help` to see all available options.

## About

*dckr* is free and open-source software distributed under the [MIT License](LICENSE.txt).
