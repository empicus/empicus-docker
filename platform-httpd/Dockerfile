# The Base Image used to create this Image
FROM httpd:latest

# Just my name who wrote this file
MAINTAINER SaravAK (aksarav@middlewareinventory.com)

# to Copy a file named httpd.conf from present working directory to the /usr/local/apache2/conf inside the container
# I have taken the Standard httpd.conf file and enabled the necassary modules and adding Support for an additional Directory
COPY httpd.conf /usr/local/apache2/conf/httpd.conf

# This is the Additional Directory where we are going to keep our Virtualhost configuraiton files
# You can use the image to create N number of different virtual hosts
RUN mkdir -p /usr/local/apache2/conf/sites/
RUN mkdir -p /usr/local/apache2/conf/sites-certs/client-portal
RUN mkdir -p /usr/local/apache2/conf/sites-certs/admin-portal
RUN mkdir -p /usr/local/apache2/conf/sites-certs/platform

RUN mkdir -p /var/www

COPY config/client-portal/client-portal.conf /usr/local/apache2/conf/sites/client-portal.conf
COPY config/client-portal/domain.key /usr/local/apache2/conf/sites-certs/client-portal/client-portal.com.key
COPY config/client-portal/domain-chain.crt /usr/local/apache2/conf/sites-certs/client-portal/client-portal.com.crt

COPY config/admin-portal/admin-portal.conf /usr/local/apache2/conf/sites/admin-portal.conf
COPY config/admin-portal/domain.key /usr/local/apache2/conf/sites-certs/admin-portal/admin-portal.com.key
COPY config/admin-portal/domain-chain.crt /usr/local/apache2/conf/sites-certs/admin-portal/admin-portal.com.crt

COPY config/platform/platform.conf /usr/local/apache2/conf/sites/platform.conf
COPY config/platform/domain.key /usr/local/apache2/conf/sites-certs/platform/platform.com.key
COPY config/platform/domain-chain.crt /usr/local/apache2/conf/sites-certs/platform/platform.com.crt

# To tell docker to expose this port
EXPOSE 80
EXPOSE 443

# The Base command, This command should be used to start the container
# Remember, A Container is a Process.As long as the base process (started by base cmd) is live the Container will be ALIVE.
CMD ["httpd", "-D", "FOREGROUND"]