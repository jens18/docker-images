FROM oraclelinux:7
MAINTAINER Jens Kaemmerer <jens.kaemmerer@oracle.com>

### set the root password
RUN echo "root:cgbu-dev" | chpasswd

### setup Yum HTTP proxy
#RUN echo "proxy=http://www-proxy.us.oracle.com:80" >> /etc/yum.conf

### setup curl HTTP proxy
#ENV http_proxy http://www-proxy.us.oracle.com:80

### install Oracle Linux dependencies 
# tar/unzip - install JDK, Coherence
# emacs - to create
# minimal set of X11 libs - to run graphical applications such as jconsole, xterm
RUN yum -y install tar unzip emacs-nox libXrender libXext libXtst xterm

### set DISPLAY variable:
ENV DISPLAY :0

### create regular user account:
RUN groupadd cgbu-dev && useradd -g cgbu-dev cgbu-dev
RUN echo "cgbu-dev:cgbu-dev" | chpasswd

# create directory for additional software packages
RUN mkdir /home/cgbu-dev/opt
WORKDIR /home/cgbu-dev/opt/

### install Java 8
COPY jdk-8u65-linux-x64.tar.gz /home/cgbu-dev/opt/
RUN cd /home/cgbu-dev/opt; tar -zxf jdk-8u65-linux-x64.tar.gz
# set Java environment variables
ENV JAVA_HOME /home/cgbu-dev/opt/jdk1.8.0_65
ENV PATH $JAVA_HOME/bin:$PATH
# test
RUN java -version

### install Maven 3.3.3
RUN cd /home/cgbu-dev/opt && curl -O http://apache.mirrors.tds.net/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.zip && unzip -q apache-maven-3.3.3-bin.zip && rm apache-maven-3.3.3-bin.zip
# configure Maven
RUN mkdir /home/cgbu-dev/.m2
COPY settings.xml /home/cgbu-dev/.m2/
# set Maven environment variables
ENV M2_HOME /home/cgbu-dev/opt/apache-maven-3.3.3
ENV PATH /home/cgbu-dev/opt/apache-maven-3.3.3/bin:$PATH
# test
RUN mvn -version

### install Weblogic + Coherence 12.2.1
COPY fmw_12.2.1.0.0_wls_Disk1_1of1.zip /home/cgbu-dev/opt/ 
RUN unzip fmw_12.2.1.0.0_wls_Disk1_1of1.zip
COPY wl_coh_12.2.1.rsp /home/cgbu-dev/opt/
COPY oraInst.loc /home/cgbu-dev/opt/

# change permissions for all files in the cgbu-dev home directory 
# (all files are still owned by root)
RUN chown -R cgbu-dev:cgbu-dev /home/cgbu-dev

# start the Docker container with this user
USER cgbu-dev 
# run OUI installer
RUN java -jar fmw_12.2.1.0.0_wls.jar -silent -responseFile \
/home/cgbu-dev/opt/wl_coh_12.2.1.rsp -invPtrLoc /home/cgbu-dev/opt/oraInst.loc

# set Coherence environment variables
ENV COHERENCE_HOME=/home/cgbu-dev/opt/wlcoh_12.2.1/coherence

### configure jVisualVM plugins: Visual GC, VisualVM-JConsole, VisualVM-MBeans, Coherence-JVisualVM
RUN cd ${JAVA_HOME} && \
mkdir -p lib/visualvm/visualvm/update/download && \
cd lib/visualvm/visualvm/update/download && \
curl -O https://java.net/downloads/visualvm/release136/com-sun-tools-visualvm-modules-visualgc_1.nbm && \
curl -O https://java.net/downloads/visualvm/release136/com-sun-tools-visualvm-modules-jconsole.nbm && \
curl -O https://java.net/downloads/visualvm/release136/com-sun-tools-visualvm-modules-mbeans_1.nbm 
RUN cp ${COHERENCE_HOME}/plugins/jvisualvm/coherence-jvisualvm.nbm ${JAVA_HOME}/lib/visualvm/visualvm/update/download/

### compile Coherence 12.2.1 examples

# federated caching 
RUN cd ${COHERENCE_HOME}/examples/java; bin/build federation
# REST
RUN mvn install:install-file \
-Dfile=${COHERENCE_HOME}/lib/coherence.jar \
-DpomFile=${COHERENCE_HOME}/plugins/maven/com/oracle/coherence/coherence/12.2.1/coherence.12.2.1.pom
RUN mvn install:install-file \
-Dfile=${COHERENCE_HOME}/lib/coherence-rest.jar \
-DpomFile=${COHERENCE_HOME}/plugins/maven/com/oracle/coherence/coherence-rest/12.2.1/coherence-rest.12.2.1.pom
RUN cd ${COHERENCE_HOME}/examples/rest; mvn clean compile
RUN echo "cd ${COHERENCE_HOME}/examples/rest; mvn exec:exec -DhttpProxy" \
> /home/cgbu-dev/run_rest_example.sh
RUN chmod 755 /home/cgbu-dev/run_rest_example.sh

# remove software package files
RUN rm /home/cgbu-dev/opt/fmw_12.2.1.0.0_wls.jar \
/home/cgbu-dev/opt/fmw_12.2.1.0.0_wls_Disk1_1of1.zip \
/home/cgbu-dev/opt/wl_coh_12.2.1.rsp \
/home/cgbu-dev/opt/jdk-8u65-linux-x64.tar.gz

