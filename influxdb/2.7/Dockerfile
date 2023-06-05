FROM cr.loongnix.cn/library/debian:buster

RUN export DEBIAN_FRONTEND=noninteractive && \
    # install dependencies \
    apt-get update -y && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      gnupg && \
    # cleanup apt \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV arch loong64

# Install dasel for configuration parsing
Run curl -fLo /usr/local/bin/dasel https://github.com/Loongson-Cloud-Community/dasel/releases/download/v2.1.2/dasel_linux_loong64 && \
    chmod +x /usr/local/bin/dasel && \
    dasel --version

RUN groupadd -r influxdb --gid=1000 && \
    useradd -r -g influxdb --uid=1000 --create-home --home-dir=/home/influxdb --shell=/bin/bash influxdb

# Install gosu for easy step-down from root
RUN curl -fLo /usr/local/bin/gosu https://github.com/Loongson-Cloud-Community/gosu/releases/download/1.12/gosu-loongarch64 && \
    chmod +x /usr/local/bin/gosu && \
    gosu --version && \
    gosu nobody true

# Install the influxd server
ENV INFLUXDB_VERSION 2.7.1
RUN curl -fLO https://github.com/Loongson-Cloud-Community/influxdb/releases/download/v2.7.1/influxdb2-2.7.1-linux-loong64.tar.gz && \
    tar xzf "influxdb2-${INFLUXDB_VERSION}-linux-${arch}.tar.gz" && \
    cp "influxdb2_linux_${arch}/influxd" /usr/local/bin/influxd && \
    rm -rf "influxdb2-${INFLUXDB_VERSION}-linux-${arch}.tar.gz" \
           "influxdb2_linux_${arch}" && \
    influxd version

# Install the influx CLI
ENV INFLUX_CLI_VERSION 2.7.3
RUN curl -fLO https://github.com/Loongson-Cloud-Community/influx-cli/releases/download/v2.7.3/influxdb2-client-2.7.3-linux-loong64.tar.gz && \
    tar xzf "influxdb2-client-${INFLUX_CLI_VERSION}-linux-${arch}.tar.gz" && \
    cp influx /usr/local/bin/influx && \
    rm -rf "influxdb2-client-${INFLUX_CLI_VERSION}-linux-${arch}" \
           "influxdb2-client-${INFLUX_CLI_VERSION}-linux-${arch}.tar.gz" && \
    influx version

# Create standard directories expected by the entry-point.
RUN mkdir /docker-entrypoint-initdb.d && \
    mkdir -p /var/lib/influxdb2 && \
    chown -R influxdb:influxdb /var/lib/influxdb2 && \
    mkdir -p /etc/influxdb2 && \
    chown -R influxdb:influxdb /etc/influxdb2
VOLUME /var/lib/influxdb2 /etc/influxdb2

COPY default-config.yml /etc/defaults/influxdb2/config.yml
COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["influxd"]

EXPOSE 8086

ENV INFLUX_CONFIGS_PATH /etc/influxdb2/influx-configs
ENV INFLUXD_INIT_PORT 9999
ENV INFLUXD_INIT_PING_ATTEMPTS 600
ENV DOCKER_INFLUXDB_INIT_CLI_CONFIG_NAME default
