# dev build
FROM ghcr.io/immich-app/base-server-dev:20231214@sha256:cd5be516b27c0c402bee3a6a93d8c83dfd5a827c18a2343cb97b55f3be98151b as dev

RUN apt-get install --no-install-recommends -yqq tini
WORKDIR /usr/src/app
COPY server/package.json server/package-lock.json ./
RUN npm ci && \
    # sharp-linux-x64 and sharp-linux-arm64 are the only ones we need
    # they're marked as optional dependencies, so we need to copy them manually after pruning
    rm -rf node_modules/@img/sharp-libvips* && \
    rm -rf node_modules/@img/sharp-linuxmusl-x64
COPY server .
ENV PATH="${PATH}:/usr/src/app/bin"
ENTRYPOINT ["tini", "--", "/bin/sh"]


FROM dev AS prod

RUN npm run build
RUN npm prune --omit=dev --omit=optional
COPY --from=dev /usr/src/app/node_modules/@img ./node_modules/@img

# web build
FROM node:iron-alpine3.18 as web

WORKDIR /usr/src/app
COPY web/package*.json web/svelte.config.js .
RUN npm ci
COPY web .
RUN npm run build


# prod build
FROM ghcr.io/immich-app/base-server-prod:20231214@sha256:b214f86683fde081b09beed2d7bfc28bec55c829751ccf2e02ad7dd18293f5e0

WORKDIR /usr/src/app
ENV NODE_ENV=production
COPY --from=prod /usr/src/app/node_modules ./node_modules
COPY --from=prod /usr/src/app/dist ./dist
COPY --from=prod /usr/src/app/bin ./bin
COPY --from=web /usr/src/app/build ./www
COPY server/resources resources
COPY server/package.json server/package-lock.json ./
COPY server/start*.sh ./
RUN npm link && npm cache clean --force
COPY LICENSE /licenses/LICENSE.txt
COPY LICENSE /LICENSE
ENV PATH="${PATH}:/usr/src/app/bin"
VOLUME /usr/src/app/upload
EXPOSE 3001
ENTRYPOINT ["tini", "--", "/bin/sh"]
