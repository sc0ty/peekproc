# PeekProc
Peek at process details.
It will print executed commands (with arguments) and other information about given process and its childs.

PeekProc is similar to [snoopy](https://github.com/a2o/snoopy), but no library injection is needed.

## Features
 - monitor commands executed by given processes and by its childs;
 - monitor command arguments, working directory, process and parent PIDs, owner information and more;
 - monitor process variables;
 - configurable set of parameters to report;
 - configurable output format.

## Installation
It is Python script therefore no compilation is needed. It depends on `psutil` version 2.0 or newer.
Note that Ubuntu 14.04 provides version 1.5 of this package, but you still could install current version with `pip`:
	```
	sudo apt update
	sudo apt install python-pip
	sudo pip install psutil
	```
PeekProc is compatible with Python 2.x and 3.x.

## Usage
`peekproc PID [PID ...] [-h] [-n INTERVAL] [-q] [-s] [-a ATTR [ATTR ...]] [-e ENV [ENV ...]] [-f FORMAT]`
Arguments:
	- `PID` - one or more process ID to peek at;
	- `-h`, `--help`	- print help message;
	- `-n INTERVAL`, `--interval INTERVAL`	- update interval in seconds, default is 0.2;
	- `-q`, `--quick`	- run without delay between scans (could result in high CPU load);
	- `-s`, `--single`	- single pass, scan process tree once and exit;
	- `-a ATTR [ATTR ...]`, `--attrs ATTR [ATTR ...]`	- one or more attribute to read; list available attributes with `peekproc -h`, for detailed description see [psutil documentation](http://pythonhosted.org/psutil/#process-class);
	- `-e ENV [ENV ...]`, `--env ENV [ENV ...]`	- environment variables to read;
	- `-f FORMAT`, `--format FORMAT`	- output format string composed with Python [format string syntax](https://docs.python.org/2/library/string.html#format-string-syntax); list available field names with `peekproc -h`.

## Examples
- monitor process 4872:
	```
	sc0ty@lap:~ $peekproc 4872
	0.002:   4872  tmux
	0.030:   4873  zsh
	0.059:   6830  vim peekproc
	0.115:   6999  git gui
	```
- extra information about process using custom format:
	```
	sc0ty@lap:~ $peekproc 4872 -f "{time:.3f}: pid:{pid}, name:{name}, exe:{exe}, ppid:{ppid}, cwd:{cwd}, user:{username}, '{cmd}'"
	0.002: pid:4872, name:tmux, exe:/usr/local/bin/tmux, ppid:2835, cwd:/home/sc0ty, user:sc0ty, tmux
	0.031: pid:4873, name:zsh, exe:/bin/zsh5, ppid:4872, cwd:/home/sc0ty/projects/peekproc, user:sc0ty, zsh
	0.058: pid:6830, name:vim, exe:/usr/local/bin/vim, ppid:4873, cwd:/home/sc0ty/projects/peekproc, user:sc0ty, 'vim peekproc'
	0.114: pid:6999, name:git, exe:/usr/bin/git, ppid:6943, cwd:/home/sc0ty/projects/peekproc, user:sc0ty, 'git gui'
	```
- extra information about process (multi-line format):
	```
	sc0ty@lap:~ $peekproc 8089 --attrs cwd exe ppid --env SHELL TERM
	0.002:   8089  zsh
		cwd:         '/home/sc0ty/projects/peekproc'
		exe:         '/bin/zsh5'
		ppid:        4872
		$SHELL:      '/usr/bin/zsh'
		$TERM:       'screen-256color'

	38.745:   8147  htop
		cwd:         '/home/sc0ty/projects/peekproc'
		exe:         '/usr/bin/htop'
		ppid:        8089
		$SHELL:      '/usr/bin/zsh'
		$TERM:       'screen-256color'

	53.074:   8151  /usr/local/bin/mc -P /tmp/mc-sc0ty/mc.pwd.8089
		cwd:         '/home/sc0ty/projects/peekproc'
		exe:         '/usr/local/bin/mc'
		ppid:        8089
		$SHELL:      '/usr/bin/zsh'
		$TERM:       'screen-256color'
	```

## Limitations
1. This tool is gathering information by periodically polling the data. It could result with not reporting every changes. Especially quickly exiting tasks (like `ls`) will probably be missed. One could tune the scan interval with `-n` parameter, or even disable the delay completely with `-q`, but still this limitation will occur.
Note that using parameter `-q` or `-n` with very small interval will result with high CPU load up to 100% on single core.
2. For given process only command line string is monitored. Changes of other parameters alone will not trigger printing. E.g. execution of `peekproc PID -a cwd` will report current directory changes only if command line will change as well.

