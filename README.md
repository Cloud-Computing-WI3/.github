![Logo](https://avatars.githubusercontent.com/u/117459812?s=200&v=4)     
# Newsify




## Contributors
![Kevin Eberhardt](https://avatars.githubusercontent.com/u/47750689?s=48&v=4) [Kevin Eberhardt](https://github.com/kevin-eberhardt) 

![Danial Eshete](https://avatars.githubusercontent.com/u/47521900?s=48&v=4) [Danial Eshete](https://github.com/danialeshete)

![Harri Fassbender]() [Harri Fassbender](https://github.com/harrif020)

![Jonathan Kessel](https://avatars.githubusercontent.com/u/64253062?s=48&v=4) [Jonathan Kessel](https://github.com/JonathanKessel) 


![System Architecture](https://raw.githubusercontent.com/Cloud-Computing-WI3/.github/3ab0687e1920980902927117906bacd48e96b45e/images/system_architecture.svg)


## Components
### 1) Front-End
The frontend shows a logged-in user news articles from his and from certain categories. 
In addition, relevant articles are displayed to him based on his entered keywords.
### 2) Middleware
The middleware takes care of the communication between Fontend and the backend logics. 
It integrates a profile management service and a service for retrieving news articles based on user settings.

#### Profile Management Service
The Profile Management Service is based on Django and is used to store user information and for authentication. User information is stored in a relational database (MySQL) on Amazon Redshift as well as on an Amazon S3 bucket (profile images).

#### Profile News Feed Provider Service
The News Feed Provider Service connects the frontend with the Elasticsearch database (more about Elasticsearch under [point 3](#3-Elasticsearch)). Via an API in Python ([FASTAPI](https://github.com/tiangolo/fastapi)), the frontend requests the service as soon as a user wants to display articles in the frontend. If the user has already requested the data, Redis is used. Redis is an open-source in-memory database that can be used to cache data. If the user queries data already available in redis, the data is not loaded from Elasticsearch, but from redis. This has the advantage that the query is much faster and therefore the loading of the data is much more performant.
***TODO***: *Nochmals drüber lesen ob das so stimmt*

#### User Data DB
A relational database on Amazon Redshift that stores user information.

#### Redis
[Redis](https://redis.io/) is an open-source in-memory database that can be used to cache data. If the user queries data already available in redis, the data is not loaded from Elasticsearch, but from redis. This has the advantage that the query is much faster and therefore the loading of the data is much more performant.
### 3) Elasticsearch
[Elasticsearch](https://www.elastic.co/) is a distributed RESTful search engine and analytics engine that can address a growing number of use cases. As the core of the Elastic Stack, it stores your data and enables fast searches, fine-tuned relevance, and powerful and effortlessly scalable analytics. In our use case it is implemented as a cloud instance, fully managed by elastic.

### 4) Keyword/Category labeling
This Microservice is responsible for labeling the articles with categories and keywords, it utilizes the [Google Natural Language API](https://cloud.google.com/natural-language). To generate keywords the [Entities](https://cloud.google.com/natural-language/docs/analyzing-entities) endpoint is utilized. To generate categories the [category endpoint](https://cloud.google.com/natural-language/docs/reference/rest/v1/ClassificationCategory) is used. 


### 5) Kafka
[Apache Kafka](https://kafka.apache.org/) is an open-source distributed event streaming platform used by thousands of companies for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications. In our use case it receives the articles from different sources and writes them into the elastic database [3](#3-Elasticsearch). It is implemented as a cloud service on [confluent-kafka](
### 6) Connector controller
This Microservice is responsible for activating the connection sink between between elasticsearch and kafka/confluent. This is a cost saving measure since the connector usage is billed by the minute. It calls the elasticsearch connector before and after the scheduler is running, so the articles in the kafka queue get sent to the elasticsearch database. The controller is implemented as [google cloud function](https://cloud.google.com/functions).


### 7) Schedulers
These schedulers call the defined endpoints at predefined intervalls, they are implemented as [google cloud scheduler](https://cloud.google.com/scheduler?hl=en). 
#### Connector Scheduler
Activates and pauses the [connector controller](#6-Connector controller). 
#### News Scheduler
Ativates and pauses the [news service](#3-News Service).
#### Scraper Scheduler
Activates and pauses the [Scraper Service](#9-Scraper Service).
### 8) News Service
This service is calling the [top headlines](https://newsapi.org/docs/endpoints/top-headlines) endpoint from newsapi.org. It acts as a producer for [kafka](#5-kafka). 
The News Serice is implemented as [google cloud function](https://cloud.google.com/functions). 

### 9) Scraper Service
This service currently scrapes https://www.chefkoch.de/wochenrezepte/ and writes the newest recipes as articles into the kafka queue. The service is implemented as [google cloud function](https://cloud.google.com/functions). 
