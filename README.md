# HASS-Watchdog
This repository describes how to use Watchdog with Home-Assistant (HA).

[Watchdog](https://github.com/gorakhargosh/watchdog) is a python package for monitoring file-system events, such as the creation or deletion of files or directories. Under the hood it uses a variety of platforms to efficiently monitor the system, for example using [inotify](https://en.wikipedia.org/wiki/Inotify) on Linux. Watchdog can be used from the command line with the shell utility `watchmedo` or using a python API. In this repo I use the python API to integrate with HA. My long term aim is to add a component to HA which directly integrates Watchdog, but for the time-being I am using the following workflow:

1. A python script will start Watchdog and log file system events to a text file
2. Run the python script using a [Shell command](https://home-assistant.io/components/shell_command/)
2. Use a [file sensor](https://home-assistant.io/components/sensor.file/) to periodically read the text file and display the logged information.

Simple! Lets get started.

### Python script
I slightly modified the following script from the Watchdog [Quickstart](https://pythonhosted.org/watchdog/quickstart.html#quickstart) example. I added logging to a text file, and commented out the date-time string as I'm not interested in exact timings and HA will display the approximate time of events.
