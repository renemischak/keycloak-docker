FROM jboss/base-jdk:8

ENV KEYCLOAK_VERSION 4.0.0.Final
# Enables signals getting passed from startup script to JVM
# ensuring clean shutdown when container is stopped.
ENV LAUNCH_JBOSS_IN_BACKGROUND 1
ENV PROXY_ADDRESS_FORWARDING false
USER root

RUN yum install -y epel-release && yum install -y jq && yum install -y git && yum clean all

USER jboss

RUN cd /opt/jboss && curl -s http://apache.uib.no/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz | tar xz && mv apache-maven-3.5.3 /opt/jboss/maven

RUN git clone https://github.com/stianst/keycloak.git -b openshift-integration /opt/jboss/keycloak-clone
RUN cd /opt/jboss/keycloak-clone && export M2_HOME=/opt/jboss/maven && /opt/jboss/maven/bin/mvn clean install -DskipTests -Pdistribution

ENV UPDATE=3

RUN cd /opt/jboss/keycloak-clone && git fetch origin && git reset --hard origin/openshift-integration
RUN cd /opt/jboss/keycloak-clone && export M2_HOME=/opt/jboss/maven && /opt/jboss/maven/bin/mvn clean install -DskipTests -Pdistribution
RUN cd /opt/jboss && tar xfz /opt/jboss/keycloak-clone/distribution/server-dist/target/keycloak-*.tar.gz
RUN rm -rf /opt/jboss/keycloak && mv /opt/jboss/keycloak-?.?.?.Final-SNAPSHOT /opt/jboss/keycloak

ADD docker-entrypoint.sh /opt/jboss/

ADD cli /opt/jboss/keycloak/cli
RUN cd /opt/jboss/keycloak && bin/jboss-cli.sh --file=cli/standalone-configuration.cli && rm -rf /opt/jboss/keycloak/standalone/configuration/standalone_xml_history
RUN cd /opt/jboss/keycloak && bin/jboss-cli.sh --file=cli/standalone-ha-configuration.cli && rm -rf /opt/jboss/keycloak/standalone/configuration/standalone_xml_history

ENV JDBC_POSTGRES_VERSION 42.2.2
ENV JDBC_MYSQL_VERSION 5.1.46
ENV JDBC_MARIADB_VERSION 2.2.3

ADD databases/change-database.sh /opt/jboss/keycloak/bin/change-database.sh

RUN mkdir -p /opt/jboss/keycloak/modules/system/layers/base/com/mysql/jdbc/main; cd /opt/jboss/keycloak/modules/system/layers/base/com/mysql/jdbc/main && curl -O http://central.maven.org/maven2/mysql/mysql-connector-java/$JDBC_MYSQL_VERSION/mysql-connector-java-$JDBC_MYSQL_VERSION.jar
ADD databases/mysql/module.xml /opt/jboss/keycloak/modules/system/layers/base/com/mysql/jdbc/main/module.xml

RUN mkdir -p /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main; cd /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main; curl -L http://central.maven.org/maven2/org/postgresql/postgresql/$JDBC_POSTGRES_VERSION/postgresql-$JDBC_POSTGRES_VERSION.jar > postgres-jdbc.jar
ADD databases/postgres/module.xml /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main

RUN mkdir -p /opt/jboss/keycloak/modules/system/layers/base/org/mariadb/jdbc/main; cd /opt/jboss/keycloak/modules/system/layers/base/org/mariadb/jdbc/main; curl -L http://central.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/$JDBC_MARIADB_VERSION/mariadb-java-client-$JDBC_MARIADB_VERSION.jar > mariadb-jdbc.jar
ADD databases/mariadb/module.xml /opt/jboss/keycloak/modules/system/layers/base/org/mariadb/jdbc/main

ENV JBOSS_HOME /opt/jboss/keycloak
ENV LANG en_US.UTF-8

EXPOSE 8080

ENTRYPOINT [ "/opt/jboss/docker-entrypoint.sh" ]

CMD ["-b", "0.0.0.0"]
