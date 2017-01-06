---
title:  "Setup Minecraft server on Raspberry"
date:   2017-01-06
categories: [raspberry, minecraft]
tags: [raspberry, minecraft]
---

In this article I want to document how to setup a Minecraft server on raspberry pi (3).
The content is heavily borrowed by [jankarres](https://jankarres.de).

### Step 1

First of all you should configure your raspberry pi

```
sudo raspi-config
```
1. Expand Filesystem
2. Change user password
3. Change hostname
4. Set boot to CLI
5. Set GPU Memory to 16
6. Update your localisation settings


### Step 2 (optional)

Expand your swap file-size.

```
sudo su -c 'echo "CONF_SWAPSIZE=2048" > /etc/dphys-swapfile'
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
```


### Step 3

Install dependencies.

```
sudo apt-get install git screen
```


### Step 4

Download & patch the Spigot server.

```
wget https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar
java -Xmx1G -jar BuildTools.jar
```

### Step 5

Create a run script to start the Minecraft server.

> Keep in mind that you have to update the spigot version number inside the run script @ **spigot-1.8.jar**. 

```
#!/bin/sh
BINDIR=$(dirname "$(readlink -fn "$0")")
cd "$BINDIR"
java -Xmx1024M -XX:ParallelGCThreads=8 -Xincgc -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSIncrementalPacing -XX:+AggressiveOpts -XX:+CMSParallelRemarkEnabled -XX:+DisableExplicitGC -XX:MaxGCPauseMillis=500 -XX:SurvivorRatio=16 -XX:TargetSurvivorRatio=90 -XX:+UseAdaptiveGCBoundary -XX:-UseGCOverheadLimit -Xnoclassgc -XX:UseSSE=3 -XX:LargePageSizeInBytes=4m -jar spigot-1.8.jar nogui
```

### Step 6

Add permission.

```
sudo chmod +x run.sh
```


### Step 7

Start your server for the first time.

```
sudo ./run.sh
```

### Step 8

Accept the EULA by setting `eula=true` in the `eula.txt`.

```
vi eula.txt
```

### Step 9

Update the default config of the spigot server to improve the performance.

Config download: [spigot_config_2015.02.tar.gz](../../files/spigot_config_2015_02.tar.gz)


### Step 10 (optional)

Start your Minecraft server in a screen session so it keeps running in the 'background'.


1. Navigate to your server's directory.

2. Run the command: `screen -S ScreenNameGoesHere`

3. Then you'll be in your screen session, so run the server startup script.

4. To leave the screen session (and leave it running) press and hold CTRL, A, D (in that order).

5. To resume the screen session run the command: screen -r ScreenNameGoesHere

6. To kill the screen session run the command: screen -S ScreenNameGoesHere -X kill



### Availability
> If you want to make your server public available you need to use a dynip service like: [noip.com](https://www.noip.com/)