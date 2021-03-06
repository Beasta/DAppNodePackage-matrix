# Dockerfile to build the matrixdotorg/synapse docker images.
#
# To build the image, run `docker build` command from the root of the
# synapse repository:
#
#    docker build -f docker/Dockerfile .
#
# There is an optional PYTHON_VERSION build argument which sets the
# version of python to build against: for example:
#
#    docker build -f docker/Dockerfile --build-arg PYTHON_VERSION=3.6 .
#

ARG PYTHON_VERSION=3
ARG SYNAPSE_TAG=v0.99.0

###
### Stage 0: builder
###
FROM docker.io/python:${PYTHON_VERSION}-alpine3.8 as builder

# install the OS build deps

RUN apk add \
        git \
        build-base \
        libffi-dev \
        libjpeg-turbo-dev \
        libressl-dev \
        libxslt-dev \
        linux-headers \
        postgresql-dev \
        zlib-dev

# build things which have slow build steps, before we copy synapse, so that
# the layer can be cached.
#
# (we really just care about caching a wheel here, as the "pip install" below
# will install them again.)

RUN pip install --prefix="/install" --no-warn-script-location \
        cryptography \
        msgpack-python \
        pillow \
        pynacl

# now install synapse and all of the python deps to /install.

RUN git clone https://github.com/matrix-org/synapse && \
        cd synapse && \
        git checkout ${SYNAPSE_TAG}

#RUN ls && ls synapse
#COPY synapse/synapse /synapse/synapse/
#COPY synapse/scripts /synapse/scripts/
#COPY MANIFEST.in README.rst setup.py synctl /synapse/

RUN pip install --prefix="/install" --no-warn-script-location \
        /synapse[all]

###
### Stage 1: runtime
###

FROM docker.io/python:${PYTHON_VERSION}-alpine3.8

RUN apk add --no-cache --virtual .runtime_deps \
        libffi \
        libjpeg-turbo \
        libressl \
        libxslt \
        libpq \
        zlib \
        su-exec

COPY --from=builder /install /usr/local
COPY ./start.py /start.py
COPY ./conf /conf

VOLUME ["/data"]

EXPOSE 8008/tcp 8009/tcp 8448/tcp

ENTRYPOINT ["/start.py"]
