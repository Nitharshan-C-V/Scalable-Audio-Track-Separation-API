# Scalable-Audio-Track-Seperation-API
![separation](images/music_separation.png)
Music-Separation-as-a-service (MSaaS)
## Overview
This project involves building a Kubernetes-based system that provides a REST API for automatic music separation. The system processes and separates different tracks within a song, preparing them for retrieval. Additionally, there is an option to use a gRPC API for communication, following a structured implementation approach.
## Project Components

The project consists of the following key services deployed as containers:

+ **rest** - Accepts API requests for music separation and handles MP3 queries. The REST worker will queue tasks to workers using `redis` queues. Full details are provided in [rest/README.md](rest/README.md).

+ **worker** - These process MP3 files using Facebook's open-source waveform source separation software and store the results in an object store (MinIO).Full details are provided in [worker/README.md](worker/README.md).
  
+ **redis** - Used as a message queue to coordinate tasks between the REST API and worker nodes. Full details are provided in [redis/README.md](redis/README.md.)

### Waveform Source Separation Analysis
The worker nodes perform waveform source separation using Facebook's open-source tool. Since this process is computationally expensive (taking about 3-4 times the song's duration), it is implemented as a microservice to optimize performance and scalability

### Setting up Kubernetes
To run this project, a Kubernetes cluster is required. Development was conducted using a local Docker and Kubernetes setup, followed by deployment to Google Kubernetes Engine (GKE) for scalability. Instructions for deploying a GKE cluster are provided in the Kubernetes tutorial.

See the [directions in the Kubernetes tutorial on deploying a GKE cluster](https://github.com/cu-csci-4253-datacenter/kubernetes-tutorial/tree/master/07-guestbook-on-gke).

### Cloud object Storage

Rather than send the large MP3 files through Redis, I suggest using the Min.io object storage system (or other object store) to store the song contents ***and*** the output of the waveform separation. 
We suggest you use the [min.io python interface](https://min.io/docs/minio/linux/developers/python/API.html) to connect to an object storage system like [Min.io](https://github.com/cu-csci-4253-datacenter/kubernetes-tutorial/tree/master/06-minio), [Google object store](https://cloud.google.com/storage) and Amazon S3.
The library [has several example code snippets](https://github.com/minio/minio-py/tree/release/examples) that you can repurpose.

One benefit of an object store is that you can control access to those objects & direct the user to download the objects directly from *e.g.* S3 rather than relaying the data through your service. We're not going to rely on that feature.

For organization, the object store is structured into two main buckets:

•	Queue Bucket: Holds MP3 files that are queued for processing.

•	Output Bucket: Stores separated tracks named in the format <songhash>-<track>.mp3


## Development Steps

1.	Deploy Redis - Redis is deployed using the provided script deploy-local-dev.sh to simplify development.
2.	Implement the REST API - The REST API is developed to handle client requests and connect to the Redis queue.
3.	Develop the Worker Node - Worker nodes are initially tested locally before deployment in Kubernetes. To optimize debugging, the Redis connection is forwarded to a local machine.
4.	Logging and Debugging - A simple Python script (logs/log.py) is used to monitor Redis logs and track job execution. Additionally, Kubernetes logging tools assist in debugging performance issues.
5.	Versioning and Deployment - For every code update, a new container image is pushed with a version number, and the affected Kubernetes pods are redeployed.

### Port Forwarding

To enable local development while using Kubernetes services, Redis and MinIO ports are forwarded to a local machine:
```
kubectl port-forward --address 0.0.0.0 service/redis 6379:6379 &
kubectl port-forward --namespace minio-ns svc/myminio-proj 9000:9000 &
```
This allows the REST API and worker nodes to connect to localhost:6379 and localhost:9000 during development.

### Sample Data
Two sample request scripts are provided:
•	sample-requests.py: Simulates API requests for full-length MP3 files.
•	short-sample-requests.py: Processes short audio clips, making it ideal for local testing.
Since music separation is memory-intensive (requiring at least 6GB of RAM), short samples are tested locally while full-length files are tested in GKE.

