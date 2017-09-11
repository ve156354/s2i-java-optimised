FROM rhel:7.4

MAINTAINER Veerendra K Akula <vakula@atpco.net>

ENV JAVA_HOME="/usr/java/jdk1.8.0_144" \
    JAVA_VENDOR="oracle" \
    JAVA_VERSION="1.8.0" \
    PATH=$PATH:"/usr/local/s2i" \   
    JAVA_DATA_DIR=/deployments/data
	
ENV TZ=America/New_York

# Some version information
LABEL io.k8s.description="Platform for running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080" \
      org.jboss.deployments-dir="/deployments"


# Need to install Yum Base Packages, and JRE
RUN yum repolist > /dev/null && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    yum clean all && \
    INSTALL_PKGS="tar \
    unzip \
    wget \
    which \
    rsync" && \   
    yum install -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum --enablerepo=\* clean all && \
			
    # Install Java
    wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jre-8u144-linux-x64.rpm" \
	      -O /tmp/jre-8u144-linux-x64.rpm  && \
    yum -y --setopt=tsflags=nodocs install /tmp/jre-8u144-linux-x64.rpm  && \
    yum --enablerepo=\* clean all  && \
    rm  /tmp/jre-8u144-linux-x64.rpm && \		
	
    # Add Java User
    groupadd -r java -g 1000 && \
    useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
    usermod -a -G java java && \
    
    #Timezone fix -- RHEL default is UTC.
    rm -f /etc/localtime && ln -s /usr/share/zoneinfo/$TZ /etc/localtime

RUN  rm -rf /var/cache/yum 
	
COPY run-java.sh run-env.sh debug-options container-limits java-default-options /opt/run-java/
RUN chmod 755 /opt/run-java/run-java.sh /opt/run-java/run-env.sh /opt/run-java/java-default-options /opt/run-java/container-limits /opt/run-java/debug-options

# Expose Ports
EXPOSE 8080

# S2I scripts + README
COPY s2i /usr/local/s2i
RUN chmod 755 /usr/local/s2i/*
ADD README.md /usr/local/s2i/usage.txt

RUN mkdir -p /deployments/data && \
 chmod -R "g+rwX" /deployments && \
 chown -R java:root /deployments

# S2I requires a numeric, non-0 UID. This is the UID for the java user in the base image
USER 185

WORKDIR /opt/java

# Use the run script as default 
CMD [ "/usr/local/s2i/run" ]
