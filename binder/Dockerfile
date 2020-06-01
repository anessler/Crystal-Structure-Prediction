FROM openjdk:11.0.7

# Version of FFX to Download
ENV FFX_VERSION 1.0.0-beta

# Version of TINKER to Download
ENV TINKER_VERSION 8.7.2

# Use GraalVM for JavaScript and Python
# FROM findepi/graalvm:20.0.0-java11-polyglot

RUN apt-get update
RUN apt-get install -y python3-pip

# add requirements.txt, written this way to gracefully ignore a missing file
COPY . .
RUN ([ -f requirements.txt ] \
    && pip3 install --no-cache-dir -r requirements.txt) \
        || pip3 install --no-cache-dir jupyter jupyterlab kotlin-jupyter-kernel

USER root

# Download Force Field X
ENV FFX_TAR ffx-$FFX_VERSION-bin.tar.gz
RUN wget https://ffx.biochem.uiowa.edu/$FFX_TAR

# Download/Install TINKER
ENV TINKER_TAR tinker-$TINKER_VERSION.tar.gz
ENV TINKER_BIN_TAR bin-linux.tar.gx
# Download Tinker
RUN wget https://dasher.wustl.edu/tinker/downloads/$TINKER_TAR
RUN wget https://dasher.wustl.edu/tinker/downloads/$TINKER_BIN_TAR

RUN tar xzf $TINKER_TAR
RUN tar xzf $TINKER_BIN_TAR

ENV TINKERDIR /tinker
ENV TINKER_BIN /bin-linux

# Setup Tinker Environment
RUN cd tinker/
RUN mkdir bin
RUN cd fftw/
RUN ./configure --prefix=/tinker/fftw/ --enable-openmp --enable-threads
RUN make -j 8
RUN make install
RUN cd ..
RUN cp ./make/Makefile ./source
RUN cd source

# Edit Makefile to comment out default operating system in favor of linux
RUN sed -i 's/#F77 = gfortran/F77 = gfortran/g' Makefile
RUN sed -i 's/#F77FLAGS = -c/F77FLAGS = -c/g' Makefile
RUN sed -i 's/#OPTFLAGS = -Ofast -mssse3 -fopenmp/OPTFLAGS = -Ofast -mssse3 -fopenmp/g' Makefile
RUN sed -i 's/#LIBS =/LIBS =/g' Makefile
RUN sed -i 's/#LIBFLAGS = -crusv/LIBFLAGS = -crusv/g' Makefile
RUN sed -i 's/#RANLIB = ranlib -c/RANLIB = ranlib -c/g' Makefile
RUN sed -i 's/#LINKFLAGS = $(OPTFLAGS) -static-libgcc/LINKFLAGS = $(OPTFLAGS) -static-libgcc/g' Makefile

RUN cp ../linux/gfortran/*.make .
RUN cp ../fftw
RUN make -j 8
RUN make install

# Setup GDMA2
RUN cd $HOME
#TODO: ensure stable version
RUN git clone https://gitlab.com/anthonyjs/gdma.git
RUN cd gdma
RUN make
ENV GDMADIR /gdma/bin

# Setup POLTYPE2
RUN cd $HOME
RUN wget https://github.com/pren/poltype/archive/poltype2.zip

RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
RUN bash Miniconda3-latest-Linux-x86_64.sh

RUN conda create -n p4env -c psi4/label/dev psi4 python=3.6
RUN conda activate p4env
RUN conda install scipy
RUN conda install matplotlib
RUN conda install -c openbabel openbabel=2.4.1
RUN conda install git pip
RUN conda install -c rdkit rdkit
RUN conda install -c conda-forge mdanalysis
RUN conda install -c conda-forge svgutils
RUN conda install -c conda-forge cairosvg
RUN pip install git+git://github.com/ParmEd/ParmEd.git

ENV PATH="/opt/software/miniconda3/bin:/opt/gdma-2.2/bin:/tinker/source/:${PATH}"

# Unpack Force Field X
RUN tar xzf $FFX_TAR
RUN mv ffx-$FFX_VERSION ffx

# Set FFX_HOME and the CLASSPATH
ENV FFX_HOME /ffx
ENV FFX_BIN $FFX_HOME/bin
ENV CLASSPATH $FFX_BIN/ffx-all-$FFX_VERSION.jar

# Download the IJava kernel
RUN wget https://github.com/SpencerPark/IJava/releases/download/v1.3.0/ijava-1.3.0.zip

# Set IJava kernel environment variables.
ENV IJAVA_CLASSPATH $CLASSPATH
ENV IJAVA_STARTUP_SCRIPTS_PATH $FFX_BIN/startup.jshell

# Unpack and install the IJava kernel
RUN unzip ijava-1.3.0.zip -d ijava-kernel \
  && cd ijava-kernel \
  && python3 install.py --sys-prefix --startup-scripts-path $IJAVA_STARTUP_SCRIPTS_PATH --classpath $CLASSPATH

# Set up the user environment
ENV NB_USER ffx
ENV SHELL /bin/bash
ENV NB_UID 1000
ENV HOME /home/$NB_USER

RUN adduser --disabled-password \
    --gecos "Default user" \
    --uid $NB_UID \
    $NB_USER

COPY . $HOME
RUN chown -R $NB_UID $HOME

# Set up the FFX Kotlin library
RUN mkdir $HOME/.jupyter_kotlin
RUN mkdir $HOME/.jupyter_kotlin/libraries
RUN cp binder/ffx.json $HOME/.jupyter_kotlin/libraries/.

USER $NB_USER

# Launch the notebook server
WORKDIR $HOME
# CMD ["jupyter", "notebook", "--ip", "0.0.0.0"]
