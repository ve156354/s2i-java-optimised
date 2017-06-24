FROM rhel:7.3

ENV MAVEN_VERSION="3.3.3" \    
    JAVA_HOME="/usr/java/jdk1.8.0_131" \
    JAVA_VENDOR="oracle" \
    JAVA_VERSION="1.8.0" \
    PATH=$PATH:"/usr/local/s2i" \   
    JAVA_DATA_DIR=/deployments/data

# Some version information
LABEL io.k8s.description="Platform for building and running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080" \
      org.jboss.deployments-dir="/deployments"


# Need to install Yum Base Packages, and Java
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
    yum clean all && \
	
	  
	# Install Java
	wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm" -O /tmp/jdk-8-linux-x64.rpm  && \
    yum -y --setopt=tsflags=nodocs install /tmp/jdk-8-linux-x64.rpm  && \
    yum clean all  && \
    rm  /tmp/jdk-8-linux-x64.rpm && \
		
	
    # Add Java User
    groupadd -r java -g 1000 && \
    useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
    usermod -a -G java java && \
    
    # Install Maven
    curl https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz | \
    tar -xzf - -C /opt && \
    ln -s /opt/apache-maven-${MAVEN_VERSION} /opt/maven && \
    ln -s /opt/maven/bin/mvn /usr/bin/mvn && \
         
    echo securerandom.source=file:/dev/urandom >> /usr/java/jdk1.8.0_131/jre/lib/security/java.security

	
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
 
COPY settings.xml /opt/java/.m2/settings.xml

# S2I requires a numeric, non-0 UID. This is the UID for the java user in the base image
USER 185

WORKDIR /opt/java

# Use the run script as default since we are working as an hybrid image which can be
# used directly to. (If we were a plain s2i image we would print the usage info here)
CMD [ "/usr/local/s2i/run" ]
