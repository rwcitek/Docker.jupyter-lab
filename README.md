# Docker.jupyter-lab
Running Jupyter lab from within Docker

Initial notes.  Needs cleanup.

```bash
  ( true
SHARED=~/src/github/projects/rwcitek/random_notes/anaconda/
mkdir -p "${SHARED}"
docker \
    run \
    --rm \
    -p 127.0.0.1:5150:8888 \
    -e JUPYTER_ENABLE_LAB=yes \
    -v "${SHARED}":/home/jovyan/shared \
    -v /tmp/zfoo/:/tmp/zfoo/ \
    -w /home/jovyan/shared \
    --name jupyter-lab \
    rwc-jupyter-notebook:latest \
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

echo
echo http://localhost:5150/lab?$( grep -m1 -o token=.* /tmp/docker.log )
echo




  ( true
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
    rwc-jupyter-notebook:latest \
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

echo
echo http://192.168.1.8:5150/lab?$( grep -m1 -o token=.* /tmp/docker.log )
echo
```



