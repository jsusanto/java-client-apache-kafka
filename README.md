# java-client-apache-kafka
Java Client Application (Producer, Consumer) and Setting up Apache Kafka Linux Ubuntu
# Description
1. Setting up Apache Kafka Server on Amazon AWS Linux Ubuntu Free-Tier.
Reference: https://www.linkedin.com/pulse/kafka-aws-free-tier-steven-aranibar/
3. Java Client Application (Consumer) - Build using Netbeans IDE and Maven Project
Reference: https://www.conduktor.io/kafka/complete-kafka-consumer-with-java/
4. Java Client Application (Producer) - Build using Netbeans IDE and Maven Project
Reference: https://www.conduktor.io/kafka/complete-kafka-producer-with-java/
# Setting up Apache Kafka Server on Amazon AWS Linux Ubuntu Free-Tier
<h3>The Problem</h3>
By default, Kafka's config files require a gigabyte of memory and the free tier t2 micro instances only have a single gigabyte of memory for the whole instance! If you successfully install kafka, you won't get it up and running with the default config settings.
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/90331eb3-7b15-479f-9a98-5d87e42aa01b)
<h3>STEP 1 - Run Update and Upgrade in your current installed OS </h3>
<pre>
$sudo apt-get update

$sudo apt-get upgrade
</pre>
<h3>STEP 2 - Install Java</h3>
<pre>
$sudo apt-get install openjdk-8-jdk

ubuntu@ip-172-31-1-82:~$ java -version
openjdk version "1.8.0_382"
OpenJDK Runtime Environment (build 1.8.0_382-8u382-ga-1~22.04.1-b05)
OpenJDK 64-Bit Server VM (build 25.382-b05, mixed mode)
</pre>
<h3>Zookeeper and Kafka Installation</h3>
Kafka Installation, go to https://kafka.apache.org/downloads
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/494b2086-724d-4503-b0df-7a850a235af9)
We're using Binary Version (kafka_2.12-3.5.1.tgz), This java client should be compatible with any kafka version.

<pre>
  $wget https://downloads.apache.org/kafka/3.5.1/kafka_2.12-3.5.1.tgz

  $tar -zxvf kafka_2.12-3.5.1.tgz
</pre>

<b>Altering .bashrc file</b>
<pre>
  ubuntu@ip-172-31-1-82:~$ vi .bashrc  
</pre>
At the end of the config, add the following configurations:
<pre>
  # adding Kafka to PATH
export PATH=/home/ubuntu/kafka_2.12-3.5.1/bin:$PATH

# Kafka and zookeeper environment variables
export KAFKA_HEAP_OPTS=-Xms32M

export ZK_CLIENT_HEAP=128, ZK_SERVER_HEAP=128
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/c1e10ee6-d277-4ef9-baa7-43f480e3ff77)

<b>Testing or Validate</b>
If you run $kafka-topics.sh while not in the Kafka directory and then the $kafka-topics.sh options and descriptions appear: you were successful!
<pre>
  ubuntu@ip-172-31-1-82:~$ kafka-topics.sh
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/2b773409-fc25-47fb-8603-4ee7af814b67)
<h3>Altering Default Kafka Configurations</h3>
Create data/zookeeper/ directory
<pre>
  $mkdir data/kafka
  
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ ls data/
kafka  zookeeper
</pre>
Go to config/zookeeper.properties from Kafka directory. <br/>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/1efc1904-952b-449b-984e-1dbb16841647) <br/>
Find the dataDir configuration and replace with your newly created data folder.
<pre>
  dataDir=/home/ubuntu/kafka_2.12-3.5.1/data/zookeeper
</pre>
<h3>Edit the zookeeper-server-start.sh file in the kafka bin/ directory</h3>
<pre>
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ vi bin/zookeeper-server-start.sh
</pre>
Replace with a different value
<pre>
  if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    #export KAFKA_HEAP_OPTS="-Xmx512M -Xms512M"
    export KAFKA_HEAP_OPTS="-Xms32M -Xmx64M"
  fi
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/3fc7ed82-c37c-4a0d-8ea0-5b2360fada48)
<h3>Run Zookeeper</h3>
<pre>
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ zookeeper-server-start.sh config/zookeeper.properties
</pre>
If everything goes well you will see a message <br/>

INFO binding to port 0.0.0.0/0.0.0.0:2181
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/db0ac996-b0c8-48f7-bcbe-fefde04a8604)
<h3>Edit and Configure Kafka Data Folder and Properties</h3>
Now open up another window to your instance and create a Kafka sub directory in your newly made data/ folder the same way you did with Zookeeper. 
<br/>
Remember to get the full path name prior to using nano text editor.
<pre>
  ubuntu@ip-172-31-1-82:~$ cd kafka_2.12-3.5.1/
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ mkdir data/kafka
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ vi config/server.properties
</pre>
scroll to the log.dirs and change it to your new Kafka directory
<pre>
  log.dirs=/home/ubuntu/kafka_2.12-3.5.1/data/kafka
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/10da02b7-30c7-4881-88b1-e8bf76cb7969)
We'll edit the kafka-server-start.sh file found in the Kafka bin/ directory.
<pre>
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ vi bin/kafka-server-start.sh
</pre>
Update the following config <br/>
As we can see by the bit of code I've highlighted, the default memory is set to 1GB which our t2micro free instance cannot support. Change to:
<br/>
export KAFKA_HEAP_OPTS="-Xms32M -Xmx64M"
<pre>
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
    export KAFKA_HEAP_OPTS="-Xms32M -Xmx64M"
fi
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/191a99d6-a4f5-4827-b4ea-06e10f0fc806)
<h3>Run the Kafka</h3>
<pre>
  ubuntu@ip-172-31-1-82:~/kafka_2.12-3.5.1$ kafka-server-start.sh config/server.properties
</pre>
![image](https://github.com/jsusanto/java-client-apache-kafka/assets/1523220/a7a4b327-0ff0-42a7-8d40-8cc7dea6a41f)
Both Zookeeper and Kafka are running.
