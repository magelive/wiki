
##时区设置(通过Dockerfile)
```
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
或者也可以直接这样：
```
# Set the timezone.
RUN sudo echo "America/New_York" > /etc/timezone
RUN sudo dpkg-reconfigure -f noninteractive tzdata
```
更多参考：
http://serverfault.com/questions/683605/docker-container-time-timezone-will-not-reflect-changes
https://github.com/docker/docker/issues/12084
http://tedwise.com/2015/05/02/setting-the-timezone-in-a-docker-image

##中文支持
新建Dockerfile：
```
FROM ubuntu:trusty

# Ensure UTF-8 locale
#COPY locale /etc/default/locale
RUN locale-gen zh_CN.UTF-8 &&\
  DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales
RUN locale-gen zh_CN.UTF-8  
ENV LANG zh_CN.UTF-8  
ENV LANGUAGE zh_CN:zh  
ENV LC_ALL zh_CN.UTF-8
```
  
然后执行构建
```
docker build -t xbyubuntu .
```

更多参考:
http://stackoverflow.com/questions/28405902/how-to-set-the-locale-inside-a-docker-container
https://github.com/abevoelker/docker-ubuntu-locale/blob/master/Dockerfile
http://blog.shiqichan.com/Input-Chinese-character-in-docker-bash/