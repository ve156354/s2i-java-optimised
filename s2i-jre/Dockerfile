FROM  registry.access.redhat.com/rhel7-atomic:7.4-54

MAINTAINER Veerendra K Akula <vakula@atpco.net>

ENV JAVA_HOME="/usr/java/jdk1.8.0_131" \
    JAVA_VENDOR="oracle" \
    JAVA_VERSION="1.8.0" \
    PATH=$PATH:"/usr/local/s2i" \   
    JAVA_DATA_DIR=/deployments/data
	
ENV TZ=America/New_York

# version information
LABEL io.k8s.description="Platform for running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080" \
      org.jboss.deployments-dir="/deployments"


# install Base Packages and JRE
RUN microdnf --nodocs --enablerepo=rhel-7-server-rpms install wget rsync which tar unzip && microdnf clean all  && \

# download oracle JRE
	wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
	http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jre-8u131-linux-x64.rpm  -O /tmp/jre-8u131-linux-x64.rpm && \
	rpm -ivh /tmp/jre-8u131-linux-x64.rpm --nodocs && \   
    rm  /tmp/jre-8u131-linux-x64.rpm && \		
	
    # Add Java User
    groupadd -r java -g 1000 && \
    useradd -u 185 -r -g root -m -d /opt/java -s /sbin/nologin -c "Java user" java && \
    usermod -a -G java java && \
    
    #Timezone fix -- RHEL default is UTC.
    rm -f /etc/localtime && ln -s /usr/share/zoneinfo/$TZ /etc/localtime

COPY run-java.sh run-env.sh debug-options container-limits java-default-options /opt/run-java/
RUN  chmod 755 /opt/run-java/run-java.sh /opt/run-java/run-env.sh /opt/run-java/java-default-options /opt/run-java/container-limits /opt/run-java/debug-options

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
