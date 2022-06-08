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
```bash
docker exec -i jupyter-lab /bin/bash <<'eof'
    pip install --upgrade $( pip list | awk 'NR > 2 {print $1}' )
    pip install openpyxl
eof
```

## Dockerfile

```bash
docker \
    run \
    --rm \
    --entrypoint cat \
    rwcitek/jupyter-notebook:latest \
    /Dockerfile
```
## Docker build and push to Dockerhub
```bash
docker build --tag rwcitek/jupyter-notebook:latest docker/
docker login
docker push rwcitek/jupyter-notebook:latest
```
