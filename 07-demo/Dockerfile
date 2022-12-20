FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y python3 python3-pip openssh-client sudo vim && \
    python3 -m pip install ansible-core==2.13.6 && \
    groupadd -f -g 1010 ansible && \
    useradd ansible -u 1000 -g ansible -m -s /bin/bash && \
    sudo -u ansible ansible-galaxy collection install community.digitalocean:=1.22.0

COPY ansible_project /opt/

RUN chmod +xw /opt/

USER ansible

