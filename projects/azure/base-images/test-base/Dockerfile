# This is development dockerfile
# original copy is kept inside build-config
FROM docker.io/centos:7

ENV HOME=/tmp/home \
    USER_UID=1001 \
    GOROOT=/usr/local/go \
    GOPATH=/go \
    GOCACHE=/go/.cache
ENV PATH=$GOPATH/bin:$GOROOT/bin:$PATH

# install golang-1.11
RUN mkdir -p /go /tmp/home && chmod -R 777 /go /tmp/home && \
    curl https://storage.googleapis.com/golang/go1.11.2.linux-amd64.tar.gz | tar -C /usr/local -xzf -

# install glide
RUN curl -L https://github.com/Masterminds/glide/releases/download/v0.13.1/glide-v0.13.1-linux-amd64.tar.gz | tar -C /tmp/ -xzf -
RUN mv /tmp/linux-amd64/glide /usr/bin

# install tooling
RUN yum update -y && \
    yum install epel-release -y && \
    yum install -y --setopt=skip_missing_names_on_install=False \
    gcc \
    git \
    jq \
    lsof \
    make \
    tar \
    wget \
    which \
    && \
    # enable git clone with random user (glide dependency)
    git config --system core.logallrefupdates false

# install azure cli
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc && \
    sh -c 'echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo' && \
    yum install azure-cli -y

RUN yum clean all

# install oc client for tests
RUN curl -L https://github.com/openshift/origin/releases/download/v3.10.0/openshift-origin-client-tools-v3.10.0-dd10d17-linux-64bit.tar.gz | tar -C /tmp/ -xzf - && \
    cp /tmp/openshift-origin-client-tools-v3.10.0-dd10d17-linux-64bit/oc /usr/local/bin/

RUN go get github.com/go-bindata/go-bindata/go-bindata && \
    go get golang.org/x/tools/cmd/goyacc && \
    go get github.com/golang/mock/gomock && \
    go install github.com/golang/mock/mockgen && \
    mv $GOPATH/bin/* /usr/bin/

# pin goimports to 04b5d21e commit, see discussion https://github.com/openshift/openshift-azure/pull/862
RUN go get golang.org/x/tools/cmd/goimports && \
    pushd $GOPATH/src/golang.org/x/tools/cmd/goimports/ && \
    git checkout -q 04b5d21e && \
    go build && \
    cp ./goimports /usr/bin/goimports && \
    rm -rf $GOPATH/* $GOPATH/.cache && \
    popd

# Add image scripts and files for running as non priv container
RUN chown -R $USER_UID:$USER_UID $HOME && \
    chown -R $USER_UID:$USER_UID $GOPATH && \
    chmod ug+rwx -R $HOME && \
    chmod g+xw -R $GOPATH && \
    chmod g+rw /etc/passwd

COPY entrypoint /usr/local/bin/
USER $USER_UID
ENTRYPOINT [ "/usr/local/bin/entrypoint" ]
