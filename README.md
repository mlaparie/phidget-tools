# phidget-tools
__*Easy recording/monitoring from Phidget 1048_1B USB-thermocouple devices directly from your comfy terminal*__

It aims at providing a fast, flexible and efficient way to record and monitor thermocouple data in situations where a computer (including a tiny and inexpensive one like the Pi Zero) is in reach. This saves the usual hassle of using GUI and/or proprietary software with limited features from commercial providers, having to set recording missions without live feedback, and then unload and view the data only when the experiment is over.

In addition to better portability and compatibility with headless machines, using command line tools allows easily automating execution for complex monitoring cycles, when combined with other tools such as `crontab` and `ssh`, for instance.

<p align="center">
    <a href="https://asciinema.org/a/447554">
        <img width="800" src="https://github.com/mlaparie/phidget-tools/raw/main/pics/phidget-rec.svg">
    </a>
</p>

<p align="center">
    <a href="https://raw.githubusercontent.com/mlaparie/phidget-tools/main/pics/phidget-plot.mp4">
    <img width="1000" src="https://github.com/mlaparie/phidget-tools/raw/main/pics/phidget-plot.gif">
    </a>
</p>

## Usage
- `phidget-rec` (Python)

```txt
$ phidget-rec -h
Usage: $ phidget-rec [options]

A program to facilitate recording and monitoring of Phidget 1048_1B USB-
thermocouple devices directly from the comfort of your favourite terminal

Options:
  -h, --help            show this help message and exit
  -i, --interactive     Enable interactive input prompts for all options
  -m MODE, --mode=MODE  [d]uration, scheduled end [t]ime, or [c]ontinuous
                        until manually stopped, default=d
  -r RATE, --rate=RATE  Poll rate in seconds (min 0.1), default=1
  -n NAME, --name=NAME  Your name; will be prefixed to the output file name to
                        facilitate sorting data on shared computers
  -l LABEL, --label=LABEL
                        Label (experiment, species, etc.) to include in the
                        csv file name, default=Phidget S/N
  -t TAG, --tag=TAG     Add any relevant tags (keywords, etc.) or summary
                        information to the csv description; use quotes in case
                        of multiple words
  -s SEP, --sep=SEP     Column separator, either [c]omma, [t]ab, or [s]pace,
                        default=c
  -f FILENAME, --file=FILENAME
                        Custom output file name (ignores --name and --label),
                        default=NAME_LABEL_RRATE_MODEDURATION_DATETIME.csv
  -p PHIDGET, --phidget=PHIDGET
                        6-digit serial number of the Phidget to use, if
                        multiple Phidgets 1048_1B are connected
  -d DAYS, --days=DAYS  Number of days to record, meant for use in combination
                        with duration mode in a non-interactive and therefore
                        easily programmable way
  -y, --yes             Skip confirmation prompt before recording data, meant
                        to allow suppressing all interactive prompts when
                        options are set in command line to facilitate the
                        automatic execution of the program
  -q, --quiet           Hide sensor readings while recording

Improve me at https://github.com/mlaparie/phidget-tools
```

- `phidget-plot` (Bash)

```txt
$ phidget-plot -h
phidget-plot (live) plots csv data from phidget-rec directly in the terminal.

Usage: $ phidget-plot [FILE] [CHANNEL(S)]

       [FILE]        path to input csv file
       [CHANNEL(S)]  space-separated list of channel(s) from 0 to 4 to plot, e.g., 0 1 2 3 4

       Arguments are facultative, suppling none enables interactive mode (requires fzf).
       Interactive mode looks for input csv files in present directory and takes no [CHANNEL] argument.

       Refresh rate of live plots defaults to 0.5s but can be overriden with
       an environment variable: 'phidgetplotrate=60 phidget-plot [ARGS]'

Improve me:
   https://github.com/mlaparie/phidget-tools
```

## Getting started
### Installation

```bash
git clone https://github.com/mlaparie/phidget-tools && cd phidget-tools
ln -s phidget-rec ~/.local/bin/phidget-rec
ln -s phidget-plot ~/.local/bin/phidget-plot
```

### Dependencies
#### `phidget-rec`
- `python3`
- Phidget libraries: follow "[Part 1](https://www.phidgets.com/?tier=3&catid=14&pcid=12&prodid=120R0)" instructions from the official website
- `Phidget22` Python library: `pip3 install Phidget22`

#### `phidget-plot`
- [`jp`](https://github.com/sgreben/jp)
  * `jp` can be installed with Go: `go get -u github.com/sgreben/jp/cmd/jp`
  * Alternatively, prebuilt binaries are [available](https://github.com/sgreben/jp/releases), just unzip the version for your architecture either in your `$PATH` (mandatory if you symlinked `phidget-plot` to your `$PATH` as described above and execute it from outside `phidget-tools/`), or else simply in the `phidget-tools/` directory if you run the script from there only.
- [`fzf`](https://github.com/junegunn/fzf), optional but phidget-tools will lose functionality without it.
  `fzf` is typically available in any distribution's package manager:
  * Debian: `sudo apt install fzf`
  * Solus: `sudo eopkg it fzf`
  * macOS: `brew install fzf`

### How to use
Using both scripts should be straightforward, check `--help`. In short, plug your Phidget device, run `phidget-rec` (try the `--interactive` ot `-i`  option if in doubt) to collect data, and run `phidget-plot` to select and preview previously recorded data or view live plots for ongoing monitorings. Make sure your Phidget has thermocouples connected on all four TC channels (0 to 3) because errors may arise if some channels cannot collect data.

While `phidget -i` will prompt the user for all available options (except selecting a Phidget when several are connected, because I could not test it, but adding `-p SERIAL` should work), one advantage for CLI tools is to suppress all interactions to run them from other scripts and tools. Just make sure you provide the compulsory arguments `-n NAME`, `-d DAYS` (though inconvenient, it is a float, so fractions are possible: `-d 0.25` is six hours) and `-y` to skip the final confirmation. `-r RATE` is not mandatory but will obviously be set by most users.

The example Bash script below cycles the execution of `phidget-rec` for `N` phases (first argument), each separated by `WAIT` days. These two arguments have to be given first and in that order, and followed by at least the compulsory arguments for `phidget-rec` (in any order) to suppress all input prompts and fully automate the whole routine. More complex scheduling could be achieved using `cron` for instance.

```bash
#!/usr/bin/env bash
# This script should be given 2 arguments: number of phases, wait time (in days) between phases.
# Both arguments should be separated by a space, and then followed by phidget-rec options
# example: sh cycle-rec.sh 6 0.0001157407 -nDoe -r0.2 -yd0.0001157407
# Note that the script overrides the --label argument of phidget-rec to name file after their cycle number

TMPFILE="$(mktemp)"
CYCLE=0
N="$1"
DELAY="$(echo $(bc <<< "$2 * 86400 + 0.5")/1 | bc)"

printf "\033[33;1mInitiated a cycle of "$N" phases each separated by "$2" days…\033[0m\n"
echo 0 > $TMPFILE
		shift 2
	while [ "$CYCLE" -lt "$N" ]
	do
		CYCLE=$[$(cat $TMPFILE) + 1]
		phidget-rec "$@" -l"$CYCLE""of""$N" 1>/dev/null # Remove "1>/dev/null" if you want to see phidget-rec output
		printf "\033[33;1m"$CYCLE"/"$N" completed.\033[0m\n"
		echo $CYCLE > $TMPFILE
		sleep "$DELAY"
	done
```

```bash
$ sh cycle-rec.sh 6 0.0001157407 -nDoe -r0.2 -yd0.0001157407
Initiated a cycle of 6 phases each separated by 0.0001157407 days…
Phase 1/6 over.
Phase 2/6 over.
Phase 3/6 over.
Phase 4/6 over.
Phase 5/6 over.
Phase 6/6 over.

$ tree Data/
Data/
└── Doe
    ├── Doe_1of6_r0.2_D0.0d_20211108-133540.csv
    ├── Doe_2of6_r0.2_D0.0d_20211108-133600.csv
    ├── Doe_3of6_r0.2_D0.0d_20211108-133620.csv
    ├── Doe_4of6_r0.2_D0.0d_20211108-133641.csv
    ├── Doe_5of6_r0.2_D0.0d_20211108-133701.csv
    └── Doe_6of6_r0.2_D0.0d_20211108-133722.csv

1 directory, 6 files

$ head Data/Doe/Doe_phase1of6_r0.2_D0.0d_20211108-133540.csv -n 14
"Hostname: xiaomimi-solus"
"Phidget: 647540"
"Name: Doe"
"Tag(s) or info: NA"
"Label: 1of6"
"Poll rate: 0.2s"
"Program initialization: 2021-11-08 13:35:40"
"Scheduled to record until: 2021-11-08 13:35:49.999996"
"chan0 to chan3: K-type thermocouples"
"chan4: internal temperature"

"time","chan0","chan1","chan2","chan3","chan4"
"2021-11-08 13:35:40.402383","24.8175","24.945","25.0146","25.178","26.66"
"2021-11-08 13:35:40.602828","24.83","24.94","25.0146","25.178","26.66"
…
```

This example does stupidly short cycles for test purposes, but similar routines could get useful over longer durations. *e.g.*, if you are only interested in night temperatures.

## To do
- Add a safety in `phidget-rec` to prevent overwriting an existing csv file
- Investigate implementing upcoming `jp` update and associated new features
