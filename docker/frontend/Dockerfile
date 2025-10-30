from nginx:latest
RUN rm -rf /usr/share/nginx/html/*
COPY ./expense-frontend-v2/* /usr/share/nginx/html/ 
COPY docker/frontend/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]