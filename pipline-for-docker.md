```
node { 
   // 拉取代码
   stage('Git Checkout') { 
        checkout([$class: 'GitSCM', branches: [[name: '${branch}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '52b4e1a9-f896-41ff-bb4c-bb354c90fe9e', url: 'http://113.59.224.12/java/dubbo-springboot.git']]])
   }
   // 代码编译
   stage('Maven Build') {
        sh '''
        export JAVA_HOME=/usr/
        /usr/local/maven3/bin/mvn clean package -Dmaven.test.skip=true
        '''
   }
   // 项目打包到镜像并推送到镜像仓库
   stage('Build and Push Api Image') {
        sh '''
        REPOSITORY_api=reg.harbor.io/dubbo-springboot/dubbo-api:${branch}
        echo -e 'FROM reg.harbor.io/base_image/jdk8:v1 
MAINTAINER www.harbor.io
ADD ./demo-api/target/*.jar /data/dubbo-api.jar
ENTRYPOINT [ "sh", "-c", "java  -Djava.security.egd=file:/dev/./urandom -jar /data/dubbo-api.jar" ]' |tee Dockerfile
        echo $PWD
        docker build -t $REPOSITORY_api .
        docker login reg.harbor.io -u admin -p Harbor12345
        docker push $REPOSITORY_api
'''
   }
   stage('Build and Push Service Image') {
        sh '''
        REPOSITORY_service=reg.harbor.io/dubbo-springboot/dubbo-service:${branch}
        echo -e 'FROM reg.harbor.io/base_image/jdk8:v1 
MAINTAINER www.harbor.io
ADD ./demo-service-impl/target/*.jar /data/dubbo-service.jar
ENTRYPOINT [ "sh", "-c", "java  -Djava.security.egd=file:/dev/./urandom -jar /data/dubbo-service.jar" ]' |tee Dockerfile
        docker build -t $REPOSITORY_service .
        docker login reg.harbor.io -u admin -p Harbor12345
        docker push $REPOSITORY_service
'''
   }
   // 备份镜像
   stage('Backup Images') {
       sh '''
       docker tag reg.harbor.io/dubbo-springboot/dubbo-service:${branch} reg.harbor.io/dubbo-springboot/dubbo-service:${branch}-`date +%F_%S`
       docker tag reg.harbor.io/dubbo-springboot/dubbo-api:${branch} reg.harbor.io/dubbo-springboot/dubbo-api:${branch}-`date +%F_%S`
       '''
   }
   // 部署到Docker主机
   stage('Deploy to Service Docker Machaine') {
       sh '''
        echo "部署Service"
        REPOSITORY_service=reg.harbor.io/dubbo-springboot/dubbo-service:${branch}
        docker stop  dubbo-service 
        docker rm dubbo-service 
        docker run -d --name dubbo-service $REPOSITORY_service
        '''
   }
   stage('Deploy to Api Docker Machaine') {
       sh '''
        echo "部署Api"
        REPOSITORY_api=reg.harbor.io/dubbo-springboot/dubbo-api:${branch}
        docker stop dubbo-api
        docker rm -f dubbo-api |true
        docker container run -d --name dubbo-api -p 18105:18105 $REPOSITORY_api
       '''
   }
}
```
