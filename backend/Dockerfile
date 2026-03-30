# ──────────────────────────────────────────────
# Stage 1 – Build
# ─────────────────────────────────────────────

FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production && npm cache clean --force


# ──────────────────────────────────────────────
# Stage 2 – Production (Reduce Image size)
# ──────────────────────────────────────────────

FROM node:20-alpine 

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

RUN apk --no-cache add dumb-init

WORKDIR /app

COPY --from=build /app/node_modules ./node_modules

COPY src ./src/

USER appuser

EXPOSE 5000

ENTRYPOINT ["dumb-init","--"]
CMD ["node","src/index.js"]
