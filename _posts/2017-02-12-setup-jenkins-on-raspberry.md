---
title:  "Setup Jenkins server on Raspberry"
date:   2017-01-06
categories: [raspberry, jenkins]
tags: [raspberry, jenkins, setup]
---

### Step 1

Choose java jdk-8 as your java.

```
sudo update-alternatives --config java

There are 2 choices for the alternative java (providing /usr/bin/java).
```


### Step 2

Installation

```
mkdir jenkins && cd jenkins

wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -

sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt-get update

sudo apt-get install jenkins
```


### Step 3

Start Jenkins


```
sudo nano /etc/init.d/jenkins
```

Define your ports

```
HTTP_PORT=8000  
JENKINS_ARGS="--httpPort=$HTTP_PORT"  
```

To make sure the new port is used restart Jenkins

```
sudo /etc/init.d/jenkins restart  
```

