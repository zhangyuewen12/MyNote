# Resource providers Yarn



## Introduction

[Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) is a resource provider popular with many data processing frameworks. Flink services are submitted to YARN’s ResourceManager, which spawns containers on machines managed by YARN NodeManagers. Flink deploys its JobManager and TaskManager instances into such containers.

Flink can dynamically allocate and de-allocate TaskManager resources depending on the number of processing slots required by the job(s) running on the JobManager.



