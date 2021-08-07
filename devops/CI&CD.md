### 背景

> 随着公司业务发展，研发端单体项目重构升级走向微服务架构似乎是当前不可避免的道路，然而一些初期团队由于微服务架构理解不足、领域划分不明确等等原因，导致服务规模化问题凸显、拆分过杂。
> 主要表现为服务系统数量激增、服务版本控制以及发布构建较为无章法、有研发失控风险。当然提高研发管理、技术分享，提升coder服务治理、领域划分能力是核心、是本质，但此篇幅暂且不论及这个话题。
> 在此仅聊聊如何打通CI/CD解决方案，尽量提升研发效能。此处CI/CD方案为通过Jenkins搭建持续集成环境，继而形成Git+Jenkins+maven+nexus+EDAS（阿里容器服务【docker+k8s】）方案。

### 方案设计(一图一避之)
![全景图](https://user-images.githubusercontent.com/88572123/128595034-cca13ad4-a1be-4a2e-98b0-0cf5b9fcf32b.jpg)

### 方案实施

#### 项目pom改造
公司私库顶级pom添加
```
# 插件配置
<build>
  <plugins>
    ...
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <version>3.1.2</version>
      <configuration>
        <from>
          ...
        </from>
        <to>
           <image>${registry.url}</image>
           <tags><tag>${image.tag}</tag></tags>
           <auth>
            <username>${registry.username}</username>
           	<password>${registry.password}</password>
           </auth>
        </to>
        <container>
        	<mainClass>${project.main.zlass}</mainClass>
          	<ports>
            	<port>8080</port>
            </ports>
          </container>
          <allowInsecureRegistries>true</allowInsecureRegistries>
      </configuration>
      <executions>
          <execution>
             <phase>package</phase>
             <goals>
               <goal>build</goal>
             </goals>
          </execution>
      </executions>
    </plugin>
  </plugins>      
</build>
```

#### 环境搭建&测试
> 环境准备 `自行准备略去操作`
> > 系统Centos 7  
> > yum -y update 内核升级  
> > openJDK1.8  
> > docker
- 安装GitLab及Nexus私库
```
此两着安装略去，search太多，随意查看
```
- 安装Git
```
yum -y install git
git --version
#默认安装目录/usr/bin/git
vim /etc/profile
追加export PATH=/root/git/bin:$PATH
wq!
source /etc/profile
```
- 安装maven
```
cd /opt
wget https://apache.claz.org/maven/maven-3/3.8.1/binaries/apache-maven-3.8.1-bin.tar.gz
tar xf apache-maven-3.8.1
mv apache-maven-3.8.1 maven381
vim /etc/profile
追加：
export MAVEN_HOME=/opt/maven381
export PATH=$PATH:$MAVEN_HOME/bin
wq!
source /etc/profile
```
- 安装Jenkins  
基于docker执行
```
docker pull jenkins/jenkins
#宿主机自行创建/Users/xxx/docker_wsp/jenkins,用于挂载数据卷
docker run -d  --name ijk -u root -p 9999:8080 -v /Users/xxx/docker_wsp/jenkins:/var/jenkins_home jenkins/jenkins
docker exec -it ijk bash
cat /var/jenkins_home/secrets/initialAdminPassword #查看密码
#http登录并初始化安装插件
#核心插件：git、maven、edas、dingTalk、git parameter
```
