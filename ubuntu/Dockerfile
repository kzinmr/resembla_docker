## FROM docker-brew-ubuntu-core/xenial/Dockerfile
FROM ubuntu:16.04

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap
RUN set -xe \
  \
  && echo '#!/bin/sh' > /usr/sbin/policy-rc.d \
  && echo 'exit 101' >> /usr/sbin/policy-rc.d \
  && chmod +x /usr/sbin/policy-rc.d \
  \
  && dpkg-divert --local --rename --add /sbin/initctl \
  && cp -a /usr/sbin/policy-rc.d /sbin/initctl \
  && sed -i 's/^exit.*/exit 0/' /sbin/initctl \
  \
  && echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \
  \
  && echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean \
  && echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean \
  && echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean \
  \
  && echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages \
  \
  && echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes \
  \
  && echo 'Apt::AutoRemove::SuggestsImportant "false";' > /etc/apt/apt.conf.d/docker-autoremove-suggests

# delete all the apt list files since they're big and get stale quickly
RUN rm -rf /var/lib/apt/lists/*
# this forces "apt-get update" in dependent images, which is also good

# enable the universe
RUN sed -i 's/^#\s*\(deb.*universe\)$/\1/g' /etc/apt/sources.list

# make systemd-detect-virt return "docker"
# See: https://github.com/systemd/systemd/blob/aa0c34279ee40bce2f9681b496922dedbadfca19/src/basic/virt.c#L434
RUN mkdir -p /run/systemd && echo 'docker' > /run/systemd/container


RUN set -ex \
  && buildDeps=' \
    autoconf \
    build-essential \
    ca-certificates \
    gcc \
	g++ \
    make \
    pkg-config \
    wget \
    bzip2 \
    libbz2-dev \
    libpq-dev \
    apt-transport-https \
    curl \
    jq \
    git \
	xz-utils \
	file \
	sudo \
  ' \
  && apt-get update && apt-get install -y $buildDeps --no-install-recommends


## FROM docker-library/python/3.5/Dockerfile
#FROM buildpack-deps:stretch

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
		tcl \
		tk \
	&& rm -rf /var/lib/apt/lists/*

ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
ENV PYTHON_VERSION 3.6.3

RUN set -ex \
	&& buildDeps=' \
		dpkg-dev \
		tcl-dev \
		tk-dev \
	' \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends \
	\
	&& wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm python.tar.xz \
	\
	&& cd /usr/src/python \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-loadable-sqlite-extensions \
		--enable-shared \
		--with-system-expat \
		--with-system-ffi \
		--without-ensurepip \
	&& make -j "$(nproc)" \
	&& make install \
	&& ldconfig \
	\
	&& apt-get purge -y --auto-remove $buildDeps \
	\
	&& find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' + \
	&& rm -rf /usr/src/python

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
	&& ln -s idle3 idle \
	&& ln -s pydoc3 pydoc \
	&& ln -s python3 python \
	&& ln -s python3-config python-config

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 9.0.1

RUN set -ex; \
	\
	wget -O get-pip.py 'https://bootstrap.pypa.io/get-pip.py'; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PYTHON_PIP_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py


## Setup MeCab
ENV MECAB_VERSION 0.996
ENV IPADIC_VERSION 2.7.0-20070801
ENV mecab_url https://drive.google.com/uc?export=download&id=0B4y35FiV1wh7cENtOXlicTFaRUE
ENV ipadic_url https://drive.google.com/uc?export=download&id=0B4y35FiV1wh7MWVlSDBCSXZMTXM
RUN set -ex \
  && buildDeps=' \
    g++ \
    xz-utils \
    file \
  ' \
  && apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
  \
  && wget -O mecab-${MECAB_VERSION}.tar.gz ${mecab_url} \
  && tar zxf mecab-${MECAB_VERSION}.tar.gz \
  && cd mecab-${MECAB_VERSION} \
  && ./configure --enable-utf8-only --with-charset=utf8 \
  && make \
  && make install \
  && cd \
  && ldconfig \
  # Install IPA dict
  && wget -O mecab-ipadic-${IPADIC_VERSION}.tar.gz ${ipadic_url} \
  && tar zxf mecab-ipadic-${IPADIC_VERSION}.tar.gz \
  && cd mecab-ipadic-${IPADIC_VERSION} \
  && ./configure --with-charset=utf8 \
  && make \
  && make install \
  && cd \
  # Install Neologd
  && git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git \
  && mecab-ipadic-neologd/bin/install-mecab-ipadic-neologd -n -y \
  && cd mecab-ipadic-neologd \
  && neologd_date=$(git log --pretty=format:'{%n  "commit": "%H",%n  "author": "%aN <%aE>",%n  "date": "%ad",%n  "message": "%f"%n},' $@ | perl -pe 'BEGIN{print "["}; END{print "]\n"}' | perl -pe 's/},]/}]/'|jq '.[]'|jq .date|sed 's/"[A-Za-z]* \([A-Za-z]*\) \([0-9]*\) [^ ]* \([0-9]*\) .*/\1-\2-\3/'|xargs date "+%Y%m%d" -d) \
  && echo "NEOLOGD_VERSION=${neologd_date}"  > /NEOLOGD_VERSION \
  && cd .. \
  # Clean up
  && rm -rf \
    mecab-${MECAB_VERSION}* \
    mecab-ipadic* \
    mecab-ipadic-neologd \
  && sed -i -e"s/^dicdir =  \/usr\/local\/lib\/mecab\/dic\/ipadic/dicdir =  \/usr\/local\/lib\/mecab\/dic\/mecab-ipadic-neologd/" /usr/local/etc/mecabrc \
  && pip install --no-cache mecab-python3 neologdn \
  && cd /


RUN set -ex; \
	wget -O icu4c.tgz 'http://download.icu-project.org/files/icu4c/60.2/icu4c-60_2-src.tgz'; \
	tar zxf icu4c.tgz; \
	cd icu/source; \
	./runConfigureICU Linux; \
	make; \
	make install; \
	cd /

RUN git clone https://github.com/cjlin1/libsvm.git; \
	cd libsvm; \
	make; \
	cd python; \
	make; \
	mkdir -p /usr/local/bin/libsvm/tools; \
	mv /libsvm/svm-predict /libsvm/svm-scale /libsvm/svm-train /usr/local/bin/libsvm; \
	mv /libsvm/tools/*py /usr/local/bin/libsvm/tools/; \
	mv /libsvm/libsvm.so.2 /usr/local/lib/; \
	cd /usr/local/lib; \
	ln -s  libsvm.so.2  libsvm.so; \
	export PATH=$PATH:/usr/local/bin/libsvm; \
	mkdir /usr/local/include/libsvm/; \
	mv /libsvm/svm.h /usr/local/include/libsvm/; \
	ldconfig; \
	cd /

RUN apt-get update && apt-get install -y build-essential autoconf automake libtool curl autotools-dev unzip pkg-config --no-install-recommends; \
	# git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc; \
	git clone -b v1.2.5 https://github.com/grpc/grpc; \
	cd grpc; \
	git submodule update --init; \
	cd third_party/protobuf/; \
	./autogen.sh; \
	./configure; \
	make; \
	make install; \
	cd ../../; \
	ldconfig; \
	make; \
	make install; \
	pip install --upgrade pip; \
	pip install grpcio; \
	pip install grpcio-tools; \
	pip install protobuf

RUN git clone https://github.com/tuem/resembla.git; \
	cd /resembla/src; \
	make; \
	make install; \
	#cd /resembla/misc/mecab_dic/mecab-unidic-neologd/; \
	#./install-mecab-unidic-neologd.sh; \
	cd /resembla/src/executable; \
	make; \
	make install; \
	ldconfig

RUN cd /resembla/example/grpc/src; \
	make

CMD ["/bin/bash"]
