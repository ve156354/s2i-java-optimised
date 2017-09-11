FROM FROM alpine:3.6

ENV JAVA_VERSION=8 \
    JAVA_UPDATE=131 \
    JAVA_BUILD=11 \
    JAVA_URL_HASH=d54c1d3a095b4ff2b6607d096fa80163 \
    JAVA_SHA256_CHECKSUM=355e5cdb066d4cada1f9f16f358b6fa6280ff5caf7470cf0d5cdd43083408d35

# version information
LABEL io.k8s.description="Platform for running plain Java applications" \
      io.k8s.display-name="Java Applications" \
      io.openshift.tags="builder,java" \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i" \
      io.openshift.s2i.destination="/tmp" \
      io.openshift.expose-services="8080" \
      org.jboss.deployments-dir="/deployments"
	  

ENV JAVA_HOME="/usr/java/jdk1.8.0_131"

RUN apk --no-cache add --virtual=build-dependencies wget rsync which tar unzip ca-certificates \

 && wget http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION}u${JAVA_UPDATE}-b${JAVA_BUILD}/${JAVA_URL_HASH}/jre-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz \
         --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
         --directory-prefix=/tmp/ \
         --quiet \

 && echo "${JAVA_SHA256_CHECKSUM}  /tmp/jre-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz" | sha256sum -c - \

 && mkdir -p /usr/java/jdk1.8.0_131 \

 && tar -xzf /tmp/jre-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz \
        -C /usr/java/jdk1.8.0_131 && \
		
	mkdir -p /opt/java && \
    addgroup  java -g 1000 && \
	adduser -u 185  -D -S -G  root  java java -h /opt/java && \
	
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