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
```

## Installation and Usage

There are two ways to use this script: you can run it manually for a single session, or you can set it up to run automatically every time you log in, which is the recommended approach.

### Manual Usage (for testing)

1.  **Save the script:** Copy the script above and save it to a file named `plasmashell-watcher.sh`.
2.  **Make it executable:** Open a terminal and run the following command in the directory where you saved the file:
    ```sh
    chmod +x plasmashell-watcher.sh
    ```
3.  **Run the script:**
    ```sh
    ./plasmashell-watcher.sh
    ```

The script will now be running in your terminal and will restart `plasmashell` if it crashes. You will need to keep this terminal window open.

### Automatic Startup with systemd (Recommended)

For a "set it and forget it" solution, you can run the script as a systemd user service. This will automatically start the script when you log in and manage it in the background.

**Step 1: Place the script in a permanent location.**

It's a good practice to store user scripts in `~/.local/bin`. If this directory doesn't exist, create it:

```sh
mkdir -p ~/.local/bin
```

Now, move the script into this directory:

```sh
mv plasmashell-watcher.sh ~/.local/bin/
```

Make sure it's executable:

```sh
chmod +x ~/.local/bin/plasmashell-watcher.sh
```

**Step 2: Create the systemd service file.**

Create a new directory for user systemd services if it doesn't already exist:

```sh
mkdir -p ~/.config/systemd/user/
```

Now, create a new service file with a text editor:

```sh
nano ~/.config/systemd/user/plasmashell-watcher.service
```

Paste the following content into the file. **Make sure to replace `YOUR_USERNAME` with your actual username.**

```ini
[Unit]
Description=Plasmashell Watcher and Restarter

[Service]
ExecStart=/home/YOUR_USERNAME/.local/bin/plasmashell-watcher.sh

[Install]
WantedBy=plasma-workspace.target
```

Save the file and exit the text editor.

**Step 3: Enable and start the service.**

Now, you need to tell systemd to recognize the new service and start it.

```sh
systemctl --user daemon-reload
systemctl --user enable --now plasmashell-watcher.service
```

The `enable` command makes it start on login, and the `--now` flag starts it immediately for the current session.

You can check the status of the service at any time with:

```sh
systemctl --user status plasmashell-watcher.service
```

That's it! The script will now run automatically in the background every time you log in, ensuring your Plasma Shell is always running.

## Customization

If you want to change how often the script checks for `plasmashell`, you can edit the `plasmashell-watcher.sh` script and change the value of the `CHECK_INTERVAL` variable.

## License

This project is licensed under the MIT License.
