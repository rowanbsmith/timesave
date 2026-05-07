# timesave

Persists the system clock to disk every 60 seconds and on shutdown,
replicating the clock-saving behaviour of systemd-timesyncd for systems
using chrony as the NTP daemon.

## Background

When systemd-timesyncd is removed in favour of chrony, the periodic saving
of the system clock to `/var/lib/systemd/clock` is also lost. This means
the kernel has no reference timestamp on next boot until NTP sync occurs,
which can cause issues with SSL certificates, log timestamps, and other
time-sensitive operations.

This package restores that behaviour using a systemd timer and service.

## Requirements

- systemd
- chrony

## Install via apt

```bash
echo "deb [trusted=yes] https://rowanbsmith.github.io/timesave stable main" \
  | sudo tee /etc/apt/sources.list.d/timesave.list

sudo apt update
sudo apt install timesave
```

## What it installs

- `timesave.timer` — runs every 60 seconds after chrony has started
- `timesave.service` — touches `/var/lib/systemd/clock` to update its mtime
- `timesave-shutdown.service` — saves the clock on shutdown

## Uninstall

```bash
sudo apt remove timesave
sudo rm /etc/apt/sources.list.d/timesave.list
```
