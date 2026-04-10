# ===========================================================
# Default tags
# ===========================================================
ARG LR_IMAGE_TAG="2026.q1.2-lts"
ARG UBI9_TAG="9.4"

FROM liferay/dxp:${LR_IMAGE_TAG} AS liferay

FROM redhat/ubi9:${UBI9_TAG}

# ===========================================================
# Args
# ===========================================================
ARG JBOSS_EAP_ZIP_FILE_PATH=./files/jboss/jboss-eap-8.1.0.zip

# ===========================================================
# Install dependencies
# ===========================================================
RUN dnf -y install --allowerasing \
    unzip \
    file \
    curl \
    fontconfig \
    java-21-openjdk \
    && dnf clean all && rm -rf /var/cache/dnf

# UPDATED: The correct path for OpenJDK 21 on UBI 9
ENV JAVA_HOME=/usr/lib/jvm/jre-21-openjdk \
    PATH=/usr/lib/jvm/jre-21-openjdk/bin:$PATH

ENV JAVA_FONTS=/usr/share/fonts

# ===========================================================
# JBoss setup
# ===========================================================
ENV JBOSS_HOME="/opt/liferay/jboss-eap"

COPY ${JBOSS_EAP_ZIP_FILE_PATH} /tmp/jboss.zip

RUN unzip /tmp/jboss.zip -d /tmp && \
    mkdir -p /opt/liferay && \
    # The wildcard * handles 'jboss-eap-8.1' or 'jboss-eap-8.1.0'
    mv /tmp/jboss-eap-* ${JBOSS_HOME} && \
    rm /tmp/jboss.zip

# ===========================================================
# Liferay env
# ===========================================================
ENV JPDA_ADDRESS=0.0.0.0:8000
ENV LIFERAY_HOME="/opt/liferay"
ENV LIFERAY_PID="${LIFERAY_HOME}/liferay.pid"
ENV LIFERAY_MOUNT_DIR="/mnt/liferay"

# ===========================================================
# Copy Liferay (same approach as before)
# ===========================================================
COPY --from=liferay /opt/liferay/osgi ${LIFERAY_HOME}/osgi
COPY --from=liferay /opt/liferay/tomcat/webapps/ROOT ${JBOSS_HOME}/standalone/deployments/ROOT.war
COPY --from=liferay /usr/local/bin/*.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/*.sh

# ===========================================================
# Fake tree (keep your workaround)
# ===========================================================
RUN echo '#!/bin/bash' > /usr/bin/tree && chmod +x /usr/bin/tree

# ===========================================================
# PostgreSQL driver (FIXED)
# ===========================================================
RUN mkdir -p ${JBOSS_HOME}/modules/org/postgresql/main && \
    curl -L -o ${JBOSS_HOME}/modules/org/postgresql/main/postgresql.jar \
    https://jdbc.postgresql.org/download/postgresql-42.7.3.jar

RUN cat <<'EOF' > ${JBOSS_HOME}/modules/org/postgresql/main/module.xml
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.3" name="org.postgresql">
  <resources>
    <resource-root path="postgresql.jar"/>
  </resources>
  <dependencies>
    <module name="jakarta.api"/>
    <module name="jakarta.transaction.api"/>
  </dependencies>
</module>
EOF

# ===========================================================
# Configure JBoss (UPDATED for EAP 8)
# ===========================================================
RUN TMP="${JBOSS_HOME}/standalone/configuration/standalone.xml" && \
    sed -i 's/<jsp-config\/>/<jsp-config development="true" source-vm="21" target-vm="21" \/>/' "$TMP" && \
    sed -i '/<\/extensions>/a \
<system-properties>\
<property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8"/>\
<property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true"/>\
</system-properties>' "$TMP" && \
    sed -i 's/<deployment-scanner/<deployment-scanner deployment-timeout="600"/' "$TMP" && \
    sed -i '/<location name="\/" handler="welcome-content"\/>/d' "$TMP"

# ===========================================================
# JVM config (Java 21 safe)
# ===========================================================
RUN echo '\nJAVA_OPTS="$JAVA_OPTS \
-Dfile.encoding=UTF-8 \
-Djava.locale.providers=COMPAT,CLDR \
--add-opens=java.base/java.lang=ALL-UNNAMED \
--add-opens=java.base/java.util=ALL-UNNAMED \
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED \
-Djava.net.preferIPv4Stack=true \
-Xms4g -Xmx4g"' >> "${JBOSS_HOME}/bin/standalone.conf"

RUN touch "${JBOSS_HOME}/standalone/deployments/ROOT.war.dodeploy"

# ===========================================================
# Adapt Liferay startup (same logic, fixed)
# ===========================================================
RUN sed -i 's|exec "${LIFERAY_HOME}"/tomcat/bin/catalina.sh.*run|'"${JBOSS_HOME}"'/bin/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0|' \
    /usr/local/bin/start_liferay.sh && \
    mkdir -p ${LIFERAY_HOME}/deploy /mnt/liferay && \
    rm -rf ${LIFERAY_HOME}/osgi/state

# Debug port
RUN echo '\nif [ "$LIFERAY_JPDA_ENABLED" = "true" ]; then \
JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,address=${JPDA_ADDRESS},server=y,suspend=n"; fi' \
>> ${JBOSS_HOME}/bin/standalone.conf

# Admin user
RUN ${JBOSS_HOME}/bin/add-user.sh test test --silent

# ===========================================================
# Start
# ===========================================================
CMD ["/usr/local/bin/liferay_entrypoint.sh"]