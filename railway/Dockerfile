# Railway build of this fork (see AGENTS-fork.md). It mirrors ../Dockerfile-build with two differences:
# - No BuildKit cache mounts: Railway rejects cache mounts whose id lacks an "s/<service-id>-" prefix,
#   and hardcoding a service ID would tie the repo to a single Railway service.
# - No docs build: the embedded /docs page is optional, and skipping it drops the Python/mkdocs toolchain.

FROM node:24-bookworm AS web
WORKDIR /app
COPY Makefile ./
COPY web ./web
# "make web-build" moves the bundle into ../server/site, so the parent directory must exist.
RUN mkdir -p server && make web-deps web-build

FROM golang:1.26-bookworm AS builder
# Railway passes the deployed commit as a build arg when the Dockerfile declares it.
ARG RAILWAY_GIT_COMMIT_SHA=unknown
WORKDIR /app
COPY . .
COPY --from=web /app/server/site ./server/site
RUN make VERSION=dev COMMIT=$RAILWAY_GIT_COMMIT_SHA cli-linux-server

FROM alpine
RUN apk add --no-cache tzdata
COPY --from=builder /app/dist/ntfy_linux_server/ntfy /usr/bin/ntfy
# ntfy reads /etc/ntfy/server.yml by default, for both "ntfy serve" and the "ntfy user/token" admin commands.
COPY railway/server.yml /etc/ntfy/server.yml
EXPOSE 8080/tcp
ENTRYPOINT ["ntfy"]
CMD ["serve"]
