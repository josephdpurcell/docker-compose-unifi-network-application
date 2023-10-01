# Unifi controller via docker

This is a how-to for using https://hub.docker.com/r/linuxserver/unifi-network-application

# Background

Let's start with the fact of the Unifi controller software. Why. Why do I need all this state to control my network. I just want to go to the AP's IP and control it just like I do for the edge router. I have such a simple setup. But, alas, I am not a networking expert, and I know network orchestration is a big deal. There are probably good reasons.

Then, how to run it? Fortunately I have experience getting applications to run, so thats not a problem just a bit of a hassle since it's not like you can just have a simple binary to run.

I tried https://hub.docker.com/r/jacobalberty/unifi but it requires forcing APs into override notify and that felt not awesome. I wasn't even seeing the AP show up. I tried linuxserver's and I saw the AP but it got stuck in "Adopting" and then realized you have to do the same thing -- force override the notification thing.

Based on what I see it seems like a great approach is to use https://github.com/linuxserver/docker-unifi-network-application with docker compose. I personally like that, at least. Thank you `linuxserver`!

In the `linuxserver` docs I couldn't find a complete docker-compose example that included mongo, so I'm making this repo.

Goals:

* Use docker-compose
* Any state is mounted as a volume (so the host machine can backup data)
* Assume all required containers are in the docker compose (the linuxserver one assumes mongo is external)

But, honestly this repo is really just help me remember how to do this again.

# DISCLAIMER

You are at your own risk! I ran through this and it worked for me, but some steps don't make sense. For example, I'm not seeing db/config dir get populated. And, what does recovery look like if I lose all the data for the unifi controller? I think my plan is just to reset, because now that I know the setup it would probably take me 10 mins to set it up again as opposed to the 200 mins it's taken me to figure out how to just configure it in the first place.

# Assumptions

## About your network

I tested these instructions with Ubuntu 18 or so and Unifi 6 Professional. The Ubuntu machine runs the Unifi Controller in a docker container that I'll spin up when I need to, and is plugged in hard-wired to a Ubiquity Edge Router on port 3. Edge router port 2 has the 48 v POE injector (because Unifi no longer comes with them! that got me). Edge router port 1 is configured to receive DHCP from my ISP's router on a `192.168.1.*`, so Edge Router is handing out DHCP on ports 2, 3, and 4 on `192.168.2.*`. Very basic setup.

These instructions assumes 192.168.2.39 is the Unifi AP and 192.168.2.38 is the machine running the docker container Unifi controller. Your IPs will most likely differ.

## About security

I'm OK using defaults for the controller and mongo stuff. I am hoping I boot the container once, configure the AP, and then never have to boot the software again because either (a) my Unifi AP outlives me, or (b) Ubiquity offers something easier and I can deprecate this repo.

You COULD still use this composer example just make sure you don't use default passwords and apply all the magic sauces that keep things secure.

# Getting started

## 1. Setup the folder structure

You COULD just clone this repo, but lets pretend you didn't and you just want to start fresh. Here's what to do:

```
mkdir -p app/config
mkdir -p db/data
mkdir -p db/config
```

Copy the `docker-compose.yml` from this repo into `./` and copy `db/init-mongo.js` from the repo into `db/init-mongo.js`.

Your cwd should have folders/files like:

```
app/
└── config/
db/
├── config/
├── data/
└── init-mongo.js
docker-compose.yml
```

## 2. Start the docker the first time

You need to understand docker to make sense of all this, but short version if you never booted the container do this:

NOTE: I run this in the foreground so I can see things happening, if you want it in background add `-d`

```
docker compose up
```

Eventually output should hit a pause with a line like:

```
unifi-network-application  | [ls.io-init] done.
```

The container should now be setup. Yay.

Go to: https://localhost:8443/, accept risk and continue (its self-signed SSL)

I like to click "manual" then skip so I don't manage it through Ubiquity's identity, just lets me have local access

At this point you should be logged into the Unifi controller website thing

## 3. Adopt

In the UI you should get prompted to adopt an AP, go ahead, and then it'll be stuck in "Adopting"

SSH into the AP (you can find the IP for it on the Edge Router), note the algorithm may need to be specified like this:

```
ssh -oHostKeyAlgorithms=+ssh-dss ubnt@192.168.2.39
```

Then run this command, using the IP of the machine running the docker service:

```
set-inform http://192.168.2.38:8080/inform
```

Tip: before running that command make sure 8080 is open:

```
nc -w 1 -z 192.168.2.38 8080
echo $?
```

You should get `0` from `echo $?`

Ok, at this point the unifi controller website should show the access point is all good.

### 4. Shut it down

At this point you can shut down the Unifi controller docker container:

```
docker compose stop
```

NOTE: do NOT run down, just use stop

### 5. Bring it back up when you need it

Want to see how your AP is doing? Start the unifi application again:

```
docker compose start
```

You must be in the root directory, the one with the `docker-compose.yml` file.

NOTE: do NOT run up, just do start.

NOTE: if `docker compose` command doesn't work try `docker-compose` instead, not sure why there are two variants.

# Contributing

Nah. Well, maybe? Hopefully some day this repo gets deleted and put into https://github.com/linuxserver/docker-unifi-network-application#usage as a recipe?

