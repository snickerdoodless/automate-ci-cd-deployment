FROM mysql:5.7

COPY feane.sql /docker-entrypoint-initdb.d/

# change the root pw 
ENV MYSQL_ROOT_PASSWORD=root 
ENV MYSQL_DATABASE=feane
