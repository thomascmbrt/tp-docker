FROM node:12-alpine3.9

WORKDIR /app

COPY package*.json .

RUN npm install --production

COPY src/ ./src


CMD ["node", "src/index.js"]