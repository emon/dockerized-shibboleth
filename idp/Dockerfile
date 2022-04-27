FROM alpine:latest

LABEL maintainer="Komura Takaaki"

# specified in docker-compose.yml
#  or docker-compose build --build-arg

ARG JETTY_KEYSTORE_PASSWORD
ARG IDP_INSTALL_DSTDIR
ARG IDP_INSTALL_SCOPE
ARG IDP_INSTALL_HOST_NAME
ARG IDP_INSTALL_SEALER_PASSWORD
ARG IDP_INSTALL_KEYSTORE_PASSWORD
ARG IDP_INSTALL_ENTITY_ID
ARG IDP_VERSION

ENV JETTY_KEYSTORE_PASSWORD=${JETTY_KEYSTORE_PASSWORD}
ENV IDP_INSTALL_DSTDIR=${IDP_INSTALL_DSTDIR}
ENV IDP_INSTALL_SCOPE=${IDP_INSTALL_SCOPE}
ENV IDP_INSTALL_HOST_NAME=${IDP_INSTALL_HOST_NAME}
ENV IDP_INSTALL_SEALER_PASSWORD=${IDP_INSTALL_SEALER_PASSWORD}
ENV IDP_INSTALL_KEYSTORE_PASSWORD=${IDP_INSTALL_KEYSTORE_PASSWORD}
ENV IDP_INSTALL_ENTITY_ID=${IDP_INSTALL_ENTITY_ID}


# install version and hash
ENV JETTY_VERSION=9.4.45.v20220203
ENV JETTY_SHA1=97fb0b0456d21ba18df4b89707157e02ebbc1a60
ENV IDP_VERSION=4.1.7
ENV IDP_SHA256=115a5e9838e873193af0743965a0e59cc84fe1eea184a62136cf6cb5384a5418

ENV IDP_JETTY_BASE_VERSION=9.4.1
ENV IDP_JETTY_BASE_SHA1=c7661b28976cd0bbede31faae54107b7305e8804

ENV IDP_JETTY_BASE_URL=https://build.shibboleth.net/maven/snapshots/net/shibboleth/idp/idp-jetty-base/9.4.1-SNAPSHOT/idp-jetty-base-9.4.1-20211028.163025-766.tar.gz
# from https://build.shibboleth.net/nexus/service/local/repositories/releases/content/net/shibboleth/idp/idp-jetty-base/9.4.0/idp-jetty-base-9.4.0.tar.gz.sha1
ENV JETTY_BASE_DIFF=jetty-base-9.4.0.diff

# environment
ENV JAVA_HOME=/usr/lib/jvm/default-jvm
#ENV JAVA=/usr/lib/jvm/jre/bin/java
ENV JETTY_HOME=/opt/jetty
ENV JETTY_BASE=/opt/jetty-base
ENV JETTY_RUN=/opt/jetty-base/tmp
ENV JETTY_STATE=/opt/jetty-base/tmp/jetty.state

ENV JETTY_KEYSTORE_PATH=etc/keystore

ENV IDP_INSTALL_TARGET_DIR=${IDP_INSTALL_DSTDIR}/shibboleth-idp
ENV IDP_INSTALL_SRC_DIR=$IDP_INSTALL_DSTDIR/shibboleth-identity-provider-${IDP_VERSION}

ENV JETTY_JAVA_ARGS="jetty.home=$JETTY_HOME \
    jetty.base=$JETTY_BASE \
    -Djetty.sslContext.keyStorePassword=$JETTY_KEYSTORE_PASSWORD \
    -Djetty.sslContext.keyStorePath=$JETTY_KEYSTORE_PATH"

ENV IDP_TGZ_NAME=shibboleth-identity-provider-${IDP_VERSION}.tar.gz
ENV IDP_DOWNLOAD_URL=https://shibboleth.net/downloads/identity-provider/archive/${IDP_VERSION}/${IDP_TGZ_NAME}

COPY jetty-base-9.4.0.diff /root/

#
# install openjdk11 and jetty
#
RUN set -xe ;\
    apk --no-cache add openjdk11-jre-headless curl tar bash patch ;\
#
    curl -sLO https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-distribution/$JETTY_VERSION/jetty-distribution-$JETTY_VERSION.tar.gz ;\
    echo "$JETTY_SHA1  jetty-distribution-$JETTY_VERSION.tar.gz" | sha1sum -c - ;\
    tar zxv -C /opt -f jetty-distribution-$JETTY_VERSION.tar.gz ;\
    ln -s /opt/jetty-distribution-$JETTY_VERSION/ $JETTY_HOME ;\
    rm jetty-distribution-$JETTY_VERSION.tar.gz

#
# install jetty-base for Shibboleth-IdP
#
RUN set -xe ;\
    curl -sLO $IDP_JETTY_BASE_URL ;\
    IDP_JETTY_BASE_FILENAME=`basename $IDP_JETTY_BASE_URL` ;\
    echo "$IDP_JETTY_BASE_SHA1  $IDP_JETTY_BASE_FILENAME" | sha1sum -c - ;\
    tar zxv -C /opt -f $IDP_JETTY_BASE_FILENAME;\
    addgroup -g 110 jetty ;\
    adduser  -u 110 -G jetty -h /opt/jetty-base -s /sbin/nologin -g "Jetty daemon" -D jetty ;\
    cd "${JETTY_BASE}" ; patch -p1 < /root/jetty-base-9.4.0.diff ;\
    #
    chown -R jetty:jetty ${JETTY_BASE} ;\
    chown -R root:root /opt/jetty-distribution-9.4.* $JETTY_BASE ;\
    chmod -R g-w /opt/jetty-base ;\
    chown jetty:jetty $JETTY_BASE/logs $JETTY_BASE/tmp
#
# install Shibboleth IdP
#
RUN set -xe ;\
    curl -sLO ${IDP_DOWNLOAD_URL} ;\
    echo "$IDP_SHA256  shibboleth-identity-provider-$IDP_VERSION.tar.gz" | sha256sum -c - ;\
    tar -zxv -C /opt -f shibboleth-identity-provider-$IDP_VERSION.tar.gz ;\
    bash /opt/shibboleth-identity-provider-$IDP_VERSION/bin/install.sh \
      -Didp.conf.credentials.filemode=640 \
      -Didp.conf.credentials.group=jetty \
      -Didp.scope=$IDP_INSTALL_SCOPE \
      -Didp.target.dir=$IDP_INSTALL_TARGET_DIR \
      -Didp.src.dir=/opt/shibboleth-identity-provider-$IDP_VERSION \
      -Didp.host.name=$IDP_INSTALL_HOST_NAME \
      -Didp.noprompt=true \
      -Didp.sealer.password=$IDP_INSTALL_SEALER_PASSWORD \
      -Didp.keystore.password=$IDP_INSTALL_KEYSTORE_PASSWORD \
      -Didp.entityID=$IDP_INSTALL_ENTITY_ID \
      ;\
    rm shibboleth-identity-provider-${IDP_VERSION}.tar.gz ;\
    rm -rf /tmp/shibboleth-identity-provider-${IDP_VERSION} ;\
    #
    cd $IDP_INSTALL_TARGET_DIR ;\
    chown -R jetty:jetty logs ;\
    chgrp jetty metadata ;\
    chmod g+w   metadata ;\
    chmod +t    metadata ;\
    #
    # duplicate config files of IdP
    #
    cp -pr $IDP_INSTALL_TARGET_DIR/conf $IDP_INSTALL_TARGET_DIR/conf.original ;\

    #
    # Cleanup
    #
    #
#    apt-mark auto '.*' > /dev/null ;\
#    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null ;\
#    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false ;\
#    rm -rf /var/lib/apt/lists/*
    echo "DONE"

WORKDIR $JETTY_BASE
USER jetty
EXPOSE 8080 8443

# Jettyを起動する。
CMD $JAVA_HOME/bin/java -jar $JETTY_HOME/start.jar $JETTY_JAVA_ARGS
