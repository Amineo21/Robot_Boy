# Étape 1 : builder l'app TanStack Start
FROM node:22-alpine AS builder
WORKDIR /app
COPY health-robot-front/package*.json ./
RUN npm ci
COPY health-robot-front/ ./
RUN npm run build

# Étape 2 : runner Node.js pour TanStack Start
FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.output ./.output
COPY --from=builder /app/package*.json ./
RUN npm ci --omit=dev
EXPOSE 3000
CMD ["node", ".output/server/index.mjs"]
