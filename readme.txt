External service demo with NodeJS and MongoDB


External DB preparation

Start MongoDB

mongod --bind_ip 192.168.64.1

Create db and user


mongo 192.168.64.1

use sampledb

db.createUser({user: "userU5Y", pwd: "dlYIciGWGt2O234v", roles: [ { role: "readWrite", db: "sampledb" } ] })

Test connectivity 

mongo 192.168.64.1:27017/sampledb -u userU5Y -p dlYIciGWGt2O234v 


oc new-project external-service

# oc new-app nodejs-mongodb-example.json
#  nodejs-mongodb-example.json is the same as  mark.json except mark.json uses my own version which fix the DB ENV for external DB
oc new-app mark.json -p DATABASE_USER=userU5Y -p DATABASE_PASSWORD=dlYIciGWGt2O234v


oc logs -f bc/nodejs-mongodb-example

oc logs -f dc/nodejs-mongodb-example


Delete the MongoDB in OpenShift created by mark.json
oc delete dc/mongodb
oc delete svc/mongodb

Setup the external service to point to external db using the endpoints

oc create -f endpoints.yml
oc create -f mongo-service.yml


oc describe svc/external-mongo
oc describe dc/nodejs-mongodb-example


Set the NodeJS app to point to the external DB.  It will trigger a new deployment

oc set env dc/nodejs-mongodb-example DATABASE_SERVICE_NAME=EXTERNAL_MONGO


oc logs -f dc/nodejs-mongodb-example



URL for testing

http://nodejs-mongodb-example-external-service.apps-crc.testing


Look at the MongoD log to see the successful login message from the NodeJS app


oc delete project external-service




Recommend to use external domain name

Using external domain names make it easier to manage an external service linkage, because you do not have to worry about the external service’s IP addresses changing.


https://docs.openshift.com/container-platform/3.11/dev_guide/integrating_external_services.html

