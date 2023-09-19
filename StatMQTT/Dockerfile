FROM public.ecr.aws/docker/library/python:3.11-alpine AS builder

RUN apk add build-base linux-headers

WORKDIR /build
COPY ./requirements.txt .
RUN python -m pip install --upgrade pip
RUN pip install --target=/build/deps -r requirements.txt
RUN rm -r requirements.txt

# Cleanup files we dont want to bring over
WORKDIR /build/deps
RUN rm -rf \
    __pycache__ \
    src/__pycache__ \
    pip \
    pip* \
    src/*_test.py \
    psutil/tests

FROM public.ecr.aws/docker/library/python:3.11-alpine
RUN apk add bash

WORKDIR /opt/StatMQTT
COPY --from=builder /build .
COPY *.py ./
COPY StatMQTT.service .
COPY logging.conf .

# add user StatMQTT to image
RUN addgroup -S StatMQTT && adduser -S StatMQTT -G StatMQTT
RUN chown -R StatMQTT:StatMQTT /opt/StatMQTT

# process run as StatMQTT user
USER StatMQTT

# set conf path
ENV StatMQTTCONFIG="/opt/StatMQTT/conf/StatMQTT.conf"

# add deps to PYTHONPATH
ENV PYTHONPATH="${PYTHONPATH}:/opt/StatMQTT/deps"

# run process
CMD python StatMQTT.py
