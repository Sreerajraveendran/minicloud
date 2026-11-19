# syntax=docker/dockerfile:1

# ---- build stage: install deps (cached layer), then build static files -----
FROM node:22-alpine AS build
WORKDIR /src
COPY package.json package-lock.json ./
RUN npm ci --no-fund --no-audit
COPY . .
RUN npm run build

# ---- runtime stage: static files served by unprivileged NGINX on :8080 ------
FROM nginxinc/nginx-unprivileged:1.31-alpine AS runtime
COPY nginx-static.conf /etc/nginx/conf.d/default.conf
COPY --from=build /src/dist /usr/share/nginx/html
EXPOSE 8080
