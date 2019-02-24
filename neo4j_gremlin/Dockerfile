# Neo4j Server
# Repository http://github.com/jedivind/docker-neo4j
FROM java:openjdk-7-jdk

MAINTAINER Vinay Bharadwaj <vbharadwaj@milcord.com>

ENV PATH $PATH:/var/lib/neo4j/bin

ENV NEO4J_VERSION 2.1.5

RUN apt-get update \
    && apt-get install -y curl \
    && curl -fSL -o neo4j-community.tar.gz http://dist.neo4j.org/neo4j-community-$NEO4J_VERSION-unix.tar.gz \
    && tar xzf neo4j-community.tar.gz -C /var/lib \
    && mv /var/lib/neo4j-* /var/lib/neo4j \
    && ln -s /var/lib/neo4j/data /data \
    && touch /tmp/rrd \
    && rm neo4j-community.tar.gz \
    && curl -fSL -o neo4j-2.1.5-sparql-plugin.jar https://github.com/jedivind/neo4j-sparql-extension/releases/download/1.0.0b/neo4j-sparql-extension-1.0.0.jar \
    && mv neo4j-2.1.5-sparql-plugin.jar /var/lib/neo4j/plugins/ \
    && apt-get purge -y --auto-remove curl && rm -rf /var/lib/apt/lists/* 

RUN apt-get update
RUN apt-get install -y maven

RUN sed -i -e "s|.*dbms.pagecache.memory=.*|dbms.pagecache.memory=512M|g" /var/lib/neo4j/conf/neo4j.properties \
    && sed -i -e "s|.*keep_logical_logs=.*|keep_logical_logs=100M size|g" /var/lib/neo4j/conf/neo4j.properties \
    && sed -i -e "s|#*remote_shell_enabled=.*|remote_shell_enabled=true|g" /var/lib/neo4j/conf/neo4j.properties \
    && sed -i -e "s|org.neo4j.server.webadmin.rrdb.location=.*|org.neo4j.server.webadmin.rrdb.location=/tmp/rrd|g" /var/lib/neo4j/conf/neo4j-server.properties \
    && sed -i -e "s|org.neo4j.server.database.location=.*|org.neo4j.server.database.location=/opt/joern-0.3.1/.joernIndex/|g" /var/lib/neo4j/conf/neo4j-server.properties \
    && sed -i -e "s|org.neo4j.server.webserver.address=.*|org.neo4j.server.webserver.address=127.0.0.1|g" /var/lib/neo4j/conf/neo4j-server.properties \    
    && sed -i -e "s|Dneo4j.ext.udc.source=.*|Dneo4j.ext.udc.source=docker|g" /var/lib/neo4j/conf/neo4j-wrapper.conf \
    && sed -i -e "s|#org.neo4j.server.thirdparty_jaxrs_classes=.*|org.neo4j.server.thirdparty_jaxrs_classes=de.unikiel.inf.comsys.neo4j=/rdf|g" /var/lib/neo4j/conf/neo4j-server.properties


RUN apt-get install -y git
WORKDIR /home/
RUN git clone https://github.com/neo4j-contrib/gremlin-plugin.git
WORKDIR /home/gremlin-plugin
RUN find . -type f -print0 |xargs -0 sed -i 's,2016,2017,'
RUN mvn -e clean package -Dlicense.skip=true -Dhttps.protocols=TLSv1.2
RUN unzip target/neo4j-gremlin-plugin-2.1-SNAPSHOT-server-plugin.zip -d /var/lib/neo4j/plugins/gremlin-plugin
VOLUME /data

COPY neo4j.sh /neo4j.sh
EXPOSE 7474 7473

RUN apt-get update
RUN apt-get upgrade -y && apt-get install -y python-dev python-setuptools python-pip graphviz libgraphviz-dev ant python-pygraphviz python-pip && apt-get clean
RUN apt-get install -y tmux 
RUN pip install virtualenv --upgrade
RUN pip install --upgrade pip
RUN apt-get install -y build-essential libssl-dev libffi-dev
# install joern
RUN cd /opt/ && wget https://github.com/fabsx00/joern/archive/0.3.1.tar.gz && tar xfzv 0.3.1.tar.gz
RUN cd /opt/joern-0.3.1 && wget http://mlsec.org/joern/lib/lib.tar.gz && tar xfzv lib.tar.gz
RUN cd /opt/joern-0.3.1 && ant
# Install gremlin plugin
# RUN cd /tmp && wget -O neo4j-gremlin.zip http://mlsec.org/joern/lib/neo4j-gremlin-plugin-2.1-SNAPSHOT-server-plugin.zip && unzip neo4j-gremlin.zip -d /usr/share/neo4j/plugins/gremlin-plugin
#install joern tools
RUN pip install cryptography==1.3.4
RUN cd /tmp/ && git clone https://github.com/fabsx00/joern-tools && cd joern-tools && python2 setup.py install
RUN pip uninstall -y py2neo
WORKDIR /opt/joern-0.3.1
RUN mkdir py2neo
WORKDIR /opt/joern-0.3.1/py2neo
RUN wget https://codeload.github.com/technige/py2neo/zip/release/2.0.7
RUN unzip 2.0.7
WORKDIR py2neo-release-2.0.7
RUN python setup.py install

WORKDIR /opt/joern-0.3.1/
RUN mkdir tutorial
WORKDIR tutorial
RUN wget http://download.videolan.org/pub/videolan/vlc/2.1.4/vlc-2.1.4.tar.xz
RUN tar xfJ vlc-2.1.4.tar.xz
WORKDIR /opt/joern-0.3.1
RUN ./joern tutorial/vlc-2.1.4/
RUN apt-get install -y emacs-nox
RUN chmod -R 777 /opt/joern-0.3.1/
#CMD ["/neo4j.sh"]
