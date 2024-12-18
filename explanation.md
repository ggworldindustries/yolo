## 1. Choice of Base Image
 The base image used to build the containers is `node:16-alpine3.16`. It is derived from the Alpine Linux distribution, making it lightweight and compact. 
 Used 
 1. Client:`node:16-alpine3.16`
 2. Backend: `node:16-alpine3.16`
 3.Mongo : `mongo:6.0 `
       

## 2. Dockerfile directives used in the creation and running of each container.
 I used two Dockerfiles. One for the Client and the other one for the Backend.

**Client Dockerfile**

```
# Build stage
FROM node:16-alpine3.16 as build-stage

# Set the working directory inside the container
WORKDIR /client

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies and clear the npm cache and remove any temporary files
RUN npm install --only=production && \
  npm cache clean --force && \
  rm -rf /tmp/*

# Copy the rest of the application code
COPY . .

# Build the application and remove development dependencies
RUN npm run build && \
  npm prune --production

# Production stage
FROM node:16-alpine3.16 as production-stage

WORKDIR /client

# Copy only the necessary files from the build stage
COPY --from=build-stage /client/build ./build
COPY --from=build-stage /client/public ./public
COPY --from=build-stage /client/src ./src
COPY --from=build-stage /client/package*.json ./

# Set the environment variable for the app
ENV NODE_ENV=production

# Expose the port used by the app
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

**Backend Dockerfile**

```dockerfile
# Set base image
FROM node:16-alpine3.16

# Set the working directory
WORKDIR /backend

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies and clear the npm cache and remove any temporary files
RUN npm install --only=production && \
  npm cache clean --force && \
  rm -rf /tmp/*

# Copy the rest of the application code
COPY . .

# Set the environment variable for the app
ENV NODE_ENV=production

# Expose the port used by the app
EXPOSE 5000

# Prune the node_modules directory to remove development dependencies and clear the npm cache and remove any temporary files
RUN npm prune --production && \
  npm cache clean --force && \
  rm -rf /tmp/*

# Start the application
CMD ["npm", "start"]
```

## 3. Docker Compose Networking
The `docker-compose.yml` defines the networking configuration for the project. It includes the allocation of application ports. The relevant sections are as follows:

```yaml
services:
  backend:
  # ...
  ports:
    - "5000:5000"
  networks:
    - yolo-network

  client:
  # ...
  ports:
    - "3000:3000"
  networks:
    - yolo-network
  
  mongodb:
  # ...
  ports:
    - "27017:27017"
  networks:
    - yolo-network

networks:
  yolo-network:
  driver: bridge
```

In this configuration, the backend container is mapped to port 5000 of the host, the client container is mapped to port 3000 of the host, and the MongoDB container is mapped to port 27017 of the host. All containers are connected to the `yolo-network` bridge network.

## 4. Docker Compose Volume Definition and Usage
The Docker Compose file includes volume definitions for MongoDB data storage. The relevant section is as follows:

```yaml
volumes:
  mongodata:  mongodata
    driver: local
```

This volume, `mongodata`, is designated for storing MongoDB data. It ensures that the data remains intact and is not lost even if the container is stopped or deleted.

