# ==============================================================================================================

# This Dockerfile is used to build a Docker image for Liferay DXP with JBoss EAP.
# By default this Dockerfile uses Liferay DXP 2024.Q4.2, Red Hat UBI 7.9 images and Java 21.
# In order to change the Java version. 
# Search for 'zulu21-jdk' and change 21. e.g. for Java 17, use zulu17-jdk.
# Search for 'source-vm=' and 'target-vm='. Change the values to the desired Java version.
# e.g. for Java 17, use source-vm="17" target-vm="17".

# Compatibility with Java JDK 8 runtime has been removed in 2024.Q2 and later releases.
# Java JDK 11 runtime was deprecated in 2024.Q1 and removed in 2024.Q3 and later releases.
# Java JDK 17 and JDK 21 runtimes are certified on 2024.Q2 or later. JDK 21 is the recommended runtime.
# https://help.liferay.com/hc/en-us/articles/4411310034829-Liferay-DXP-Quarterly-Releases-Compatibility-Matrix

# ==============================================================================================================

# Default Liferay and Red Hat UBI image tags
ARG LR_IMAGE_TAG="2024.q4.2"
ARG REDHAT_UBI_IMAGE_TAG="7.9"

# Prepare Liferay files to be extracted to UBI image
FROM liferay/dxp:${LR_IMAGE_TAG} AS liferay

# Prepare UBI image with JBoss + Liferay
FROM registry.access.redhat.com/ubi7/ubi:${REDHAT_UBI_IMAGE_TAG}

# Download JBoss EAP 7.4.0 from https://files.liferay.com/private/apps/redhat/jbosseap/7/
# and place it in the files/jboss folder.
# Argument used for JBoss installation files
ARG JBOSS_EAP_ZIP_FILE_PATH=./files/jboss/jboss-eap-7.4.0.zip

# Install Java package needed by JBoss EAP
RUN yum -y install https://cdn.azul.com/zulu/bin/zulu-repo-1.0.0-1.noarch.rpm && \
    yum -y install unzip file zulu21-jdk && \
    yum -y clean all && \
    rm -rf /var/cache

# Install Fontconfig and Fonts
RUN yum install -y fontconfig \
    liberation-fonts \
    && yum clean all

# Set JAVA_FONTS environment variable
ENV JAVA_FONTS=/usr/share/fonts

# JBoss env variables
ENV JBOSS_HOME="/opt/liferay/jboss-eap"

# Extract JBoss EAP
COPY ${JBOSS_EAP_ZIP_FILE_PATH} /tmp/jboss-eap.zip
RUN unzip /tmp/jboss-eap.zip -d /tmp && \
    rm /tmp/jboss-eap.zip && \
    mkdir -p /opt/liferay && \
    mv /tmp/jboss-eap* /opt/liferay/jboss-eap

# Liferay env variables
ENV JPDA_ADDRESS=0.0.0.0:8000
ENV LIFERAY_HOME="/opt/liferay"
ENV LIFERAY_PID="${LIFERAY_HOME}/liferay.pid"
ENV LIFERAY_MOUNT_DIR="/mnt/liferay"

# Copy Liferay bundle files + Liferay scripts to JBoss
COPY --from=liferay /opt/liferay/osgi ${LIFERAY_HOME}/osgi
COPY --from=liferay /opt/liferay/tomcat/webapps/ROOT ${JBOSS_HOME}/standalone/deployments/ROOT.war
COPY --from=liferay /usr/local/bin/*.sh /usr/local/bin/

# Emulate tree command (since installing tree looks complicated when using UBI Docker image)
# (tree is a command used by Liferay scripts)
RUN TMP_FILE_PATH="/usr/local/bin/configure_liferay.sh" && \
    TMP_CMD="find \"\${LIFERAY_MOUNT_DIR}\"/files | awk -F'/' '{ \
        if(NR==1) { \
            first=NF; \
            print \
        } else { \
            line=\"|\"; \
            for(i=0; i< NF-first-1; i++) { \
                line=line\"   |\" \
            }; \
            line=line\"-- \"; \
            print line\$NF \
        } \
    }'" && \
    awk -v pattern="^[\t]+tree " -v cmd="${TMP_CMD}" '{ \
        if($0 ~ pattern) { \
            print "\t\t\t" cmd \
        } else { \
            print \
        } \
    }' "${TMP_FILE_PATH}" > "${TMP_FILE_PATH}.tmp" && \
    mv "${TMP_FILE_PATH}.tmp" "${TMP_FILE_PATH}" && \
    unset TMP_CMD TMP_FILE_PATH

RUN echo '#!/bin/bash' > /usr/bin/tree && \
    chmod +x /usr/bin/tree

# Add mysql.jar to JBoss modules
RUN if [ -d "${JBOSS_HOME}/standalone/deployments/ROOT.war/WEB-INF/shielded-container-lib" ]; then \
        TMP_MYSQL_JAR_LINK="$( \
            unzip -c "${JBOSS_HOME}/standalone/deployments/ROOT.war/WEB-INF/shielded-container-lib/portal-impl.jar" portal.properties | \
            awk '/=https.*mysql.*\.jar/{split($0,a,"="); print a[2]; exit}' \
        )" && \
        if [ -n "${TMP_MYSQL_JAR_LINK}" ]; then \
            curl -o "${JBOSS_HOME}/standalone/deployments/ROOT.war/WEB-INF/shielded-container-lib/mysql.jar" "${TMP_MYSQL_JAR_LINK}"; \
        fi && \
        unset TMP_MYSQL_JAR_LINK; \
    else \
        TMP_MYSQL_JAR_LINK="$( \
            unzip -c "${JBOSS_HOME}/standalone/deployments/ROOT.war/WEB-INF/lib/portal-impl.jar" portal.properties | \
            awk '/=https.*mysql.*\.jar/{split($0,a,"="); print a[2]; exit}' \
        )" && \
        if [ -n "${TMP_MYSQL_JAR_LINK}" ]; then \
            curl -o "${JBOSS_HOME}/modules/com/liferay/portal/main/mysql.jar" "${TMP_MYSQL_JAR_LINK}"; \
        fi && \
        unset TMP_MYSQL_JAR_LINK; \
    fi

# Configuring standalone.xml
RUN TMP_CONFIG_PATH="${JBOSS_HOME}/standalone/configuration/standalone.xml" && \
    sed -i 's/<jsp-config\/>/<jsp-config development="true" source-vm="21" target-vm="21" \/>/' "${TMP_CONFIG_PATH}" && \
    sed -i '/<\/extensions>/a \
        <system-properties>\n\
            <property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8" \/>\n\
            <property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true" \/>\n\
        <\/system-properties>' "${TMP_CONFIG_PATH}" && \
    sed -i '/<\/console-handler>/i \
        <filter-spec value="not(any(match(\&quot;WFLYSRV0059\&quot;),match(\&quot;WFLYEE0007\&quot;)))" \/>' "${TMP_CONFIG_PATH}" && \
    sed -i 's/<deployment-scanner/<deployment-scanner deployment-timeout="600"/' "${TMP_CONFIG_PATH}" && \
    sed -i '/<subsystem xmlns="urn:jboss:domain:security:2.0">/!b;n;/<security-domains>/n;i \
        <security-domain name="PortalRealm">\n\
            <authentication>\n\
                <login-module code="com.liferay.portal.security.jaas.PortalLoginModule" flag="required" \/>\n\
            <\/authentication>\n\
        </security-domain>' "${TMP_CONFIG_PATH}" && \
    sed -i '/<location name="\/" handler="welcome-content"\/>/d' "${TMP_CONFIG_PATH}" && \
    sed -i '/<file name="welcome-content" path="${jboss.home.dir}\/welcome-content"\/>/d' "${TMP_CONFIG_PATH}" && \
    unset TMP_CONFIG_PATH

# Configuring JVM standalone.conf
RUN echo -e '\nJAVA_OPTS="$JAVA_OPTS \
    -Dfile.encoding=UTF-8 \
    -Djava.locale.providers=JRE,COMPAT,CLDR \
    --add-opens=java.base/sun.util.calendar=ALL-UNNAMED \
    --add-opens=jdk.zipfs/jdk.nio.zipfs=ALL-UNNAMED \
    --add-opens=java.base/java.lang=ALL-UNNAMED \
    --add-opens=java.base/java.net=ALL-UNNAMED \
    --add-opens java.base/java.util=ALL-UNNAMED \
    --add-opens=java.base/java.lang.invoke=ALL-UNNAMED \
    --add-opens=java.base/sun.net.www.protocol.http=ALL-UNNAMED \
    --add-opens=java.base/java.lang.reflect=ALL-UNNAMED \
    -Djava.net.preferIPv4Stack=true \
    -Dlog4j2.formatMsgNoLookups=true \
    -Duser.timezone=GMT \
    -Xms4096m \
    -Xmx4096m \
    -XX:MaxNewSize=1536m \
    -XX:MaxMetaspaceSize=768m \
    -XX:MetaspaceSize=768m \
    -XX:NewSize=1536m \
    -XX:SurvivorRatio=7"' >> "${JBOSS_HOME}/bin/standalone.conf"

RUN touch "${JBOSS_HOME}/standalone/deployments/ROOT.war.dodeploy"

RUN TMP_CONFIG_PATH="${JBOSS_HOME}/standalone/configuration/standalone.xml" && \
    sed -i '/<\/system-properties>/i \        <property name="jboss.as.management.blocking.timeout" value="600"/>' "${TMP_CONFIG_PATH}" && \
    unset TMP_CONFIG_PATH

# Adapt Liferay scripts to run with JBoss instead of Tomcat
RUN echo -e '\nif [ "${LIFERAY_JPDA_ENABLED}" == "true" ]; then' >> "${JBOSS_HOME}/bin/standalone.conf" && \
    echo -e '   JAVA_OPTS="$JAVA_OPTS -agentlib:jdwp=transport=dt_socket,address="${JPDA_ADDRESS}",server=y,suspend=n"' >> "${JBOSS_HOME}/bin/standalone.conf" && \
    echo -e 'fi' >> "${JBOSS_HOME}/bin/standalone.conf" && \
    sed -i 's/exec \"${LIFERAY_HOME}\"\/tomcat\/bin\/catalina.sh.*run/${JBOSS_HOME}\/bin\/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0/' /usr/local/bin/start_liferay.sh && \
    mkdir -p ${LIFERAY_HOME}/deploy /mnt/liferay && \
    rm -rf ${LIFERAY_HOME}/osgi/state

# Add admin user (test/test) for JBoss, can be used to configure JBoss via Web UI
# On https://localhost:9990
RUN ${JBOSS_HOME}/bin/add-user.sh test test --silent 2> /dev/null

# Override JBoss CMD with Liferay entrypoint CMD
CMD /usr/local/bin/liferay_entrypoint.sh
