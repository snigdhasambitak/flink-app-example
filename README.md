# flink-app-example

simple flink app and kubernetes deployment configurations

## check java version

```
java -version

java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)

```

## install flink

If using macbook

```
brew install apache-flink

...
$ flink --version
Version: 1.2.0, Commit ID: 1c659cf
```

If using linux

Download a binary from the downloads page. You can pick any Scala variant you like. For certain features you may also have to download one of the pre-bundled Hadoop jars and place them into the /lib directory.
Go to the download directory.

Unpack the downloaded archive.

```
$ cd ~/Downloads        # Go to download directory
$ tar xzf flink-*.tgz   # Unpack the downloaded archive
$ cd flink-1.9.0
```

## start a Local Flink Cluster

```
./bin/start-cluster.sh  # Start Flink

Starting cluster.
Starting standalonesession daemon on host d-c02xw0aljgh7.
Starting taskexecutor daemon on host d-c02xw0aljgh7
```

If you have installed using brew then run:

```
/usr/local/Cellar/apache-flink/1.9.0/libexec/bin/start-cluster.sh
```

It is a good idea to write down the bin directory somewhere or so you can find it easily next time, or set it to path of your Mac OS X

```
export PATH=/usr/local/Cellar/apache-flink/1.9.0/libexec/bin:$PATH
source .bash_profile  // in case you use .bash_profile
```

## create a simple streaming job, that reads data from socket, and prints the count of words every 5 seconds

```
DataStream<Tuple2<String, Integer>> dataStream = env
    .socketTextStream("192.168.99.1", 9999)
    .flatMap(new Splitter())
    .keyBy(0)
    .timeWindow(Time.seconds(5))
    .sum(1);

dataStream.print();
```

IP 192.168.99.1 allows container to access services running on minikube host. For this example to work, you need to run the following on your host before creating the JobManager pod

```
nc -lk 9999

```


## build the  code using maven

```
mvn clean package
```

The compiled job jar can be found in target/flink-app-0.0.1-SNAPSHOT-jar-with-dependencies.jar


## run the example

First of all, we use netcat to start local server via

```
$ nc -l 9999
```

Submit the Flink program:

```
$ flink run target/flink-app-0.0.1-SNAPSHOT-jar-with-dependencies.jar --port 9999

WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.apache.flink.api.java.ClosureCleaner (file:/usr/local/Cellar/apache-flink/1.12.2/libexec/lib/flink-dist_2.12-1.12.2.jar) to field java.lang.String.value
WARNING: Please consider reporting this to the maintainers of org.apache.flink.api.java.ClosureCleaner
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Job has been submitted with JobID ead6c5087d7ddfb4f1a293cd34ba3c36
```

The program connects to the socket and waits for input. You can check the web interface to verify that the job is running as expected
