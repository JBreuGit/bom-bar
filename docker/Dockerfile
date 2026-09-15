FROM eclipse-temurin:21-jre

# start.sh relies on curl and jq for the AWS / docker-dev profiles
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl jq && \
    rm -rf /var/lib/apt/lists/*

RUN addgroup --system java && \
    adduser --system --ingroup java --home /home/java java && \
    mkdir /app && \
    chown java:java /app

COPY target/BOM-bar*.jar /app/service.jar
COPY docker/start.sh /app/start.sh

USER java
WORKDIR /app

ENV JAVA_TOOL_OPTIONS="-XX:+UseContainerSupport"

EXPOSE 8080

ENTRYPOINT ["/app/start.sh"]
