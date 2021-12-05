FROM golang:1.17.1-alpine as builder

WORKDIR /ProjectAlias


RUN apk update \
    && apk add git yarn build-base gcc abuild binutils binutils-doc gcc-doc \
    && git clone --recurse-submodules https://github.com/Xhofe/alist-web.git \
    && git clone --recurse-submodules https://github.com/Xhofe/alist.git \
    && cd /ProjectAlias/alist-web \
    && yarn install --network-timeout 1000000 \
    && yarn run build \
    && mv /ProjectAlias/alist-web/dist/* /ProjectAlias/alist/public \
    && cd /ProjectAlias/alist \
    && appName="alist" \
    && builtAt="$(date +'%F %T %z')" \
    && goVersion=$(go version | sed 's/go version //') \
    && gitAuthor=$(git show -s --format='format:%aN <%ae>' HEAD) \ 
    && gitCommit=$(git log --pretty=format:"%h" -1) \
    && gitTag=$(git describe --long --tags --dirty --always) \
    && ldflags="\
    -w -s \
    -X 'github.com/Xhofe/alist/conf.BuiltAt=$builtAt' \
    -X 'github.com/Xhofe/alist/conf.GoVersion=$goVersion' \
    -X 'github.com/Xhofe/alist/conf.GitAuthor=$gitAuthor' \
    -X 'github.com/Xhofe/alist/conf.GitCommit=$gitCommit' \
    -X 'github.com/Xhofe/alist/conf.GitTag=$gitTag' \
    " \
    && go build -ldflags="$ldflags" -o alist-main alist.go

FROM lsiobase/alpine:3.13

ENV PUID=1000
ENV PGID=1000
ENV TZ="Asia/Shanghai"

LABEL MAINTAINER="dusk"

WORKDIR /alist

COPY --from=builder /ProjectAlias/alist/alist-main /alist/

VOLUME ["/alist/data"]

RUN echo ">>>>>> update dependencies" \
    && apk update \
    && apk add tzdata \
    && echo ">>>>>> set up timezone" \
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && echo ">>>>>> fix cloudreve-main premission" \
    && chmod +x /alist/alist-main

EXPOSE 5244

ENTRYPOINT ["./alist-main"]

