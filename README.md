# Host Open-source Applications on Compute Engine

Description: This is a hands-on workshop on the steps required to run open-source docker applications on a Compute Engine instance in Google Cloud Platform and serve the apps via https.

When you read this, it's safe to assume that you have met the pre-requisites:

- Created a Compute Engine instance from GCP
- Able to SSH into the Compute Engine instance
- Own a subdomain (e.g., domain.com) and have access to its DNS settings
- Have API token key for the domain registrar (e.g., Cloudflare, Namecheap, etc) to authorize issuance of SSL certificates

Note: If you are experienced, some of these steps are interchangeable if you want to host apps from your own personal computer.

So, what are the next steps?

- [Docker Installation](#docker-installation)
- [Setup Docker Network](#setup-docker-network)
- [Setup the env files](#setup-the-env-files)
- [Run the applications](#run-the-applications)
- [Configure Nginx Proxy Manager](#configure-nginx-proxy-manager)

## Docker Installation

Now that we've SSH-ed into our Compute Engine instance, we need to install Docker before we can run the apps that we want. So we need to run the following commands

Note: installation command taken from official Docker installation instructions for Linux Debian 12 [here](https://docs.docker.com/engine/install/debian/)

```sh
# Remove pre-existing docker installation from the machine
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done &&

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Give our user permission to run `docker` without sudo privileges
sudo groupadd docker
sudo usermod -aG docker $USER

# Ensure docker always run when compute instance is turned on
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

exec $SHELL
```

If everything went well, you should be able to verify if the installation went well with this command:

```sh
docker run hello-world
```

## Setup Docker Network

To achieve our goal of having our docker apps accessible by our web server (`nginx-proxy-manager`) when there's an incoming request, we need to create a docker network and put all our docker containers within the same network

```sh
docker network create --driver bridge services-network
```

## Setup the env files

To make it easier to manage our own apps, let's create `.env` files from the given `.env.example` files in each app subdirectories

## Run the applications

Now that we have everything set up, we can now run our applications. For each subdirectory, run the following commands:

```sh
cd <app-directory>
docker compose up -d
```

## Configure Nginx Proxy Manager

Now from your actual PC, run the following command to forward the `nginx-proxy-manager` dashboard app to your machine:

```sh
ssh user@ip-address -L 4001:localhost:81 # forward cloud VM's port 81 to your machine's port 4001
```

Then access your machine there and start configuring the following the items:

- Proxy hosts (helps NGINX understand which app to forward the incoming request to)
- SSL Certificates (helps secure the connection between the user and the NGINX server)

# Conclusion

You have reached the end of this README, meaning that you don't have to deal with the editor anymore, phew!
