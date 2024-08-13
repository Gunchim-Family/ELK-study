FROM docker.elastic.co/logstash/logstash:7.17.23

RUN rm -f /usr/share/logstash/pipeline/logstash.conf
RUN rm -f /usr/share/logstash/config/pipelines.yml
COPY pipeline/ /usr/share/logstash/pipeline/
COPY config/ /usr/share/logstash/config/

# 도커 파일 명령어
# docker build -t logstash_docker_image .
# docker run -p 5002:5002 -p 5010:5010 --name docker_container_name --network docker-compose_elk-network -d logstash_docker_image
# docker exec -it docker_container_name /bin/bash
# docker logs docker_container_name
