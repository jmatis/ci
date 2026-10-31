# Socket CLI used by .github/workflows/socket.yml. Dependabot keeps the Node image and package.json current.
FROM docker.io/library/node:26.12.1-alpine@sha256:40cd011b55415bc894e45b02fdd5a5dfde056921b7d23a64c74f80497af48797
WORKDIR /opt/socket
COPY package.json package-lock.json ./
RUN npm ci --ignore-scripts --no-audit --no-fund
ENV PATH=/opt/socket/node_modules/.bin:$PATH
