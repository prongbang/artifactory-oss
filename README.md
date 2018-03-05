# JFROG ARTIFACTORY OPEN SOURCE FOR ARTIFACT LIFE-CYCLE MANAGEMENT
JFrog’s Artifactory open source project was created to speed up development cycles using binary repositories. It’s the world’s most advanced repository manager, creating a single place for teams to manage all their binary artifacts efficiently.
[https://jfrog.com/open-source/](https://jfrog.com/open-source/)

# Docker Image
```bash
FROM frogops-dockerv2.jfrog.io/os/centos-artifactory:6.6

# MAINTAINER matank@jfrog.com

EXPOSE 8081

ADD runArtifactory.sh /tmp/run.sh
RUN chmod +x /tmp/run.sh && \
    yum install -y --disablerepo="*" --enablerepo="Artifactory-oss" jfrog-artifactory-oss

LABEL Create folders needed by Artifactory and set owner to artifactory user. \
Without this action, Artifactory will not be able to write to external mounts
RUN mkdir -p /etc/opt/jfrog/artifactory /var/opt/jfrog/artifactory/{data,logs,backup} && \
    chown -R artifactory: /etc/opt/jfrog/artifactory /var/opt/jfrog/artifactory/{data,logs,backup} && \
    mkdir -p /var/opt/jfrog/artifactory/defaults/etc && \
    cp -rp /etc/opt/jfrog/artifactory/* /var/opt/jfrog/artifactory/defaults/etc/

ENV ARTIFACTORY_HOME /var/opt/jfrog/artifactory

CMD /tmp/run.sh
```

# Create Container
```
docker build -t artifactory-oss .
```

# Start Container
```
docker run -it --name artifactory-oss -p 8081:8081 artifactory-oss
```
or 
```
docker start artifactory-oss
```

# Docker Compose
```
docker-compose up -d
```

# Test
```
http://127.0.0.1:8081/artifactory/webapp/#/home
```

# Account
```
u: admin
p: itrepos
```

# Screen Shot
<img src="https://github.com/prongbang/artifactory-oss/blob/master/screen-shot.png?raw=true"/>


# อัปโหลดไลบรารี่จาก Android Studio ขึ้น Artifactory

> เพิ่ม properties ในไฟล์ local.properties ตามนี้

```gradle
# jfrog artifactory
artifactory_url=http://IPADDRESS:8081/artifactory
artifactory_username=YOUR_USERNAME
artifactory_password=YOUR_ENCRYPTED_PASSWORD
```

> เปิดไฟล์ ```build.gradle``` ระดับโปรเจค เพิ่มบรรทัด classpath ไปดังนี้

```gradle
buildscript {
    dependencies {
        // Add this line
        classpath "org.jfrog.buildinfo:build-info-extractor-gradle:4.0.1"
    }
}
```

> เปิดไฟล์ build.gradle ของ Library Module ที่เราต้องการจะเอาขึ้น Artifactory ขึ้นมา เพิ่มสองบรรทัดนี้เข้าไปที่ด้านบนสุดของไฟล์

```gradle
apply plugin: 'com.jfrog.artifactory'
apply plugin: 'maven-publish'
```

> และเพิ่มด้านล่างสุดของไฟล์ ให้ใส่สคริปต์เพิ่มตามรูปแบบดังนี้

```gradle
def libraryGroupId = 'my.group.package'
def libraryArtifactId = 'thelibrary'
def libraryVersion = '1.0.0'
```

> โดยโค้ดด้านบนจะทำให้เราสามารถดึงไลบรารี่ตัวนี้มาใช้ได้ด้วยคำสั่ง 

```gradle
compile 'my.group.package:thelibrary:1.0.0'
```

> จากนั้นให้ใส่ต่อท้ายไฟล์เดิม

```gradle
publishing {
    publications {
        aar(MavenPublication) {
            groupId libraryGroupId
            version libraryVersion
            artifactId libraryArtifactId

            artifact("$buildDir/outputs/aar/${artifactId}-release.aar")
        }
    }
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

artifactory {
    contextUrl = properties.getProperty('artifactory_url')
    publish {
        repository {
            repoKey = 'libs-release-local'

            username = properties.getProperty('artifactory_username')
            password = properties.getProperty('artifactory_password')
        }
        defaults {
            publications('aar')
            publishArtifacts = true

            properties = ['qa.level': 'basic', 'q.os': 'android', 'dev.team': 'core']
            publishPom = true
        }
    }
}
```

# อัปโหลดไลบรารี่
## Windows:
```
gradlew.bat assembleRelease artifactoryPublish
```

## Mac OS X & Linux:
```
./gradlew assembleRelease artifactoryPublish
```

> รอจนกระทั่งมันทำงานเสร็จและขึ้นว่า BUILD SUCCESSFUL  ก็เป็นอันเรียบร้อยและพร้อมใช้งานแล้ว


# ตรวจสอบความถูกต้อง
> กดเข้าไปที่เว็บเดิม http://IPADDRESS:8081/artifactory และกดไปที่ Artifacts -> libs-release-local

# การเรียกใช้งาน
> ประกาศเพิ่ม Maven Repository ไว้ในไฟล์ ```build.gradle``` ระดับโปรเจคไว้ดังนี้

```gradle
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

allprojects {
    repositories {
        jcenter()
        // Add these lines
        maven {
            url "${properties.getProperty('artifactory_url')}/libs-release-local"
            credentials {
                username = properties.getProperty('artifactory_username')
                password = properties.getProperty('artifactory_password')
            }
        }
    }
}
```

> ไปที่ไฟล์ ```build.gradle``` ของโมดูลที่คุณต้องการจะเรียกไลบรารี่เอามาใช้งาน และใส่ dependency ประมาณนี้

```gradle
dependencies {
    compile 'my.group.package:thelibrary:1.0.0'
}
```

# docker-compose: command not found (Linux)
> [https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)
```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
```