FROM node:16.9.1-alpine3.11

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
RUN npm install -g nodemon
COPY package.json package-lock.json ./
RUN npm install
COPY . .

CMD nodemon server
