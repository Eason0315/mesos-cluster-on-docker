
# Multi-node Setup
For this setup, we will need 3 servers with Docker installed on it.

1. Export out the 2 servers' IP that we will be using on each server

        export  HOST_IP_1=10.211.55.3
        export  HOST_IP_2=10.211.55.4
        export  HOST_IP_3=10.211.55.5

2. Start ZooKeepers

    On host #1

         docker run -d \
         --net="host" \
         -e SERVER_ID=1 \
         -e ADDITIONAL_ZOOKEEPER_1=server.1=${HOST_IP_1}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_2=server.2=${HOST_IP_2}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_3=server.3=${HOST_IP_3}:2888:3888 \
         garland/zookeeper

    On host #2

         docker run -d \
         --net="host" \
         -e SERVER_ID=2 \
         -e ADDITIONAL_ZOOKEEPER_1=server.1=${HOST_IP_1}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_2=server.2=${HOST_IP_2}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_3=server.3=${HOST_IP_3}:2888:3888 \
         garland/zookeeper

	On host #3

         docker run -d \
         --net="host" \
         -e SERVER_ID=3 \
         -e ADDITIONAL_ZOOKEEPER_1=server.1=${HOST_IP_1}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_2=server.2=${HOST_IP_2}:2888:3888 \
         -e ADDITIONAL_ZOOKEEPER_3=server.3=${HOST_IP_3}:2888:3888 \
         garland/zookeeper
         
    The only difference is the "SERVER_ID".  You can repeat this for the next X number of ZooKeepers you want to run.

3. Start Mesos Masters

    On host #Master1

          docker run --net="host" --name=mesos-master \
          -p 5050:5050 \
          -e "MESOS_HOSTNAME=${HOST_IP_1}" \
          -e "MESOS_IP=${HOST_IP_1}" \
          -e "MESOS_ZK=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/mesos" \
          -e "MESOS_PORT=5050" \
          -e "MESOS_LOG_DIR=/var/log/mesos" \
          -e "MESOS_QUORUM=2" \
          -e "MESOS_REGISTRY=in_memory" \
          -e "MESOS_WORK_DIR=/var/lib/mesos" \
          -d \
          guyan/mesos /usr/sbin/mesos-master

    On host #Master2

          docker run --net="host" --name=mesos-master\
          -p 5050:5050 \
          -e "MESOS_HOSTNAME=${HOST_IP_2}" \
          -e "MESOS_IP=${HOST_IP_2}" \
          -e "MESOS_ZK=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/mesos" \
          -e "MESOS_PORT=5050" \
          -e "MESOS_LOG_DIR=/var/log/mesos" \
          -e "MESOS_QUORUM=2" \
          -e "MESOS_REGISTRY=in_memory" \
          -e "MESOS_WORK_DIR=/var/lib/mesos" \
          -d \
          guyan/mesos /usr/sbin/mesos-master

    On host #Master3

          docker run --net="host" --name=mesos-master\
          -p 5050:5050 \
          -e "MESOS_HOSTNAME=${HOST_IP_3}" \
          -e "MESOS_IP=${HOST_IP_3}" \
          -e "MESOS_ZK=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/mesos" \
          -e "MESOS_PORT=5050" \
          -e "MESOS_LOG_DIR=/var/log/mesos" \
          -e "MESOS_QUORUM=2" \
          -e "MESOS_REGISTRY=in_memory" \
          -e "MESOS_WORK_DIR=/var/lib/mesos" \
          -d \
          guyan/mesos /usr/sbin/mesos-master

4. Start Meso Slaves in a container

		docker run --privileged --name mesos-slave \
  		--entrypoint=mesos-slave \
  		--net=host -p 5051:5051 -d \
  		-e "MESOS_LOG_DIR=/var/log/mesos" \
  		-e "MESOS_MASTER=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/mesos" \
  		-e "MESOS_CONTAINERIZERS=docker" \
  		-v /var/run/docker.sock:/var/run/docker.sock \
  		-v /sys:/sys \
  		guyan/mesos
  		
5. Start Marathon

         docker run \
         -d \
         --net="host" \
         -p 8080:8080 \
         guyan/marathon \
         /usr/bin/marathon --master zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/mesos --zk zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181,${HOST_IP_3}:2181/marathon

6. Goto the Meso's Web page

         http://localhost:5050

7. Goto the Marathon's Web page

         http://localhost:8080
         

