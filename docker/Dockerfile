#beegfs-client docker
FROM centos:centos8

LABEL beegfs_component="client"

ENV BEEGFS_VERSION 7.2.5

RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum -q -y update
RUN yum install -y sudo

RUN dnf -y install openssh-server openssh-clients; \
    sed -i 's/^\(UsePAM yes\)/# \1/' /etc/ssh/sshd_config; \
    ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' && \
    ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' && \
    ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N ''; \
    dnf clean all;

# entrypoint
RUN { \
    echo '#!/bin/bash -eu'; \
    echo 'ln -fs /usr/share/zoneinfo/${TIMEZONE} /etc/localtime'; \
#    echo 'echo "root:${ROOT_PASSWORD}" | chpasswd'; \
    echo 'echo "root:password" | chpasswd'; \
    echo 'exec "$@"'; \
    } > /usr/local/bin/entry_point.sh; \
    chmod +x /usr/local/bin/entry_point.sh;

ENV TIMEZONE Asia/Seoul

ENV ROOT_PASSWORD root


RUN sudo yum install -y autoconf automake bash curl git openssh openssh-server openssh-clients
RUN yum install -y wget git pkg-config nano gcc bzip2 patch gcc-c++ make
RUN yum groupinstall "Development Tools" -y
RUN yum -y install mpich*
RUN echo "export PATH=$PATH:/usr/lib64/mpich/bin/" >> /root/.bashrc \
	&& source /root/.bashrc && wget https://github.com/hpc/ior/releases/download/3.3.0/ior-3.3.0.tar.gz \
	&& tar xvzf ior-3.3.0.tar.gz && cd ior-3.3.0 && ./configure && make && sudo make install
#RUN source /root/.bashrc
#RUN cat /root/.bashrc
#RUN wget https://github.com/hpc/ior/releases/download/3.3.0/ior-3.3.0.tar.gz \
#        && tar xvzf ior-3.3.0.tar.gz && cd ior-3.3.0 && ./configure && make && sudo make install
#RUN git clone https://github.com/hpc/ior.git \
#        && cd ior && ./bootstrap && ./configure && make && sudo make install


RUN curl -L -o /etc/yum.repos.d/beegfs-rhel8.repo \
  http://www.beegfs.com/release/beegfs_${BEEGFS_VERSION}/dists/beegfs-rhel8.repo
RUN yum install -q -y beegfs-client beegfs-helperd beegfs-utils; yum clean all

RUN /opt/beegfs/sbin/beegfs-setup-client -m 192.168.61.83

ADD beegfs/*.conf /etc/beegfs/
ADD run.sh /

# Generate keys]
#RUN sed -i 's/^\(UsePAM yes\)/# \1/' /etc/ssh/sshd_config; \
#    ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' && \
#    ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' && \
#    ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N '';
#RUN echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
#RUN ssh-keygen -A
#RUN echo "root:root" | chpasswd

#RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key
#RUN ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa
#RUN cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

#ADD start.sh /start.sh
#RUN chmod +x /start.sh
RUN cp /usr/local/bin/entry_point.sh /
RUN chmod +x /run.sh

RUN { \
    echo '#!/bin/bash'; \
#    echo 'set -m'; \
#    echo './entry_point.sh &'; \
    echo './run.sh &'; \
    echo './entry_point.sh &'; \
    echo 'wait'; \
    } > final_entry_point.sh; \
    chmod +x final_entry_point.sh;
#CMD /start.sh
#RUN  /etc/init.d/sshd start
#CMD ["/usr/sbin/sshd", "-D"]

RUN { \
    echo '#!/bin/bash'; \
#    echo 'set -m'; \
#    echo './entry_point.sh &'; \
    echo 'ssh-keygen -A'; \
    echo 'mkdir -p .ssh'; \
    echo 'echo "$AUTHORIZED_KEYS" > .ssh/authorized_keys'; \
    echo 'chmod 0700 .ssh'; \
    echo 'chmod 0600 .ssh/authorized_keys'; \
#    sed -i 's/^\(# UsePAM yes\)/^# \1/' /etc/ssh/sshd_config; \
    echo 'exec /usr/sbin/sshd -D -e'; \
    echo 'exec "$@"'; \
    } > entrypoint_passwordless.sh; \
    chmod +x entrypoint_passwordless.sh;

#ENV AUTHORIZED_KEYS

ENTRYPOINT ["/entrypoint_passwordless.sh"]

EXPOSE 8004 8006 22
CMD ["sh", "run.sh"]
#CMD ["/usr/sbin/sshd", "-D", "-e"]
