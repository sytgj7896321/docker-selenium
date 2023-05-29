ARG NAMESPACE
ARG VERSION
ARG AUTHORS
FROM ${NAMESPACE}/node-base:${VERSION}
LABEL authors=${AUTHORS}

USER root

#============================================
# Google Chrome
#============================================
# can specify versions by CHROME_VERSION;
#  e.g. google-chrome-stable=53.0.2785.101-1
#       google-chrome-beta=53.0.2785.92-1
#       google-chrome-unstable=54.0.2840.14-1
#       latest (equivalent to google-chrome-stable)
#       google-chrome-beta  (pull latest beta)
#============================================
ARG CHROME_VERSION="google-chrome-stable"
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
  && apt-get update -qqy \
  && apt-get -qqy install git \
    ${CHROME_VERSION:-google-chrome-stable} \
  && rm /etc/apt/sources.list.d/google-chrome.list \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

ARG GOLANG_VERSION="1.20.4"
RUN wget --no-verbose -O /tmp/golang_linux64.tar.gz https://dl.google.com/go/go${GOLANG_VERSION}.linux-amd64.tar.gz \
    && tar -zxf /tmp/golang_linux64.tar.gz -C /opt \
    && mkdir /go \
    && chown 1200:1201 /go \
    && chown -R 1200:1201 /opt/go

ENV GOROOT /opt/go
ENV GOPATH /go
ENV PATH $GOROOT/bin:$PATH

#=================================
# Chrome Launch Script Wrapper
#=================================
COPY wrap_chrome_binary /opt/bin/wrap_chrome_binary
RUN /opt/bin/wrap_chrome_binary

USER 1200

#============================================
# Chrome webdriver
#============================================
# can specify versions by CHROME_DRIVER_VERSION
# Latest released version will be used by default
#============================================
ARG CHROME_DRIVER_VERSION
RUN if [ -z "$CHROME_DRIVER_VERSION" ]; \
  then CHROME_MAJOR_VERSION=$(google-chrome --version | sed -E "s/.* ([0-9]+)(\.[0-9]+){3}.*/\1/") \
    && NO_SUCH_KEY=$(curl -ls https://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROME_MAJOR_VERSION} | head -n 1 | grep -oe NoSuchKey) ; \
    if [ -n "$NO_SUCH_KEY" ]; then \
      echo "No Chromedriver for version $CHROME_MAJOR_VERSION. Use previous major version instead" \
      && CHROME_MAJOR_VERSION=$(expr $CHROME_MAJOR_VERSION - 1); \
    fi ; \
    CHROME_DRIVER_VERSION=$(wget --no-verbose -O - "https://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROME_MAJOR_VERSION}"); \
  fi \
  && echo "Using chromedriver version: "$CHROME_DRIVER_VERSION \
  && wget --no-verbose -O /tmp/chromedriver_linux64.zip https://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip \
  && rm -rf /opt/selenium/chromedriver \
  && sudo unzip /tmp/chromedriver_linux64.zip -d /opt/selenium \
  && rm /tmp/chromedriver_linux64.zip \
  && sudo mv /opt/selenium/chromedriver /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && sudo chmod 755 /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION \
  && sudo ln -fs /opt/selenium/chromedriver-$CHROME_DRIVER_VERSION /usr/bin/chromedriver

#============================================
# Dumping Browser name and version for config
#============================================
RUN echo "chrome" > /opt/selenium/browser_name

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
