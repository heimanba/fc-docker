FROM aliyunfc/fc-runtime:stretch-base

ENV LANG C.UTF-8
# Install python3.6
RUN apt-get update && apt-get install -y --no-install-recommends \
		libbluetooth-dev \
		tk-dev \
		libbz2-dev \
	&& rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y \
       gnupg \
       dirmngr \
	&& rm -rf /var/lib/apt/lists/*

ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
ENV PYTHON_VERSION 3.6.15

RUN set -ex \
	\
	&& wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$GPG_KEY" \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
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
		--enable-optimizations \
		--enable-option-checking=fatal \
		--enable-shared \
		--with-system-expat \
		--with-system-ffi \
		--without-ensurepip \
	&& make -j "$(nproc)" \
# setting PROFILE_TASK makes "--enable-optimizations" reasonable: https://bugs.python.org/issue36044 / https://github.com/docker-library/python/issues/160#issuecomment-509426916
		PROFILE_TASK='-m test.regrtest --pgo \
			test_array \
			test_base64 \
			test_binascii \
			test_binhex \
			test_binop \
			test_bytes \
			test_c_locale_coercion \
			test_class \
			test_cmath \
			test_codecs \
			test_compile \
			test_complex \
			test_csv \
			test_decimal \
			test_dict \
			test_float \
			test_fstring \
			test_hashlib \
			test_io \
			test_iter \
			test_json \
			test_long \
			test_math \
			test_memoryview \
			test_pickle \
			test_re \
			test_set \
			test_slice \
			test_struct \
			test_threading \
			test_time \
			test_traceback \
			test_unicode \
		' \
	&& make install \
	&& rm -rf /usr/src/python \
	\
	&& find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.a' \) \) \
			-o \( -type f -a -name 'wininst-*.exe' \) \
		\) -exec rm -rf '{}' + \
	\
	&& ldconfig \
	\
	&& python3 --version

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
	&& ln -s idle3 idle \
	&& ln -s pydoc3 pydoc \
	&& ln -s python3 python \
	&& ln -s python3-config python-config

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 21.2.4
# https://github.com/docker-library/python/issues/365
ENV PYTHON_SETUPTOOLS_VERSION 57.5.0
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://bootstrap.pypa.io/get-pip.py
ENV PYTHON_GET_PIP_SHA256 c518250e91a70d7b20cceb15272209a4ded2a0c263ae5776f129e0d9b5674309

RUN set -ex; \
	\
	wget -O get-pip.py "$PYTHON_GET_PIP_URL"; \
	echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict -; \
	\
	python get-pip.py \
		--disable-pip-version-check \
		--no-cache-dir \
		"pip==$PYTHON_PIP_VERSION" \
		"setuptools==$PYTHON_SETUPTOOLS_VERSION" \
	; \
	pip --version; \
	\
	find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +; \
	rm -f get-pip.py


COPY commons/pip.conf /etc/pip.conf

# Change work directory.
WORKDIR ${FC_FUNC_CODE_PATH}

# Install dev dependencies.
RUN pip install coverage debugpy -i https://mirrors.aliyun.com/pypi/simple/

# Install third party libraries for user function.
RUN pip install -U six
RUN pip install -U pip && pip install -U setuptools

# fc-docker fix: cprotobuf 0.1.10, dependency of pydatahub, could not be installed, so using 0.1.9
RUN pip install --user cython==0.29.14 cprotobuf==0.1.9 -i https://mirrors.aliyun.com/pypi/simple/
RUN pip install --user \
    scipy==1.0.0 \
    numpy==1.13.3 \
    wand==0.4.4 \
    opencv-python==3.4.0.14 \
    matplotlib==2.0.2 \
    scrapy==1.4.0 \
    cbor==1.0.0 \
    aliyun-fc==0.6 \
    meinheld==0.6.1 \
    aliyun-fc2==2.3.0 \
    tablestore==5.1.0 \
    aliyun-python-sdk-core-v3==2.13.11 \
    aliyun-python-sdk-iot==7.8.0 \
    aliyun-python-sdk-ecs==4.10.1 \
    aliyun-python-sdk-vpc==3.0.2 \
    aliyun-python-sdk-rds==2.1.4 \
    aliyun-python-sdk-kms==2.5.0 \
    aliyun-python-sdk-imm==1.3.4 \
    aliyun-log-python-sdk==0.6.38 \
    aliyun-python-sdk-ram==3.0.0 \
    aliyun-python-sdk-sts==3.0.0 \
    aliyun-python-sdk-fnf==1.3.0 \
    aliyun-python-sdk-dds==2.0.7 \
    aliyun-mns==1.1.5 \
    aliyun-python-sdk-cdn==3.0.8 \
    oss2==2.9.1 \
    pydatahub==2.11.2 \
    -i https://mirrors.aliyun.com/pypi/simple/

# iot sdk support 6.1.0
RUN curl --connect-timeout 60  -m 3600 -fsSL http://lambda-public.oss-cn-hangzhou.aliyuncs.com/lambda/aliyunsdkiot_py3_v20170420.tar.gz | tar -xzC /root/.local/lib/python3.6/site-packages/aliyunsdkiot/request/

# Server path.
ENV FC_SERVER_PATH=/var/fc/runtime/python3
ENV FC_SERVER_LOG_PATH=${FC_SERVER_PATH}/var/log
# Create directory.
RUN mkdir -p ${FC_SERVER_LOG_PATH}
ENV LD_LIBRARY_PATH=${FC_FUNC_CODE_PATH}:${FC_FUNC_CODE_PATH}/lib:/usr/local/lib

# Start a shell by default
CMD ["bash"]

