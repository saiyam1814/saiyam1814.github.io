---
layout: post
title:  "How to Overcome Memory Usage Challenges with the Time Series Index"
date:   2019-09-03
categories:  Time Series
---




[Source](https://thenewstack.io/how-to-overcome-memory-usage-challenges-with-the-time-series-index/ "Permalink to How to Overcome Memory Usage Challenges with the Time Series Index")

# How to Overcome Memory Usage Challenges with the Time Series Index

[InfluxData][1] sponsored this post.

InfluxDB is a leading open source [time series databases][2]. In case you're unfamiliar with InfluxDB, it is designed to be fast but it uses an in-memory index, which comes at the cost of RAM usage as your datasets grow. So, for optimum performance and RAM usage, InfluxData introduced a special indexing mechanism for InfluxDB called [time series index (TSI)][3]. TSI optimizes the RAM usage saturation for larger data sets.

InfluxData supports customers using InfluxDB with tens of millions of time series data points. InfluxData's goal, however, is to expand this capability to hundreds of millions, and eventually, billions of data points. This is why InfluxData has added the new TSI. The aim is to support a large number of time series (a very high cardinality in the number of unique time series that the database stores). Cardinality is a measurement of unique series in your database.

## The Time Series Index (TSI) in Context

[

![][4]

Saiyam Pathak

Saiyam is software engineer of the large-scale multi-cloud Kubernetes project, and works on Kubernetes at Walmart Labs with a focus on creating and managing the project ecosystem. Previously at HP and Oracle, Saiyam has worked on many facets of Kubernetes including scaling, multicloud, managed kubernetes services, K8s documentation and testing. He's also been instrumental in implementing the major managed services (GKE/AKS/OKE) in different organizations.

][5]

With InfluxData's TSI storage engine, users are able to have millions of unique time series. As the [_documentation][6]_ states, the goal is that the number of series should be unaffected by the amount of memory the server hardware has. Importantly, the number of series that exist in the database should also ideally have a negligible impact on database startup time. The development of the TSI  thus represents the most significant technical advancement in the database since InfluxData released the TSM storage engine in 2016.

As mentioned before, when InfluxDB ingests data, it stores not only the value but also indexes the measurement and tag information so that it can be queried quickly.  In earlier versions, indexed — data could only be stored in-memory — and again, that requires a lot of RAM and places an upper bound on the number of series a machine can hold. The TSI was developed to allow it to surpass that upper bound. TSI stores index data on disk so that we are no longer restricted by RAM. TSI uses the operating system's page cache to pull hot data into memory and let cold data rest on disk.

In this article, we focus on converting TSM storage engine in-memory indexing to time series index (TSI) indexing. This conversion improves performance and reduces memory problems. We will also provide some tips to reduce the memory overload for InfluxDB. Containers are used for this article.

Let's begin by discussing the default installation of InfluxData's open source platform, the [TICK Stack][7] (an acronym of the platform's components Telegraf, InfluxDB, Chronograf and Kapacitor).

Prerequisites:

* **Virtual machine with Docker installed:** If you haven't installed Docker before, you can do that with official Docker [documentation][8].
* **TICK Stack installation:** Before we explain the conversion from TSM to TSI indices, we first need to have the TICK Stack installed. The TICK Stack is an open source platform by InfluxData consisting of Telegraf, InfluxDB, Chronograf and Kapacitor.

## Platform Component Definitions and Installation

**InfluxDB:** InfluxDB is a time series database built from the ground up to handle high write and query loads. InfluxDB is meant to be used as a backing store for any use case involving large amounts of time-stamped data, including DevOps monitoring, application metrics, IoT sensor data and real-time analytics.

**Installation:**  
> docker network create influxdb

![][9]

Docker network create.

> docker run -d –name=influxdb -p 8086:8086 -v $PWD:/var/lib/influxdb –net=influxdb –restart=always influxdb

![][10]

InfluxDB is running.

**Telegraf:** Telegraf is an open source agent written in Go for collecting metrics and data on the system it's running on or from other services. Telegraf writes data that it collects to InfluxDB in the correct format.

## Installation

Dumping the **telegraf.conf** file to enable docker monitoring and changing the influx endpoint.

`> docker run --rm telegraf telegraf config > telegraf.conf`

Edit the telegraf.conf file with the InfluxDB container name, "influxdb," as the host for the URL update. For enabling the docker monitoring, uncomment the docker endpoint.

![][11]

![][12]

Uncomment endpoint.

**Run the Telegraf container:**

`> docker run -d --name=telegraf --net=influxdb -v /var/run/docker.sock:/var/run/docker.sock -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro telegraf`

![][13]

Telegraf is running.

**Chronograf**: Chronograf is InfluxData's open source web application. Use Chronograf with the other components of the TICK Stack for alert management, data visualization and database management.

**Installation:**

`> docker run -d --name=chronograf -p 8000:8888 --net=influxdb --restart=always chronograf --influxdb-url`

![][14]

Now that we have the TICK Stack running, how do we see if TSM or TSI is being used? To check whether the index version of Influx is in-memory or TSI, we can simply run the following command:

![][15]

TICK Stack is running.

`> docker logs influxdb | grep index_version`

![][16]

As you can see above, the "index_version=inmem" shows that InfluxDB does not have TSI enabled.

**The Five Steps to Enable TSI Indices**

There is a five-step procedure to enable TSI indices for InfluxDB container.

Step 1: Stop the container:

`docker stop influxdb`

`docker rm influxdb`

![][17]

Removing the container.

Step 2: Start container again but with entrypoint as a Bash and pass the environment variable to enable TSI index version.

`> sudo docker run -it --name influx-db --restart unless-stopped `

`-e INFLUXDB_DATA_INDEX_VERSION="tsi1" `

`-v $PWD:/var/lib/influxdb `

`\--entrypoint=bash -it `

`-p 8086:8086 -p 8083:8083 `

`influxdb`

Step 3: Run the conversion from TSM to TSI:

`> influx_inspect buildtsi -datadir=/var/lib/influxdb/data -waldir=/var/lib/influxdb/wal`

![][18]

TSM to TSI.

You can also add -database flag if you want to convert only for one database or one database at a time.

As you can see, all the shards have been indexed, and you can also see the index folder created:

`find /var/lib/influxdb -type d -name index`

![][19]

The index folder is created for both the databases.

Step 4: Exit and remove the container.

![][20]

Remove the container.

Step 5: Start the InfluxDB container without entrypoint flag.

`sudo docker run -itd --name influx-db --restart unless-stopped `

`-e INFLUXDB_DATA_INDEX_VERSION="tsi1" -v $PWD:/var/lib/influxdb -p 8086:8086 -p 8083:8083 influxdb`

![][21]

TSI check.

As you can see above, TSM is successfully changed to TSI.

Few important points regarding TSI:

* There is no  change to the current data.
* While a shard is being indexed, a temporary file (.index) is created. If for whatever reason this process crashes or fails, partials are removed and attempted again. Completed indices are left intact. This helps if the VM or container crashes while performing the conversion operation, preventing data from getting corrupted.
* -database flag can be used to convert a specific database or one database at a time. If you do not specify that then all the databases are converted.
* -e INFLUXDB_DATA_INDEX_VERSION="tsi1″ should be passed while starting up the InfluxDB container.
* Alternatively, conversion can be changed with the conf file as well: 
    * docker run –rm influxdb influxd config > influxdb.conf
    * Edit the conf and change the index-version = "inmem" > index-version = "tsi1"
    * Start the container with this conf file :

`docker run -p 8086:8086 `

`-v $PWD/influxdb.conf:/etc/influxdb/influxdb.conf:ro `

`influxdb -config /etc/influxdb/influxdb.conf`

* Indices still grow with the number of series, but memory isn't needed to grow linearly with those indices. This is where TSI helps.
* General behavior: InfluxDB will use as much memory as is available to maintain an in-memory index and fall back to disk for anything else.
* Without TSI, there is only one index per database and it consumes all memory present and can not do anything else. Now, after TSI, when the memory limit gets hit, InfluxDB starts referring to those indices. Also, it loads the WAL in-mem, indices are paged in as required.

If you face any issues or have any questions regarding TSM to TSI conversion, head over to InfluxDB's [Slack channel][22] and start a discussion. TSI is breaking new ground and helping InfluxDB lead the way to over a billion series.

Feature image from [Pixabay][23].

[1]: http://bit.ly/2xvLV3w
[2]: https://www.influxdata.com/time-series-database/
[3]: https://docs.influxdata.com/influxdb/v1.7/concepts/tsi-details/#sidebar
[4]: https://cdn.thenewstack.io/media/2019/09/284fe6e2-cab2b8f5-saiyam-pathak.png
[5]: https://www.linkedin.com/in/saiyam-pathak-97685a64/
[6]: https://docs.influxdata.com/influxdb/v1.7/concepts/time-series-index/
[7]: https://www.influxdata.com/products/
[8]: https://docs.docker.com/install/
[9]: https://cdn.thenewstack.io/media/2019/09/9e29bd65-influx1.png
[10]: https://cdn.thenewstack.io/media/2019/09/dab40ad7-influx2.png
[11]: https://cdn.thenewstack.io/media/2019/09/d65144c6-influx3.png
[12]: https://cdn.thenewstack.io/media/2019/09/b04faae3-influx4.png
[13]: https://cdn.thenewstack.io/media/2019/09/0abf42f6-influx5.png
[14]: https://cdn.thenewstack.io/media/2019/09/26ef8d36-influx6.png
[15]: https://cdn.thenewstack.io/media/2019/09/5f2ebca4-influx7.png
[16]: https://cdn.thenewstack.io/media/2019/09/9a439fae-influx8.png
[17]: https://cdn.thenewstack.io/media/2019/09/7031015e-influx9.png
[18]: https://cdn.thenewstack.io/media/2019/09/87dc4ac2-influx10.png
[19]: https://cdn.thenewstack.io/media/2019/09/3eff34c3-influx11.png
[20]: https://cdn.thenewstack.io/media/2019/09/b094fbac-influx12.png
[21]: https://cdn.thenewstack.io/media/2019/09/c88b1117-influx13.png
[22]: https://influxcommunity.slack.com/
[23]: https://pixabay.com/photos/metropolis-new-york-apartments-usa-498406/

  
