FROM jenkins/jenkins
USER root
RUN apt-get update && \
    apt-get install -y uidmap
RUN curl -sSL https://get.docker.com/ | sh
RUN usermod -a -G docker jenkins
# USER jenkins
# RUN dockerd-rootless-setuptool.sh install
