# onedrive docker image

Thats right folks onedrive is now dockerized ;)

This container offers simple monitoring-mode service for 'Free Client for OneDrive on Linux'.

## Usage instructions

### 0. Install the docker under your own platform's instructions

### 1. Pull the image

```bash
docker pull driveone/onedrive
```

**NOTE:** SELinux context needs to be configured or disabled for Docker, to be able to write to OneDrive host directory.

### 2. Prepare required stuff

Onedrive needs two volumes. One of them is the config volume. 

If you dont't need an extra config file, You can create a docker volume:

```bash
docker volume create onedrive_conf;
```

This will create a docker volume labeled 'onedrive_conf', which we will use it later.

The second one is your data folder that needs to sync with. Keep in mind that:

-   The owner of the folder must **NOT** be root

-   The owner have permission to its parent directory  
    (because onedrive will try to setup a monitor for the sync folder).

### 3. First run

Onedrive also needs to be authorized with your account.  
This is done by running docker in interactive mode. 

**make sure to change onedriveDir to your own.**

```bash
onedriveDir="/FULL/PATH/TO/YOUR/LOCAL/SYNC/DIR"
docker run -it --restart unless-stopped --name onedrive -v onedrive_conf:/onedrive/conf -v "${onedriveDir}:/onedrive/data" driveone/onedrive
```

-   You will be asked to open a specific link using your web browser 
-   login into your Microsoft Account and give the application the permission  
-   After giving the permission, you will be redirected to a blank page.  
-   Copy the URI of the blank page into the application.

If your onedrive is working as expected, you can detach from the container with Ctrl+p, Ctrl+q.

### 4. Status, stop, and restart

Check if monitor service is running

```bash
docker ps -f name=onedrive
```

Show monitor run logs

```bash
docker logs onedrive
```

Stop running monitor

```bash
docker stop onedrive
```

Resume monitor

```bash
docker start onedrive
```

Unregister onedrive monitor

```bash
docker rm -f onedrive
```

### 5. Edit the config

Onedrive should run in default configuration, but however you can change your configuration.  

First download the default config from [here](https://raw.githubusercontent.com/abraunegg/onedrive/master/config)  
Then put it into your onedrive_conf volume path, which can be found with:  

```bash
docker volume inspect onedrive_conf
```

Or you can map your own config folder to config volume (copy stuffs from docker volume first)

The detailed document for the config can be found here: [additional-configuration](https://github.com/abraunegg/onedrive#additional-configuration)

## Run or update with one script

If you are experienced with docker and onedrive, you can use the following script:

```bash
# Update onedriveDir with correct existing OneDrive directory path
onedriveDir="${HOME}/OneDrive"

firstRun='-d'
docker pull driveone/onedrive
docker inspect onedrive_conf > /dev/null || { docker volume create onedrive_conf; firstRun='-it'; }
docker inspect onedrive > /dev/null && docker rm -f onedrive
docker run $firstRun --restart unless-stopped --name onedrive -v onedrive_conf:/onedrive/conf -v "${onedriveDir}:/onedrive/data" driveone/onedrive
```

## Build instructions

```bash
git clone https://github.com/abraunegg/onedrive
cd onedrive/docker
git clone https://github.com/abraunegg/onedrive
docker build . -t driveone/onedrive
```
