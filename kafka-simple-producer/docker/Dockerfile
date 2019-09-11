FROM ubuntu:16.04

RUN apt-get update && \
  apt-get install -y software-properties-common && \
  add-apt-repository ppa:jonathonf/python-3.6
RUN apt-get update

RUN apt-get install -y build-essential python3.6 python3.6-dev python3-pip python3.6-venv
RUN apt-get install -y git librdkafka-dev

# update pip
RUN python3.6 -m pip install pip --upgrade
RUN python3.6 -m pip install wheel
RUN pip install numpy confluent-kafka arrow
COPY src /opt/bin

#CMD ["/opt/bin/es_test_data.py", "--es_url=http://elasticsearch-master:9200"]