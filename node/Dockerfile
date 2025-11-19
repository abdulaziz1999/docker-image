# Gunakan Node LTS (alpine supaya ringan)
FROM node:24-alpine

# Install tool tambahan yang sering diperlukan
RUN apk add --no-cache bash git curl

# Set workdir
WORKDIR /app

# Default NODE_ENV development (bisa override)
ARG NODE_ENV=development
ENV NODE_ENV=$NODE_ENV

# Copy package.json dan lock file jika ada
COPY package*.json ./

# Install dependencies (jika tidak ada lock file tetap jalan)
RUN npm install || true

# Copy semua source code
COPY . .

# Jika production, build project
RUN if [ "$NODE_ENV" = "production" ]; then npm run build; fi

# Expose port
EXPOSE 3000

# Default command (bisa override saat run)
CMD [ "npm", "run", "dev" ]

