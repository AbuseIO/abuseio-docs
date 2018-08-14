# Installation via Docker

## Prerequisites 

This guide assumes that you have already installed docker and it is running.

If this is not the case for you, find a guide for your OS that runs through the basic installation of Docker.

Here are a couple below:

Ubuntu
```
https://docs.docker.com/install/linux/docker-ce/ubuntu/
```

CentOS
```
https://docs.docker.com/install/linux/docker-ce/centos/
```

This guide also assumes that you have installed git on your server.


## Installation

The first step is to download the docker container for AbuseIO:
```bash
git clone https://github.com/AbuseIO/abuseio-docker.git
```

Next, change into the directory that was just created and build it:
```bash
cd abuseio-docker;
docker build -t abuseio:4.1 --build-arg GITHUB_TOKEN=github_token_goes_here .
```

You will need to create 3 directories, data, config and logs. See the example below:
```bash
mkdir -p /var/abuseio_data/{config,data,logs}
```

Now it is time to run the installer:
```bash
docker run -d -p 8000:8000 -p 3306:3306 -e "APP_URL=http://localhost:8000" -v /var/abuseio_data/config/:/config -v /var/abuseio_data/data/:/data -v /var/abuseio_data/logs/:/log abuseio:latest
```

Check that the container is definitely running:
```bash
docker ps
```

Now you will be able to connect to your instance of AbuseIO via a web browser on port 8000:
```
http://YOUR_IP_ADDRESS:8000
```
