# jenkins.md



### 

#### Jenkins agent Docker 镜像重新命名, 按需取用
jenkins/agent 是旧的 jenkins/slave 镜像从 4.3-2 开始的新名称
jenkins/inbound-agent 是旧的 jenkins/jnlp-slave 镜像从 4.3-2 开始的新名称
jenkins/ssh-agent 是旧的 jenkins/ssh-slave 镜像从 2.0.0 开始的新名称


### 试验
docker run --init jenkins/inbound-agent:4.3-7-alpine \
  -url http://121.36.34.122:32080 \
  158669499d7e6c3876e72fbaffb667dd8b47d8b13e2ca2ba8d4058578c268bd7 \
  linux-docker-1

