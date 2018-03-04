# HASS-Watchdog
This repository describes how to use Watchdog with Home-Assistant (HA).

[Watchdog](https://github.com/gorakhargosh/watchdog) is a python package for monitoring file-system events, such as the creation or deletion of files or directories. Under the hood it uses a variety of platforms to efficiently monitor the system, for example using [inotify](https://en.wikipedia.org/wiki/Inotify) on Linux. Watchdog can be used from the command line with the shell utility `watchmedo` or using a python API. In this repo I use the python API to integrate with HA. My long term aim is to add a component to HA which directly integrates Watchdog, but for the time-being I am using the following workflow:

1. A python script will start Watchdog and log file system events to a text file
2. Run the python script using a [Shell command](https://home-assistant.io/components/shell_command/)
2. Use a [file sensor](https://home-assistant.io/components/sensor.file/) to periodically read the text file and display the logged information.

Simple! Lets get started.

### Install Watchdog
You need to ssh into your HASS environment then `pip install watchdog`. To check that everything is installed and ready, run the command line utility help command with `watchmedo --help`. If he help message pops up you are OK.

### Python script
I modified a couple of scripts I found online to create the `hass_watchdog.py` script. This script can be run with or without a directory path as an argument, for example I want to watch a particular folder so I run `python hass_watchdog.py /Users/robincole/Documents/test_dir`. Events detected by Watchdog are both printed to the terminal, and written to the `data.json` text file. An example of the contents of this text file are shown below:
```json
{"event": "deleted", "full_path": "/Users/robincole/Documents/test_dir/test copy.txt", "file": "test copy.txt"}
{"event": "created", "full_path": "/Users/robincole/Documents/test_dir/test copy.txt", "file": "test copy.txt"}
```
I've decided to place `hass_watchdog.py` in the `www` folder of my Home-assistant route directory so I can easily view the `data.json` file from the HA web GUI if I need to. I then created the following shell command to start the script:
```yaml
shell_command:
  watchdog: python ~/.homeassistant/www/hass_watchdog.py /Users/robincole/.homeassistant/www

```
Note that HA is a bit picky about file paths, so I am passing the absolute path for Watchdog to watch.
