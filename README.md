# Spark Structured Streaming 
## Real-Time Data Processing Pipeline

---

##  Table of Contents

1. [Project Overview](#project-overview)
4. [Project Structure](#project-structure)
5. [Deployment Guide](#deployment-guide)
6. [HDFS Data Management](#hdfs-data-management)
7. [Spark Streaming Application](#spark-streaming-application)
8. [Running the Application](#running-the-application)
9. [Monitoring & Verification](#monitoring--verification)


---

##  Project Overview

This implementation showcases an end-to-end Big Data solution that combines:

- **Hadoop HDFS** for scalable distributed file storage
- **Apache Spark Standalone** cluster for stream processing
- **Structured Streaming** for real-time data consumption
- **Docker containerization** for simplified infrastructure setup

### Key Features

-  Containerized Hadoop HDFS infrastructure (NameNode + DataNode)  
-  Containerized Spark Standalone environment (Master + Workers)  
-  Continuous CSV file processing via Spark Structured Streaming  
-  Dynamic file discovery in HDFS  
-  Maven-based Java Spark application  
-  Real-time console output display

### Data Flow

```
CSV Files → HDFS Storage → Spark Structured Streaming → Console Output
```

---

## Project structure
<img width="435" height="485" alt="image" src="https://github.com/user-attachments/assets/723df133-b15b-4dcf-aa79-0246ee79e934" />




### Container Services

| Container | Role | Ports | Description |
|-----------|------|-------|-------------|
| `namenode` | HDFS NameNode | 9870, 8020 | Handles file system metadata operations |
| `datanode` | HDFS DataNode | - | Manages data block storage |
| `resourcemanager` | YARN ResourceManager | 8088 | Controls cluster resource allocation |
| `nodemanager` | YARN NodeManager | - | Handles task execution on nodes |
| `spark-master` | Spark Master | 7077, 8080 | Orchestrates Spark job execution |
| `spark-worker-1` | Spark Worker | - | Performs Spark task processing |

---

##  Prerequisites

### Required Software

- **Docker** (v20.10+)
- **Docker Compose** (v2.0+)

### Verify Installation

```bash
docker --version
docker-compose --version

```

---

### Maven Dependencies (`pom.xml`)

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.5.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>3.5.0</version>
        </dependency>


    </dependencies>
```

---

##  Deployment Guide

### Step 1: Start Docker Containers

```bash

# Start all services
docker-compose up -d

# Verify all containers are running
docker ps
```

output:
<img width="1350" height="567" alt="image" src="https://github.com/user-attachments/assets/638172b3-5280-460e-b58d-540b087774c7" />



### Step 2: Verify Web Interfaces

| Service | URL | Purpose |
|---------|-----|---------|
| HDFS NameNode | http://localhost:9870 | Browse files in distributed storage |
| Spark Master | http://localhost:8080 | View cluster status and jobs |
| YARN ResourceManager | http://localhost:8088 | Monitor resource allocation |

---

##  HDFS Data Management

### Access NameNode Container

```bash
docker exec -it tp5-spark-structured-streaming-main-namenode-1 bash
```

### Create HDFS Directory

```bash
hdfs dfs -mkdir -p /data
```

### Upload CSV Files to HDFS

```bash
# Upload single file
hdfs dfs -put -f /data/orders1.csv /data/

# Upload multiple files
hdfs dfs -put -f /data/orders*.csv /data/
```

### Verify Upload

```bash
# List files in HDFS
hdfs dfs -ls /data

# Preview file content
hdfs dfs -cat /data/orders1.csv | head -10

# Check file size
hdfs dfs -du -h /data
```

###  Output
<img width="1032" height="618" alt="image" src="https://github.com/user-attachments/assets/91978a90-9592-402d-951f-2db195395d09" />



---

##  Spark Streaming Application

### Application Code (`Main.java`)

```java
package org.example;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.streaming.OutputMode;
import org.apache.spark.sql.streaming.StreamingQuery;
import org.apache.spark.sql.streaming.StreamingQueryException;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

public class Main {
    public static void main(String[] args) throws Exception {
        SparkSession ss=SparkSession.builder()
                .appName("Structured streaming App")
                .master("spark://spark-master:7077")
                .getOrCreate();
        ss.sparkContext().setLogLevel("ERROR");
        StructType schema = new StructType(new StructField[]{
                new StructField("order_id",DataTypes.LongType,false, Metadata.empty()),
                new StructField("client_id",DataTypes.LongType,false, Metadata.empty()),
                new StructField("client_name",DataTypes.StringType,false, Metadata.empty()),
                new StructField("product",DataTypes.StringType,false, Metadata.empty()),
                new StructField("quantity",DataTypes.IntegerType,false, Metadata.empty()),
                new StructField("price",DataTypes.DoubleType,false, Metadata.empty()),
                new StructField("order_date",DataTypes.StringType,false, Metadata.empty()),
                new StructField("status",DataTypes.StringType,false, Metadata.empty()),
                new StructField("total",DataTypes.DoubleType,false, Metadata.empty())
        });
        Dataset<Row> inputDF = ss.readStream().schema(schema).option("header",true).csv("hdfs://namenode:8020/data");
        StreamingQuery query= inputDF.writeStream().format("console")
                .outputMode(OutputMode.Append())
                .start();
        query.awaitTermination();
    }
}
```



##  Running the Application

### Step 1: Copy JAR to Spark Master

```bash
docker cp .\target\structured-Streaming-Spark-1.0-SNAPSHOT.jar spark-master:/opt/ 
```

### Step 2: Access Spark Master Container

```bash
docker exec -it spark-master bash
```

### Step 3: Submit Spark Job

```bash
/opt/spark/bin/spark-submit  --class org.example.Main --master spark://spark-master:7077 /opt/structured-Streaming-Spark-1.0-SNAPSHOT.jar
```

### Command Parameters Explained

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--master` | `spark://spark-master:7077` | URL of the Spark cluster master node |
| `--class` | `org.example.Main` | Fully qualified name of the entry class |


---

##  Monitoring & Verification

### Console Output Example

<img width="1251" height="614" alt="image" src="https://github.com/user-attachments/assets/471cdf80-6c18-4dff-867c-0727e96464fe" />




## Real-Time Streaming Demonstration

### Test Continuous Processing

1. **Keep the Spark job running**

2. **In a new terminal, add a new CSV file:**

```bash
docker exec -it namenode bash
hdfs dfs -put -f /data/orders4.csv /data/
```

3. **Observe automatic processing** in Spark console:

<img width="1281" height="485" alt="image" src="https://github.com/user-attachments/assets/9f04d06b-330c-49be-bc00-c3e4501c6c99" />


### How It Works

1. **File Detection:** Spark monitors the `/data` directory in HDFS continuously
2. **Automatic Processing:** Newly added files are detected and processed immediately
3. **Schema Validation:** Every file is checked against the predefined schema structure
4. **Output Generation:** Processing results are displayed in the console in real-time


