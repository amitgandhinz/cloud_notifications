#Cloud Notifications
**Cloud Notifications is a Notifications as a Service offering built on top of Open Stack's Marconi Project.**

The concept is simple:

- Publisher (the application/website) will publish() messages on to a specific queue
- The application will be responsible for advertising the queue's to the public

- Subscribers will subscribe to queue's that they will be interested in (topics)
- Subscribers will be constantly running, listening to each queue for messages. (Eventually support push for push based queues) 
- Subscribers will contain pluugable middleware for
  - Analysing/Transforming a Message
  - Passing on the message (eg send to another queue for furthur processing, sending an email, sending an sms)

- Publishers and Subscribers will be authenticated using Keystone.

Note : This is a hack day project, and is very much a proof of concept only.


*Execution*
To run this locally: >gunicorn app:app

#API

Subscriber Actions

    PUT /v1/topics/{topic}/subscribers/{subscriber_name} # subscribe to a topic (may require confirmation based on topic permissions)
    DELETE /v1/topics/{topic}/subscribers/{subscriber_name} # unsubscribe from a topic
    GET /v1/topics/{topic}/messages # get list of unseen messages for the topic
    
Publisher Actions

    GET /v1/topics # return a list of topics
    PUT /v1/topics/{topic} # create/update a topic to publish to
    DELETE /v1/topics/{topic} # delete a topic
    POST /v1/topics/{topic}/messages # publish messages to the topic. All subscribers to this topic receive all messages.
    
    GET /v1/{topic}/subscribers # lists all subscribers to the topic
    PATCH /v1/{topic}/subscribers/{subscriber_name} # confirms a subscriber subscribing to the topic
    DELETE /v1/{topic}/subscribers/{subscriber_name} # removes a subscriber from the topic
    PUT /v1/{topic}/permissions/{project_id} # allows a keystone projectid to publish to this topic
    DELETE /v1/{topic}/permissions/{project_id} # removes a granted projectid the ability to publish to this topic

Subscriber Protocols

    Email (Subscriber who Sends an Email)
    SMS (Subscriber who Sends a text message)
    Marconi (Subscriber who Sends message on to a Marconi Queue)
    URI (Subscriber who Calls an URI endpoint with a JSON encoded message)
    Twitter (Subscriber that sends out a tweet)
    Facebook (Subscriber that Posts to a Facebook wall)
    
    Application_{UniqueIdentifier} (Subscriber who handles the Messages their own way - must implement themselves)
    
    *Subscriber Protocols are used to target different customized messages for different protocols, and is optional.*
    *Workers supporting the above protocols will be provided*
    
*We do not currently plan to support attribute based subscriptions.*

#Custom Subscriber Applications

Custom Subscribers can be built using the python library provided with driver based (pluggable) components.  The drivers that
can be substituted in the application are

- Processor -> This driver allows the developer to define what processing or transformation should be done on the message before it is passed on to the next module.
- Notifier -> This driver is responsible for sending the notification to the next subsystem (eg Billing System, a Marconi Queue, etc)

#Scaling Workers

As load increases in a topic (lots of notifications posted), workers will need to scale out to support handling the high load of notifications.  As topics are created, workers will need to be spawned to monitor that topic to handle notifications added.

- Workers autoscaled for each topic and to handle n messages per topic.
- As workers scale out (eg n Email Workers), how do we ensure dupicate notifications are not sent?

 

