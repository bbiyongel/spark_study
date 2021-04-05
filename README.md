# spark_study
all of spark 




cd .ssh 
rm -f known_hosts

sudo (관리자 권한으로 , super user do)




실습환경
- 환경이 다르다보니 trouble shooting이 안됨
virtual box centOS



hadoop대비 spark가 100배 빠름
- 경우에따라 3~4배



보통 yarn 위에서 flick, presto 돌림

ELK, casandra(NoSQL), hdfs, dw 하고도 가능

메모리기반 엔진 : flink, presto

RDD하고 spark dataframe은 기능,성능차이가 있는지요

DAG : Directed Acyclic Graph, 데이터 workflow를 의미

필터링 : map, 20명 학생 : 이름 영문명으로 변경
집계는 reduce -> count





신규 클러스터 구성해야되는 상황


장점 : 인메모리(캐싱) 기반 - 빠름
cf. MR은 spark과 비교됨
알아서 분산 병렬 처리함


입문수준 vs 실무에서 고민하는것

AWS EC2
- VM instance 4개
(분산처리환경 : 머신이 여러개)


csv : comma sepetate value
tsv : tab


cd /kikang
ls -alh
https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/HG7NV7
http://stat-computing.org/dataexpo/

python2는 디프리케이트됨 : 지금존재하나 나중에 없어질꺼다 
한줄입력하는건 interpreter 언어 : CLI 인터펙티브함


import glob
import pandas as pd
import time
start_time = time.time()
files = glob.glob("/kikang/data/airline_on_time/2008.csv")
df = pd.DataFrame()
for f in files:
    df = df.append(pd.read_csv(f))

df = df.reset_index(drop=True)
df.groupby([df.Year, df.Month]).agg({'FlightNum':'size','ArrDelay':['sum','mean','max', 'min']})
elapsed_time = time.time() - start_time
print (time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))

00:00:23



du -sh 
6.5GB

top
1
m

즉는다 

spark single machine 도 됨

https://mirrors.sonic.net/apache/spark/spark-3.1.1/spark-3.1.1-bin-hadoop3.2.tgz

https://apache.claz.org/spark/spark-3.1.1/spark-3.1.1-bin-hadoop3.2.tgz

wget 

tar svfx spark-3.1.1-bin-hadoop3.2.tgz

cd spark3. /bin

pyspark

./pyspark

import pyspark.sql.functions as fn
import time
start_time = time.time()
df = spark.read.option("header", True).option("inferSchema", False).csv("file:///kikang/data/airline_on_time/2008.csv")
df.groupBy(df.Year, df.Month).agg(fn.count(df.FlightNum), fn.sum(df.ArrDelay), fn.avg(df.ArrDelay), fn.max(df.ArrDelay.cast("double")), fn.min(df.ArrDelay.cast("double"))).orderBy(df.Year.cast("integer").asc(), df.Month.cast("integer").asc()).toDF("Year", "Month", "FlightNum_Cnt", "ArrDelay_Sum", "ArrDelay_Avg", "ArrDelay_Max", "ArrDelay_Min").show(1000)
elapsed_time = time.time() - start_time
print (time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))


import time
start_time = time.time()
df = spark.read.option("header", True).option("inferSchema", True).csv("file:///kikang/data/airline_on_time/*.csv")
df.createOrReplaceTempView("bigdata")
sql("select Year, Month, count(FlightNum) as FlightNum_Cnt, sum(ArrDelay) as ArrDelay_Sum, avg(ArrDelay) as ArrDelay_Avg, max(CAST(ArrDelay AS DOUBLE)) as ArrDelay_Max, min(CAST(ArrDelay AS DOUBLE)) as ArrDelay_Min from bigdata where 1=1 group by Year, Month order by Year, Month").show(100)
elapsed_time = time.time() - start_time
print (time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))


<2일차>

java -version

스칼라

pyspark : 주피터 노트북 세팅 
R서버에서 사용하는 방법 
제플린 : spark에 특화된 web notebook
     : python, r, scala 모두 됨 
	 
apt로 설치하고 환경구성하면됨 

yarn 기동 
/kikang/hadoop3/sbin/start-all.sh

spark@spark-master-01:/kikang$ /kikang/hadoop3/sbin/start-all.sh
WARNING: Attempting to start all Apache Hadoop daemons as spark in 10 seconds.
WARNING: This is not a recommended production deployment configuration.
WARNING: Use CTRL-C to abort.
Starting namenodes on [spark-master-01]
Starting datanodes
Starting secondary namenodes [spark-master-01]
Starting resourcemanager
Starting nodemanagers
spark@spark-master-01:/kikang$

네임도드 기동


spark@spark-master-01:/kikang$ jps
1985 ResourceManager
2275 Jps
1811 SecondaryNameNode
1546 NameNode
spark@spark-master-01:/kikang$

워커
spark@spark-worker-01:~$ jps
1619 NodeManager
1715 Jps
1437 DataNode
spark@spark-worker-01:~$

yarn관련 리소스매니저,노드매니저 기동

역할 다시 리뷰 : 

웹ui

35.80.21.18	mnode
34.223.66.211 worker1
44.233.150.74 worker2
44.242.147.33 worker3

35.80.21.18	spark-master-01
34.223.66.211 spark-worker-01
44.233.150.74 spark-worker-02
44.242.147.33 spark-worker-03


원래 mnode:5017
http://mnode:50170/
http://spark-master-01:50170

원래 http://mnode:8088/
http://mnode:8188/cluster
http://spark-master-01:8188/cluster



http://spark-master-01:4040


싱글머신 코어2개 : 9분
>>> 꺽쇄 3개는 파이썬 환경 
exit()

cd /kikang/spark3
bin/pyspark --help


option
 --master url : spark:/host:port  --> standalone 
                mesos://
                yarn  => 추가설정 필요 
                k8s:// => 쿠버네티스 
				디폴트는 local (분산아님 병렬)
		

bin/pyspark --master yarn => 에러남. 환경설정파일만들어야함

mkdir -p  conf2 (없으면 만들어라)
cd conf2
하둡설정정보 참고해서 만들것임

cd /kikang/hadoop3/etc/hadoop


yarn-site.xml


cp /kikang/hadoop3/etc/hadoop/core-site.xml /kikang/spark3/conf2
cp /kikang/hadoop3/etc/hadoop/yarn-site.xml /kikang/spark3/conf2



YARN_CONF_DIR=/kikang/spark3/conf2 bin/pyspark --master yarn		

YARN_CONF_DIR=/kikang/spark3/conf2 bin/pyspark --master yarn

YARN_CONF_DIR=/kikang/spark3/conf2 ./bin/pyspark --master yarn
 
에러남 : 확인필요
py4j.protocol.Py4JJavaError: An error occurred while calling None.org.apache.spark.api.java.JavaSparkContext.
: org.apache.spark.SparkException: Application application_1616808765005_0004 failed 2 times due to AM Container for appattempt_1616808765005_0004_000002 exited with  exitCode: -1000
Failing this attempt.Diagnostics: [2021-03-27 02:47:37.652]File file:/home/spark/.sparkStaging/application_1616808765005_0004/pyspark.zip does not exist
java.io.FileNotFoundException: File file:/home/spark/.sparkStaging/application_1616808765005_0004/pyspark.zip does not exist

hdfs

file:/ -> 로컬파일시스템
hdfs:/ -> HDFS




YARN_CONF_DIR=/kikang/spark3/conf2 bin/pyspark --master yarn --executor-memory 2G --executor-cores 2 --num-executors 4

YARN_CONF_DIR=/kikang/spark3/conf2 bin/pyspark --master yarn --executor-memory 2G --executor-cores 4 --num-executors 3


worker에 spark이 설치안된 상태임 


/user/spark/.sparkStaging 여기에 모든 라이브러리(kikang/spark3)를 가져다놓고 실행함 
작업종료되면 staging은 지워짐

100대서버에 spark설치 안해도 되고 기동하는 driver만 설치하면됨

hdfs://spark-master-01:9000

앞에 hdfs 빼도 경로 hdfs로 인식함

import pyspark.sql.functions as fn
import time
start_time = time.time()
df = spark.read.option("header", True).option("inferSchema", False).csv("hdfs://spark-master-01:9000/kikang/data/airline_on_time/2008.csv")
df.groupBy(df.Year, df.Month).agg(fn.count(df.FlightNum), fn.sum(df.ArrDelay), fn.avg(df.ArrDelay), fn.max(df.ArrDelay.cast("double")), fn.min(df.ArrDelay.cast("double"))).orderBy(df.Year.cast("integer").asc(), df.Month.cast("integer").asc()).toDF("Year", "Month", "FlightNum_Cnt", "ArrDelay_Sum", "ArrDelay_Avg", "ArrDelay_Max", "ArrDelay_Min").show(1000)
elapsed_time = time.time() - start_time
print (time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))


 컨트롤 + 링크 : 새탭
 

scp -r /kikang/data spark-worker-01:/kikang

scp -r /kikang/data spark-worker-02:/kikang

scp -r /kikang/data spark-worker-03:/kikang

사실 이렇게 보내지말고 hdfs에 보내면됨(3copy)

dfs 

spark@spark-master-01:/kikang/hadoop3/bin$ hdfs dfs -mkdir /kikang
 ./hdfs dfs -mkdir /kikang
 
 ./hdfs dfs -put /kikang/data/ /kikang/
 



병렬로하면 2분걸리

단일노드에서 병렬처리가 가장빠름 : 네트워크 부하가 크기때문


파일 실행된거 보면 

파일 range별로 읽어서 실행 (driver가 계산해서 줌. 128M 오프셋까지 단위처리)

shuffle : word count 에서 설명 


---------
3일차 4/3

pyspark는 싱글머신에서도 한줄한줄 처리하므로 처리가능
- 아무리 큰 파일이라도 한줄한줄 접근하면 느리지만 언젠가는 처리됨
- 메모리에 다 올리지않고 한줄씩 올린다는 의미
- 파이썬은 모두 메모리에 올림
- out of memory가 날수도 있긴함

stand alone 클러스터 리소스 매니저 띄우기 
stand alone 은 yarn 대신하는 기능 

=> 노드에 spark깔아야함


scp -r /kikang/spark3 spark-worker-01:/kikang/


jps

cd spark3/sbin
./start-master.sh 


spark@spark-master-01:/kikang/spark3/sbin$ ./start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /kikang/spark3/logs/spark-spark-org.apache.spark.deploy.master.Master-1-spark-master-01.out


spark@spark-master-01:/kikang/spark3/sbin$ jps
5856 Jps
5807 Master


/kikang/spark3/logs/spark-spark-org.apache.spark.deploy.master.Master-1-spark-master-01.out



http://spark-master-01:8080/ (디폴트 메모리)
http://spark-master-01:8180 (변경)
7177

spark@spark-master-01:~$ cd /kikang/spark3/conf
spark@spark-master-01:/kikang/spark3/conf$ ll
total 44
drwxr-xr-x  2 spark spark 4096 Feb 22 02:11 ./
drwxr-xr-x 16 spark spark 4096 Apr  3 01:41 ../
-rw-r--r--  1 spark spark 1105 Feb 22 02:11 fairscheduler.xml.template
-rw-r--r--  1 spark spark 2023 Feb 22 02:11 log4j.properties.template
-rw-r--r--  1 spark spark 9141 Feb 22 02:11 metrics.properties.template
-rw-r--r--  1 spark spark 1292 Feb 22 02:11 spark-defaults.conf.template
-rwxr-xr-x  1 spark spark 4428 Feb 22 02:11 spark-env.sh.template*
-rw-r--r--  1 spark spark  865 Feb 22 02:11 workers.template
spark@spark-master-01:/kikang/spark3/conf$


환경변수 

spark-env.sh.template

cp spark-env.sh.template spark-env.sh 

vi spark-env.sh 

: set nu

# add by jyw 
SPARK_MASTER_PORT=7177
SPARK_MASTER_WEBUI_PORT=8180

spark@spark-worker-01:/kikang/spark3/sbin$ ./stop-master.sh
no org.apache.spark.deploy.master.Master to stop
spark@spark-worker-01:/kikang/spark3/sbin$ ./start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /kikang/spark3/logs/spark-spark-org.apache.spark.deploy.master.Master-1-spark-worker-01.out


spark@spark-master-01:/kikang/spark3/sbin$ ./start-worker.sh
Usage: ./sbin/start-worker.sh <master> [options]
21/04/03 02:00:22 INFO SignalUtils: Registering signal handler for TERM
21/04/03 02:00:22 INFO SignalUtils: Registering signal handler for HUP
21/04/03 02:00:22 INFO SignalUtils: Registering signal handler for INT

Master must be a URL of the form spark://hostname:port

Options:
  -c CORES, --cores CORES  Number of cores to use
  -m MEM, --memory MEM     Amount of memory to use (e.g. 1000M, 2G)
  -d DIR, --work-dir DIR   Directory to run apps in (default: SPARK_HOME/work)
  -i HOST, --ip IP         Hostname to listen on (deprecated, please use --host or -h)
  -h HOST, --host HOST     Hostname to listen on
  -p PORT, --port PORT     Port to listen on (default: random)
  --webui-port PORT        Port for web UI (default: 8081)
  --properties-file FILE   Path to a custom Spark properties file.
                           Default is conf/spark-defaults.conf.
spark@spark-master-01:/kikang/spark3/sbin$


./start-worker.sh spark://spark-master-01:7177


hostname 

 
# add by jyw
SPARK_MASTER_PORT=7177
SPARK_MASTER_WEBUI_PORT=8180

SPARK_WORKER_PORT=40001
SPARK_WORKER_WEBUI_PORT=8181
SPARK_PUBLIC_DNS=${hostname}



spark@spark-master-01:/kikang/spark3/sbin$ ./stop-master.sh
stopping org.apache.spark.deploy.master.Master
spark@spark-master-01:/kikang/spark3/sbin$ scp /kikang/spark3/conf/spark-env.sh spark-worker-01:/kikang/spark3/conf/spark-env.sh
spark-env.sh                                  100% 4582     8.5MB/s   00:00
spark@spark-master-01:/kikang/spark3/sbin$ scp /kikang/spark3/conf/spark-env.sh spark-worker-02:/kikang/spark3/conf/spark-env.sh
spark-env.sh                                  100% 4582     5.7MB/s   00:00
spark@spark-master-01:/kikang/spark3/sbin$ scp /kikang/spark3/conf/spark-env.sh spark-worker-03:/kikang/spark3/conf/spark-env.sh
spark-env.sh                                  100% 4582     9.1MB/s   00:00
spark@spark-master-01:/kikang/spark3/sbin$




spark@spark-worker-01:~$ cd /kikang/spark3/sbin
spark@spark-worker-01:/kikang/spark3/sbin$ ./start-worker.sh spark://spark-master-01:7177
starting org.apache.spark.deploy.worker.Worker, logging to /kikang/spark3/logs/spark-spark-org.apache.spark.deploy.worker.Worker-1-spark-worker-01.out



spark@spark-master-01:/kikang/spark3/sbin$ ./start-worker.sh -h
Usage: ./sbin/start-worker.sh <master> [options]
21/04/03 02:22:47 INFO SignalUtils: Registering signal handler for TERM
21/04/03 02:22:47 INFO SignalUtils: Registering signal handler for HUP
21/04/03 02:22:47 INFO SignalUtils: Registering signal handler for INT

Master must be a URL of the form spark://hostname:port

Options:
  -c CORES, --cores CORES  Number of cores to use
  -m MEM, --memory MEM     Amount of memory to use (e.g. 1000M, 2G)
  -d DIR, --work-dir DIR   Directory to run apps in (default: SPARK_HOME/work)
  -i HOST, --ip IP         Hostname to listen on (deprecated, please use --host or -h)
  -h HOST, --host HOST     Hostname to listen on
  -p PORT, --port PORT     Port to listen on (default: random)
  --webui-port PORT        Port for web UI (default: 8081)
  --properties-file FILE   Path to a custom Spark properties file.
                           Default is conf/spark-defaults.conf.
spark@spark-master-01:/kikang/spark3/sbin$


starting org.apache.spark.deploy.worker.Worker, logging to /kikang/spark3/logs/spark-spark-org.apache.spark.deploy.worker.Worker-1-spark-worker-01.out
spark@spark-worker-01:/kikang/spark3/sbin$


/kikang/spark3/sbin/start-worker.sh spark://spark-master-01:7177 -c 8 -m 8G

통신포트확인
spark@spark-worker-03:~$ lsof -i
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    5935 spark  298u  IPv6  66475      0t0  TCP spark-worker-03:40001 (LISTEN)
java    5935 spark  307u  IPv6  66583      0t0  TCP *:8181 (LISTEN)
java    5935 spark  312u  IPv6  66586      0t0  TCP spark-worker-03:40094->spark-master-01:7177 (ESTABLISHED)
spark@spark-worker-03:~$







spark@spark-master-01:/kikang/spark3$ bin/pyspark -h
Usage: ./bin/pyspark [options]

Options:
  --master MASTER_URL         spark://host:port, mesos://host:port, yarn,
                              k8s://https://host:port, or local (Default: local[*]).
  --deploy-mode DEPLOY_MODE   Whether to launch the driver program locally ("client") or
                              on one of the worker machines inside the cluster ("cluster")
                              (Default: client).
  --class CLASS_NAME          Your application's main class (for Java / Scala apps).
  --name NAME                 A name of your application.
  --jars JARS                 Comma-separated list of jars to include on the driver
                              and executor classpaths.
  --packages                  Comma-separated list of maven coordinates of jars to include
                              on the driver and executor classpaths. Will search the local
                              maven repo, then maven central and any additional remote
                              repositories given by --repositories. The format for the
                              coordinates should be groupId:artifactId:version.
  --exclude-packages          Comma-separated list of groupId:artifactId, to exclude while
                              resolving the dependencies provided in --packages to avoid
                              dependency conflicts.
  --repositories              Comma-separated list of additional remote repositories to
                              search for the maven coordinates given with --packages.
  --py-files PY_FILES         Comma-separated list of .zip, .egg, or .py files to place
                              on the PYTHONPATH for Python apps.
  --files FILES               Comma-separated list of files to be placed in the working
                              directory of each executor. File paths of these files
                              in executors can be accessed via SparkFiles.get(fileName).
  --archives ARCHIVES         Comma-separated list of archives to be extracted into the
                              working directory of each executor.

  --conf, -c PROP=VALUE       Arbitrary Spark configuration property.
  --properties-file FILE      Path to a file from which to load extra properties. If not
                              specified, this will look for conf/spark-defaults.conf.

  --driver-memory MEM         Memory for driver (e.g. 1000M, 2G) (Default: 1024M).
  --driver-java-options       Extra Java options to pass to the driver.
  --driver-library-path       Extra library path entries to pass to the driver.
  --driver-class-path         Extra class path entries to pass to the driver. Note that
                              jars added with --jars are automatically included in the
                              classpath.

  --executor-memory MEM       Memory per executor (e.g. 1000M, 2G) (Default: 1G).

  --proxy-user NAME           User to impersonate when submitting the application.
                              This argument does not work with --principal / --keytab.

  --help, -h                  Show this help message and exit.
  --verbose, -v               Print additional debug output.
  --version,                  Print the version of current Spark.

 Cluster deploy mode only:
  --driver-cores NUM          Number of cores used by the driver, only in cluster mode
                              (Default: 1).

 Spark standalone or Mesos with cluster deploy mode only:
  --supervise                 If given, restarts the driver on failure.

 Spark standalone, Mesos or K8s with cluster deploy mode only:
  --kill SUBMISSION_ID        If given, kills the driver specified.
  --status SUBMISSION_ID      If given, requests the status of the driver specified.

 Spark standalone, Mesos and Kubernetes only:
  --total-executor-cores NUM  Total cores for all executors.

 Spark standalone, YARN and Kubernetes only:
  --executor-cores NUM        Number of cores used by each executor. (Default: 1 in
                              YARN and K8S modes, or all available cores on the worker
                              in standalone mode).

 Spark on YARN and Kubernetes only:
  --num-executors NUM         Number of executors to launch (Default: 2).
                              If dynamic allocation is enabled, the initial number of
                              executors will be at least NUM.
  --principal PRINCIPAL       Principal to be used to login to KDC.
  --keytab KEYTAB             The full path to the file that contains the keytab for the
                              principal specified above.

 Spark on YARN only:
  --queue QUEUE_NAME          The YARN queue to submit to (Default: "default").

spark@spark-master-01:/kikang/spark3$


spark@spark-master-01:/kikang/spark3$ bin/pyspark --master spark://spark-master-01:7177
Python 3.8.5 (default, Jan 27 2021, 15:41:15)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
21/04/03 02:46:18 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 3.1.1
      /_/

Using Python version 3.8.5 (default, Jan 27 2021 15:41:15)
Spark context Web UI available at http://spark-master-01:4040
Spark context available as 'sc' (master = spark://spark-master-01:7177, app id = app-20210403024621-0000).
SparkSession available as 'spark'.
>>> sc.master






core 24 모두 할당 
memory 는 1G씩 할당 



21/04/03 03:00:41 WARN TaskSchedulerImpl: Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources





bin/pyspark --master spark://spark-master-01:7177 --executor-cores 2 --executor-memory 2g --total-executor-cores 4



spark@spark-worker-02:~$ jps
6708 Jps
6406 Worker
6571 CoarseGrainedExecutorBackend

bin/pyspark --master spark://spark-master-01:7177 --executor-cores 2 --executor-memory 2g --total-executor-cores 4 --name jyw




spark@spark-worker-02:~$ cd /kikang/spark3/work
spark@spark-worker-02:/kikang/spark3/work$ ll
total 12
drwxrwxr-x  3 spark spark 4096 Apr  3 02:46 ./
drwxr-xr-x 17 spark spark 4096 Apr  3 02:40 ../
drwxrwxr-x  5 spark spark 4096 Apr  3 03:07 app-20210403024621-0000/
spark@spark-worker-02:/kikang/spark3/work$



spark stand alone으로 구성하면 빠르고 가볍고 좋은데 버전호환성이 없음
- 최신버전 spark 3.1.1 
spark2 버전으로 과거버전 app은 어떻게 돌리면 되나?

spark에서 최소 8core 8giga 권장 

spark.apache.org/docs/latest/
api scala


sql 사용할때 functions 

 sql.functions 
 
 
 낮은 버전에서 높은 버전 function이나 api 못쓰나요?
 
 dayofweek 요일, 공휴일 sunday1, => java은 월요일이 1임 
 dayofmonth 2월 28일까지 있다 
 
 
 2.2.5였는데 
 
 
 timestamp 
 
 
 없는건 custom 함수로 만들어서 쓸수있다
 
 
 spark2 버전 받아서 호환성 한번 보자 
 
 archive.apache.org/dist/spark/
 
 
 우린 spark 1.6인데 
 
 spark : 버클리 BSD 라이센스 => 아파치 라이센스로 변경됨 
 
 
 wget https://archive.apache.org/dist/spark/spark-2.2.3/spark-2.2.3-bin-hadoop2.7.tgz

tar -xvf XXX

mv spark-2.2.3-bin-hadoop2.7/ spark2



spark@spark-master-01:/kikang/spark2$ bin/pyspark --master spark://spark-master-01:7177 --excutor-cores 2
bin/pyspark: line 45: python: command not found
Exception in thread "main" java.lang.IllegalArgumentException: pyspark does not support any application options.
        at org.apache.spark.launcher.CommandBuilderUtils.checkArgument(CommandBuilderUtils.java:241)
        at org.apache.spark.launcher.SparkSubmitCommandBuilder.buildPySparkShellCommand(SparkSubmitCommandBuilder.java:288)
        at org.apache.spark.launcher.SparkSubmitCommandBuilder.buildCommand(SparkSubmitCommandBuilder.java:147)
        at org.apache.spark.launcher.Main.main(Main.java:86)
spark@spark-master-01:/kikang/spark2$



문서찾는법 
more > configuration 

spark.pyspark.driver.python 

spark.pyspark.python =>요거 설정 


spark@spark-master-01:/kikang/spark2$ bin/pyspark --master spark://spark-master-01:7177 --excutor-cores 2 --conf spark.pyspark.python=python3


결론은 : 실행안됨 
 /kikang/spark2/bin/pyspark --master spark://spark-master-01:7177 --executor-cores 2 --conf spark.pyspark.python=python3

 /kikang/spark3/bin/pyspark --master spark://spark-master-01:7177 --executor-cores 2 --conf spark.pyspark.python=python3



3.1.1
메이저버전-마이너버전-버그픽스용


yarn은 버전 안타나? 

/kikang/hadoop3/sbin/start-all.sh

http://spark-master-01:50170/dfshealth.html#tab-overview

http://spark-master-01:8188/cluster/apps    : yarn 

전사에서 사용하는 클러스터라면 
- 버전이 다른게 돌고 있음 
- yarn이 각 버전을 가져와서 실행하므로 문제없음


YARN_CONF_DIR=/kikang/spark3/conf2

/kikang/spark2/bin/pyspark --master yarn --executor-cores 2 --executor-memory 2G --num-executors 3 --conf spark.pyspark.python=python3

/kikang/spark3/bin/pyspark --master yarn --executor-cores 2 --executor-memory 2G --num-executors 3 --conf spark.pyspark.python=python3 --name second_spark3
/kikang/spark3/bin/pyspark --master yarn --executor-cores 2 --executor-memory 2G --num-executors 3 --conf spark.pyspark.python=python3 --name fastcampus

>>> sql("select current_date()").show()
>>> sql("select dayofweek(current_date())").show()

버전학인 
>>> sc.version

>>> sc.tab

sc.appName 
sc.app
 
 
 
jypyter-notebook
rstudio-server
zeppelin


aws : 목6시~일6시


