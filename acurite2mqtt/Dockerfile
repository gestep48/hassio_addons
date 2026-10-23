FROM ghcr.io/home-assistant/base:latest AS builder

RUN apk add --no-cache build-base cmake git pkgconf libusb-dev openssl-dev && \
    (apk add --no-cache rtl-sdr-dev || apk add --no-cache librtlsdr-dev)

RUN git clone --depth 1 https://github.com/merbanan/rtl_433.git /src/rtl_433

# TEMPORARY: Detlef's Optimus 6-in-1 decoder (rtl_433 #3729) until the official fix is merged.
# To go back to stock rtl_433, delete this COPY line and the acurite.c file in this folder.
COPY acurite.c /src/rtl_433/src/devices/acurite.c

RUN cmake -S /src/rtl_433 -B /src/rtl_433/build -DCMAKE_BUILD_TYPE=Release && \
    cmake --build /src/rtl_433/build -j2 && \
    DESTDIR=/out cmake --install /src/rtl_433/build

FROM ghcr.io/home-assistant/base:latest

# Define environment variables
# Use this variable when creating a container to specify the MQTT broker host.
ENV MQTT_HOST=127.0.0.1 \
    MQTT_PORT=1883 \
    MQTT_USERNAME="" \
    MQTT_PASSWORD="" \
    MQTT_RETAIN=True \
    MQTT_TOPIC=rtl_433 \
    PROTOCOL="" \
    WHITELIST_ENABLE=False \
    EXPIRE_AFTER=0 \
    WHITELIST="" \
    DISCOVERY_PREFIX=homeassistant \
    DISCOVERY_INTERVAL=600 \
    AUTO_DISCOVERY=True \
    DEBUG=False \
    LANG=C.UTF-8

LABEL Maintainer="Jeffrey Stone" \
Description="This image is used to start the RTL433 to HASS script that will monitor for 433Mhz devices and send the data to an MQTT server"

WORKDIR /data

# Copy scripts
COPY entry.sh rtl_433_mqtt_hass.py /scripts/

# Runtime packages (rtl_433 itself is compiled in the first stage)
RUN apk add --no-cache rtl-sdr libusb openssl mosquitto-clients python3 py3-paho-mqtt && \
    chmod +x /scripts/entry.sh && \
    chmod +x /scripts/rtl_433_mqtt_hass.py

# Newest rtl_433 built from source
COPY --from=builder /out/usr/local/ /usr/local/

# Execute entry script
ENTRYPOINT [ "/scripts/entry.sh" ]
