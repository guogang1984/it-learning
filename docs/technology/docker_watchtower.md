docker_watchtower.md

# 参考
https://www.jianshu.com/p/67ca0b521154#!/xh

````bash
docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
````

# https://github.com/containrrr/watchtower