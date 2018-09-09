FROM confluentinc/cp-kafka:5.0.0

ENV PROMETHEUS_EXPORTER_VERSION=0.3.1
RUN mkdir -p /opt/prometheus/conf \
    && curl -SLs "https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${PROMETHEUS_EXPORTER_VERSION}/jmx_prometheus_javaagent-${PROMETHEUS_EXPORTER_VERSION}.jar" \
          -o /opt/prometheus/jmx_prometheus_javaagent.jar \
    && chmod 444 /opt/prometheus/jmx_prometheus_javaagent.jar \
    && chmod 555 /opt/prometheus/conf

ENV KAFKA_OPTS="-javaagent:/opt/prometheus/jmx_prometheus_javaagent.jar=5555:/opt/prometheus/conf/prometheus-config.yml"

ADD prometheus-config.yml /opt/prometheus/conf/
