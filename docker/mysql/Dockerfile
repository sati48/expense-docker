FROM mysql:8.0
ENV MYSQL_ROOT_PASSWORD=ExpenseApp@1
ENV MYSQL_DATABASE=transactions
ENV MYSQL_USER=expense
ENV MYSQL_PASSWORD=ExpenseApp@1
COPY ./expense-backend-v2/schema/backend.sql /docker-entrypoint-initdb.d/   
EXPOSE 3306