FROM jboss/wildfly

ARG DB_URL

ENV DS_NAME JbpmDS
ENV WP_DS_NAME EmpicusPlatformDS

ENV MYSQL_DATABASE jbpmDB
ENV MYSQL_USER jbpmDB
ENV MYSQL_PASSWORD kwesta89634893wale3467893@f567hj

ENV WP_MYSQL_DATABASE empicusDB
ENV WP_MYSQL_USER empicusDB
ENV WP_MYSQL_PASSWORD kwesta89634893wale3467893@f567hj

ENV MYSQL_CONNECTOR mysql-connector-java-8.0.11.jar
ENV MYSQL_VERSION 6.0.6
ENV WILDFLY_USER empicus-platform-admin
ENV WILDFLY_PASS Andromeda89634893nxuj88638@7836
ENV JBOSS_HOME /opt/jboss/wildfly
ENV JBOSS_CLI /opt/jboss/wildfly/bin/jboss-cli.sh
ENV DEPLOYMENT_DIR /opt/jboss/wildfly/standalone/deployments/
# Add the mysql connector
ADD $MYSQL_CONNECTOR /opt/

# Add the admin user to the server
RUN $JBOSS_HOME/bin/add-user.sh -u $WILDFLY_USER -p $WILDFLY_PASS --silent

ADD alstracoin_io/keycloak.jks /opt/jboss/wildfly/standalone/configuration/
# ADD standalone.xml /opt/jboss/wildfly/standalone/configuration/
# Configure Wildfly server
RUN echo "=> Starting WildFly server" && \
    bash -c '$JBOSS_HOME/bin/standalone.sh &' && \
    echo "=> Waiting for the server to boot" && \
    bash -c 'until `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null | grep -q running`; do echo `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null`; sleep 1; done' && \
    echo "=> Adding MySQL module" && \
    $JBOSS_CLI --connect --command="module add --name=com.mysql --resources=/opt/$MYSQL_CONNECTOR --dependencies=javax.api,javax.transaction.api" && \
    echo "=> Adding MySQL driver" && \
    $JBOSS_CLI --connect --command="/subsystem=datasources/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql,driver-class-name=com.mysql.cj.jdbc.Driver)" && \
    echo "=> Creating a new JBPM datasource" && \
    $JBOSS_CLI --connect --command="data-source add \
    --name=${DS_NAME} \
    --jndi-name=java:jboss/datasources/${DS_NAME} \
    --user-name=${MYSQL_USER} \
    --password=${MYSQL_PASSWORD} \
    --driver-name=mysql \
    --connection-url=jdbc:mysql://${DB_URL}/${MYSQL_DATABASE} \
    --use-ccm=false \
    --max-pool-size=25 \
    --blocking-timeout-wait-millis=5000 \
    --enabled=true" && \
    echo "=> Creating a new WP datasource" && \
    $JBOSS_CLI --connect --command="data-source add \
    --name=${WP_DS_NAME} \
    --jndi-name=java:jboss/datasources/${WP_DS_NAME} \
    --user-name=${WP_MYSQL_USER} \
    --password=${WP_MYSQL_PASSWORD} \
    --driver-name=mysql \
    --connection-url=jdbc:mysql://${DB_URL}/${WP_MYSQL_DATABASE} \
    --use-ccm=false \
    --max-pool-size=25 \
    --blocking-timeout-wait-millis=5000 \
    --enabled=true" && \
    echo "=> Adding system properties" && \ 
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.persistence.dialect:add(value=org.hibernate.dialect.MySQLInnoDBDialect)" && \ 
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.persistence.dialect:read-resource" && \ 
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.persistence.ds:add(value=jdbc:mysql://${DB_URL}/${MYSQL_DATABASE})" && \ 
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.persistence.ds:read-resource" && \
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.id:add(value=myserver)" && \ 
    $JBOSS_CLI --connect --command="/system-property=org.kie.server.id:read-resource" && \
    echo "=> Configuring key store" && \ 
    $JBOSS_CLI --connect --command="/core-service=management/security-realm=UndertowRealm:add()" && \
    echo "=> Adding key store" && \
    $JBOSS_CLI --connect --command="/core-service=management/security-realm=UndertowRealm/server-identity=ssl:add(keystore-path=keycloak.jks, keystore-relative-to=jboss.server.config.dir, keystore-password=Cloderia9@2019)" && \
    echo "=> Adding HTTPS Listener" && \
    $JBOSS_CLI --connect --command="/subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=security-realm, value=UndertowRealm)" && \
    echo "=> Enabling Compression" && \
    $JBOSS_CLI --connect --command="/subsystem=undertow/configuration=filter/gzip=gzipfilter:add()" && \
    echo "=> Enabling Proxy Header" && \
    $JBOSS_CLI --connect --command="/subsystem=undertow/configuration=filter/response-header=vary-header:add(header-name="Vary", header-value="Accept-Encoding")" && \
    $JBOSS_CLI --connect --command="/subsystem=undertow/server=default-server/host=default-host/filter-ref=vary-header:add" && \
    echo "=> Shutting down WildFly and Cleaning up" && \
    $JBOSS_CLI --connect --command=":shutdown" && \
    rm -rf $JBOSS_HOME/standalone/configuration/standalone_xml_history/ $JBOSS_HOME/standalone/log/* && \
    rm -f /tmp/*.jar

# Add the web application
ADD com.alstradocs.empicus-empicus-web-1.0-SNAPSHOT.war $DEPLOYMENT_DIR

# Add the templates directory
ADD templates  /enterprise-platform/templates/

# Run the server
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]

MAINTAINER Edward Banfa (ebanfa@gmail.com)
