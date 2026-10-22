FROM node:24.22.1-bookworm-slim AS build

WORKDIR /usr/local/app

ENV NODE_ENV=development
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

COPY .yarn ./.yarn
COPY yarn.lock package.json .yarnrc.yml ./

RUN yarn install --immutable

COPY tsconfig.json ./
COPY src ./src

RUN yarn build

ENV NODE_ENV=production

# install dependencies with frozen lockfile
RUN yarn install --immutable

FROM node:24.22.1-bookworm-slim

RUN apt-get update \
     && apt-get install -y wget gnupg ca-certificates procps libxss1 \
     # apt-key is deprecated on Debian 12 (and removed in newer releases),
     # thus the signing key is stored in a dedicated keyring instead.
     && wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome.gpg \
     && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] https://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
     && apt-get update \
     # We install Chrome to get all the OS level dependencies, but Chrome itself
     # is not actually used as it's packaged in the node puppeteer library.
     # Alternatively, we could could include the entire dep list ourselves
     # (https://github.com/puppeteer/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-unix)
     # but that seems too easy to get out of date.
     && apt-get install -y google-chrome-stable \
     # Font installation, support chinese font characters for now
     && apt-get install -y fonts-wqy-zenhei fonts-indic fonts-noto fonts-noto-cjk \
     && rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

ENV PORT=3000
ENV NODE_ENV=production
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/google-chrome

COPY --from=build /usr/local/app/node_modules ./node_modules
COPY --from=build /usr/local/app/.yarn ./.yarn
COPY --from=build /usr/local/app/build ./
COPY yarn.lock package.json .yarnrc.yml ./

CMD node index.js
