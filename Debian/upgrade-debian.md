# Debian - Upgrading Major Release/version

```
Documentation and Guide on how to upgrade from your current version to any other version

Preferably to the next version
```

## Table of Contents
- [Setup](#setup)
	+ [Pre-Requisites](#pre-requisites)
- [Documentation](#documentation)
	+ [Steps](#steps)

## Setup
### Pre-Requisites
- Backup 
	- System
		```
		Always backup your system before proceeding

		- Be prepared
		```

		- Using tar
			```
			Syntax: tar -cvzf [output-file] [target-files]
			```
			tar -cvzf /tmp/home.tar.gz /home
	
	- Packages
		```
		To create a 'requirements.txt' list to restore all installed packages
		```

		- Distros
			- Debian
				```
				List all installed packages and output
				Syntax: apt-get list | tee -a requirements.txt
				```

			- Arch
				```
				List all installed packages and output
				Syntax: pacman -Qq | tee -a requirements.txt
				```

- Update and Upgrade Existing Packages
	- Synopsis/Syntax:
		```console
		sudo apt update && sudo apt ugrade
		```

- Reboot system
	```console
	sudo reboot now
	```

## Documentation
### Version Change
- Edit the file /etc/apt/sources.list 
	- Replace instance of current version codename with the codename of the next version	
		- Information
			- let
				+ current version be [buster] (debian 10) and
				+ next version be [bullseye] (debian 11)
			- using a text editor and
				+ replace each instance of [buster] with [bullseye]. 
		- Automatic
			- Synopsis/Syntax:
				```console
				sed -i 's/old-text/new-text/g' filename
				```
			```console
			sed -i 's/buster/bullseye/g' /etc/apt/sources.list
			```

	- Next find the security line,
		```
		replace keyword [buster/updates] with [bullseye-security]
		```
		- Automatic
			- Synopsis/Syntax:
				```console
				sed -i 's/old-text/new-text/g' filename
				```
			- Explanation:
				+ s : Substitute command of sed for find all occurences of 'old-text' and replace with 'new-text'
			```console
			sed -i 's/"buster/updates"/"bullseye-security"/g' /etc/apt/sources.list
			```


- Update the packages index on Debian Linux
	```console
	sudo apt update
	```

- Prepare for the operating system upgrade
	```console
	sudo apt upgrade
	```

- Finally, update Debian 10 to Debian 11 bullseye
	```console
	sudo apt full-upgrade
	```

- Reboot the Linux system so that you can boot into Debian 11 Bullseye
	```console
	sudo reboot now
	```

- Verify that everything is working correctly


