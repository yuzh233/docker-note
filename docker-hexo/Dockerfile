FROM centos
MAINTAINER harry<yuzh233@gmail>
EXPOSE 4000
RUN cd / && mkdir docker-hexo && cd docker-hexo

WORKDIR /docker-hexo

# install latest node
RUN yum install -y gcc-c++ make \
&& curl -sL https://rpm.nodesource.com/setup_12.x | bash - \
&& yum -y install nodejs \
&& node -v

# install git
RUN yum -y install git \
&& git --version

# install hexo
RUN npm install -g hexo-cli

# CMD ["sh","-c","hexo g && hexo s"]
ENTRYPOINT ["sh","-c","cd /harrys-blog && hexo g && hexo s"]
