# Google Cloud Platform
Google Certified Professional - Data Engineer 

## Labs

### 1. Start Compute Engine instance

**Install Software**

```
sudo apt-get update
sudo apt-get -y -qq install git
```

**Verify that git is now installed**

```
git --version
```
### 2. Interact with Cloud Storage

ingest-transform-and-publish data pipeline
 
**Ingest USGS data**:

```
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
cd training-data-analyst/CPB100/lab2b
less ingest.sh
bash ingest.sh
head earthquakes.csv
```
> *ingest.sh*:

```
wget http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_week.csv -O earthquakes.csv
```

**Transform the data**
[Data transformation Notebook](https://github.com/GoogleCloudPlatform/datalab-samples/blob/master/basemap/earthquakes.ipynb)

```
bash install_missing.sh
python transform.py
ls -l
```
**Store data**:

```
gsutil cp earthquakes.* gs://cpb100-162913/
```
**Publish data**:

[Earthquakes](https://storage.googleapis.com/cpb100-162913/earthquakes.htm)

---

**Delete de instance**

```
gcloud compute instances delete instance-1
```

### 3. Setup rentals data in Cloud SQL

Populate rentals data in Cloud SQL for the rentals recommendation engine to use. The recommendations engine itself will run on Dataproc using Spark ML.

```
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
cd training-data-analyst/CPB100/lab3a
less cloudsql/table_creation.sql
```

```sql
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;
CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId) 
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId) 
    REFERENCES Accommodation(id)
);

```

```
head cloudsql/*.csv
```

```
==> cloudsql/accommodation.csv <==
1,Comfy Quiet Chalet,Vancouver,50,3,3.1,cottage
2,Cozy Calm Hut,London,65,2,4.1,cottage
3,Agreable Calm Place,London,65,4,4.8,house
4,Colossal Quiet Chateau,Paris,3400,16,2.7,castle
5,Homy Quiet Shack,Paris,50,1,1.1,cottage
6,Pleasant Quiet Place,Dublin,35,5,4.3,house
7,Vast Peaceful Fortress,Seattle,3200,24,1.9,castle
8,Giant Quiet Fortress,San Francisco,3400,12,4.1,castle
9,Giant Peaceful Palace,London,1500,20,3.5,castle
10,Sizable Calm Country House,Auckland,650,9,4.9,mansion
==> cloudsql/rating.csv <==
10,1,1
18,1,2
13,1,1
7,2,2
4,2,2
13,2,3
19,2,2
12,2,1
11,2,1
1,2,2
```

```
gsutil cp cloudsql/* gs://cpb100-162913/sql/
```

**Create Cloud SQL instance**

```
bash ./find_my_ip.sh
```

> Note: If you lose your Cloud Shell VM due to inactivity, you will have to reauthorize your new Cloud Shell VM with Cloud SQL. For your convenience, lab3a includes a script called `authorize_cloudshell.sh` that you can run.


**Create database and import data**

**Explore Cloud SQL**

```
mysql --host=35.184.59.111 --user=root --password
```

```sql
use recommendation_spark;
show tables;
select * from Accommodation where type = 'castle' and price < 1500;
```

```
+----+--------------------------+--------------+-------+-------+--------+--------+
| id | title                    | location     | price | rooms | rating | type   |
+----+--------------------------+--------------+-------+-------+--------+--------+
| 14 | Colossal Peaceful Palace | Melbourne    |  1200 |    21 |    1.5 | castle |
| 15 | Vast Private Fort        | London       |  1300 |    18 |    2.6 | castle |
| 26 | Enormous Peaceful Palace | Paris        |  1300 |    18 |    1.1 | castle |
| 31 | Colossal Private Castle  | Buenos Aires |  1400 |    15 |    3.3 | castle |
| 45 | Vast Quiet Chateau       | Tokyo        |  1100 |    19 |    2.3 | castle |
+----+--------------------------+--------------+-------+-------+--------+--------+
5 rows in set (0.10 sec)

```

### 4.Recommendations ML with Dataproc

Carry out recommendations machine learning using Dataproc

* Launch DataprocRun Spark
* ML jobs using Dataproc

```
cd ~/training-data-analyst/CPB100/lab3b
bash authorize_dataproc.sh   cluster-1   us-central1-b   2
```

**Run ML model**

Change MySql connection parameters:

```
nano sparkml/train_and_apply.py
```

```
gsutil cp sparkml/tr*.py gs://cpb100-162913/
```

Submit job to Dataproc: `gs://cpb100-162913/train_and_apply.py`

**Explore ML results**

```
bash ../lab3a/authorize_cloudshell.sh
mysql --host=35.184.59.111 --user=root --password
```

```sql
use recommendation_spark;
select r.userid, r.accoid, r.prediction, a.title, a.location, a.price, a.rooms, a.rating, a.type from Recommendation as r, Accommodation as a where r.accoid = a.id and r.userid = 10;
```

```
+--------+--------+------------+-----------------------------+--------------+-------+-------+--------+---------+
| userid | accoid | prediction | title                       | location     | price | rooms | rating | type    |
+--------+--------+------------+-----------------------------+--------------+-------+-------+--------+---------+
| 10     | 74     |   1.789905 | Giant Calm Fort             | Melbourne    |  2400 |    12 |    2.3 | castle  |
| 10     | 43     |  1.6393561 | Nice Private Hut            | Melbourne    |    60 |     3 |    2.8 | cottage |
| 10     | 2      |  1.5926311 | Cozy Calm Hut               | London       |    65 |     2 |    4.1 | cottage |
| 10     | 31     |  1.5924639 | Colossal Private Castle     | Buenos Aires |  1400 |    15 |    3.3 | castle  |
| 10     | 77     |  1.5140096 | Great Private Country House | Dublin       |  1150 |    10 |    2.4 | mansion |
+--------+--------+------------+-----------------------------+--------------+-------+-------+--------+---------+
5 rows in set (0.10 sec)
```

### 5. Create ML dataset with BigQuery

Use TensorFlow to predict taxicab demand on a day-by-day basis.

Using BigQuery from within Datalab to create a Pandas dataframe that will be the training data for predicting taxicab demand.

* Use **BigQuery** and **Datalab** to explore and visualize data

* Build a Pandas dataframe that will be used as the training dataset for machine learning using **TensorFlow**.

**Run Datalab from Cloud Shell**

```
git clone http://github.com/GoogleCloudPlatform/training-data-analyst
cd training-data-analyst/datalab/cloudshell
./create_vm.sh
./start_tunnel.sh
```

[Demand forecasting with BigQuery and TensorFlow](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab4a/demandforecast.ipynb)

[Notebook](notebooks/demandforecast.ipynb)

### 6. Machine Learning APIs

[Using Machine Learning APIs](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/CPB100/lab4c/mlapis.ipynb)

[Notebook](notebooks/gcp_machine_apis.ipynb)




