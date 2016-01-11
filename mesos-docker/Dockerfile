#Mesos
FROM centos:latest
MAINTAINER guyan <guyan0315@163.com>

#Add Mesos resource
RUN rpm -Uvh 'http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm'

RUN yum install -y docker
RUN yum install -y mesos

CMD [mesos]
