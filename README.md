# coffeinate

<p align="center">
  <img src="coffee.gif" alt="coffeinate animated coffee mug" width="333">
</p>

Keep your Mac awake, with style. A small wrapper around `/usr/bin/caffeinate` that adds a friendly prompt animation and a few ergonomic flags. Anything that works with `caffeinate` works here.

## Install

```
brew install genericJE/tools/coffeinate
```

Or grab the script directly:

```
curl -L https://raw.githubusercontent.com/genericJE/coffeinate/main/coffeinate -o /usr/local/bin/coffeinate
chmod +x /usr/local/bin/coffeinate
```

## Usage

```
coffeinate [options] [command ...]
```

| Flag | Effect |
|------|--------|
| `-d` | Prevent display sleep (default) |
| `-i` | Prevent idle sleep |
| `-m` | Prevent disk idle sleep |
| `-s` | Prevent sleep (AC power only) |
| `-u` | Declare user active |
| `-L` | Keep running with the lid closed (implies `-i`, needs `sudo`) |
| `-t <mins>` | Timeout in minutes |
| `-T <HH:MM>` | Shut down at given time |
| `-w <pid>` | Wait for a process to exit |
| `-q` | Silent mode (no animation, no output) |
| `-h` | Show help |

Press Ctrl-C to stop.

## Closing the lid

`caffeinate` cannot survive a lid close. Shutting the lid raises a clamshell event that overrides every userspace power assertion, so the Mac sleeps however many assertions are held. The only lever is the kernel's `SleepDisabled` flag, and that is what `-L` sets:

```
coffeinate -L -- ./long-build.sh
```

That asks for `sudo` once, disables lid sleep for as long as coffeinate runs, and puts it back on the way out. `-L` implies `-i`, because with the lid shut the display is off anyway and what you actually want is for the system to stay up.

Worth knowing:

* `SleepDisabled` persists across reboots, so restoring it matters. coffeinate restores it on Ctrl-C and on `TERM`, `HUP` and `QUIT`. A `kill -9` gives it no chance to, in which case clear the flag yourself with `sudo pmset -a disablesleep 0`.
* If lid sleep was already disabled before coffeinate started, coffeinate says so and leaves it alone rather than clearing something it did not set.
* A closed lid traps heat and the battery still drains. Prefer AC power, and keep the machine out of a bag while it runs.
* For unattended runs where nobody can type a password, allow just the one command in sudoers via `sudo visudo`:

```
%admin ALL=(root) NOPASSWD: /usr/bin/pmset -a disablesleep *
```

## License

MIT.

If anything here ends up being useful to you and you feel like saying thanks, my PayPal is https://paypal.me/genericJE. Truly no expectation either way, just leaving the option here in case.
