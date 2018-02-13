# Installation Computers
Set Up & Monitoring Guide

[Computer Settings]() <br/>
[Hide Menus]() <br/>
[Prevent Crash Dialogs]() <br/>
[Keep Application Open]() <br/>
[Sending Screenshots]() <br/>
[Remote Access]() <br/>

## Computer Settings
#### Mac
We follow Step 1 and (optional) Step 4 from this guide: <br/>
https://github.com/laserpilot/Installation_Up_4evr  </br>
to turn off screensaver, prevent notifications, and other necessary mac settings. <br/>
Optionally to shutdown the computer on a daily schedule. 
#### Linux
* Set computer to power on after power failure
  * Hold `F2` as the computer starts up to enter BIOS
  * Advanced Power → After Power Failure → Power On
  * Press `F10` to save and exit BIOS
* Clear the desktop and set a background
* Disable auto update in Software and Update settings
* Disable firewall: `sudo ufw disable`


## Hide Menus
#### Mac
* Right-click application and select ‘Show Package Contents’
* Open Info.plist
* Add key `Application UI Presentation Mode` with value `3 (all hidden)` to never show the system menu or dock over application
#### Linux
* System Settings → Appearance → Behavior → Autohide Launcher → ON
* Set Autohide Launcher Reveal Sensitivity to lowest
* Run the app fullscreen to hide the system menu


## Prevent Crash Dialogs
#### Mac Terminal Commands
* `defaults write com.apple.CrashReporter DialogType none`
* `defaults write NSGlobalDomain NSQuitAlwaysKeepsWindows -bool false`


## Keep Application Open
We run a script at the top of every minute, which opens the application if it is closed. To allow script to run as an executable: `chmod +x script.ext`
#### Mac
* Add to crontab: <br/>
  `* * * * * open -a /absolute/path/to/application`
#### Linux
* Install run-one: `sudo apt-get install run-one`
* Add to crontab: <br/>
  `* * * * * DISPLAY=:0 run-one /absolute/path/to/application`

## Sending Screenshots
We have a slack channel that receives a screenshot of each exhibit every morning after it starts up. We check this for anything unusual before the museums are open to the public. In the future we hope to also report frame rate, cpu usage, crashes, and network connection.
#### Screenshots on Mac
* Add to crontab to take a screenshot at 7:00am: <br/>
  `0 7 * * * "/usr/sbin/screencapture" -f /absolute/path/to/screenshot.jpg`
#### Screenshots on Linux
* Install scrot: `sudo apt-get install scrot`
* Add to crontab: <br/>
  `0 7 * * * DISPLAY=:0 /usr/bin/scrot /absolute/path/to/screenshot.jpg`
#### Sending Screenshots to Slack (Mac & Linux)
* Get a slack token
* Install slackclient with pip: `pip install slackclient`
* Save this python script with the slack token, channel, and path to screenshot:
```
from slackclient import SlackClient
import os
slack_token = “YOUR SLACK TOKEN”
compName = os.environ[“COMP_NAME”]
sc = SlackClient(slack_token)
myFile = open(“/absolute/path/to/screenshot.jpg”,”rb”)
sc.api_call(
 “files.upload”,
 channels=[“#channel-name”],
 title=compName,
 file=myFile
)
```
* Turn script to executable: `chmod +x /absolute/path/to/script.py`
* Add to crontab to run at 7:01am: <br/>
`1 7 * * * COMP_NAME="name" python /absolute/path/to/script.py`

#### Sending Screenshots to Email (Mac & Linux)
Before integrating with Slack, we sent screenshots via email.
* Save this python script with the information filled in:
```
# Change these values:

SMTP_SERVER = 'smtp.server.com'
SMTP_PORT = 000
SMTP_USERNAME = 'FROM@email.address'
SMTP_PASSWORD = '******'
SMTP_FROM = 'FROM@email.address'
SMTP_TO = 'TO@email.address'
SUBJECT = "SCREEN STATUS OF ___________"

# Make sure you are saving the screenshot as email.jpg

TEXT_FILENAME = 'email.jpg'

# Constructing the message
import smtplib, email
from email import encoders
from email.mime.image import MIMEImage
import os

msg = email.MIMEMultipart.MIMEMultipart()
fp = open(os.path.basename(TEXT_FILENAME), 'rb')
img = MIMEImage(fp.read())
fp.close()
msg['Subject'] = SUBJECT
msg.attach(img)
msg.add_header('From', SMTP_FROM)
msg.add_header('To', SMTP_TO)

# Now send the message
mailer = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
mailer.starttls()
# EDIT: mailer is already connected
# mailer.connect()
mailer.login(SMTP_USERNAME, SMTP_PASSWORD)
mailer.sendmail(SMTP_FROM, [SMTP_TO], msg.as_string())
mailer.close()
```

* Turn script to executable: `chmod +x /absolute/path/to/script.py`
* Add to crontab to run at 7:01am: <br/>
`1 7 * * * COMP_NAME="name" python /absolute/path/to/script.py`


## Remote Access
We use TeamViewer. Setting up SSH is another good idea.
