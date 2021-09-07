#Kamailio Test Stuff
FROM kamailio/kamailio:5.5.0-stretch


#Add Additional Modules

#Copy the config file onto the Filesystem of the Docker instance
COPY kamailio.cfg /etc/kamailio/
COPY dispatcher.list /etc/kamailio/
COPY kamctlrc /etc/kamailio/

#Print out the current IP Address info
RUN ip add

# append /etc/hosts
RUN echo "172.19.0.4    sip1.example.com" | tee -a "/etc/hosts"
RUN echo "172.19.0.5    sip2.example.com" | tee -a "/etc/hosts"
RUN echo "172.19.0.3    sip3.example.com" | tee -a "/etc/hosts"

#Expose port 5060 (SIP) for TCP and UDP
EXPOSE 5060
EXPOSE 5060/udp
