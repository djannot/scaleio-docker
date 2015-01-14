FROM centos:centos6
RUN yum -y install tar wget numactl libaio mutt python python-paramiko java-1.6.0-openjdk vi which
RUN wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
RUN wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
RUN rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
RUN yum -y install bash-completion
ADD . /opt/emc/scaleio/siinstall/

RUN printf "#!/bin/sh \n\
umount /dev/shm \n\
mount -t tmpfs -o rw,nosuid,nodev,noexec,relatime,size=524288k shm /dev/shm \n\
rpm -qa | egrep -i 'ecs|scaleio' \n\
if ((\$? != 0 )); then \n\
  rpm -Uvh /opt/emc/scaleio/siinstall/EMC-ScaleIO-mdm-*.x86_64.rpm \n\
  rpm -Uvh /opt/emc/scaleio/siinstall/EMC-ScaleIO-sds-*.x86_64.rpm \n\
fi \n\
/opt/emc/scaleio/mdm/bin/run_bin.sh & \n\
/opt/emc/scaleio/sds/bin/run_bin.sh & \n\
while true;do \n\
  ps -ef \n\
  netstat -an \n\
  sleep 10 \n\
done" > /start.sh

RUN chmod +x /start.sh
EXPOSE 6611
EXPOSE 9011
EXPOSE 7072
EXPOSE 25620
EXPOSE 25640
ENTRYPOINT ["/start.sh"]
