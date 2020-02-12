# apcupsd-callhome

## Why
I wanted an Telegram message and email when the power went out at any of my sites.

## How
There are only two requirements here, SMTP server for sending emails and Telegram API + ConversationID for sending the messages to Telegram.

It works by changing the default scripts that come with apcupsd, setting those scripts to call our script. From our one script, we can send emails and Telegram messages regarding all of the power messages that apcupsd can send.

Normally apcupsd has a script for each condition, so it can be a little tedious to edit and maintain each one. Thus, each script just calls our script with the specific condition.

## SELinux
SELinux blocks the apcupsd program from certain things like using curl to talk to the internet. I have created a module to allow this, however you should read through the text file to see exactly what would be allowed.
```
module apcupsd-callhome-selinux 1.0;

require {
	type http_port_t;
	type apcupsd_t;
	type cert_t;
	class tcp_socket name_connect;
	class dir write;
	class file write;
}

#============= apcupsd_t ==============
allow apcupsd_t cert_t:dir write;
allow apcupsd_t cert_t:file write;

#!!!! This avc can be allowed using the boolean 'nis_enabled'
allow apcupsd_t http_port_t:tcp_socket name_connect;

```

## Telegram
I can't cover the methods of creating Bots or getting your conversation ID from Telegram because these methods may change in the future. Here are the basic steps:
1. Create Telegram Bot, get the Bot Token ID
2. Create a conversation or channel, get the ID

## Installation & Usage
* Clone git repository
  * `git clone git@github.com:spmfox/apcupsd-callhome.git`
* Movie directory to wherever you want it to run from
  * `mv apcupsd-callhome /opt/`
* Set root ownership
  * `sudo chown -R root:root /opt/apcupsd-callhome`
* Set execute bit on main script
  * `sudo chmod 700 /opt/apcupsd-callhome/apcupsd-callhome.sh`
* Install SELinux module (if using SELinux)
  * `sudo semodule -i /opt/apcupsd-callhome/apcupsd-callhome-selinux.pp`
* Configure apcupsd to call our script on each alert
  * `sudo /opt/apcupsd-callhome/apcupsd-callhome.sh -create-event-files`
* Edit the following variables:
  * `sudo vim /opt/apcupsd-callhome/apcupsd-callhome.sh`

| Variable | Purpose |
| ---------| ------- |
|opt_FromEmail|Email address to sent from|
|opt_ToEmail|Email address to send to|
|opt_TelegramBotToken|Token of Telegram Bot|
|opt_TelegramMessageID|ID of the Telegram conversation to post the alert to|
|opt_SendEmail|If not set to yes, email wont be sent (default is yes)|
|opt_SendTelegram|If not set to yes, Telegram message wont be sent (default is yes)|


## Troubleshooting
Be sure to have apcupsd configured before trying to run the script. The script can be tested by manually invoking any of the arguments, such as:
`sudo /opt/apcupsd-callhome/apcupsd-callhome.sh -onbattery`
That should trigger the script to send and email and Telegram message.
