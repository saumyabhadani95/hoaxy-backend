FROM frolvlad/alpine-glibc:alpine-3.9

ENV CONDA_DIR="/opt/conda"
ENV PATH="$CONDA_DIR/bin:$PATH"

RUN apk add --no-cache --update \
                     alpine-sdk \
                     apache-ant \ 
                     bash \
                     ca-certificates \ 
                     curl \ 
                     g++ \ 
                     gcc \ 
                     git \
                     htop \ 
                     jq \ 
                     libffi-dev \
                     libssl1.1 \ 
                     libxml2-dev \
                     libxslt-dev \
                     musl-dev \ 
                     nano \
                     openssl \ 
                     openssl-dev \
                     openssl-dev \ 
                     postgresql-dev \
                     py-pip \
                     python-dev \
                     vim \
                     yaml-dev \ 
                     zlib-dev 
# Install conda2

RUN CONDA_VERSION="4.6.14" && \
    CONDA_MD5_CHECKSUM="718259965f234088d785cad1fbd7de03" && \
    \
    apk add --no-cache --virtual=.build-dependencies wget ca-certificates bash && \
    \
    mkdir -p "$CONDA_DIR" && \
    wget "http://repo.continuum.io/miniconda/Miniconda3-${CONDA_VERSION}-Linux-x86_64.sh" -O miniconda.sh && \
    echo "$CONDA_MD5_CHECKSUM  miniconda.sh" | md5sum -c && \
    bash miniconda.sh -f -b -p "$CONDA_DIR" && \
    echo "export PATH=$CONDA_DIR/bin:\$PATH" > /etc/profile.d/conda.sh && \
    rm miniconda.sh && \
    \
    conda update --all --yes && \
    conda config --set auto_update_conda False && \
    rm -r "$CONDA_DIR/pkgs/" && \
    \
    apk del --purge .build-dependencies && \
    \
    mkdir -p "$CONDA_DIR/locks" && \
    chmod 777 "$CONDA_DIR/locks"

ENV JAVA_HOME=/usr/lib/jvm/default-jvm
RUN apk add --no-cache openjdk8 && \
    ln -sf "${JAVA_HOME}/bin/"* "/usr/bin/"

ENV ANT_HOME /usr/share/java/apache-ant
ENV PATH $PATH:$ANT_HOME/bin

ENV ANT_OPTS="-Dhttp.proxyHost=proxy -Dhttp.proxyPort=8080"

# Install dependencies for hoaxy

RUN mkdir -p /root/conda
COPY requirements.txt /root/conda
WORKDIR /root/conda

RUN conda install --file requirements.txt

RUN conda install -y conda-forge::'tweepy>=3.5,<4'

WORKDIR /usr/src/pylucene

RUN mkdir -p /root/.ant/lib
RUN curl https://downloads.apache.org/ant/ivy/2.5.0/apache-ivy-2.5.0-bin.tar.gz | tar -xz --strip-components=1 \
    && cp ivy-2.5.0.jar /root/.ant/lib/

RUN curl https://downloads.apache.org/lucene/pylucene/pylucene-7.7.1-src.tar.gz | tar -xz --strip-components=1 \
    && cd jcc \
    && export JCC_ARGSEP=";" \
    && export JCC_CFLAGS="-v;-fno-strict-aliasing;-Wno-write-strings;-D__STDC_FORMAT_MACROS" \
    && JCC_JDK=/usr/lib/jvm/default-jvm python setup.py install \
    && cd .. \
        && make all install JCC='python -m jcc' ANT=ant PYTHON=python NUM_FILES=8 \
        && rm -rf /usr/src/pylucene

ENV HOAXY_VCS_REMOTE="https://github.com/saumyabhadani95/hoaxy-backend.git"

RUN git clone ${HOAXY_VCS_REMOTE} \
    && cd hoaxy-backend  \
    && python setup.py install

RUN mkdir /home/hoaxy
WORKDIR /home/hoaxy

ENV HOAXY_HOME=/home/hoaxy
EXPOSE 5432
CMD ["bash"]

