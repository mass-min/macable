FROM --platform=linux/amd64 mysql:5.7
COPY docker/mysql/my.cnf /etc/mysql/conf.d/my.cnf
RUN touch /var/log/slow.log
