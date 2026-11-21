FROM node:24-alpine AS builder
WORKDIR /src
RUN npm install -g pnpm@12
COPY package.json pnpm-lock.yaml ./
RUN pnpm ci
COPY . .
RUN pnpm install --frozen-lockfile
COPY . .
RUN pnpm run build

FROM node:24-alpine AS runner
WORKDIR /src
RUN npm install -g pnpm@12
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --prod --frozen-lockfile
COPY --from=builder /src/dist ./dist
CMD [ "node", "dist/main.js" ]
