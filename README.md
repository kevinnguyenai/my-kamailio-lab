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

