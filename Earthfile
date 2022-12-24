VERSION 0.6

# This Earthfile was generated using docker2earthly
# the conversion is done on a best-effort basis
# and might not follow best practices, please
# visit https://docs.earthly.dev for Earthfile guides
subbuild1:
    FROM golang:1.19-alpine3.17
    ARG GOPROXY
    ENV GOPROXY ${GOPROXY:-direct}
    ARG GITEA_VERSION
    ARG TAGS="sqlite sqlite_unlock_notify"
    ENV TAGS "bindata timetzdata $TAGS"
    ARG CGO_EXTRA_CFLAGS
    RUN apk --no-cache add build-base git nodejs npm
    COPY . ${GOPATH}/src/code.gitea.io/gitea
    WORKDIR ${GOPATH}/src/code.gitea.io/gitea
    RUN if [ -n "${GITEA_VERSION}" ]; then git checkout "${GITEA_VERSION}"; fi  && make clean-all build
    RUN go build contrib/environment-to-ini/environment-to-ini.go
    SAVE ARTIFACT /go/src/code.gitea.io/gitea/gitea gitea

    SAVE ARTIFACT /go/src/code.gitea.io/gitea/environment-to-ini environment-to-ini

subbuild2:
    FROM alpine:3.17
    LABEL maintainer="maintainers@gitea.io"
    EXPOSE 22 3000
    RUN apk --no-cache add     bash     ca-certificates     curl     gettext     git     linux-pam     openssh     s6     sqlite     su-exec     gnupg
    RUN addgroup     -S -g 1000     git &&   adduser     -S -H -D     -h /data/git     -s /bin/bash     -u 1000     -G git     git &&   echo "git:*" | chpasswd -e
    ENV USER git
    ENV GITEA_CUSTOM /data/gitea
    VOLUME ["/data"]
    ENTRYPOINT ["/usr/bin/entrypoint"]
    CMD ["/bin/s6-svscan", "/etc/s6"]
    COPY docker/root /
    COPY +subbuild1/gitea /app/gitea/gitea
    COPY +subbuild1/environment-to-ini /usr/local/bin/environment-to-ini
    RUN chmod 755 /usr/bin/entrypoint /app/gitea/gitea /usr/local/bin/gitea /usr/local/bin/environment-to-ini
    RUN chmod 755 /etc/s6/gitea/* /etc/s6/openssh/* /etc/s6/.s6-svscan/*
    SAVE IMAGE 

build:
    BUILD +subbuild2
