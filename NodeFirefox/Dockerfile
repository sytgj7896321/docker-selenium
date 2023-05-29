ARG NAMESPACE
ARG VERSION
ARG AUTHORS
FROM ${NAMESPACE}/node-base:${VERSION}
LABEL authors=${AUTHORS}

USER root

#=========
# Firefox
#=========
ARG FIREFOX_VERSION=latest
RUN FIREFOX_DOWNLOAD_URL=$(if [ $FIREFOX_VERSION = "latest" ] || [ $FIREFOX_VERSION = "beta-latest" ] || [ $FIREFOX_VERSION = "nightly-latest" ] || [ $FIREFOX_VERSION = "devedition-latest" ] || [ $FIREFOX_VERSION = "esr-latest" ]; then echo "https://download.mozilla.org/?product=firefox-$FIREFOX_VERSION-ssl&os=linux64&lang=en-US"; else echo "https://download-installer.cdn.mozilla.net/pub/firefox/releases/$FIREFOX_VERSION/linux-x86_64/en-US/firefox-$FIREFOX_VERSION.tar.bz2"; fi) \
  && apt-get update -qqy \
  && apt-get -qqy --no-install-recommends install git firefox libavcodec-extra \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/* \
  && wget --no-verbose -O /tmp/firefox.tar.bz2 $FIREFOX_DOWNLOAD_URL \
  && apt-get -y purge firefox \
  && rm -rf /opt/firefox \
  && tar -C /opt -xjf /tmp/firefox.tar.bz2 \
  && rm /tmp/firefox.tar.bz2 \
  && mv /opt/firefox /opt/firefox-$FIREFOX_VERSION \
  && ln -fs /opt/firefox-$FIREFOX_VERSION/firefox /usr/bin/firefox

ARG GOLANG_VERSION="1.20.4"
RUN wget --no-verbose -O /tmp/golang_linux64.tar.gz https://dl.google.com/go/go${GOLANG_VERSION}.linux-amd64.tar.gz \
    && tar -zxf /tmp/golang_linux64.tar.gz -C /opt \
    && mkdir /go \
    && chown 1200:1201 /go \
    && chown -R 1200:1201 /opt/go

ENV GOROOT /opt/go
ENV GOPATH /go
ENV PATH $GOROOT/bin:$PATH

#============
# GeckoDriver
#============
ARG GECKODRIVER_VERSION=latest
RUN GK_VERSION=$(if [ ${GECKODRIVER_VERSION:-latest} = "latest" ]; then echo "0.33.0"; else echo $GECKODRIVER_VERSION; fi) \
  && echo "Using GeckoDriver version: "$GK_VERSION \
  && wget --no-verbose -O /tmp/geckodriver.tar.gz https://github.com/mozilla/geckodriver/releases/download/v$GK_VERSION/geckodriver-v$GK_VERSION-linux64.tar.gz \
  && rm -rf /opt/geckodriver \
  && tar -C /opt -zxf /tmp/geckodriver.tar.gz \
  && rm /tmp/geckodriver.tar.gz \
  && mv /opt/geckodriver /opt/geckodriver-$GK_VERSION \
  && chmod 755 /opt/geckodriver-$GK_VERSION \
  && ln -fs /opt/geckodriver-$GK_VERSION /usr/bin/geckodriver

USER 1200

#============================================
# Dumping Browser name and version for config
#============================================
RUN echo "firefox" > /opt/selenium/browser_name

ARG GHE_USER
ARG GHE_AUTH_TOKEN
ARG UI_AUTOMATION_BRANCH="master"
ARG UI_AUTOMATION_COMMIT_ID
ARG GIT_HOST
ARG GIT_URL
ARG GIT_REPO
RUN echo "machine $GIT_HOST login $GHE_USER password $GHE_AUTH_TOKEN" > $HOME/.netrc \
    && cd /tmp \
    && git clone --branch $UI_AUTOMATION_BRANCH --single-branch $GIT_URL \
    && cd $GIT_REPO \
    && git checkout $UI_AUTOMATION_COMMIT_ID \
    && sed -i 's/PortNo: 8081/PortNo: 4444/g' config/*.yaml \
    && sed -i 's/PortNo: 8081/PortNo: 4444/g' config/*.yml \
    && go mod tidy \
    && rm -f $HOME/.netrc
