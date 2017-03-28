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