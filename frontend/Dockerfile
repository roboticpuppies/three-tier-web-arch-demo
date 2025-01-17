FROM node:18.15.0-alpine3.17 AS builder
ENV REACT_APP_BACKEND_URL=https://api.realworld.io/api
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:1.23.4-alpine
COPY --from=builder /app/build /usr/share/nginx/html