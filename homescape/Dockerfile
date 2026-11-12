# Stage 1: the web app. Uses the files in ./www when present (npm run build:app), else builds them from the repository.
FROM node:22-alpine AS web
# Pinned fork: the app is built from one exact, reviewed commit of this fork, never from a branch.
# To update: review the upstream changes, then change HOMESCAPE_REF and bump `version` in config.yaml.
ARG HOMESCAPE_REPO=https://github.com/arrowedisgaming/homescape
ARG HOMESCAPE_REF=9b469cf0298c5254de48566f4c3b0d2e412d709e
COPY . /files
RUN if [ -f /files/www/index.html ]; then \
      cp -r /files/www /out; \
    else \
      apk add --no-cache git \
      && git init -q /src \
      && cd /src \
      && git fetch -q --depth 1 "${HOMESCAPE_REPO}" "${HOMESCAPE_REF}" \
      && git checkout -q FETCH_HEAD \
      && [ "$(git rev-parse HEAD)" = "${HOMESCAPE_REF}" ] \
      && npm ci --no-audit --no-fund \
      && npm run build \
      && cp -r dist /out; \
    fi

# Stage 2: a small nginx that serves the files to Home Assistant's ingress
FROM ghcr.io/home-assistant/base:3.24
RUN apk add --no-cache nginx && mkdir -p /run/nginx
COPY nginx.conf /etc/nginx/http.d/default.conf
COPY --from=web /out/ /var/www/homescape/
CMD ["nginx", "-g", "daemon off;"]
