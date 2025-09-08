# # -------------------
# # Stage 1: Node.js build environment
# # -------------------
# FROM node:18-alpine AS build

# # Set working directory
# WORKDIR /app

# # Copy package files and install dependencies
# COPY package*.json ./
# RUN npm install

# # Copy all source code
# COPY . .

# # Build React app (env variables must be set at build time)
# ARG REACT_APP_API_URL
# ENV REACT_APP_API_URL=${REACT_APP_API_URL}
# RUN npm run build


# # -------------------
# # Stage 2: Nginx runtime server
# # -------------------
# FROM nginx:stable-alpine AS production

# # Remove default nginx static assets
# RUN rm -rf /usr/share/nginx/html/*

# # Copy built React app from previous stage
# COPY --from=build /app/build /usr/share/nginx/html

# # Copy custom nginx config (optional)
# # COPY nginx.conf /etc/nginx/conf.d/default.conf

# # Expose port
# EXPOSE 80

# # Start Nginx
# CMD ["nginx", "-g", "daemon off;"]


# -------------------
# Stage 1: Dependencies (Node.js)
# -------------------
FROM node:18-alpine AS deps

WORKDIR /app

# Install dependencies only
COPY package*.json ./
RUN npm install


# -------------------
# Stage 2: Build React app
# -------------------
FROM node:18-alpine AS build

WORKDIR /app

# Copy installed deps from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy all source code
COPY . .

# Pass API URL at build time
ARG REACT_APP_API_URL
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

# Build React app
RUN npm run build


# -------------------
# Stage 3: Nginx runtime
# -------------------
FROM nginx:stable-alpine AS production

# Remove default nginx static assets
RUN rm -rf /usr/share/nginx/html/*

# Copy built React app
COPY --from=build /app/build /usr/share/nginx/html

# (Optional) custom nginx.conf
# COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
