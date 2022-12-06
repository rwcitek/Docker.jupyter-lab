# syntax=docker/dockerfile:1.3-labs

FROM       jupyter/datascience-notebook:ubuntu-22.04
MAINTAINER robert.citek@gmail.com

USER  root
RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        file \
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
        tree \
        vim \
        ;
RUN   pip install bash_kernel graphviz lxml html5lib
RUN   python -m bash_kernel.install

USER  jovyan

CMD   [ "start-notebook.sh" ]

ENTRYPOINT [ "tini", "-g", "--" ]

COPY Dockerfile /

