openshift-lightblue-all
=============

All lightblue rest and web apps bundled into a single WAR for deployment on OpenShift.

# App Setup
Create your application and add the mongo cartridge.  This can be done via the web console or on the command line with the `rhc` tool.  Steps with the cli:

```
OPENSHIFT_LIGHTBLUE_APP_NAME=services
OPENSHIFT_LIGHTBLUE_NAMESPACE=lightblue

# protect for changing versions, extract cart names from rhc app
CART_JBOSSEAP=`rhc cartridge list | grep jbosseap | sort | tail -n1 | awk '{print $1}'`
CART_MONGODB=`rhc cartridge list | grep mongodb | sort | tail -n1 | awk '{print $1}'`

# create application
rhc app create $OPENSHIFT_LIGHTBLUE_APP_NAME $CART_JBOSSEAP -n $OPENSHIFT_LIGHTBLUE_NAMESPACE

# add mongo cart
rhc cartridge add $CART_MONGODB -a $OPENSHIFT_LIGHTBLUE_APP_NAME -n $OPENSHIFT_LIGHTBLUE_NAMESPACE
```

# Environment: No Jenkins
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

# verify
rhc env-show MAVEN_ARGS --namespace $OPENSHIFT_LIGHTBLUE_NAMESPACE --app $OPENSHIFT_LIGHTBLUE_APP_NAME
```

# Environment: Jenkins
TODO, but something similar to the "No Jenkins" option above where an env variable MAVEN_ARGS is setup.  Need to try it out to have it completely documented.

# Initial Code Push
Simply force push the master branch from this repo into your app.

```
OPENSHIFT_LIGHTBLUE_APP_NAME=services
OPENSHIFT_LIGHTBLUE_NAMESPACE=lightblue

rhc git-clone --namespace $OPENSHIFT_LIGHTBLUE_NAMESPACE --app $OPENSHIFT_LIGHTBLUE_APP_NAME
cd $OPENSHIFT_LIGHTBLUE_APP_NAME
git remote add github https://github.com/lightblue-platform/openshift-lightblue-all.git
git fetch github
git reset --hard github/master
git push origin master -f
```

# Push updates
The openshift-lightblue-all source uses overlays with the maven war plugin, so pushing updates is as simple as forcing a build.  Note that if it's dependent on snapshot versions the `MAVEN_ARGS` env variable set earlier should include `-U` for update snapshot dependencies.

```
OPENSHIFT_LIGHTBLUE_APP_NAME=services
OPENSHIFT_LIGHTBLUE_NAMESPACE=lightblue

rhc deploy HEAD -a $OPENSHIFT_LIGHTBLUE_APP_NAME -n $OPENSHIFT_LIGHTBLUE_NAMESPACE
```

# Verification
You can verify your deployment by simply running functional tests for lightblue against your newly deployed application!  See how [here](https://github.com/lightblue-platform/lightblue-tests#functional-tests).
