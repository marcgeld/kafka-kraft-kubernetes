# Build: docker build --platform=linux/amd64 -t kafka:latest .
FROM ubuntu:latest as builder
ENV KAFKA_VERSION=3.6.1
ENV SCALA_VERSION=2.13
ENV KAFKA_HOME=/opt/kafka
ENV KAFKA_DATA=/var/lib/kafka
ENV PATH=${PATH}:${KAFKA_HOME}/bin
ENV JAVA_DL="https://github.com/graalvm/graalvm-ce-builds/releases/download/jdk-21.0.1/graalvm-community-jdk-21.0.1_linux-x64_bin.tar.gz"

RUN apt-get update && apt-get install -y \
    binutils \
    curl \
    unzip

WORKDIR /tmp

# Install Java
RUN curl --fail -L --silent -o java.tgz ${JAVA_DL} && \
    mkdir /tmp/javaexec && \
    tar -zxvf java.tgz --strip-components=1 -C /tmp/javaexec && \
    rm -rf java.tgz

# Install Kafka
RUN curl --fail -L --silent -o kafka.tgz \
    https://downloads.apache.org/kafka/${KAFKA_VERSION}/kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz && \
    mkdir /tmp/kafka && \
    tar -zxvf kafka.tgz --strip-components=1 -C /tmp/kafka && \
    rm -rf kafka.tgz .tgz

FROM ubuntu:latest
LABEL name="kafka" version=${KAFKA_VERSION}

ENV KAFKA_NODE_ID=1
ENV KAFKA_INTER_BROKER_LISTENER_NAME=BROKER

ENV LANG=C.UTF-8
ENV JAVA_HOME=/app/javaexec
ENV KAFKA_HOME=/app/kafka
ENV KAFKA_DATA=/var/lib/kafka
ENV KAFKA_CID_VOL=/app/clusterId
ENV PATH=${PATH}:${KAFKA_HOME}/bin:${JAVA_HOME}/bin
ENV KAFKA_LOG_DIRS="${KAFKA_DATA}/logs"
ENV KAFKA_HEAP_OPTS="-Xmx1G -Xms512m"
ENV KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"


RUN apt update && apt install -y sudo \
    && groupadd --gid 1000 app \
    && adduser --gid 1000 --disabled-password app \
    && echo 'app ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers \
    && mkdir /app \
    && sudo chown -R app:app /app \
    && mkdir ${KAFKA_CID_VOL} \
    && sudo chown -R app:app ${KAFKA_CID_VOL}

COPY --from=builder --chown=app /tmp/javaexec /app/javaexec 
COPY --from=builder --chown=app /tmp/kafka /app/kafka 
USER app
WORKDIR ${KAFKA_HOME}
EXPOSE 9092 9093

VOLUME [ "${KAFKA_DATA}", "${KAFKA_CID_VOL}" ]

# Kafka initialization and startup in ENTRYPOINT
ENTRYPOINT if [ ! -f /kid/KAFKA_CLUSTER_ID ]; then \
               ${KAFKA_HOME}/bin/kafka-storage.sh random-uuid > ${KAFKA_CID_VOL}/KAFKA_CLUSTER_ID; \
               KAFKA_CLUSTER_ID=$(cat "${KAFKA_CID_VOL}/KAFKA_CLUSTER_ID"); \
               ${KAFKA_HOME}/bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties; \
           fi; \
           echo "KAFKA_CLUSTER_ID: $KAFKA_CLUSTER_ID"; \
           exec ${KAFKA_HOME}/bin/kafka-server-start.sh ${KAFKA_HOME}/config/kraft/server.properties

