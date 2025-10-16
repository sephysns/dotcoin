# --- Stage 1: install dependencies (cached layer) ---
FROM node:20-alpine AS deps
WORKDIR /app
# Copy only the files needed to resolve dependencies
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile

# --- Stage 2: build the Next.js app ---
FROM node:20-alpine AS builder
WORKDIR /app
# Reuse node_modules from deps to speed up builds
COPY --from=deps /app/node_modules ./node_modules
# Copy the full app source
COPY . .
# Build production bundle
RUN corepack enable && pnpm build

# --- Stage 3: minimal runtime image ---
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy production artifacts and static assets
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY package.json ./package.json
COPY pnpm-lock.yaml ./pnpm-lock.yaml

# Install only production dependencies
RUN corepack enable && pnpm install --prod --frozen-lockfile

EXPOSE 3000
CMD ["pnpm","start"]
