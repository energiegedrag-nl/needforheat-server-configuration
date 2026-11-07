FROM linuxserver/duplicati:2.4.0

WORKDIR /scripts

COPY ./script.sh /scripts/script.sh

RUN chown abc /scripts/script.sh && \
    chmod 700 /scripts/script.sh && \
    mkdir /dump_prod && \
    mkdir /dump_dev
