# Creates an Ambari Server base on vanilla centos
FROM sequenceiq/dnsmasq
MAINTAINER SequenceIQ

ADD hdp.repo /etc/yum.repos.d/
ADD ambari.repo /etc/yum.repos.d/
ADD epel-apache-maven.repo /etc/yum/repos.d/
RUN yum install -y ambari-server ambari-agent
RUN yum install -y tar git curl
RUN ambari-server setup --silent

ADD serf /usr/local/serf

RUN echo root:password | chpasswd
# instead of starting the docker file FROM sequenceiq/ssh:
RUN yum install -y curl
RUN curl -L https://api.github.com/repos/sequenceiq/docker-ssh/tarball/master|tar -xz -C /usr/local/serf --strip-components=2 --touch \*/serf
RUN curl -L https://raw.githubusercontent.com/sequenceiq/docker-ssh/master/install-ssh.sh | bash

# fix annoying PAM error 'couldnt open session'
RUN sed -i "/pam_limits/ s/^/#/" /etc/pam.d/*

# warm up the image
RUN yum install -y ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi cyrus-sasl-plain gcc gcc-c++ krb5-devel libtidy libxml2-devel libxslt-devel mysql mysql-devel openldap-devel python-devel python-simplejson sqlite-devel apache-maven wget

RUN yum install -y ambari-log4j hadoop hadoop-libhdfs hadoop-lzo hadoop-lzo-native hadoop-mapreduce hadoop-mapreduce-historyserver hadoop-yarn hadoop-yarn-nodemanager hadoop-yarn-proxyserver hadoop-yarn-resourcemanager lzo net-snmp net-snmp-utils snappy snappy-devel unzip zookeeper

# add ambari shell to the image so new users don't need the 1GB java image
RUN curl -Ls https://raw.githubusercontent.com/sequenceiq/ambari-shell/master/latest-snap.sh | bash
ADD install-cluster.sh /tmp/
ADD wait-for-host-number.sh /tmp/
ADD ambari-shell.sh /tmp/
ENV JAVA_HOME /usr/jdk64/jdk1.7.0_45
ENV PATH $PATH:$JAVA_HOME/bin
WORKDIR /tmp

# fixing pgsql issue
RUN rm -rf /tmp/.s.PGSQL.5432.*

# install hue
RUN \
  useradd hue && \
  cd /tmp && \
  wget https://dl.dropboxusercontent.com/u/730827/hue/releases/3.6.0/hue-3.6.0.tgz && \
  tar xvzf hue-3.6.0.tgz && \
  cd hue-3.6.0 && \
  PREFIX=/usr/local make install && \
  chown -R hue:hue /usr/local/hue/desktop 
   
ADD hue.ini /usr/local/hue/desktop/conf/


# EXPOSE 50020 50090 50070 50010 50075 50030 8021 8080 8020 50111
EXPOSE 50020 50090 50070 50010 50075 50030 50111 8021 8080 8020 8088 8000 8888

ENTRYPOINT ["/usr/local/serf/bin/start-serf-agent.sh"]
CMD ["--log-level", "debug"]
