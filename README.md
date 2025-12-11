# Plasmashell Watcher

A simple Bash script that monitors `plasmashell` and automatically restarts it if it crashes. This is useful for KDE Plasma users who occasionally experience a frozen or disappearing panel, which is often a result of `plasmashell` crashing and failing to restart on its own.

## How It Works

The script runs in a continuous loop and checks if the `plasmashell` process is running. If the process is not detected, it executes `plasmashell --replace` to launch a new instance of the Plasma Shell. There is a configurable delay between each check to ensure minimal resource usage.

## The Script

Below is the complete Bash script. You can save this to a file, for example, `plasmashell-watcher.sh`.

```bash
#!/bin/bash

# A script to monitor and restart plasmashell if it's not running.
# This is a workaround for situations where plasmashell crashes and doesn't
# automatically restart.

# The time in seconds to wait between checks.
CHECK_INTERVAL=5

while true; do
    # Check if plasmashell is running.
    # pgrep -x will match the exact process name.
    if ! pgrep -x "plasmashell" > /dev/null; then
        echo "Plasmashell is not running. Restarting..."
        # Using 'plasmashell --replace &' to start it in the background.
        # The DISPLAY environment variable might be necessary if running from a context
        # where it's not set, such as certain startup scripts.
        export DISPLAY=:0
        plasmashell --replace &
    fi
    # Wait for the specified interval before checking again.
    sleep $CHECK_INTERVAL
done
