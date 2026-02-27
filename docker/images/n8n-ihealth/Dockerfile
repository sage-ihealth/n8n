# =============================================================================
# iHealth Labs — n8n (built from source)
# Source: https://github.com/sage-ihealth/n8n
# =============================================================================

# ── Stage 1: Build ────────────────────────────────────────────────────────────
FROM node:22-alpine AS builder

# System deps for native modules
RUN apk add --no-cache git python3 make g++ curl

# Install pnpm
RUN npm install -g pnpm@10.22.0

WORKDIR /app
COPY . .

# Build the monorepo → outputs to ./compiled
RUN CI=true pnpm install --frozen-lockfile
RUN CI=true node scripts/build-n8n.mjs

# ── Stage 2: Runtime ──────────────────────────────────────────────────────────
FROM node:22-alpine AS runtime

ARG N8N_VERSION=2.10.0
ARG BUILD_SHA=local
ARG BUILD_DATE

ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable
ENV SHELL=/bin/sh

# Runtime deps (tini for proper PID 1, sqlite3 headers)
RUN apk add --no-cache tini graphicsmagick ghostscript openssl

WORKDIR /home/node

# Copy compiled output from builder
COPY --from=builder /app/compiled /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /docker-entrypoint.sh

RUN cd /usr/local/lib/node_modules/n8n && \
    npm rebuild sqlite3 2>/dev/null || true && \
    ln -s /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    mkdir -p /home/node/.n8n && \
    chown -R node:node /home/node && \
    chmod +x /docker-entrypoint.sh && \
    rm -rf /root/.npm /tmp/*

EXPOSE 5678/tcp
USER node

LABEL org.opencontainers.image.title="iHealth n8n" \
      org.opencontainers.image.description="iHealth Labs Workflow Automation Platform — built from source" \
      org.opencontainers.image.source="https://github.com/sage-ihealth/n8n" \
      org.opencontainers.image.vendor="iHealth Labs Inc." \
      org.opencontainers.image.version="${N8N_VERSION}" \
      org.opencontainers.image.revision="${BUILD_SHA}" \
      org.opencontainers.image.created="${BUILD_DATE}" \
      com.ihealthlabs.component="n8n-workflow-automation"

ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
