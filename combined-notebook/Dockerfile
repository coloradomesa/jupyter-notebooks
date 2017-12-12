FROM ubuntu:rolling
COPY . /tmp/src/
WORKDIR /tmp/src/
USER root
RUN apt-get update && apt-get install python python3 python3-pip cmake make wget curl bzip2 build-essential zlib1g-dev subversion texinfo git -y
RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt

# Install Cling for C++11, C++14, and C++17 Kernels
RUN mkdir -p cling && cd cling/ && \
    curl -O https://root.cern.ch/download/cling/cling_2017-11-13_ubuntu16.tar.bz2 && \
    tar xf cling* && \
    cd cling_2017-11-13_ubuntu16 && \
    cp -R * /usr/local/ && \
    cd /usr/local/share/cling/Jupyter/kernel/ && \
    pip3 install -e . && \
    jupyter-kernelspec install cling-cpp11 && \
    jupyter-kernelspec install cling-cpp14 && \
    jupyter-kernelspec install cling-cpp17

# Install Anaconda Python
RUN cd /tmp/src &&\
    wget https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh && \
    bash Anaconda3-5.0.1-Linux-x86_64.sh -b -p /usr/local/share/conda && \
    export PATH="$HOME/conda/bin:$PATH"

# Install scijava jupyter kernel
RUN conda config --add channels conda-forge && \
    conda create --name java_env scijava-jupyter-kernel \
    source activate java_env && \
    echo "List of installed jupyter kernels:" && \
    jupyter kernelspec list

RUN jupyter notebook --generate-config && \
    cat >> $HOME/.jupyter/jupyter_notebook_config.py << EOF \
import os\
password = os.environ.get('JUPYTER_NOTEBOOK_PASSWORD')\
if password:\
    import notebook.auth\
    c.NotebookApp.password = notebook.auth.passwd(password)\
    del password\
    del os.environ['JUPYTER_NOTEBOOK_PASSWORD']\
EOF


# Cleanup
RUN rm -rf /tmp/src && \
    apt-get remove --purge -y $BUILD_PACKAGES $(apt-mark showauto) && \
    rm -rf /var/lib/apt/lists/*

COPY ./builder /opt/app-root/builder
CMD ['exec', '/opt/app-root/builder/run']