FROM jboss/wildfly

ARG DB_URL
ARG MYSQL_DATABASE
ARG MYSQL_USER
ARG MYSQL_PASSWORD
ARG PLATFORM_DS_NAME
ARG JBPM_MYSQL_DATABASE
ARG JBPM_MYSQL_USER
ARG JBPM_MYSQL_PASSWORD
ARG JBPM_DS_NAME
ARG MYSQL_CONNECTOR
ARG MYSQL_VERSION
ARG WILDFLY_USER
ARG WILDFLY_PASSWORD
ARG WILDFLY_HOME
ARG WILDFLY_CLI
ARG WILDFLY_DEPLOYMENT_DIR

# Add the mysql connector
ADD $MYSQL_CONNECTOR /opt/

RUN env
# Add the admin user to the server
RUN $WILDFLY_HOME/bin/add-user.sh -u $WILDFLY_USER -p $WILDFLY_PASSWORD --silent

ADD alstracoin_io/keycloak.jks /opt/jboss/wildfly/standalone/configuration/
# ADD standalone.xml /opt/jboss/wildfly/standalone/configuration/
# Configure Wildfly server
RUN echo "=> Starting WildFly server" && \
    bash -c '$WILDFLY_HOME/bin/standalone.sh &' && \
    echo "=> Waiting for the server to boot" && \
    bash -c 'until `$WILDFLY_CLI -c ":read-attribute(name=server-state)" 2> /dev/null | grep -q running`; do echo `$WILDFLY_CLI -c ":read-attribute(name=server-state)" 2> /dev/null`; sleep 1; done' && \
    echo "=> Adding MySQL module" && \
    $WILDFLY_CLI --connect --command="module add --name=com.mysql --resources=/opt/$MYSQL_CONNECTOR --dependencies=javax.api,javax.transaction.api" && \
    echo "=> Adding MySQL driver" && \
    $WILDFLY_CLI --connect --command="/subsystem=datasources/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql,driver-class-name=com.mysql.cj.jdbc.Driver)" && \
    echo "=> Creating a new JBPM datasource" && \
    $WILDFLY_CLI --connect --command="data-source add \
    --name=${JBPM_DS_NAME} \
    --jndi-name=java:jboss/datasources/${JBPM_DS_NAME} \
    --user-name=${JBPM_MYSQL_USER} \
    --password=${JBPM_MYSQL_PASSWORD} \
    --driver-name=mysql \
    --connection-url=jdbc:mysql://${DB_URL}/${JBPM_MYSQL_DATABASE} \
    --use-ccm=false \
    --max-pool-size=25 \
    --blocking-timeout-wait-millis=5000 \
    --enabled=true" && \
    echo "=> Creating a new WP datasource" && \
    $WILDFLY_CLI --connect --command="data-source add \
    --name=${PLATFORM_DS_NAME} \
    --jndi-name=java:jboss/datasources/${PLATFORM_DS_NAME} \
    --user-name=${MYSQL_USER} \
    --password=${MYSQL_PASSWORD} \
    --driver-name=mysql \
    --connection-url=jdbc:mysql://${DB_URL}/${MYSQL_DATABASE} \
    --use-ccm=false \
    --max-pool-size=25 \
    --blocking-timeout-wait-millis=5000 \
    --enabled=true" && \
    echo "=> Adding system properties" && \ 
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.persistence.dialect:add(value=org.hibernate.dialect.MySQLInnoDBDialect)" && \ 
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.persistence.dialect:read-resource" && \ 
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.persistence.ds:add(value=jdbc:mysql://${DB_URL}/${JBPM_MYSQL_DATABASE})" && \ 
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.persistence.ds:read-resource" && \
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.id:add(value=myserver)" && \ 
    $WILDFLY_CLI --connect --command="/system-property=org.kie.server.id:read-resource" && \
    echo "=> Configuring key store" && \ 
    $WILDFLY_CLI --connect --command="/core-service=management/security-realm=UndertowRealm:add()" && \
    echo "=> Adding key store" && \
    $WILDFLY_CLI --connect --command="/core-service=management/security-realm=UndertowRealm/server-identity=ssl:add(keystore-path=keycloak.jks, keystore-relative-to=jboss.server.config.dir, keystore-password=Cloderia9@2019)" && \
    echo "=> Adding HTTPS Listener" && \
    $WILDFLY_CLI --connect --command="/subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=security-realm, value=UndertowRealm)" && \
    echo "=> Enabling Compression" && \
    $WILDFLY_CLI --connect --command="/subsystem=undertow/configuration=filter/gzip=gzipfilter:add()" && \
    echo "=> Enabling Proxy Header" && \
    $WILDFLY_CLI --connect --command="/subsystem=undertow/configuration=filter/response-header=vary-header:add(header-name="Vary", header-value="Accept-Encoding")" && \
    $WILDFLY_CLI --connect --command="/subsystem=undertow/server=default-server/host=default-host/filter-ref=vary-header:add" && \
    echo "=> Shutting down WildFly and Cleaning up" && \
    $WILDFLY_CLI --connect --command=":shutdown" && \
    rm -rf $WILDFLY_HOME/standalone/configuration/standalone_xml_history/ $WILDFLY_HOME/standalone/log/* && \
    rm -f /tmp/*.jar

# Add the web application
ADD com.alstradocs.empicus-empicus-web-1.0-SNAPSHOT.war $WILDFLY_DEPLOYMENT_DIR

# Add the templates directory
ADD templates  /enterprise-platform/templates/

# Run the server
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]

MAINTAINER Edward Banfa (ebanfa@gmail.com)
