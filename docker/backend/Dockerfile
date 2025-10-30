FROM node:20
workdir /app
COPY ./expense-backend-v2/package*.json ./  
RUN npm install
COPY ./expense-backend-v2 . 
ENV DB_HOST=mysql
CMD ["node", "server.js"]