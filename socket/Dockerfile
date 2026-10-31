# Socket CLI used by .github/workflows/socket.yml. Dependabot keeps the Node image and package.json current.
FROM docker.io/library/node:22.23.4-alpine@sha256:fe4f6d74e20370acb04ad889077d14d0f64a9c8ea6a42bc60cddf1e844699914
WORKDIR /opt/socket
COPY package.json package-lock.json ./
RUN npm ci --ignore-scripts --no-audit --no-fund
ENV PATH=/opt/socket/node_modules/.bin:$PATH
