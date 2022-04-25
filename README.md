
# 一分钟自建zerotier-planet

私有部署zeroteir-planet服务
基于 [ztncui](https://github.com/key-networks/ztncui-aio) 整理成 docker-compose.yml 文件.


# 必要条件

- 具有公网ip的服务器(需要开放4000/tcp端口,亦可自定义端口）
- 安装 docker
- 安装 docker-compose

# 用法

纯小白建议直接执行这个:

`docker run --restart=on-failure:3 -d --name ztncui -e HTTP_PORT=4000 -e HTTP_ALL_INTERFACES=yes -e ZTNCUI_PASSWD=mrdoc.fun -p 4000:4000 keynetworks/ztncui`

有基础的,可二选一.

```
git clone https://github.com/Jonnyan404/zerotier-planet
OR
git clone https://gitee.com/Jonnyan404/zerotier-planet

cd zerotier-planet
docker-compose up -d
```

然后访问 `http://ip:4000` 访问web界面.

- 用户名:admin
- 密码:mrdoc.fun

# 关联云服务器(带公网IP)

[【腾讯云】云产品限时秒杀，爆款2核4G云服务器，首年74元](https://curl.qcloud.com/S2Db7PLK)

## 后记

经实测,window/Android客户端可直接连接,无需修改任何文件.理论上其它客户端同理.

### 私有 zerotier-planet 的优势:
- 解除官方 50 的设备连接数限制
- 提升手机客户端连接的稳定性

# Reference Link

- <https://www.mrdoc.fun/doc/443/>
- <https://github.com/key-networks/ztncui-aio>




使用作者的docker, 最终中心节点显示为leaf, 测试移动4g和电信宽带延迟为500ms, 到中心节点分别为120ms和40ms, 实际没有走自定义的节点.
经过研究, 需要再进行设置. 提供下思路, 如下:

compose需要增加ports: '9993:9993'和'9993:9993/udp', 服务器和防火墙也得放行
进入容器, 生成moon.json
拷贝moon.json到宿主机, 修改stableEndpoints
在宿主机用mkmoonworld-x86生成行星文件
把修改后的moon.json拷回容器, 在容器内生成moon文件, 创建moons.d文件夹, 放进去. 拷贝一份到宿主机备用
把行星文件替换回容器
重启容器
把客户端的planet文件替换
安卓端的话, 实测单独加载planet不生效. 加载moon文件, 关闭官方行星节点, 生效
具体参考 https://github.com/xubiaolin/docker-zerotier-planet 里面的代码实现和各种生成moon教程

######## 仅供参考 #########

下载

git clone https://github.com/Jonnyan404/zerotier-planet
cd zerotier-planet
vim docker-compose.yml
修改

###　参考
### date:2021年11月29日
### author: www.mrdoc.fun | jonnyan404
### 转载请保留来源
version: '2.0'
services:
    ztncui:
        container_name: ztncui
        restart: always
        environment:
            # - MYADDR=公网地址(不设置该项自动获取)
            - MYADDR=127.0.0.1  # 改成自己的服务器公网ip
            - HTTP_PORT=3443
            - HTTP_ALL_INTERFACES=yes
            - ZTNCUI_PASSWD=root
        ports:
            - '3443:3443'  # 设置网页的端口
            - '9993:9993'  # 作为中心节点，提供9993端口给客户端用，一般是9993
            - '9993:9993/udp'
        volumes:
            - './zerotier-one:/var/lib/zerotier-one'
            - './ztncui/etc:/opt/key-networks/ztncui/etc'
            # 按实际路径挂载卷， 冒号前面是宿主机的， 支持相对路径
        image: keynetworks/ztncui
运行

docker-compose up -d

docker images #　查看镜像
docker container ps -a # 查看容器

docker exec -it ztncui bash # 进入容器
# 在容器内操作
cd /var/lib/zerotier-one
ls -l
# 生成moon配置文件
zerotier-idtool initmoon identity.public > moon.json
chmod 777 moon.json
新建一个terminal, 在容器外修改moon.json, 位置对应挂载位置

修改stableEndpoints, 注意格式和实际公网ip

{
 "id": "b72b5e9e1a",
 "objtype": "world",
 "roots": [
  {
   "identity": "b72b5e9e1a:0:a892e51d2ef94ef941e4c499af01fbc2903f7ad2fd53e9370f9ac6260c2f5d2484fd90756bec0c410675a81b7cf61d2bb885783bd6a8c28bce83bcab5f03fe14",
   "stableEndpoints": ["127.0.0.1/9993"]
  }
 ],
 "signingKey": "45f0613e569a0549c74293c39b30495b594a003534290e8ade9ef82877aa7505d7a73eeabfc22c97c404e4caaf9f3c9eed2b134d696935c966e28f523364f15f",
 "signingKey_SECRET": "cc6afd67e7b7f84a92e2c8d3c2e7212c71e2ad0a4f5b3c03bf60ab1cd3b99281b57d9a2958d2bd8fc2bc77fdf2a1160099c2c61d3d9acc8cb311673ee120b4a6",
 "updatesMustBeSignedBy": "45f0613e569a0549c74293c39b30495b594a003534290e8ade9ef82877aa7505d7a73eeabfc22c97c404e4caaf9f3c9eed2b134d696935c966e28f523364f15f",
 "worldType": "moon"
}
在容器内生成moon文件

zerotier-idtool genmoon moon.json
mkdir moons.d
cp *.moon moons.d/
在容器外生成planet文件

拷贝一份moon文件， 客户端可以用到

下载[mkmoonworld](https://github.com/kaaass/ZeroTierOne/releases/tag/mkmoonworld-1.0), 拷贝moon.json， 放在一个目录下

./mkmoonworld-x86_64 ./moon.json
mv world.bin planet
# 复制到容器内
docker cp ./planet ztncui:/var/lib/zerotier-one
重启容器

docker restart ztncui
docker exec -it ztncui bash # 进入容器
# 在容器内操作
cd /var/lib/zerotier-one
#　查看ｍoon
zerotier-cli listmoons
访问ip+端口对应的设置页面

替换客户端的planet文件并重启服务， 再加入网络， 在网页端授权
