# Socket CLI used by .github/workflows/socket.yml. Dependabot keeps the Node image and package.json current.
FROM docker.io/library/node:22.23.2-alpine@sha256:b6f26b36c8ff49624cfdac716b8ea1138d606df02586a77d364bb5536a634f85
WORKDIR /opt/socket
COPY package.json package-lock.json ./
RUN npm ci --ignore-scripts --no-audit --no-fund
ENV PATH=/opt/socket/node_modules/.bin:$PATH
