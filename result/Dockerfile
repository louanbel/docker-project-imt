FROM node:18-slim

WORKDIR /usr/local/app

COPY package-lock.json package.json ./

RUN npm install -g nodemon && \
    npm ci && \
    npm cache clean --force  && \
    mv /usr/local/app/node_modules /node_modules

ENV PORT=4000

COPY . .

EXPOSE $PORT

CMD ["node", "server.js"]
