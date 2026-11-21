FROM node:20-alpine AS builder
WORKDIR /src
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /src
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /src/dist ./dist
CMD [ "node", "dist/main.js" ]
