# Distributed System Design Patterns: Sidecar Pattern 



## Overview

Since starting the book, "Designing Distributed systems" by Brendan Burns, It has enhanced my understanding of building distributed systems and the theory behind the design patterns associated with them. In this blog I will explain my interpretation of a single-node pattern called  the "Sidecar pattern".  The sidecar pattern is a single-node pattern made up of two containers.  

- Application Container
  - This is the container where you application  or service logic will live. If you been have been using Docker or any container technology then you have probably launched your own application container at some point. 
- Sidecar Container 
  - This container can be literally anything you want it to be. The idea of the sidecar is to add functionality to the Application Container.  An example of this can be adding SSL on to an service that does not natively support it. You see this when a developer deploys an NGNIX container that acts as a SSL forwarding proxy to the  application container that is configured without SSL. 



Excerpt from Designing Distributed Systems:

[The role of the sidecar is to augment and improve the application container, often without the application container’s knowledge.  In its simplest form, a sidecar container can be used to add functionality to a container that might otherwise be difficult to improve.](https://learning.oreilly.com/library/view/designing-distributed-systems/9781491983638/)



## Design

![](D:\blog_posts\programming\distributed_systems\imgs\sidecar_1.png)



In this example it is assumed that you understand the following technologies 

- Docker
- docker-compose
- YAML 
- Python3



Lets say we have a User registration service that currently handles incoming HTTP request to generate a user for the system to utilize.  Users have been complaining that they cannot confirm whether their registration has been completed and requested that some kind of functionality is added  to do so. The developer who wrote the service is currently on vacation and the company has a strict policy about modifying code you are not the originator of.  What to do? Well Lets add a Sidecar on that baby and give the people what they want!



## Implementation



First lets take a look at the service logic.  In the case of the sidecar, it does not care about what the service is doing but from our understanding we need to make sure that the change being implemented with the sidecar has no negative or duplicate effects on the service. 

service.py

```python
import time
import csv
from bottle import route, run, request
import logging
import os

APP_PORT = os.getenv("APP_PORT")
DEBUG = os.getenv("DEBUG", False)

logging.basicConfig(level=logging.DEBUG)


@route('/register')
def service():
    lname = request.params.get("lname")
    fname = request.params.get("fname")

    with open('/data/dump.txt', 'a+') as f:
        writter = csv.DictWriter(f, fieldnames=['fname', 'lname'])
        writter.writerow({
            'fname': fname,
            'lname': lname
        })
        logging.info(f'Created {fname} {lname}')
    return "Success"


def main():
    logging.info("Starting Server on port 8080")
    run(host='0.0.0.0', port=APP_PORT, debug=DEBUG)


if __name__ == "__main__":
    main()


```



As you can see the service is pretty straight forward. This code uses bottle framework to serve request to the client. The client will send request to the **/register** endpoint using the URL **params('fname', 'lname')** . This request will trigger the service to open the file 'dump.txt' and write the user contents to the file. 



sidecar.py

```python
import csv
import time
import logging

logging.basicConfig(level=logging.DEBUG)


def sidecar():
    collection = {}
    logging.info("Starting Sidecar")
    while True:
        time.sleep(2)
        with open('/data/dump.txt', 'r') as f:
            reader = csv.DictReader(f, fieldnames=['fname', 'lname'])

            for row in reader:
                if row.get("fname") not in collection:
                    logging.info(f"Sending Email Sent to '{row.get('fname')}'")
                    collection[row.get("fname")] = row


def main():
    sidecar()


if __name__ == "__main__":
    main()

```



In the sidecar, this service will preform the email notification.   We will start an infinite loop. In this loop we will open the same dump.txt file that the service container used to store the user information, loop through each item and  if the item is not in the collection, add and send notification email.  As you can probably tell, there are no SMTP libraries, so we will be using a simple print statement to simulate the sending of the email. 



Next We will need to create the Dockerfiles that will be used to deploy the service and sidecar containers. 

**sidecar.Dockerfile**

The Sidecar dockerfile will be used to build the sidecar container.  

```dockerfile
FROM python

COPY sidecar.py /

CMD python3 sidecar.py
```



```shell
docker build -f sidecar.Dockerfile -t sidecar .
```

**service.Dockerfile**

The service Dockerfile  will be used to build a container use the service module.

```dockerfile
FROM python

COPY service.py /

RUN pip install bottle

CMD python3 service.py
```



```shell
docker build -f service.Dockerfile -t service .
```



## Deployment



Finally we will need to tie this all together. This docker-compose file will bind the 2 services created  and connect them to the same shared volume.  The

```yaml
version: "3.1"

services:
  service:
    image: service
    environment:
      APP_PORT=8080
      DEBUG=true
    volumes:
      - ~/shared:/data # Must match sidecar
    ports:
      - $APP_PORT:$APP_PORT


  sidecar:
    image: sidecar
    volumes:
      - ~/shared:/data # Must match sidecar

volumes: # Required 
  data:

```

Next lets run the docker compose YAML file. You will notice we used the '-d' flag to indicate to Docker that we want this compose file to run the services as daemons in the background. 

```shell
docker-compose.yml up -d
```

 Now lets check that the services are indeed running. 

```shell
cbaxter@ubuntudev01:~/sidecar_example$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                          NAMES
0dbc7454dc28        sidecar                     "/bin/sh -c 'python3…"   6 seconds ago       Up 6 seconds                                       sidecar_example_sidecar_1
10a14d9edcb8        service                     "/bin/sh -c 'python3…"   2 minutes ago       Up 2 minutes        0.0.0.0:8080->8080/tcp         sidecar_example_service_1
```



Next we need to send a request to the service container to create a user account. We will do this with the following script 



**client.py**

```python
import requests

SERVICE_HOST = 'http://{{ YOU_HOST_IP }}:{{ APP_PORT }}'

def main():
    response = requests.get(f"{SERVICE_HOST}/register?fname=jack&lname=doe")


if __name__ == "__main__":
    main()

```

```shell
python3 client.py
```



Now lets see if the service container created the user 'jack doe'

```shell
cbaxter@ubuntudev01:~/scripts$ docker logs 10a14d9edcb8
INFO:root:Starting Server on port 8080
Bottle v0.12.18 server starting up (using WSGIRefServer())...
Listening on http://0.0.0.0:8080/
Hit Ctrl-C to quit.

INFO:root:Created jack doe
```



New lets see if the sidecar sent the notification email to jack. 

```shell
cbaxter@ubuntudev01:~/scripts$ docker logs 0dbc7454dc28
INFO:root:Starting Sidecar
INFO:root:Sending Email Sent to 'johns'
INFO:root:Sending Email Sent to 'jack'
```



As you can se without having to interact with the service container, the sidecar was able to add the extra functionality to the container the the implementation would've been transparent to the consumer of the service. 



## Summary 

Takeaways

- The sidecar pattern is a single-node pattern made up of two containers.  
- Application Container
  - This is the container where you application  or service logic will live. 
- Sidecar Container 
  - The role of the sidecar is to augment and improve the application container, often without the application container’s knowledge

