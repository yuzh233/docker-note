FROM centos
MAINTAINER harry<yuzh233@gmail.com>

ADD apache-tomcat-7.0.75.tar.gz /usr/local
ADD jdk-8u121-linux-x64.tar.gz /usr/local

RUN yum -y install vim
ENV WORK_PATH /usr/local
WORKDIR $WORK_PATH

# 设置 java 环境目录，此文件夹名可以在本地将 jdk-8u121-linux-x64.tar.gz 得到
ENV JAVA_HOME /usr/local/jdk1.8.0_121
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.75
ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.75

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

# ENTRYPOINT ["/usr/local/apache-tomcat-7.0.75/bin/startup.sh"]
# CMD ["/usr/local/apache-tomcat-7.0.75/bin/startup.sh","run"]

# tail 命令：查看文件最后指定行，-f 参数不停的读取。
CMD /usr/local/apache-tomcat-7.0.75/bin/startup.sh \ && tail -f /usr/local/apache-tomcat-7.0.75/logs/catalina.out

