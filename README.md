# Mob Station Setup

This guide describes how to configure a Mac-based _Mob Station_ for shared use.

The guide is written for _macOS 26_. The _Out-of-Box Experience (OOBE)_ and _System Settings_ are not stable.

The _Mob Station_ is a shared device. Security is handled by completely resetting all credentials and browser history at both startup and shutdown.

## Configuration

### Hardware

Start with a blank canvas: Either a new (shiny) Mac (mini) or a repurposed one, but ensure it has been factory reset.

### Operating System Configuration

1. Follow the onscreen guide to configure language, networking etc., until you reach account creation.

1. Create a Mac Account:

    - Full Name: `Mob Station`
    - Account Name: `mobstation`
    - Password: Memorable for the whole team. Store in your password manager of choice. Rotate monthly.
    - Allow computer account password to be reset with your Apple Account: `unchecked`

1. Sign In to Your Apple Account:

    - Select _Other Sign-In Options_ --> _Sign in Later in Settings_ and _Skip_

1. Continue until _FileVault_

1. Your Mac is Ready for FileVault:

    - Turn On
    - Store recovery key in your password manager of choice

1. Continue until _Welcome_ and select _Get Started_

1. Go to _System Settings_ --> _General_ --> _About_:

    - _Name_: `mob1` or similar

1. Update and restart the machine

### Configure Mob Station

1. Sign in as `Mob Station`.

1. Go to _System Settings_ --> _Network_ --> _Firewall_:

    - Firewall: `on`

1. Go to _System Settings_ --> _Energy_:

    - Start up when power is connected: `Always`

### Install Software

1. Open [Safari] and select _File_ --> _New Private Window_ to avoid having to reset afterwards.

1. Go to [Homebrew](https://brew.sh/) and install. _Note the __Next steps___

1. Go to [Oh My Zsh](https://ohmyz.sh/) and install.

1. Go to [Helium Browser](https://helium.computer/) and install.

1. Quit _Safari_

1. Install _git_ and _Visual Studio Code_:

    ```bash
    brew install git visual-studio-code
    ```

1. Clone this repository to `~/code/mob-station`

    ```bash
    mkdir -p ~/code
    cd ~/code
    git clone https://github.com/ondfisk/mobstation.git
    cd mobstation
    ```

1. Configure Visual Studio Code:

    - Open _Visual Studio Code_
    - Select _Continue without Signing In_
    - Select a theme and _Get Started_
    - Close the _Chat_
    - Open Command Palette (__`[Shift]`__ + __`[Command]`__ + __`[P]`__)
    - Type `path` and choose _Shell Command: Install 'code' command in __PATH___.
    - _File_ --> _Open Folder..._ --> `~/code/mob-station` and open `README.md` and select _Open as Preview_ (top right) to avoid typing the following shell commands.
    - Click _Manage_ in the _Restricted Mode_ banner and add the _code_ folder to _Trusted Folders & Workspaces_

1. Install the basic software needed:

    ```bash
    brew install \
        docker \
        gcloud-cli \
        gh \
        node \
        php@8.2 php@8.4 php@8.5 \
        phpstorm \
        slack \
        slack-cli \
        yarn
    ```

1. Configure _Helium_:

    - Open _Helium_ and select _Use defaults_
    - Quit _Helium_ (hold __`[Command]`__ + __`[Q]`__)
    - Backup _Helium_ settings:

        ```bash
        HELIUM_BUNDLE=$(defaults read "/Applications/Helium.app/Contents/Info" CFBundleIdentifier)
        mkdir -p ~/.mob-state/helium
        cp -r "$HOME/Library/Application Support/$HELIUM_BUNDLE/" ~/.mob-state/helium
        ```

    - Go to _System Settings_ --> _Desktop & Dock_ --> _Default web browser_ and select _Helium_

1. Remove unused widgets from the Desktop:

    - Calendar
    - Weather
    - Photos

1. Remove unused applications from the Dock:

    - Safari
    - Messages
    - Mail
    - Photos
    - FaceTime
    - Phone
    - Calendar
    - Contacts
    - Reminders
    - Notes
    - TV
    - Music
    - Games
    - ...

1. Keep most used applications in Dock:

    - Helium
    - Terminal
    - Slack
    - ...

### Configure git

```bash
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global credential.helper store
git config --global core.autocrlf input
git config --global commit.cleanup verbatim
```

### Configure mob scripts

1. Add `~/bin` to path:

    ```bash
    mkdir ~/bin
    echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
    source ~/.zshrc
    ```

1. Update `scripts/mob-settings.json`:

    - Configure _GitHub_ organization and repository
    - Configure _roster_ (available members for mobbing)
    - Configure _members_ (current members in mob)

1. Copy mob scripts to `~/bin`:

    ```bash
    cp -r scripts/ ~/bin
    ```

### Configure startup

1. Go to _System Settings_ --> _General_ --> _Login Items & Extensions_:

    - Open at Login: Add `~/bin/mob-startup`

### Configure teardown

1. Allow `mobstation` to run shutdown

    ```bash
    sudo tee /etc/sudoers.d/mob-shutdown > /dev/null << 'EOF'
    mobstation ALL=(root) NOPASSWD: /sbin/shutdown
    EOF
    sudo chmod 0440 /etc/sudoers.d/mob-shutdown
    sudo visudo -c    # must print "parsed OK"
    ```

1. Configure automatic teardown:

    - Create `com.mob.teardown.plist`:

        ```bash
        mkdir -p ~/Library/LaunchAgents

        cat > ~/Library/LaunchAgents/com.mob.teardown.plist << 'EOF'
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
        <plist version="1.0">
        <dict>
        <key>Label</key>
        <string>com.mob.teardown</string>

        <key>ProgramArguments</key>
        <array>
            <string>/Users/mobstation/bin/mob-teardown</string>
        </array>

        <key>StartCalendarInterval</key>
        <dict>
            <key>Hour</key>
            <integer>19</integer>
            <key>Minute</key>
            <integer>0</integer>
        </dict>

        <!-- Run Mon-Fri only -->
        <key>LimitLoadToWeekdays</key>
        <true/>

        <key>StandardOutPath</key>
        <string>/Users/mobstation/.mob-state/teardown.log</string>

        <key>StandardErrorPath</key>
        <string>/Users/mobstation/.mob-state/teardown.err</string>

        <!-- Allow running when display sleeps -->
        <key>WakeOnWakeFromSleep</key>
        <false/>
        </dict>
        </plist>
        EOF
        ```

    - Configure:

        ```bash
        launchctl load -w ~/Library/LaunchAgents/com.mob.teardown.plist
        ```

1. Verify teardown

    ```bash
    launchctl start com.mob.teardown
    ```

### Setup complete

## Usage

### Every morning

1. First one in turns on the _Mob Station_ and logs in.

1. Open _Terminal_ and run:

    ```bash
    mob morning
    ```

### Mob start

1. Verify `~/bin/mob-settings.json`.

1. Open _Terminal_ and run:

    ```bash
    mob as HANDLE
    ```

### Mob end

1. Open _Terminal_ and run:

    ```bash
    mob teardown
    ```

    __Note__: Runs automatically at 19:00 if forgotten.
