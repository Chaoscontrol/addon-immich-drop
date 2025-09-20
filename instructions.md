# Building an addon

There are basically two ways to build an addon. From scratch using the homeassistant base image, for which you can find the tutorial here: https://developers.home-assistant.io/docs/add-ons/tutorial/  
This tutorial is about the modification of an existing image to convert it in an addon. The example is based on a linuxserver image, but the basic principle are transposable to other high quality images from Dockerhub. It supposes you have read all the official documentation here: https://developers.home-assistant.io/docs/add-ons/

For this example, we’ll use this container: https://hub.docker.com/r/linuxserver/piwigo . The addon built upon it is here: https://github.com/alexbelgium/hassio-addons/tree/master/piwigo

---

## 1. Build the base addon structure

Create a repository on github based on HA provided example

---

## 2. Identify a suitable already published image

- Look on Docker Hub for a well maintained image containing preferably s6 supervision that supports multiple architectures (at least aarch64 and amd64)  
- Linuxserver.io images are usually recognized as the best quality.

---

## 3. Adapt for HA usage

### 3.1. Define the fixed parameters for your addon

By looking at the docker-compose provided by the addon, you can see the most important information to run the addon.

You can get several information from the docker-compose:

- the base image you’ll need to build your addon (you’ll need to set that in the `build.json` file; example)  
- the environment variables needed (All environment options need to be defined in a fixed manner either in the Dockerfile (through `ENV` command; example) or in the `config.json` environment key; example.)  
- the volumes where dynamic data is stored → see point after for remanent data  
- the port (will need to go in `config.json`); example  

---

### 3.2. Define the variables parameters for your addon

HA does not support passing environment variables dynamically. All environment options need to be defined in a fixed manner either in the Dockerfile (through `ENV` command) or in the `config.json` environment key.

This can be done in two ways. For example, linuxserver images have a dynamic env variable for setting PUID and GUID.

- Using `sed` to modify the scripts in the Dockerfile  
  Example:  
  ```bash
  RUN \
      # Allow UID and GID setting
      sed -i 's/bash/bashio/g' /etc/cont-init.d/10-adduser \
      && sed -i 's/{PUID:-911}/(bashio::config "PUID")/g' /etc/cont-init.d/10-adduser \
      && sed -i 's/{PGID:-911}/(bashio::config "PGID")/g' /etc/cont-init.d/10-adduser
  ```
- Pushing the variables directly through the export variable (usually done through a script).  
  ```bash
  export ALLOWED_HOSTS=$(bashio::config 'ALLOWED_HOSTS') && bashio::log.blue "ALLOWED_HOSTS=$ALLOWED_HOSTS"
  export SECRET_KEY=$(bashio::config 'SECRET_KEY') && bashio::log.blue "SECRET_KEY=$SECRET_KEY"
  ```

This code must be added in the same bash session as where the app is run. For example, either you have a `run.sh` file where you export variables, then you execute the app, or more probably you have a separate script (in `/etc/cont-init.d` probably) and services (in `/etc/services.d`). What you can do is therefore put the export in those files, for example with:

```bash
sed -i "1a export ALLOWED_HOSTS=$(bashio::config 'ALLOWED_HOSTS')" /etc/services.d/*/run
```

---

### 3.3. Create your Dockerfile

Follow the official guide. Ensure to add instructions to modify files for dynamic environment setting, and add apps if needed. In particular, the Bashio library needs to be installed as it facilitates many HA functions.

Add label.

---

### 3.4. Ensure data remanence

Create folders on your local system, and use symlink so that the app will use these folders instead of its default ones. Script example

---

### 3.5. Add any other scripts

s6 based images get any scripts in `/etc/cont-init.d` and `/etc/services.d` and execute them (if they have execute access). You can therefore add your scripts there through your Dockerfile.

---

## 4. Add codenotary signature

Optional, see here for documentation: https://developers.home-assistant.io/docs/add-ons/security?_highlight=codenotary#codenotary-cas

---

## 5. Add auto build images

Optional, see here for documentation: https://github.com/home-assistant/builder
