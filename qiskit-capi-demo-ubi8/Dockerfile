FROM registry.access.redhat.com/ubi8/python-311

USER root
RUN (dnf upgrade --disableplugin=subscription-manager -y || dnf upgrade --disableplugin=subscription-manager -y --nobest) && \
    dnf install -q -y --nodocs \
       wget \
       gcc \
       make \
       openssl-devel \
       cmake \
       clang \
       clang-devel \
       scl-utils \
       ca-certificates && \
    dnf clean all

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh && \
    bash Miniconda3-latest-Linux-aarch64.sh -b -p ~/miniconda3 && \
    rm -r Miniconda3-latest-Linux-aarch64.sh

ENV PATH=$HOME/miniconda3/bin:$PATH

COPY myenv.yaml /

RUN conda tos accept && \
    conda update -n base -c defaults conda && \
    conda env create -n sandbox -f /myenv.yaml && \
    conda init && \
    echo "conda activate sandbox" >> $HOME/.bashrc

RUN echo ". $HOME/.cargo/env" >> $HOME/.bashrc

ENV CONDA_DEFAULT_ENV=sandbox
ENV PATH=$HOME/miniconda3/envs/sandbox/bin:$PATH
