# Description

This repository provides Dockerfiles to create a ScaleIO cluster on 3 Docker hosts.

This cluster will provide MDM, TB, SDS and Gateway services. Then, other ScaleIO clients (where the SDC service will be installed) will access the ScaleIO volumes.

It has been tested with ScaleIO 1.31.

This gives anyone the ability to create a clean ScaleIO cluster for test purposes.

*Disclaimer: Don't use this for production.*

# Build

To build the docker images, you need to copy the ScaleIO RPM packages for Red Hat 6 in the 3 directories:

- scaleio-primary-mdm
- scaleio-secondary-mdm
- scaleio-tb

Then, you can simply run the following command in each directory:

```
docker build .
```

# Run

So, assume you want to create the following setup:

-	ScaleIO TB and SDS - 192.168.1.1 - Using /dev/sdb and /dev/sdc
-	ScaleIO Secondary MDM and SDS - 192.168.1.1 - Using /dev/sdb and /dev/sdc
-	ScaleIO Secondary MDM and SDS and API gateway - 192.168.1.1 - Using /dev/sdb and /dev/sdc

Existing data on /dev/sdb and /dev/sdc will be lost.

To create a new cluster, you just need to run the following commands:

- On 192.168.1.1:
```
docker run -d -p 9011:9011 -p 7072:7072 --privileged <scaleio-tb-1.31 image you built>
```

-	On 192.168.1.2:
```
docker run -d -p 6611:6611 -p 9011:9011 -p 7072:7072 --privileged <scaleio-secondary-mdm-1.31 image you built>
```

- On 192.168.1.3:
```
docker run -d –p 80:80 -p 443:443 -p 6611:6611 -p 9011:9011 -p 7072:7072 --privileged -e IP_DOCKER_HOST=192.168.1.3 -e IP_SECONDARY_MDM=192.168.1.2 -e IP_TB=192.168.1.1 -e DEVICE_LIST=/dev/sdb,/dev/sdc <scaleio-primary-mdm-1.31 image you built>
```

The docker run command return the id of the docker container. You can then use the docker logs <id> and check that all the commands are correctly executed. Then, the output of the ps and netstat commands are displayed to let you make sure everything is running correctly.

Then, you can access the Primary MDM with the credentials admin / password1?

The credentials are the same for the API gateway. You can use the ScaleIO REST API by sending requests to http://192.168.1.3 or https://192.168.1.3/

You can use also the ScaleIO UI or the scli command to administer the cluster.

If you want to execute commands on the ScaleIO containers, you can use the nsenter tool.

It assumes that you will access the SDS from other hosts where you install the ScaleIO SDC binary.

You don’t have to worry about having clean /dev/sdb and /dev/sdc devices because the SDS are added with the force_clean option.

# Licensing

Licensed under the Apache License, Version 2.0 (the “License”); you may not use this file except in compliance with the License. You may obtain a copy of the License at <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an “AS IS” BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

# Support

Please file bugs and issues at the Github issues page. For more general discussions you can contact the EMC Code team at <a href="https://groups.google.com/forum/#!forum/emccode-users">Google Groups</a>. The code and documentation are released with no warranties or SLAs and are intended to be supported through a community driven process.
