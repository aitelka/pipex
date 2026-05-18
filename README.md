# pipex

A C implementation of Unix pipe behavior, replicating the shell construct:
```
< infile cmd1 | cmd2 > outfile
```

## Usage

### Mandatory
```sh
./pipex infile "cmd1 [args]" "cmd2 [args]" outfile
```
Equivalent to:
```sh
< infile cmd1 [args] | cmd2 [args] > outfile
```

### Bonus — multiple pipes
```sh
./pipex_bonus infile "cmd1" "cmd2" ... "cmdN" outfile
```

### Bonus — here_doc
```sh
./pipex_bonus here_doc LIMITER "cmd1" "cmd2" outfile
```
Reads from stdin until `LIMITER` is encountered, then pipes through the commands, appending to `outfile`.

## Build

```sh
make          # build mandatory (./pipex)
make bonus    # build bonus    (./pipex_bonus)
make clean    # remove object files
make fclean   # remove object files and binaries
make re       # fclean + all
```

## Examples

```sh
./pipex /etc/passwd "grep root" "cut -d: -f1" output.txt
# equivalent to: < /etc/passwd grep root | cut -d: -f1 > output.txt

./pipex_bonus infile "ls -la" "grep .c" "wc -l" outfile
# chains three commands through two pipes

./pipex_bonus here_doc EOF "cat" "wc -l" outfile
# reads stdin lines until EOF, counts them, appends result to outfile
```

## Project structure

```
pipex/
├── mandatory/
│   ├── include/pipex.h
│   ├── main.c
│   └── src/
│       ├── parsing.c       # command string parsing (handles quotes)
│       ├── cmd_list.c      # linked list of commands
│       ├── executing.c     # fork/pipe/exec logic
│       ├── pipex_utils.c   # PATH resolution
│       ├── arr_utils.c     # string array helpers
│       └── assert.c        # error handling wrappers
├── bonus/
│   ├── include/pipex_bonus.h
│   ├── main_bonus.c        # here_doc + multi-pipe entry point
│   └── src/                # mirrors mandatory with bonus extensions
└── libs/
    └── libft/              # custom C standard library
```

## Key concepts

- **fork/exec** — each command runs in a child process
- **pipe(2)** — anonymous pipes connect stdout of one command to stdin of the next
- **dup2(2)** — redirects file descriptors for file I/O and pipe chaining
- **PATH resolution** — searches `PATH` env variable to find command executables
- **here_doc** — collects stdin into a temp file until the limiter line, then feeds it as input
