# https://github.com/elastic/logstash-docker
FROM docker.elastic.co/logstash/logstash:6.4.1
COPY ./config/ /usr/share/logstash/config
COPY ./pipeline/ /usr/share/logstash/pipeline
# RUN make /proxy
# CMD