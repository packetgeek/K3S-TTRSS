This is currently a work-in-progress.  This does not yet result in a working instance of TT-RSS (packetgeek, 1 Aug 2022).

# TTRSS in Minikube, on a Ryzen-based mini-PC, with NFS-based persistence

This is for my own education and personal use.  Please refer to Antonio Correia's code if you're attempting similar (I do horrible things to other people's code).

## Assumptions

- In the below, 192.168.2.105 is the IP address of my private Docker registry.

## My rig

I'm running a Minikube node on a Coofun mini-PC.  It's running Ubuntu Server 22.04 (no GUI).  It's configured with NFS-based persistence (a departure from Mr. Correia's configuration).  My intent is to track various feeds on Medium, as well as other, more "normal", RSS feeds.

# Steps - Creating the images

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

4) Perform similar for the nginx container, via:
```
cd ../web-nginx
docker build -t local/ttrss-web_nginx
```
You should now have three images built.  Running "docker images" should show something like:
```
REPOSITORY                        TAG         IMAGE ID       CREATED              SIZE
local/ttrss-web_nginx             latest      745db2243c8f   About a minute ago   23.5MB
local/ttrss-updater               latest      8760f455537b   5 minutes ago        92.6MB
local/ttrss-app                   latest      ad22edabf9ef   7 minutes ago 
```

5) Retag each image so that the private hosts IP and port are part of the tag name.  This can be done via:
```
docker images
docker tag 745db2243c8f 192.168.2.105:5000/local/ttrss-web_nginx
```
6) Perform Step #5 on the other two images.  Now running "docker images" should show something like:
```
192.168.2.105:5000/local/ttrss-web_nginx   latest      745db2243c8f   12 hours ago   23.5MB
local/ttrss-web_nginx                      latest      745db2243c8f   12 hours ago   23.5MB
192.168.2.105:5000/local/ttrss-updater     latest      8760f455537b   12 hours ago   92.6MB
local/ttrss-updater                        latest      8760f455537b   12 hours ago   92.6MB
192.168.2.105:5000/local/ttrss-app         latest      ad22edabf9ef   12 hours ago   92.6MB
local/ttrss-app                            latest      ad22edabf9ef   12 hours ago   92.6MB
```
7) Push the image to the local registry via:
```
docker push 192.168.2.105:5000/local/ttrss-app
```

8) Perform step #7 for the other two images.

9) Push all three images to your local registry.

*** in the following, suggest changing imagePullPolicy from Always to IfNotPresent ***

10) Edit the ttrss-app.yml file and, in the "image:" line, replace "ajvcorreia/ttrss-app" with "192.168.2.105:5000/local/ttrss-app"

11) Edit the ttrss-nginx.yml file and, in the "image:" line, replace "ajvcorreia/ttrss-web_nginx" with "192.168.2.105:5000/local/ttrss-web_nginx"

12) Edit the ttrss-updater.yml file and, in the "image:" line, replace "ajvcorreia/ttrss-updater" with "192.168.2.105:5000/local/ttrss-updater"

13) Edit ttrss-app-pvc.yml and change "loghorn" to "stadard".

14) Edit ttrss-db-pvc.yml and change "longhorn" to "standard".


**Development is still occurring at this point.  Below cannot be used yet.**

**** need to edit the pvc files next ***

**** need to file a bug with Mr. Correia about two imagepullpolicy lines in his db manifest ***


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
