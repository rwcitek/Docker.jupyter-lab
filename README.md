# Running Jupyter lab from within Docker

This runs a Jupyter lab within Docker and listens on port :5051 on the Docker host.
It also saves any created Jupyter notebooks on a shared folder with the host.
This allows for notebooks to persist across different Docker container instances.

```bash
SHARED=/tmp/zfoo
mkdir -p "${SHARED}"
docker \
    run \
    --rm \
    -p :5150:8888 \
    -e JUPYTER_ENABLE_LAB=yes \
    -v "${SHARED}":/home/jovyan/shared \
    -w /home/jovyan/shared \
    --name jupyter-lab \
    rwcitek/rwc-jupyter-notebook:latest \
    >& /tmp/docker.log & sleep 5

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

host=192.168.1.8         # On the Mac ( the IP of any interface on the host )
host=127.0.0.1           # On a remote cloud instance using ssh tunneling
host=penguin.linux.test  # On a Chromebook

echo
echo http://${host}:5150/lab?$( grep -m1 -o token=.* /tmp/docker.log )
echo
```

## Dockerfile

```bash
docker \
    run \
    --rm \
    -i \
    --entrypoint cat \
    rwcitek/rwc-jupyter-notebook:latest \
    /Dockerfile
```

