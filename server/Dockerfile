FROM node:lts-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

ENV MONGO_URI="mongodb://mongodb:27017/mern-app"

CMD ["npm","start"]