# Running Jupyter Lab from within Docker

This runs a Jupyter lab within Docker and listens on port :5150 on the Docker host.
The image builds upon the existing Docker image of [Jupyter Notebook](https://hub.docker.com/r/jupyter/datascience-notebook) by adding in
kernels for Julia and Bash, among others.
It also saves any created Jupyter notebooks on a shared folder with the host.
This allows for notebooks to persist across different Docker container instances.

See https://hub.docker.com/repository/docker/rwcitek/jupyter-notebook on DockerHub.

## Pull image ( optional )
```bash
docker image pull rwcitek/jupyter-notebook:latest
```

## Launch container
```bash
SHARED=~/shared.jupyter-lab
mkdir -p "${SHARED}"
docker \
    run \
    -d \
    -p :5150:8888 \
    -e JUPYTER_ENABLE_LAB=yes \
    -v "${SHARED}":/home/jovyan/shared \
#  Uncomment next line and delete this command to enable the socket for Docker-out-of-Docker
#    -v /var/run/docker.sock:/var/run/docker.sock \
    -w /home/jovyan/shared \
    --name jupyter-lab \
    rwcitek/jupyter-notebook:latest

host=192.168.1.8         # On the Mac ( the IP of any interface on the host )
host=127.0.0.1           # On a Chromebook or remote cloud instance using ssh tunneling ( -L :5150:127.0.0.1:5150 )

while true; do
  token=$( docker container logs --since 5s jupyter-lab 2>&1 | grep -m1 -o token=.* )
  [ "${token}" ] && echo -e "\n\n\nhttp://${host}:5150/lab?${token}\n\n\n" && break
  sleep 2
done |
tee /tmp/jupyter.url.token.txt

```
## Customize
```bash
docker exec -i jupyter-lab /bin/bash -c 'cat > ~/.bash_aliases' <<'eof'
  alias cls='clear';
  alias dir='ls -la';
  alias h='history';
  alias more='less -iX';
  export HISTCONTROL=ignoredups:ignorespace;
  export HISTFILESIZE=50000;
  export HISTSIZE=50000;
  export HISTTIMEFORMAT='%t%F %T%t';
  export PAGER='less -iX ';
  export IGNOREEOF=20;
  export PS1='\u@\h: \w\n\$ ';
eof
```

## Update system packages
Eventually, when the packages become too far out of date, I will update the base image.
But for now, this is a way to update the instance.
```bash
<<'eof' docker exec -i -u root jupyter-lab /bin/bash
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get dist-upgrade -y
    apt-get install -y --no-install-recommends \
        csvkit \
        ;
eof
```

### Enabling Docker-out-of-Docker
To enable "Docker-out-of-Docker", you will need to enable access to the socket ( see above comment during launch ). 
Docker is already installed.  See the Dockerfile or this [DooD gist](https://gist.github.com/rwcitek/81a942d9b7e35d104e16d1591f93018a) for installation details.

DooD allows you to manage Docker resources on the host from within a container instance.  This definitely has some security risks.  But if you own the host and are the only user on the host, it can be extremely useful.

## Update pip modules
Eventually, when the modules become too far out of date, I will update the base image.
But for now, this is a way to update the instance.
```bash
<<'eof' docker exec -i jupyter-lab /bin/bash
    # you'd think there'd be a cleaner way to upgrade all installed packages
    pip install --upgrade $( pip list | awk 'NR > 2 {print $1}' )
    pip install \
      dtale \
      mitoinstaller \
      openpyxl \
      wquantiles \
    ;
    python -m mitoinstaller install
eof
```
Blog post about [D-Tale](https://towardsdatascience.com/d-tale-one-of-the-best-python-libraries-you-have-ever-seen-c2deecdfd2b)

Blog post about [mito](https://towardsdatascience.com/mito-part-1-an-introduction-a-python-package-which-will-improve-and-speed-up-your-analysis-17d9001bbfdc)

## Container operations
#### Pausing
Pausing the container temporarily "inactivates" it.  It no longer responds to requests.
However, as a process, it is still considered running.  The upshot is that the up-time is unaffected and 
the same URL will work for connecting to it once you unpause it.
```bash
docker container pause jupyter-lab
docker container unpause jupyter-lab

# To redisplay the existing URL
host=192.168.1.8         # On the Mac ( the IP of any interface on the host )
host=127.0.0.1           # On a remote cloud instance using ssh tunneling
host=penguin.linux.test  # On a Chromebook

token=$( docker container logs jupyter-lab 2>&1 | tac | grep -m1 -o token=.* )
echo -e "\n\n\nhttp://${host}:5150/lab?${token}\n\n\n"
```
#### Stopping
Stopping the container actually shuts it down.
As a process, it is no longer running and therefore no longer responds to requests
even though the filesystem for the container still exists.
The upshot is that upon restarting, a new URL is created.
```bash
docker container stop jupyter-lab
docker container start jupyter-lab

# Wait for the new URL
host=192.168.1.8         # On the Mac ( the IP of any interface on the host )
host=127.0.0.1           # On a remote cloud instance using ssh tunneling
host=penguin.linux.test  # On a Chromebook

while true; do
  token=$( docker container logs --since 5s jupyter-lab 2>&1 | grep -m1 -o token=.* )
  [ "${token}" ] && echo -e "\n\n\nhttp://${host}:5150/lab?${token}\n\n\n" && break
  sleep 2
done |
tee /tmp/jupyter.url.token.txt
```
#### Commiting
Commiting the container creates a new image from the existing container.
This is not recommended, but can be useful in a pinch.
The container can be in any state: running, paused, or stopped.
The commit will pause the container and then unpause it upon completion.
In this example, the epoch time is used as the image tag.
```bash
docker container commit jupyter-lab jupyter-lab-commit:$( date +%s )
```
#### Removing
Removing the container does just that.
The container has to be stopped first.
```bash
docker container stop jupyter-lab
docker container rm jupyter-lab
```

## Dockerfile
Since the Dockerfile is included with the image, it can be copied out of the image.
```bash
docker \
    run \
    --rm \
    --entrypoint cat \
    rwcitek/jupyter-notebook:latest \
    /Dockerfile
```
## Docker build and push to Dockerhub
Once you have the Dockerfile, you can modify it and build a custom image.
```bash
docker build --tag rwcitek/jupyter-notebook:latest docker/
docker login
docker push rwcitek/jupyter-notebook:latest
```
Or build from the Dockerfile in Github
```bash
curl -s https://raw.githubusercontent.com/rwcitek/Docker.jupyter-lab/main/docker/Dockerfile |
docker image build --tag rwcitek/jupyter-notebook:latest -
```


