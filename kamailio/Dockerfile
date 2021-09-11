#Kamailio Test Stuff
FROM fgst/kamailio:base


#Add Additional Modules
RUN apt-get update && apt-get -y install net-tools libhiredis-dev procps

#Copy the config file onto the Filesystem of the Docker instance
COPY kamailio.cfg /etc/kamailio/
COPY kamailio-base.cfg /etc/kamailio/
COPY kamailio-dispatcher.cfg /etc/kamailio/
COPY dispatcher.list /etc/kamailio/
COPY kamctlrc /etc/kamailio/
COPY store/etc/rtpproxy/rtpproxy.conf  /etc/default/rtpproxy
COPY store/etc/default/ngcp-rtpengine-daemon /etc/default/
COPY store/etc/default/ngcp-rtpengine-daemon /etc/default/
COPY store/etc/rtpengine/rtpengine.conf /etc/rtpengine/

#Print out the current IP Address info
RUN ip add

#Expose port 5060 (SIP) for TCP and UDP
EXPOSE 5060
EXPOSE 5060/udp
#Expose port for rtpproxy

