# timesave

Restores periodic clock saving for systems using chrony instead of
systemd-timesyncd.

## Why this exists

systemd-timesyncd does two things: it syncs time via NTP, and it saves
the current clock to `/var/lib/systemd/clock` every 60 seconds and on
shutdown. The kernel reads this file at boot to set a sane initial
timestamp before any NTP sync has happened.

When you replace systemd-timesyncd with chrony — which is common on
Raspberry Pi and other single-board computers for better accuracy,
PPS support, or GPS disciplined clocks — chrony handles the NTP sync
but the periodic clock saving is lost. chrony explicitly disables
systemd-timesyncd, so the clock file stops being updated.

On a desktop or laptop with a UEFI BIOS this doesn't matter much —
the hardware RTC loads early and keeps time between boots. But on
platforms where there is no RTC, or where the RTC loads late, or where
the RTC is unreliable (many single-board computers fall into this
category), the system clock at boot can be wildly wrong until NTP
syncs. This causes problems with SSL certificate validation, log
timestamps, file modification times, and anything else that depends
on a sane clock.

This package simply restores the clock saving behaviour that
systemd-timesyncd provided, as a standalone timer and service that
works alongside chrony.

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

- `timesave.timer` — fires every 60 seconds once chrony is running,
  and waits for the hardware clock to be loaded before starting
- `timesave.service` — touches `/var/lib/systemd/clock` to update
  its modification time, which the kernel reads on next boot
- `timesave-shutdown.service` — saves the clock on shutdown so the
  most recent timestamp is always preserved

## Uninstall

```bash
sudo apt remove timesave
sudo rm /etc/apt/sources.list.d/timesave.list
```
