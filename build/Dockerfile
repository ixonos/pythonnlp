
FROM ubuntu:16.04

RUN apt-get update && apt-get -y upgrade && apt-get install -y locales \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \ 
    && apt-get install -y sudo less git vim wget man zip curl xz-utils gcc clang software-properties-common \ 
    && apt-get install -y build-essential python-dev python3-pip python3-tk

ENV LANG en_US.utf8

RUN python3 -m pip install -U pip  

# Wit NLP client
RUN pip3 install wit

# Word cloud with semantic info
RUN pip3 install wordmesh

# Spacy model for English
RUN python3 -m spacy download en_core_web_md

# Downgrade plotly to make wordmesh work
# see https://github.com/mukund109/word-mesh/issues/3
RUN pip3 install plotly==2.7.0

# Gensim
RUN pip3 install gensim

# Enable locating files
RUN apt-get install locate
RUN /etc/cron.daily/locate

RUN sed -i'' 's/\(%sudo.*\)ALL$/\1 NOPASSWD: ALL/' /etc/sudoers

RUN useradd -m nlp -G sudo
RUN mkdir /nlp
RUN chown nlp:nlp /nlp

USER nlp
WORKDIR /nlp

# FinnPos morphological tagger using omorfi and CRF
RUN git clone https://github.com/mpsilfve/FinnPos

WORKDIR FinnPos
RUN (cd share/finnpos/omorfi/ && curl -LO https://github.com/mpsilfve/FinnPos/releases/download/v0.1-alpha/morphology.omor.hfst.gz && gunzip morphology.omor.hfst.gz)
RUN make && make ftb-omorfi-tagger 
RUN sudo make install && sudo make install-models

# Voikko development and dictionary, other HFST tools
RUN sudo apt-get -y install hfst libhfst42-dev python3-libhfst python3-libvoikko voikko-fi libvoikko-dev foma-bin
RUN sudo pip3 install hfst

# Turku NLP using neural parser
WORKDIR /nlp
RUN git clone https://github.com/TurkuNLP/Turku-neural-parser-pipeline.git
WORKDIR Turku-neural-parser-pipeline
RUN git submodule update --init --recursive

# Unpack the Turku NLP requirements here due to non-GPU torch
RUN sudo pip3 install tensorflow==1.5.0
RUN sudo pip3 install wheel keras h5py matplotlib flask
RUN sudo pip3 install https://download.pytorch.org/whl/cpu/torch-1.0.0-cp35-cp35m-linux_x86_64.whl
RUN sudo pip3 install torchvision torchtext
RUN sudo pip3 install ufal.udpipe configargparse

# RUN sudo pip3 install -r requirements-cpu.txt 
RUN python3 fetch_models.py fi_tdt


# Fasttext word embeddings - download them outside the image
# Here are the command lines for including them in the image itself
# 
# RUN mkdir fasttext
# WORKDIR fasttext
# RUN curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.fi.vec
# RUN curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.se.vec
# RUN curl -O https://s3-us-west-1.amazonaws.com/fasttext-vectors/wiki.en.vec

RUN sudo /etc/cron.daily/locate
WORKDIR /nlp

