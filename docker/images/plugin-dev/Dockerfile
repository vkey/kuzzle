################################################################################
# Plugin development build image
################################################################################
FROM node:12.18.1-stretch-slim as builder

RUN  set -x \
  && apt-get update && apt-get install -y \
       curl \
       python \
       make \
       g++ \
       python \
       libzmq3-dev \
  && npm install -g --unsafe-perm --production \
    nodemon \
    kourou \
    pm2 \
  && curl -sfL https://install.goreleaser.com/github.com/tj/node-prune.sh | sh -s -- -b /usr/local/bin

ADD ./bin /app/bin
ADD ./config /app/config
ADD ./lib /app/lib
ADD ./package.json /app/package.json
ADD ./package-lock.json /app/package-lock.json
ADD ./default.config.js /app/default.config.js

WORKDIR /app

# Install dependencies
RUN  npm install --production

ADD ./docker/scripts/clean-node.sh /usr/bin/clean-node

ADD ./plugins/available/ /app/plugins/available/

RUN  set -x \
  # Link default plugins
  && mkdir -p /app/plugins/enabled/ \
  && ln -s /app/plugins/available/kuzzle-plugin-auth-passport-local /app/plugins/enabled/kuzzle-plugin-auth-passport-local \
  && ln -s /app/plugins/available/kuzzle-plugin-logger /app/plugins/enabled/kuzzle-plugin-logger \
  # Install plugins dependencies
  && for plugin in plugins/*/*; do npm install --production --prefix "$plugin"; done \
  # Clean Node.js related useless files
  && clean-node \
  && for plugin in plugins/*/*; do cd "$plugin"; echo "$plugin"; clean-node; cd /app; done \
  # Remove useless leftover dependencies
  && rm -rf node_modules/rxjs/ \
  # Strip binaries
  && strip /usr/local/bin/node \
  && strip node_modules/re2/build/Release/re2.node

################################################################################
# Plugin development image
################################################################################
FROM debian:stretch-slim

LABEL io.kuzzle.vendor="Kuzzle <support@kuzzle.io>"
LABEL description="Develop new plugin or protocol for Kuzzle with ease"

WORKDIR /var/app

ADD ./docker/scripts/entrypoint.sh /bin/entrypoint
ADD ./docker/scripts/run-plugin.sh /bin/run-plugin

COPY --from=builder /lib /lib
COPY --from=builder /usr /usr
COPY --from=builder /app /var/app

RUN  set -x \
  && rm /bin/sh \
  && ln -s /bin/bash /bin/sh \
  && ln -s /var/app /app

WORKDIR /var/app

ENV PATH=$PATH:/var/app/bin

ENTRYPOINT ["/bin/entrypoint"]

CMD ["/bin/run-plugin"]
