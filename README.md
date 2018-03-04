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
I modified a couple of scripts I found online to create the `hass_watchdog.py` script. Via SSH this script can be run with or without a directory path as an argument. However HA is a bit picky about file paths (owing to running in a venv), so when using a shell command to start the script from within HA, you need to pass the absolue path of the directory you want Watchdog to watch. Events detected by Watchdog are written to a `data.json` text file. I've decided to place `hass_watchdog.py` in the `www` folder of my Home-assistant directory so I can easily view the `data.json` file from the HA web GUI if I want to. I created the following shell command to start the script from within HA using the Service tool:

```yaml
shell_command:
  watchdog: python ~/.homeassistant/www/hass_watchdog.py /Users/robincole/.homeassistant/www
```

Once I start creating/modifying files in the `www` folder, Watchdog starts populating `data.json`. The view of that file via the HA web GUI is shown below (I've got the code for the ipanel below).

<p align="center">
<img src="https://github.com/robmarkcole/HASS-Watchdog/blob/master/images/data_json.png" width="700">
</p>

### Home-Assistant sensor
I now use a file sensor to display the most recent event (you may want to experiment with scan_interval):
```yaml
sensor:
  - platform: file
    scan_interval: 1
    name: data_json
    file_path: /Users/robincole/.homeassistant/www/data.json
    value_template: '{{ value_json.event + ": " + value_json.file }}'

panel_iframe:
  data_json:
    title: 'Watchdog'
    url: 'http://localhost:8123/local/data.json'
```

The final product is shown below:

<p align="center">
<img src="https://github.com/robmarkcole/HASS-Watchdog/blob/master/images/HA_sensor.png" width="400">
</p>

### Summary
We have seen how Home-Assistant can be configured to run Watchdog and display event data in the front end. This is a stepping stone towards a true integration of Watchdog with HA.
