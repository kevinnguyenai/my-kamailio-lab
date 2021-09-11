#Kamailio Test Stuff
FROM kamailio/kamailio:5.5.0-stretch

# install additional package
RUN apt-get update 
RUN apt-get install -y curl ntpdate lsb-release locales htop \
                    mc git gcc flex build-essential libglib2.0-dev \
                    make libssl-dev libcurl4-openssl-dev make \
                    libxml2-dev libpcre3-dev libexpat1-dev \
                    mysql-client libunistring-dev apt-utils \
		    libxmlrpc-core-c3-dev default-libmysqlclient-dev \
		    libpcap-dev libpulse-dev libswresample-dev \
		    libavformat-dev libspandsp-dev libevent-dev \
		    libjson-glib-dev libavfilter-dev libiptc-dev \
		    gperf libwebsockets-dev


WORKDIR /tmp

## RTPProxy

RUN apt-get install -y make git && \
    git clone -b master https://github.com/sippy/rtpproxy.git && \
    git -C rtpproxy submodule update --init --recursive && \
    cd rtpproxy && \
    ./configure && make && make install


COPY rtpproxy /etc/init.d/

RUN chmod 755 /etc/init.d/rtpproxy

## RTPengine 

RUN cd /tmp && \
    git clone git://github.com/redis/hiredis.git && \
    cd hiredis && \
    make && make install

RUN cd /tmp && \
    git clone https://github.com/sipwise/rtpengine.git && \
    cd rtpengine/daemon && \
    make && \
    cp -u rtpengine /usr/bin/
