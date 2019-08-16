# 한글버전(0.1)
# Kylin on Docker
이 저장소는 [Apache Kylin] (http://kylin.apache.org)을 사용하여 도커 이미지를 빌드하기 위한 코드와 파일을 추적합니다.

참고 : 이것은 스크립트와 Dockerfile이 없는 마스터 브랜치입니다. Kylin 버전과 Hadoop 릴리스 이름으로 명명 된 특정 지점을 체크 아웃해야합니다.

## Background
Apache Kylin은 매우 큰 데이터 세트를 지원하는 Hadoop에서 SQL 인터페이스 및 다차원 분석 (OLAP)을 제공하도록 설계된 오픈 소스 분산 분석 엔진입니다.
자세한 내용은 Kylin 홈페이지 (http://kylin.apache.org)를 방문하십시오.

Kylin은 일반적으로 Hadoop, HBase, Hive 및 기타 클라이언트가 클러스터와 통신하도록 올바르게 구성된 전용 Hadoop 클라이언트 노드에 배포됩니다.
Kylin은 클라이언트 Jar 및 구성 파일을 사용하여 다른 구성 요소와 작업합니다.
또한 모든 Kylin 메타 데이터와 큐브 데이터는 로컬이 아닌 HBase 및 HDFS에 유지되므로 Kylin을 빠른 배포를위한 도커 이미지로 구축하는 것이 매우 합리적입니다.

## How to make it
주요 아이디어는 Hadoop / HBase / Hive 클라이언트와 Kylin 바이너리 패키지를 하나의 이미지로 구축하는 것입니다. core-site.xml, hdfs-site.xml, yarn-site.xml, hbase-site.xml, hive-site.xml 및 kylin.properties와 같은 클라이언트 구성 파일을 유효한 경로에 추가하여 새 이미지를 만듭니다.  

시작하기 전에 몇 가지 준비가 필요합니다.
* Hadoop 버전을 확인하고 이미지의 클라이언트 라이브러리가 클러스터와 호환 가능한지 확인하십시오.
*이 배포를 위해 kylin.properties 파일을 준비하십시오.
*하둡 보안 제약으로 인해 Docker의 채택이 차단되지 않도록합니다. Kerberos가 활성화 된 경우 컨테이너에서 추가 구성 요소를 실행해야 할 수도 있습니다.

## Steps
다음은 Hortonworks HDP 2.2 클러스터에 대한 도커 이미지를 빌드하고 실행하는 샘플입니다.

1. Collect the client configuration files
  작동하는 Hadoop 클라이언트 노드에서 "-/ hadoop-conf /"라는 로컬 폴더로 * -site.xml 파일을 가져옵니다.
2. Prepare kylin.properties 
  kylin.properties 파일은 Kylin의 기본 구성 파일입니다. 이러한 파일을 준비하여 다른 conf 파일과 함께 "~ / hadoop-conf /"폴더에 넣으십시오.
  예를 들어 "kylin.metadata.url"은 올바른 메타 데이터 테이블을 가리키며 "kylin.hdfs.working.dir"은 기존 HDFS 폴더이며 쓰기 권한이 있습니다.

3. Clone this repository
  git clone https://github.com/sungminyun/kylin-docker  
  cd kylin-docker  

4. Copy the client configuration files to "kylin-docker/conf" folder, 해당 템플릿 파일을 덮어 씁니다.
  cp -rf ~/hadoop-conf/* conf/

5. Build docker image 
  docker build -t sungminyun/kylin:152 .
  
  빌드가 완료되면 "docker images"명령이로 이미지를 확인 할 수 있습니다.
  # docker images  
  REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE  
  sungminyun/kylin     152                 7ece32097fa3        About an hour ago   3.043 GB  
6. 이제 bootstrap 명령을 사용하여 contianer를 실행할 수 있습니다 (Kylin 서버 시작).

[root@ip-10-0-0-38 ~]# docker run -i -t -p 7070:7070 sungminyun/kylin:152 /etc/bootstrap.sh -bash  
Generating SSH1 RSA host key:                              [  OK  ]  
Starting sshd:                                             [  OK  ]  
KYLIN_HOME is set to /usr/local/kylin/bin/../  
kylin.security.profile is set to ldap  
16/06/30 04:50:31 WARN conf.HiveConf: HiveConf of name hive.optimize.mapjoin.mapreduce does not exist  
16/06/30 04:50:31 WARN conf.HiveConf: HiveConf of name hive.heapsize does not exist  
16/06/30 04:50:31 WARN conf.HiveConf: HiveConf of name hive.server2.enable.impersonation does not exist  
16/06/30 04:50:31 WARN conf.HiveConf: HiveConf of name hive.auto.convert.sortmerge.join.noconditionaltask does not exist  
Logging initialized using configuration in file:/etc/hive/conf/hive-log4j.properties  
HCAT_HOME not found, try to find hcatalog path from hadoop home  
A new Kylin instance is started by , stop it using "kylin.sh stop"  
Please visit http://<ip>:7070/kylin  
You can check the log at /usr/local/kylin/bin/..//logs/kylin.log  

7. 수 분 후 http://host:7070/kylin 주소로 웹 브라우저를 열 수 있습니다. 7070 포트는 "docker run"명령에서 지정한대로 contianer의 7070 포트로 리디렉션됩니다. 원하는대로 다른 포트로 변경할 수 있습니다.

8. 이제 Kylin을 사용할 수 있습니다. Hive 테이블 가져 오기, 큐브 디자인, 빌드, 쿼리 등

## Thanks
원문: https://github.com/sequenceiq/hadoop-docker/

