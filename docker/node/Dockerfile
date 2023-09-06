FROM node:18 as node
ARG USER_ID=1000
ARG GROUP_ID=1000

FROM node AS deps
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM node AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# If using npm comment out above and use below instead
# RUN npm run build

# Production image, copy all the files and run next
FROM node AS base
ARG USER_ID
ARG GROUP_ID

RUN deluser --remove-home node && addgroup --system --gid ${GROUP_ID} nextjs && adduser --system --uid ${USER_ID} --ingroup nextjs nextjs

FROM base as node-prod
ENV NODE_ENV production
WORKDIR /app

#COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
#COPY --from=builder --chown=nextjs:nextjs /app/.next/standalone ./
#COPY --from=builder --chown=nextjs:nextjs /app/.next/static ./.next/static

USER nextjs

EXPOSE 5173

ENV PORT 5173
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

CMD node server.js

FROM base as node-dev
ENV NODE_ENV development
WORKDIR /var/www/app
EXPOSE 5173
ENV PORT 5173

#COPY --chown=nextjs:nodejs . /var/www/app

USER nextjs

CMD yarn install && yarn dev
