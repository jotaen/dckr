# dckr

*dckr* is a bash utility that makes it quick and convenient to start up a docker container based on predefined configurations. The primary use-case is when you regularly spawn up docker containers on your local machine as a way to execute something in an encapsulated environment.

Basically, it’s a wrapper around `docker run ...`: if you repeatedly find yourself typing `docker run --rm -it -v $(pwd):~ ...`, then *dckr* is for you! It doesn’t do any magic, it’s just meant for lazy people to save some key strokes.

## Get dckr

1. Download the [dckr](./dckr) bash script
2. Make it executable: `chmod +x dckr`
3. Put it in your path: `mv dckr /usr/local/bin`
4. Create a config file: `touch ~/.dckr`

## Usage

Given you have a dckr configuration called “something”: whenever you run `dckr something` then *dckr* will start a container for you…

- from the image that you had associated with “something”
- in interactive mode (`--interactive`)
- with a TTY attached (`--tty`)
- transiently (`--rm`)
- with your current working directory mounted into it as volume
- with all additional arguments from your configuration file
- with any additional arguments that you pass via the command line

### Config file

You can specify a configuration file (`~/.dckr`) in your home folder and put your regularly used configs there:

```ini
[python]
    image = python:3.7-buster
    args = --entrypoint "/bin/bash"
```

### Command line utility

Based on the config file above you could now run:

```bash
dckr python -p 8080:8080
```

That would start a docker container from image `python:3.7-buster` with the entrypoint `/bin/bash` and the port binding `8080:8080`.

You can inspect the exact command that gets executed by passing `-v` after `dckr`. Or run `dckr --help` for additional information.
