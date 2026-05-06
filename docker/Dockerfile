FROM node:lts-bookworm-slim

ARG STANDALONE_SUBPATH=standalone/apps/web

ENV NODE_ENV=production
ENV HOSTNAME=0.0.0.0
ENV PORT=2323
ENV NEXT_SHARP_PATH=/usr/local/lib/node_modules/sharp
ENV YOHAKU_BASE_DIR=/app/yohaku

RUN apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates unzip \
  && rm -rf /var/lib/apt/lists/*

RUN npm install -g pnpm sharp

WORKDIR /app/yohaku

COPY release.zip ./current/release.zip

RUN cd current \
  && unzip -qq -o release.zip \
  && rm release.zip \
  && rm -rf "${STANDALONE_SUBPATH}/.env" \
  && mkdir -p /app/yohaku/.cache "${STANDALONE_SUBPATH}/.next" \
  && rm -rf "${STANDALONE_SUBPATH}/.next/cache" \
  && ln -sf /app/yohaku/.cache "${STANDALONE_SUBPATH}/.next/cache" \
  && ln -sf "/app/yohaku/current/${STANDALONE_SUBPATH}/server.js" /app/yohaku/server.js

EXPOSE 2323

CMD ["node", "server.js"]
