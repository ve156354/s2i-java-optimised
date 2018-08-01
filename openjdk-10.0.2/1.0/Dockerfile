FROM  registry.access.redhat.com/rhel7-atomic:7.4-131

MAINTAINER pawan k <pkondaveeti@atpco.net>

ENV JAVA_HOME="/usr/jdk-10.0.2" \
    JAVA_VENDOR="openjdk" \
    JAVA_VERSION="10.0.2" \
    PATH=$PATH:"/usr/local/s2i" \
    TZ=America/New_York

LABEL io.k8s.description="Platform for building Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080" \
      org.jboss.deployments-dir="/deployments"

# download required tools
RUN microdnf install wget rsync --enablerepo=rhel-7-server-rpms --nodocs && \
    microdnf update && \
    microdnf clean all && \

    # download openjdk

	wget https://download.java.net/java/GA/jdk10/10.0.2/19aef61b38124481863b1413dce1855f/13/openjdk-10.0.2_linux-x64_bin.tar.gz -O /tmp/openjdk-10.0.2_linux-x64_bin.tar.gz && \
	tar zxf /tmp/openjdk-10.0.2_linux-x64_bin.tar.gz  -C /usr && \ 
	cd /usr && ls -latr && \
    alternatives --install /usr/bin/java java /usr/jdk-10.0.2/bin/java && \
	
    # create java group and add it to user
    groupadd -r java -g 1000 && \
    useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
    usermod -a -G java java && \

    # setting timezone for container
    rm -f /etc/localtime && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \

    echo securerandom.source=file:/dev/urandom >> /usr/jdk-10.0.2/lib/security/java.security

COPY s2i/ /usr/local/s2i/
RUN chmod +x /usr/local/s2i/*

COPY run-java.sh run-env.sh debug-options container-limits java-default-options /opt/run-java/
RUN chmod 755 /opt/run-java/run-java.sh /opt/run-java/run-env.sh /opt/run-java/java-default-options /opt/run-java/container-limits /opt/run-java/debug-options

# create deployments dir to host jars and change owner to java
RUN mkdir -p /deployments && \
    chmod -R "g+rwX" /deployments && \
    chown -R java:root /deployments


USER 185
WORKDIR /opt/java
EXPOSE 8080
CMD ["/usr/local/s2i/run"]