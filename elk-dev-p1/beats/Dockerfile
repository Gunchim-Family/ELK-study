FROM docker.elastic.co/beats/filebeat:7.17.1

COPY filebeat/filebeat.yml /usr/share/filebeat/filebeat.yml
COPY filebeat/modules.d /usr/share/filebeat/modules.d
RUN filebeat modules enable logstash

USER root
RUN chown root:filebeat /usr/share/filebeat/filebeat.yml

USER filebeat
CMD ["filebeat", "-e", "-c", "/usr/share/filebeat/filebeat.yml"]

# 도커 파일 명령어
# docker build -t filebeat_docker_image .
# docker run --name docker_container_name --network docker-compose_elk-network -d logstash_docker_image
