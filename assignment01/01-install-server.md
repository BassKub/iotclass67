# Install Server and Docker


## How to install Server
1. Download Ubuntu Server from https://ubuntu.com/download/server and install it to USB to make it ready to boot ubuntu os.
2. Plug in Usb to server computer and select Ubuntu server and wait for set up.
3. install tools that need for install docker like vim, ifconfig, git.

## How to install Docker
I use this web for install Docker : https://docs.docker.com/engine/install/ubuntu/
1. Set up Docker's apt repository.
    ## Add Docker's official GPG key:
     - sudo apt-get update
     - sudo apt-get install ca-certificates curl
     - sudo install -m 0755 -d /etc/apt/keyrings
     - sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
     - sudo chmod a+r /etc/apt/keyrings/docker.asc

    ## Add the repository to Apt sources:
    - echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    - sudo apt-get update

2. Install the Docker packages.
  - sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

3. Verify that the Docker Engine installation is successful by running the hello-world image.
  - sudo docker run hello-world


