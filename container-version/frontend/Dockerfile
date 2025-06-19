FROM node:18-alpine AS build

RUN apk add --no-cache git

RUN git clone https://github.com/khaledosman/react-redux-realworld-example-app.git /app

# Set the working directory
WORKDIR /app
RUN yarn install

ENV REACT_APP_BACKEND_URL="http://192.168.200.250:8090/api"
RUN yarn run build

FROM nginx:stable-alpine AS final
COPY --from=build /app/build /usr/share/nginx/html
# COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]