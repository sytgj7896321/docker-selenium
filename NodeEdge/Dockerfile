ARG NAMESPACE
ARG VERSION
ARG AUTHORS
FROM ${NAMESPACE}/node-base:${VERSION}
LABEL authors=${AUTHORS}

USER root

#============================================
# Microsoft Edge
#============================================
# can specify versions by EDGE_VERSION;
#  e.g. microsoft-edge-beta=88.0.692.0-1
#============================================
ARG EDGE_VERSION="microsoft-edge-stable"
RUN wget -q -O - https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
  && echo "deb https://packages.microsoft.com/repos/edge stable main" >> /etc/apt/sources.list.d/microsoft-edge.list \
  && apt-get update -qqy \
  && apt-get -qqy install git ${EDGE_VERSION} \
  && rm /etc/apt/sources.list.d/microsoft-edge.list \
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
# Edge Launch Script Wrapper
#=================================
COPY wrap_edge_binary /opt/bin/wrap_edge_binary
RUN /opt/bin/wrap_edge_binary

USER 1200

#============================================
# Edge webdriver
#============================================
# can specify versions by EDGE_DRIVER_VERSION
# Latest released version will be used by default
#============================================
ARG EDGE_DRIVER_VERSION
RUN if [ -z "$EDGE_DRIVER_VERSION" ]; \
  then EDGE_MAJOR_VERSION=$(microsoft-edge --version | sed -E "s/.* ([0-9]+)(\.[0-9]+){3}.*/\1/") \
    && EDGE_DRIVER_VERSION=$(wget --no-verbose -O - "https://msedgedriver.azureedge.net/LATEST_RELEASE_${EDGE_MAJOR_VERSION}_LINUX" | tr -cd "\11\12\15\40-\176" | tr -d "\r"); \
  fi \
  && echo "Using msedgedriver version: "$EDGE_DRIVER_VERSION \
  && wget --no-verbose -O /tmp/msedgedriver_linux64.zip https://msedgedriver.azureedge.net/$EDGE_DRIVER_VERSION/edgedriver_linux64.zip \
  && rm -rf /opt/selenium/msedgedriver \
  && unzip /tmp/msedgedriver_linux64.zip -d /opt/selenium \
  && rm /tmp/msedgedriver_linux64.zip \
  && mv /opt/selenium/msedgedriver /opt/selenium/msedgedriver-$EDGE_DRIVER_VERSION \
  && chmod 755 /opt/selenium/msedgedriver-$EDGE_DRIVER_VERSION \
  && sudo ln -fs /opt/selenium/msedgedriver-$EDGE_DRIVER_VERSION /usr/bin/msedgedriver

#============================================
# Dumping Browser name and version for config
#============================================
RUN echo "MicrosoftEdge" > /opt/selenium/browser_name

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
