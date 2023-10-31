FROM node:18.18.2-alpine
WORKDIR /app
COPY package*.json ./
RUN npm clean-install
COPY . .
EXPOSE 8888
CMD [ "npm", "start" ]