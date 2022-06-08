# Running Jupyter Lab from within Docker

This runs a Jupyter lab within Docker and listens on port :5051 on the Docker host.
The image builds upon the existing Docker image of [Jupyter Notebook](https://hub.docker.com/r/jupyter/datascience-notebook) by adding in
engines for Julia and Bash, among others.
It also saves any created Jupyter notebooks on a shared folder with the host.
This allows for notebooks to persist across different Docker container instances.

See https://hub.docker.com/repository/docker/rwcitek/jupyter-notebook on DockerHub.

## Launch container
```bash
SHARED=/tmp/zfoo
mkdir -p "${SHARED}"
docker \
    run \
    -d \
    -p :5150:8888 \
    -e JUPYTER_ENABLE_LAB=yes \
    -v "${SHARED}":/home/jovyan/shared \
    -w /home/jovyan/shared \
    --name jupyter-lab \
    rwcitek/jupyter-notebook:latest

host=192.168.1.8         # On the Mac ( the IP of any interface on the host )
host=127.0.0.1           # On a remote cloud instance using ssh tunneling
host=penguin.linux.test  # On a Chromebook

while true; do
  token=$( docker container logs --since 5s jupyter-lab 2>&1 | grep -m1 -o token=.* )
  [ "${token}" ] && echo -e "\n\n\nhttp://${host}:5150/lab?${token}\n\n\n" && break
  sleep 2
done
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

## Update pip modules
Eventually, when the modules become too far out of date, I will update the base image.
But for now, this is a way to update the instance.
```bash
docker exec -i jupyter-lab /bin/bash <<'eof'
    # you'd think there'd be a cleaner way to upgrade all installed packages
    pip install --upgrade $( pip list | awk 'NR > 2 {print $1}' )
    pip install \
      dtale \
      openpyxl \
    ;
eof
```

## Container operations
#### Pausing
Pausing the container temporarily "inactivates" it.  It no longer responds to requests nor work on any data.
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
# waiting for the new URL
while true; do
  token=$( docker container logs --since 5s jupyter-lab 2>&1 | grep -m1 -o token=.* )
  [ "${token}" ] && echo -e "\n\n\nhttp://${host}:5150/lab?${token}\n\n\n" && break
  sleep 2
done
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
