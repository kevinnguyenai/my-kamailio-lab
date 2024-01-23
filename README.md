# my-kamailio-lab

## STRUTURES

|-- asterisk                    # build and configuration default for asterisk
|-- config                      # config file for repository
|-- src                         # config file and scripts
    |-- conf
    |-- scripts
    |__ examples
|-- containers                  # resources for buildin containers
|-- kamailio                    # build and configuration default for kamailio
|-- kamailio-ci                 # submodule from kamailio official repository used to build alpine / edge images
|-- docker-asterisk             # submodule from mlan/docker-asterisk used to build asterisk imgages
|-- homer5-docker               # submodule from used to build homer servers images
|-- docker-opensips             # submodule from used to build opensips images
|-- mount                       # mount directory for mount volumes configuration of container
|-- .gitmodules                 # used for git to manage submodules
|-- .gitignore                  # used for git to manage non published content files
|-- docker-compose.yml          # docker-compose file to run stacked images
|__ README                      # how-to-build and deploy


## BUILD

## TODO

after clone repository to local machine do this

`$ git submodule init`

if you already clone and init submodule of projects , please update it with latest 

`$ git submodule update --remote`

### HOW TO DEPLOY LAB

* [PLAYBOOKS][https://github.com/kevinnguyenai/my-kamailio-lab/tree/main/playbooks]
    * [PLAYBOOK MEDIA PROXY (RTPENGINE)][https://github.com/kevinnguyenai/ansible-playbooks-voip/tree/3a2394c31ad3c8d03ee5967d50f5c166050e1730]
    * [PlAYBOOK PROXY][]
        * [KAMAILIO][]
        * [OPENSIPS][]
    * [PLAYBOOK PBX][]
        * [ASTERISK][]
        * [FREESWITCH][]
        * [FUSIONPBX][]
        * [FREEBSD]
    * [LOADTEST SERVER][]
        * [VOIP-PATROL]
    * [MONITORING & METRICS][]
        * [HOMER][]
        * [Heplify-server][]
