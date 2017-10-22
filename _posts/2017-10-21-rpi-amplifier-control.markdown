---
layout: post
title: Raspberry Pi amplifier control
description: Bash script to switch ON/OFF an amplifier based on audio playback
date: '2017-10-21 01:20:02'
tags: Raspberry-Pi MPD
---


## This script can be used to automate turning ON/OFF an amplifier connected to a GPIO on the Raspberry-Pi. 

***

## How it works?

The script monitors the status of the sound card, by reading the `/proc/asound/card*/pcm*/sub*/status` file. `state: RUNNING` indicates that the sound card is in use. Based on this, it calls either `amp_on.sh` or `amp_off.sh`. 
You could decide to defer the amplifier switch OFF by a few seconds after playback stops so that a pause/stop quickly followed by a play doesn't switch off the amplifier.

***

## Usage

* Set the variables `amplifier_gpio`, `snd_card_file` and `off_timer_time`.
* Implement the `amp_on.sh` and `amp_off.sh` scripts. 
* Make sure all scripts are made executable.
* Optionally, you could setup the script to run on boot (using crontab's `@reboot` or similar mechanism).

```bash
#!/bin/bash

# Time to wait until the amplifier is turned off in seconds
off_timer_time=5
amplifier_gpio=
snd_card_file=/proc/asound/card1/pcm0p/sub0/status

last_is_playing=0
off_timer=0


# Init amplifier GPIO
/bin/echo $amplifier_gpio > /sys/class/gpio/export
/bin/echo "out" > /sys/class/gpio/gpio$amplifier_gpio/direction
/bin/echo 1 > /sys/class/gpio/gpio$amplifier_gpio/value


while true
do
	if cat $snd_card_file | grep -i RUNNING > /dev/null
	then
		is_playing=1
	else
		is_playing=0
	fi

	#0->1
	if [[ "$last_is_playing" -eq "0" && "$is_playing" -eq "1" ]]
	then
		off_timer=0
		#pid_snd=`fuser /dev/snd/pcmC1D0p 2>/dev/null | awk '{ for (i=1; i<=NF; i++) print $i }'`
        amp_on.sh
	fi

	#1->0
	if [[ "$last_is_playing" -eq "1" && "$is_playing" -eq "0" ]]
	then
		off_timer=1
	fi

	if [ "$off_timer" -ne "0" ]
	then
		if [ "$off_timer" -eq "$off_timer_time" ]
		then
			off_timer=0
			amp_off.sh
		else
			((off_timer++))
		fi
	fi

	last_is_playing=$is_playing
	sleep 1
done
```

***

## Fix MPD holds sound device indefinitely if client goes offline during playback problem

I had this problem where MPD wouldn't release the sound device when a streaming client went offline/out of the network. This left the amplifier turned ON for extended periods of time leading to over-heating problems.

To overcome this, I came up with a solution to check `mpc status` and recheck it after 2 seconds to detect if the streaming media was progressing or not. Here's my solution which I include in the above script. It isn't neat but it works.

```bash
if [[ "$is_playing" -eq "1" ]]
then
    mpc_state=`/usr/bin/mpc | grep "\[" | cut -d "[" -f2 | cut -d "]" -f1`;
    if [[ "$mpc_state" -eq "playing" ]]
    then
        mpc_progress1=`/usr/bin/mpc | grep "\[" | cut -d "#" -f2 | cut -d ")" -f1`;
        sleep 2;
        mpc_progress2=`/usr/bin/mpc | grep "\[" | cut -d "#" -f2 | cut -d ")" -f1`;
        if [ "$mpc_progress1" == "$mpc_progress2" ]; then
            #No progress
            /usr/bin/mpc pause-if-playing
        fi
    fi
fi
```

