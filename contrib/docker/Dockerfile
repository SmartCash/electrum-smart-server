FROM ubuntu:20.04

ARG DAEMON_URL=http://username:password@hostname:port/
ARG CERT_SUBJECT=/C=US/ST=Texas/L=Austin/emailAddress=smartcash@smartcash.cc

ENV DEBIAN_FRONTEND=noninteractive
ENV LC_ALL C.UTF-8

RUN apt-get update && \
    apt-get install -y python3-pip libleveldb-dev && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 install --upgrade pip setuptools wheel aiorpcX==0.10.1 attrs plyvel pylru aiohttp pycryptodomex leveldb

RUN addgroup electrumx && \
    useradd --home /home/electrumx --shell /bin/bash -g electrumx electrumx && \
    mkdir /home/electrumx && \
    chown electrumx:electrumx /home/electrumx

COPY . /home/electrumx/electrum-smart-server/
RUN chown -R electrumx:electrumx /home/electrumx/electrum-smart-server

USER electrumx
WORKDIR /home/electrumx

ENV COIN=SmartCash
ENV NET=Mainnet
ENV DB_DIRECTORY=/home/electrumx/.electrumx/db
ENV DB_ENGINE=leveldb
ENV USERNAME=electrumx
ENV HOST=0.0.0.0
ENV BANNER_FILE=/home/electrumx/.electrumx/banner.txt
ENV CACHE_MB=2048
ENV MAX_SESSIONS=1024
ENV SSL_CERTFILE=/home/electrumx/.electrumx/server.crt
ENV SSL_KEYFILE=/home/electrumx/.electrumx/server.key
ENV SSL_PORT=50002
ENV TCP_PORT=50001
ENV DAEMON_URL=$DAEMON_URL

EXPOSE 8000/tcp
EXPOSE 50001/tcp
EXPOSE 50002/tcp

RUN cd electrum-smart-server && python3 setup.py install --user

RUN mkdir -p ~/.electrumx/db && \
    cd ~/.electrumx && \
    openssl genrsa -des3 -passout pass:xyzf -out server.pass.key 2048 && \
    openssl rsa -passin pass:xyzf -in server.pass.key -out server.key && \
    rm server.pass.key && \
    openssl req -new -key server.key -out server.csr -subj $CERT_SUBJECT && \
    openssl x509 -req -days 1825 -in server.csr -signkey server.key -out server.crt

CMD /home/electrumx/.local/bin/electrumx_server
