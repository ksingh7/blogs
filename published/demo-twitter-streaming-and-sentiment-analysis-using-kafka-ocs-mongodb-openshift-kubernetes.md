
# Demo : Twitter streaming and sentiment analysis using Kafka, OCS, MongoDB & OpenShift (Kubernetes)

![](https://miro.medium.com/max/1400/1*1wetOUJZmeQTkNABJybQOw.png)

You know tech tools are cool, but unless you have defined use-case it kinda hard to put things into perspective and understand how different tools can interact with each other, help solve a problem or explore new use cases.

So to educate and motivate our technical buyers, sellers and customers, i created a fancy use case of ingesting live twitter tweets and applying sentiment analysis to it. For this demo i used the following tools

* **Twitter API** : Realtime streaming data source

* **Red Hat AMQ Streams** : Apache Kafka cluster to store real time streaming data coming into the system

* **MongoDB** : Storing tweets for long term persistence from Kafka into a schema-less NoSQL database

* **Red Hat OpenShift Container Storage** : Used for providing RWO (in this project), RWX, Object Storage persistence storage for Kafka and MongoDB apps running on OpenShift

* **Red Hat OpenShift Container Platform **: Enterprise grade k8s distribution for container apps

* **Aylien : **Sentiment analysis solution backend

* **Python** : Backend API app to trigger data sourcing from twitter, move data from Kafka to MongoDB, server data to frontend app

* **Frontend** : basic HTLM, CSS, Javascript based frontend to plot some graphs

This slide deck should give you a glimpse of how the demo would look like (demo youtube/github link below)

[Slide Deck](https://www.slideshare.net/alohamora/demo-twitter-sentiment-analysis-on-kubernetes-using-kafka-mongodb-with-openshift-container-storage)

And here is the actual demo recording that you can go through, where i have explained how these component work together and making this a viable solution if you have a real-world use case along the same lines

[YouTube Video Link](https://youtu.be/ngolragtNto)]

If you are interested in running this demo by yourself, you can find the code in my repo, Github project link : [https://github.com/ksingh7/twitter_streaming_app_on_openshift_OCS](https://github.com/ksingh7/twitter_streaming_app_on_openshift_OCS)
> Happy Analysing Live Tweets
