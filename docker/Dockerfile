# syntax=docker/dockerfile:1.3-labs

FROM       jupyter/datascience-notebook:ubuntu-22.04
MAINTAINER robert.citek@gmail.com

USER  root
RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        file \
        git \
        gnupg \
        graphviz \
        jq \
        less \
        man-db \
        mysql-client \
        pandoc \
        poppler-utils \
        postgresql-client \
        rsync \
        ruby \
        software-properties-common \
        tree \
        vim \
        ;

# install Docker
RUN   curl -fsSL https://download.docker.com/linux/$( lsb_release -is | tr A-Z a-z )/gpg | \
        gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
RUN   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/$( lsb_release -is | tr A-Z a-z ) $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN   apt-cache policy docker-ce
RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        docker-ce \
        ;
RUN   usermod -aG docker jovyan

# install mongodb client
RUN   curl -L -s https://www.mongodb.org/static/pgp/server-6.0.asc | \
      gpg --dearmor > /usr/share/keyrings/mongodb.gpg
RUN   echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" > /etc/apt/sources.list.d/mongodb-org-6.0.list
RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        mongodb-mongosh \
        ;

# install Python packages
USER  jovyan
RUN   pip install bash_kernel graphviz lxml html5lib
RUN   python -m bash_kernel.install

CMD   [ "start-notebook.sh" ]

ENTRYPOINT [ "tini", "-g", "--" ]

COPY Dockerfile /



