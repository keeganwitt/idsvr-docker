#
# Copyright (C) 2020 Curity AB. All rights reserved.
#
# The contents of this file are the property of Curity AB.
# You may not copy or use this file, in either source code
# or executable form, except in compliance with terms
# set by Curity AB.
#
# For further information, please contact Curity AB.
#

FROM quay.io/centos/centos:stream8 AS openssl-builder

RUN yum clean all && \
	yum install -y perl-core curl && \
	yum groupinstall -y "Development tools"

ENV OPENSSL_VERSION=1.1.1q

RUN cd /tmp && curl https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz --output openssl.tar.gz && \
    curl https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz.sha256 --output openssl.tar.gz.sha256 && \
    echo $(cat openssl.tar.gz.sha256) openssl.tar.gz | sha256sum -c && \
    tar -xf openssl.tar.gz && \
    rm openssl.tar.gz

RUN cd /tmp/openssl-${OPENSSL_VERSION} && ./config --prefix=/build --release \
    no-ec2m no-idea no-mdc2 no-rc5 no-ssl no-dtls no-dtls1-method no-dtls1_2-method && \
    make && \
    chmod -R go+rX . && \
    make TESTS=-test_afalg test && \
    make install_sw install_ssldirs

FROM quay.io/centos/centos:stream8

LABEL maintainer="Curity AB <info@curity.se>"

ARG TARGETARCH

EXPOSE 8443
EXPOSE 6749
EXPOSE 4465
EXPOSE 4466

RUN yum update -y && yum clean all

RUN useradd --system idsvr

ENV IDSVR_HOME /opt/idsvr
ENV JAVA_HOME $IDSVR_HOME/lib/java/jre
ENV PATH $IDSVR_HOME/bin:$JAVA_HOME/bin:$PATH
WORKDIR $IDSVR_HOME

COPY idsvr-7.0.3-${TARGETARCH}/idsvr /opt/idsvr
COPY first-run /opt/idsvr/etc/first-run
COPY --from=openssl-builder /build/bin/openssl /bin/

RUN find /opt/idsvr -type f -exec chmod a-w {} \; && \
    chmod -R o-rwx /opt/idsvr && \
    chown -R idsvr: /opt/idsvr

USER idsvr:idsvr

CMD ["idsvr"]