# syntax=docker/dockerfile:1.3-labs

FROM       jupyter/datascience-notebook:396024a4ddc1
MAINTAINER robert.citek@gmail.com

USER  root
RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        ruby \
        tree \
        less \
        file \
        rsync \
        pandoc \
        graphviz \
        man-db \
        mysql-client \
        jq \
        poppler-utils \
        vim \
        ;
RUN   pip install bash_kernel graphviz lxml html5lib
RUN   python -m bash_kernel.install

USER  jovyan

CMD   [ "start-notebook.sh" ]

ENTRYPOINT [ "tini", "-g", "--" ]

COPY Dockerfile /

