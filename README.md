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
    <a href="https://raw.githubusercontent.com/mlaparie/phidget-tools/main/pics/phidget-plot.mp4"
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

While `phidget -i` will prompt the user for all available options (except selecting a Phidget when several are connected, because I could not test it, but adding `-p SERIAL` should work), one advantage for CLI tools is to suppress all interactions to run them from other scripts and tools. Just make sure you provide the compulsory arguments `-n NAME`, `-d DAYS` (though inconvenient, it is a float, so fractions are possible: `-d 0.25` is six hours) and `-y` to skip the final confirmation. `-r RATE` is not mandatory but will obviously be set by most users. The silly example below would collect data for 10 seconds at 0.2s poll rate, then wait 10 seconds before starting a new cycle, and increment file names accordingly.

```bash
#!/usr/bin/env bash
TMPFILE="$(mktemp)"
echo 0 > $TMPFILE
	while true
	do
		INCREMENT=$[$(cat $TMPFILE) + 1]
		sleep 10 && phidget-rec -nDoe -yd0.0001157407 -r0.2 -lphase"$INCREMENT"
		echo $INCREMENT > $TMPFILE
	done
```

```txt
$ tree Data/
Data/
└── Doe
    ├── Doe_phase1_r0.2_D0.0d_20211108-024307.csv
    ├── Doe_phase2_r0.2_D0.0d_20211108-024327.csv
    ├── Doe_phase3_r0.2_D0.0d_20211108-024348.csv
    ├── Doe_phase4_r0.2_D0.0d_20211108-024408.csv
    ├── Doe_phase5_r0.2_D0.0d_20211108-024428.csv
    └── Doe_phase6_r0.2_D0.0d_20211108-024449.csv

1 directory, 6 files
```

This simple script does stupidly short cycles for test purposes, no one would need this, but similar routines could get useful over longer durations if you are only interested in night temperatures for instance.

## To-do
- Add a safety in `phidget-rec` to prevent overwriting an existing csv file
- Investigate implementing upcoming `jp` update and associated new features
