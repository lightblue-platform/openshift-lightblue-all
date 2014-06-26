lightblue-all
=============

Quickstart for deploying all applications and services together with local mongodb.

# OpenShift Integration
This project is setup for deployment on openshift with the full .openshift directory already populated.

# OpenShift Without Jenkins

## App Setup
Create your application and add the mongo cartridge.

## Environment
To get the build working correctly you need to pass some properties to maven.  Run this script to set MAVEN_ARGS env variable on your existing application.  Change the local environment variables to match your app name and namespace.

```
OPENSHIFT_LIGHTBLUE_APP_NAME=services
OPENSHIFT_LIGHTBLUE_NAMESPACE=lightblue

eval `rhc ssh $OPENSHIFT_LIGHTBLUE_APP_NAME -n $OPENSHIFT_LIGHTBLUE_NAMESPACE env | egrep -e ^OPENSHIFT_MONGODB_DB_HOST -e ^OPENSHIFT_MONGODB_DB_PORT -e ^OPENSHIFT_MONGODB_DB_USERNAME -e ^OPENSHIFT_MONGODB_DB_PASSWORD | sed 's/^/export /g'`
rhc env-set MAVEN_ARGS="clean package -U -Popenshift -DskipTests -Dmongodb.host=${OPENSHIFT_MONGODB_DB_HOST} -Dmongodb.port=${OPENSHIFT_MONGODB_DB_PORT} -Dmongodb.user=${OPENSHIFT_MONGODB_DB_USERNAME} -Dmongodb.pass=${OPENSHIFT_MONGODB_DB_PASSWORD}" --namespace $OPENSHIFT_LIGHTBLUE_NAMESPACE --app $OPENSHIFT_LIGHTBLUE_APP_NAME
unset OPENSHIFT_MONGODB_DB_HOST
unset OPENSHIFT_MONGODB_DB_PORT
unset OPENSHIFT_MONGODB_DB_USERNAME
unset OPENSHIFT_MONGODB_DB_PASSWORD
rhc env-show MAVEN_ARGS --namespace $OPENSHIFT_LIGHTBLUE_NAMESPACE --app $OPENSHIFT_LIGHTBLUE_APP_NAME
```

## Initial Code Push
Simply force push the master branch from this repo into your app.

```
OPENSHIFT_LIGHTBLUE_APP_NAME=services
OPENSHIFT_LIGHTBLUE_NAMESPACE=lightblue

rhc git-clone --namespace $OPENSHIFT_LIGHTBLUE_NAMESPACE --app $OPENSHIFT_LIGHTBLUE_APP_NAME
cd $OPENSHIFT_LIGHTBLUE_APP_NAME
git remote add github https://github.com/lightblue-platform/lightblue-all.git
git fetch github
git reset --hard github/master
git push origin master -f
```
