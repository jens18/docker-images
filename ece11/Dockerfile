FROM java8_coh12
MAINTAINER Jens Kaemmerer <jens.kaemmerer@oracle.com>

### setup curl HTTP proxy
ENV http_proxy http://www-proxy.us.oracle.com:80
ENV https_proxy http://www-proxy.us.oracle.com:80

### install Oracle Linux dependencies (already present in OL7.1 container) 
# which - required by ecc
# SSH server
USER root
# RUN yum -y install which openssh-server

### configure SSH server
RUN mkdir /var/run/sshd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22

USER cgbu-dev

### install Groovy 2.3.9
RUN cd /home/cgbu-dev/opt/ && \
curl -O https://repo1.maven.org/maven2/org/codehaus/groovy/groovy-binary/2.3.9/groovy-binary-2.3.9.zip && \
unzip groovy-binary-2.3.9.zip && \
rm /home/cgbu-dev/opt/groovy-binary-2.3.9.zip
ENV PATH /home/cgbu-dev/opt/groovy-2.3.9/bin:$PATH

### install ECE 11.2.0.8
RUN mkdir /home/cgbu-dev/opt/ece_11.2.0.8
COPY infrastructure.zip /home/cgbu-dev/opt/ece_11.2.0.8/
RUN cd /home/cgbu-dev/opt/ece_11.2.0.8 && \
unzip infrastructure.zip && \
chmod 755 bin/ecc && \
rm infrastructure.zip
# inplace update of 'rootDir' parameter
RUN cd /home/cgbu-dev/opt/ece_11.2.0.8/config && sed -i 's/..\/..\/target/\/home\/cgbu-dev\/opt\/ece_11.2.0.8/g' ece.properties

### install ECE 11.2.0.8 SDK
RUN mkdir /home/cgbu-dev/opt/ece_11.2.0.8/sdk
COPY sdk.zip /home/cgbu-dev/opt/ece_11.2.0.8/sdk/
RUN cd /home/cgbu-dev/opt/ece_11.2.0.8/sdk && \
unzip sdk.zip && \
rm sdk.zip









