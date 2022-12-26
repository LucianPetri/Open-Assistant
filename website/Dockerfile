# Install dependencies only when needed
FROM --platform=linux/amd64 node:16.18.0 as deps
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY ./package.json ./package-lock.json ./
RUN \
  if [ -f package-lock.json ]; then npm ci; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM --platform=linux/amd64 node:16.18.0 as builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY ./ .

RUN npx prisma generate
RUN npm run build

# Production image, copy all the files and run next
FROM --platform=linux/amd64 node:16.18.0 as runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

COPY --from=builder --chown=nextjs:nodejs /app/prisma/schema.prisma ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
