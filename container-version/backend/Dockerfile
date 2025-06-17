FROM node:18-alpine AS build

RUN apk add --no-cache git openssl libc6-compat

RUN git clone https://github.com/gothinkster/node-express-realworld-example-app.git /app

# Set the working directory
WORKDIR /app
RUN yarn install
RUN npx prisma generate

FROM node:18-alpine AS final

RUN apk add --no-cache openssl libc6-compat
WORKDIR /app

# Copy only the built app and necessary files from build stage
COPY --from=build /app ./

# # Set environment variable for Prisma to find DB at runtime
ENV NODE_ENV=development

CMD ["npm", "start"]