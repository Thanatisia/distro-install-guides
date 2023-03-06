# Upgrading Major Release/version

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
			+ tar -cvzf /tmp/home.tar.gz /home
	
	- Packages
		```
		To create a 'requirements.txt' list to restore all installed packages
		```
		- Distros
			- Debian
				+ List all installed packages and output
				```console
				apt-get list | tee -a requirements.txt
				```
			- Arch
				+ List all installed packages and output
				```console
				pacman -Qq | tee -a requirements.txt
				```

- Update and Upgrade Existing Packages
	- Synopsis/Syntax:
		```console
		sudo dnf upgrade
		```

- Reboot system
	```console
	sudo reboot now
	```

## Documentation

