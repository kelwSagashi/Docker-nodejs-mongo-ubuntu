FROM node

WORKDIR /src

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "start"]