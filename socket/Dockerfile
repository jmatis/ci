# Socket CLI used by .github/workflows/socket.yml. Dependabot keeps the Node image and package.json current.
FROM docker.io/library/node:22.23.4-alpine@sha256:0e1754758c2df4cfa3e68d59e2e1b7374bb7fd9ea6328eaaacf15cc934af8202
WORKDIR /opt/socket
COPY package.json package-lock.json ./
RUN npm ci --ignore-scripts --no-audit --no-fund
ENV PATH=/opt/socket/node_modules/.bin:$PATH
