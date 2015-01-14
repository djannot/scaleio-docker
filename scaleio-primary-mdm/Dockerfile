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
test=\$? \n\
if ((\$test != 0 )); then \n\
  rpm -Uvh /opt/emc/scaleio/siinstall/EMC-ScaleIO-mdm-*.x86_64.rpm \n\
  rpm -Uvh /opt/emc/scaleio/siinstall/EMC-ScaleIO-sds-*.x86_64.rpm \n\
  rpm -Uvh /opt/emc/scaleio/siinstall/EMC-ScaleIO-gateway-*.rpm \n\
  sed -i 's/mdm.ip.addresses=/mdm.ip.addresses='\$IP_DOCKER_HOST','\$IP_SECONDARY_MDM'/' /opt/emc/scaleio/gateway/webapps/ROOT/WEB-INF/classes/gatewayUser.properties \n\
  sed -i 's/gateway-admin.password=/gateway-admin.password=password1?/' /opt/emc/scaleio/gateway/webapps/ROOT/WEB-INF/classes/gatewayUser.properties \n\
  service scaleio-gateway restart \n\
fi \n\
/opt/emc/scaleio/mdm/bin/run_bin.sh & \n\
/opt/emc/scaleio/sds/bin/run_bin.sh & \n\
if ((\$test != 0 )); then \n\
  sleep 10 \n\
  scli --mdm --add_primary_mdm --primary_mdm_ip \$IP_DOCKER_HOST --accept_license \n\
  sleep 5 \n\
  scli --login --username admin --password admin \n\
  scli --set_password --old_password admin --new_password password1? \n\
  scli --login --username admin --password password1? \n\
  scli --add_secondary_mdm --mdm_ip \$IP_DOCKER_HOST --secondary_mdm_ip \$IP_SECONDARY_MDM \n\
  scli --add_tb --mdm_ip \$IP_DOCKER_HOST --tb_ip \$IP_TB \n\
  scli --switch_to_cluster_mode --mdm_ip \$IP_DOCKER_HOST \n\
  #scli --mdm --set_license --license= \n\
  scli --add_protection_domain --mdm_ip \$IP_DOCKER_HOST --protection_domain_name domain1 \n\
  sleep 10 \n\
  until scli --sds --add_sds --mdm_ip \$IP_DOCKER_HOST --sds_ip \$IP_TB --sds_ip_role all --protection_domain_name domain1 --device_path \$DEVICE_LIST --sds_name sds1 --no_test --force_clean --i_am_sure; do \n\
    echo Retrying in 5 seconds \n\
    sleep 5 \n\
  done \n\
  until scli --sds --add_sds --mdm_ip \$IP_DOCKER_HOST --sds_ip \$IP_DOCKER_HOST --sds_ip_role all --protection_domain_name domain1 --device_path \$DEVICE_LIST --sds_name sds2 --no_test --force_clean --i_am_sure; do \n\
    echo Retrying in 5 seconds \n\
    sleep 5 \n\
  done \n\
  until scli --sds --add_sds --mdm_ip \$IP_DOCKER_HOST --sds_ip \$IP_SECONDARY_MDM --sds_ip_role all --protection_domain_name domain1 --device_path \$DEVICE_LIST --sds_name sds3 --no_test --force_clean --i_am_sure; do \n\
    echo Retrying in 5 seconds \n\
    sleep 5 \n\
  done \n\
fi \n\
while true;do \n\
  ps -ef \n\
  netstat -an \n\
  sleep 10 \n\
done" > /start.sh

RUN chmod +x /start.sh
EXPOSE 443
EXPOSE 6611
EXPOSE 9011
EXPOSE 7072
ENTRYPOINT ["/start.sh"]
