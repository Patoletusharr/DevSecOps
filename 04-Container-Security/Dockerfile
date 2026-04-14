# Build stage- build the application and its dependencies
FROM node:25 AS builder

WORKDIR /build

COPY package*.json ./
RUN npm install

COPY . .

RUN groupadd -r 1000 && useradd -r -g 1000 1000
#run stage- copy only necessary files from builder stage
#FROM node:25-slim (slim image has smaller attack surface than full node image but still has node runtime and other packages)
FROM gcr.io/distroless/nodejs20-debian12  
#distroless image has minimal attack surface and only includes necessary runtime for nodejs applications)

WORKDIR /app

COPY --from=builder /build/node_modules ./node_modules
COPY --from=builder /build/app.js ./app.js

USER 1000
#run as non-root user with UID 1000 to further reduce attack surface

EXPOSE 3000

CMD ["app.js"]
