This is currently a work-in-progress.  This does not yet result in a working instance of TT-RSS (packetgeek, 1 Aug 2022).

# TTRSS on K3S Kubernetes Ryzen-based mini pc, with NFS-based persistence

This is for my own education and personal use.  Please refer to Antonio Correia's code if you're attempting similar (I do horrible things to other people's code).

## My rig
I'm running a Minikube node on a Coofun mini-PC.  It's running Ubuntu Server 22.04 (no GUI).  It's configured with NFS-based persistence (a departure from Mr. Correia's configuration).  My intent is to track various feeds on Medium, as well as other, more "normal", RSS feeds.

# Steps

1) Grab a copy of the code navigate into the folder for the app container:
```
git clone https://github.com/packetgeek/K3S-TTRSS
cd K3S-TTRSS/Containers/app
```

2) Build the app image via:
```
docker build -t local/ttrss-app .
```
    Note the "space period" at the end of the above.

3) Perform similar for the updater, via:
```
cd ../updater
docker build -t local/ttrss-updater .
```

## Installation

Once you have the containers built and stored in a local repo, the following should install TT-RSS in your K8S node.
This command should install TTRSS successfully on a kubernetes cluster
```
kubectl apply -f .
```
## From Mr. Correia's page 

Since kubernetes cannot build the containers to run in the pods, I had to build them manually from the official docker-compose repository.

I cloned the original docker compose scripts (https://git.tt-rss.org/fox/ttrss-docker-compose) and modified it to build the individual containers:
ajvcorreia/ttrss-app
ajvcorreia/ttrss-updater
ajvcorreia/ttrss-web_nginx

I had to modify the nginx container to replace the app server name in the nginx.conf on startup as this is only available after kubernetes starts the pod.
