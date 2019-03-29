
# Swagger Editor as a native application - with tab support

####Instructions for creating a native application wrapper around the web application swagger-editor.

**Credit and Thanks to:**
 - SmartBear & swagger-api for **swagger-editor**
 - Jiahao for **nativefier**


## Problems

1. The "save" mechanism in the editor only lets you download the document as "~/Downloads/openapi.yaml". Then you have to rename/move to the desired location.

2. Has to be run in a browser window

3. Does not have tab support

## Solutions

1. Using `nativefier` package to create a native application wrapper lets you run the editor outside of a browser, and because the 'browser' window is privileged, it allows for a "Save As" dialog when you download the document. Nativefier (latest version) also provides for tab support (As of this writing, only on the Mac platform).

2. Small code changes to the editor to set the document.title to the filename last imported or saved. This is just so you can navigate the tabs easier.

## Instructions

***One the machine that will be hosting the editor (either a server or your desktop):***
```
# check out repo
git clone https://github.com/swagger-api/swagger-editor.git

# apply patch
git apply tabnames.patch

# build the application
npm run build

# NOTE: Might need to run: npm install npm-run-all --save-dev (sh: 1: run-p: not found)

# Create the docker image
docker build -t myname/swagger-editor .

# Run the container with a name (referenced later with systemd script)
docker run -d --name swagger-editor -p 8080:8080 myname/swagger-editor

# Create a systemd file /etc/systemd/system/swagger-editor.service
# (see below)

# Enable the service at boot
sudo systemctl enable /etc/systemd/system/swagger-editor.service
```

#### Service file
```
[Unit]
Description=Swagger Editor
Requires=docker.service
After=docker.service

[Service]
Restart=always
RestartSec=5
ExecStart=/usr/bin/docker start -a swagger-editor
ExecStop=/usr/bin/docker stop -t 2 swagger-editor
```

***On your desktop machine:***


First, update to the latest verson of nativefier: (for tab support)
Download the sources (zip): https://github.com/jiahaog/nativefier/releases/tag/v7.6.3
```
# Dependencies
npm install

# Build
npm run build

# Install
npm install -g
```

Create the native application wrapper.
```
# Run help first - you may want to add more options to your application
nativefier --help

# Create the application wrapper
nativefier --name swagger-editor "http://where-the-editor-is-running:8080" -f --file-download-options '{"saveAs": true}'

# Move swagger-editor.app to desired location and launch it

```

## Usage
Run the application normally, you should see the editor. To enable tabs,

`View -> Show Tab Bar`

## Known issues
 - On first run, tab/document title says "null". You can do `File->Clear Editor`, then exit/restart app. Now it should say "Swagger Petstore" for that tab and be correct for all subsequent tabs.

 ### **This code is unsupported - for your experimentation only.**
