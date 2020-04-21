FROM jboss/wildfly

ENV WILDFLY_USER writing-platform-admin
ENV WILDFLY_PASS Andromeda89634893nxuj88638@7836
ENV JBOSS_HOME /opt/jboss/wildfly
ENV JBOSS_CLI /opt/jboss/wildfly/bin/jboss-cli.sh
ENV DEPLOYMENT_DIR /opt/jboss/wildfly/standalone/deployments/

# Add the admin user to the server
RUN $JBOSS_HOME/bin/add-user.sh -u $WILDFLY_USER -p $WILDFLY_PASS --silent

# Add key store
ADD alstracoin_io/keycloak.jks /opt/jboss/wildfly/standalone/configuration/

# Configure Wildfly server
RUN echo "=> Starting WildFly server" && \
    bash -c '$JBOSS_HOME/bin/standalone.sh &' && \
    echo "=> Waiting for the server to boot" && \
    bash -c 'until `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null | grep -q running`; do echo `$JBOSS_CLI -c ":read-attribute(name=server-state)" 2> /dev/null`; sleep 1; done' && \
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
ADD admin-portal-1.0-SNAPSHOT.war $DEPLOYMENT_DIR

# Run the server
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]

MAINTAINER Edward Banfa (ebanfa@gmail.com)
