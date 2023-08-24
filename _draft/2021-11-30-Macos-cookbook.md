---
layout: post
title: "MacOSX Cookbook"
categories: osx mac
toc: true
comments: true
---

Mac OSX wisdom.

## Setup auto-completion (bash-completion)

### Setup bash-completion from homebrew

```bash
 brew install bash-completion
echo "[ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion" >> ~/.bash_profile
```

### Setup bash-completion for Docker

```bash
cd /usr/local/etc/bash_completion.d \
&& ln -s /Applications/Docker.app/Contents/Resources/etc/docker.bash-completion \
&& ln -s /Applications/Docker.app/Contents/Resources/etc/docker-machine.bash-completion \
&& ln -s /Applications/Docker.app/Contents/Resources/etc/docker-compose.bash-completion
```

## Disable helpers autostart

Citrix Reciever

```bash
ls /Library/LaunchAgents/com.citrix.* | awk -F "/" '{print $4}' | xargs launchctl stop
```

Adobe Services

```bash
ls /Library/LaunchAgents/com.adobe.* | awk -F "/" '{print $4}' | xargs launchctl stop
```

Teamviewer

```bash
ls /Library/LaunchAgents/com.teamviewer.* | awk -F "/" '{print $4}' | xargs launchctl stop
```

## Files or directories moved from OSX to Synology NAS are inaccessible

If files or directory is inaccessible you need to normalize them from UTF-8 C (OSX) to D (Linux)

- Enable NFS on NAS
- Setup NFS permission on NAS for the target directory
- From Linux connect to NFS share (CentOS: `mount.nfs -w 192.168.1.100:/volume1/datastore/ /mnt/`)
- Install **convmv** tool (CentOS: `yum install convmv`)
- Open mounted directory
- Run command `convmv -r -f utf8 -t utf8 --nfc .` (dry-run)
- If `error (max: 255)` appeared modify **convmv** (`vi /usr/bin/convmv`) variable **$maxfilenamelength** from **255** to **1024**, then run the previous command again
- To perform converting run the command `convmv -r -f utf8 -t utf8 --nfc --notest .`

## Reset Bluetooth

```bash
sudo kextunload -b com.apple.iokit.BroadcomBluetoothHostControllerUSBTransport
sudo kextload -b com.apple.iokit.BroadcomBluetoothHostControllerUSBTransport
```

## Remove the dot file from the current folder and subfolder

```bash
find . -type f ! -name '.DS_Store' -name "._[^.]*" | sed -E 's|/[^/]+$||' | sort | uniq | while read folder; do echo $folder; done;
```

## DS_Store management

### Disable .DS_Store

```bash
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

### Remove .DS_Store

```bash
find /Volumes/data/ -name .DS_Store -delete
```

## Restart Bluetooth Daemon on Mac OS X without restarting

```bash
sudo kextunload -b com.apple.iokit.BroadcomBluetoothHostControllerUSBTransport
sudo kextload -b com.apple.iokit.BroadcomBluetoothHostControllerUSBTransport
```

## Mount NFS share

```bash
sudo mount -t nfs -o resvport 192.168.1.100:/source/folder /dest/folder
```

**-o resvport** is mandatory!

## Set OSX name/hostname

```bash
export HOSTNAME="dodo-mac" \
&& sudo scutil --set HostName ${HOSTNAME} \
&& sudo scutil --set LocalHostName ${HOSTNAME} \
&& sudo scutil --set ComputerName ${HOSTNAME}
```

## Force OSX to use 5Ghz

```bash
sudo /System/Library/PrivateFrameworks/Apple80211.framework/Resources/airport --channel=$(sudo /System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport /usr/sbin/airport -s | grep -E " $(sudo /System/Library/PrivateFrameworks/Apple80211.framework/Resources/airport -I | grep " SSID:" | cut -d : -f 2 | awk '{$1=$1};1') [a-z0-9]{2}:" | grep -Ev " -[0-9]{2,3}  (1[0-4]|[0-9]) " | head -n 1 | awk '{print $4}') && sleep 3 && networksetup -setairportpower airport off && sleep 3 && networksetup -setairportpower airport on
```

## Convert e-books to diferent format

Install **Calibre** app before

```bash
/Applications/calibre.app/Contents/ebook-edit.app/Contents/MacOS/ebook-convert ~/path/to/book.epub ~/path/to/converted-book.mobi
```
