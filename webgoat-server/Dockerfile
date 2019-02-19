FROM openjdk:8u181-jre-slim-stretch

ARG webgoat_version=8.0.0.M3

RUN useradd --home-dir /home/webgoat --create-home -U webgoat

RUN apt-get update && apt-get upgrade && apt-get install curl -y && apt-get clean

COPY start.sh /home/webgoat/start.sh
RUN chmod +x /home/webgoat/start.sh

USER webgoat
RUN cd /home/webgoat/; mkdir -p .webgoat
COPY target/webgoat-server-${webgoat_version}.jar /home/webgoat/webgoat.jar
