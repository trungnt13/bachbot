############################################################
# Dockerfile for the BachBot project
# Based on Ubuntu
#
# Building, pushing, and running:
#   docker build -f Dockerfile -t bachbot:base .
#   docker tag <tag of last container> fliang/bachbot:base
#   docker push fliang/bachbot:base
#   docker run -i -t fliang/bachbot:base
############################################################

FROM ubuntu:14.04
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
MAINTAINER Feynman Liang "feynman.liang@gmail.com"

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

# Required packages
RUN apt-get update
RUN apt-get -y install \
    build-essential \
    python2.7-dev \
    git \
    ssh \
    libhdf5-dev \
    libssl-dev \
    libxml2-dev \
    libxslt-dev \
    software-properties-common

# Torch and luarocks
RUN git clone https://github.com/torch/distro.git ~/torch --recursive && cd ~/torch && \
    bash install-deps && \
    ./install.sh -b

ENV LUA_PATH='~/.luarocks/share/lua/5.1/?.lua;~/.luarocks/share/lua/5.1/?/init.lua;~/torch/install/share/lua/5.1/?.lua;~/torch/install/share/lua/5.1/?/init.lua;./?.lua;~/torch/install/share/luajit-2.1.0-beta1/?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua'
ENV LUA_CPATH='~/.luarocks/lib/lua/5.1/?.so;~/torch/install/lib/lua/5.1/?.so;./?.so;/usr/local/lib/lua/5.1/?.so;/usr/local/lib/lua/5.1/loadall.so'
ENV PATH=~/torch/install/bin:$PATH
ENV LD_LIBRARY_PATH=~/torch/install/lib:$LD_LIBRARY_PATH
ENV DYLD_LIBRARY_PATH=~/torch/install/lib:$DYLD_LIBRARY_PATH
ENV LUA_CPATH='~/torch/install/lib/?.so;'$LUA_CPATH

#BachBot
RUN git clone https://github.com/feynmanliang/bachbot.git ~/bachbot

#pyenv
RUN curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | /bin/bash

# Set environment variables.
ENV PYENV_ROOT /root/.pyenv
ENV PATH $PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH
RUN echo -e '\n\n# pyenv' >> ~/.bash_profile && \
    echo 'export PYENV_ROOT="/root/.pyenv"' >> ~/.bash_profile && \
    echo 'export PATH="$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile && \
    echo 'eval "$(pyenv init -)"' >> ~/.bash_profile

RUN pyenv install 2.7.11 && \
    pyenv global 2.7.11 && \
    pyenv rehash

#BachBot virtualenv
RUN bash -c "pyenv virtualenv 2.7.11 bachbot && \
    source ~/.bash_profile && \
    pyenv activate bachbot && \
    cd ~/bachbot && \
    pip install -r requirements.txt"

#Lua requirements
RUN luarocks install nn
RUN luarocks install optim
RUN luarocks install lua-cjson

RUN git clone https://github.com/deepmind/torch-hdf5 ~/torch-hdf5 && \
    cd ~/torch-hdf5 && \
    luarocks make hdf5-0-0.rockspec

#torch-rnn and python requirements installed to bachbot venv
RUN luarocks install lua-utf8
RUN git clone https://github.com/feynmanliang/torch-rnn ~/torch-rnn
RUN bash -c "source ~/.bash_profile && \
    pyenv activate bachbot && \
    pip install -r ~/torch-rnn/requirements.txt"

#Element-Research/rnn
RUN luarocks install rnn

# Clean tmps
RUN apt-get clean && \
    rm -rf \
	/var/lib/apt/lists/* \
	/tmp/* \
	/var/tmp/* \
	~/torch-hdf5

##################### INSTALLATION END #####################
WORKDIR ~/
CMD ["bash"]
