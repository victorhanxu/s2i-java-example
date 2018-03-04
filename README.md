# s2i-java-example

This S2I example is intentionally NOT using Spring Boot, Vert.x, Dropwizard, Wildfly Swarm or whatever other simple "fat JAR" (non-WAR/EAR) Java
server framework, but for clarity simply uses the simplest possible Java server application with a main() class.  You can easily apply this example to whatever standalone Java application you want to container-ize with S2I.  (We're using the Java built-in com.sun.net.httpserver.HttpServer; *JUST* for illustration of S2I.)


## Local

For local building, install s2i either from source https://github.com/openshift/source-to-image/releases/ or e.g. via:

    sudo dnf install source-to-image

Now to build the simplest possible Java server with OpenShift Source-To-Image (S2I) using the fabric8io-images/s2i builder:

    s2i build https://github.com/vorburger/s2i-java-example fabric8/s2i-java vorburger:s2i-java-example

or

    git clone https://github.com/vorburger/s2i-java-example ; cd s2i-java-example
    s2i build --copy . fabric8/s2i-java vorburger:s2i-java-example

_NB The `--copy` ensures that the latest content of the current directory and not only it's commited .git content is used ([see S2I #418](https://github.com/openshift/source-to-image/issues/418))._

Now run it like this:

    docker run -p 8080:8080 vorburger:s2i-java-example

and see "hello, world" when accessing http://localhost:8080 - it works!


## OpenShift

To do the same as above directly inside your OpenShift instance:

    oc new-app fabric8/s2i-java~https://github.com/vorburger/s2i-java-example

_TODO oc expose svc/s2i-java-example_

or... _TODO how to this right?? This will actually fetch from git instead of using the local host sources:_

    git clone https://github.com/vorburger/s2i-java-example ; cd s2i-java-example
    oc new-app fabric8/s2i-java~.


## Advanced

### Container options

All JVM options documented on https://github.com/fabric8io-images/s2i/tree/master/java/images/jboss
are typically specified in [`.s2i/environment`](.s2i/environment), but  for quick testing can obviously also be specified on the `docker run` CLI like so:

    docker run -e "JAVA_MAIN_CLASS=ch.vorburger.openshift.s2i.example.Server" -p 8080:8080 vorburger:s2i-java-example


### fabric8io-images/s2i self build locally and in OpenShift

If you do not like to use the possibly not latest fabric8/s2i-java from hub.docker.com you can of course first also just do this for local Docker:

    docker build https://github.com/fabric8io-images/s2i.git#master:java/images/jboss

or inside OpenShift:

    oc new-build https://github.com/fabric8io-images/s2i.git --context-dir=java/images/jboss

or push latest local development changes into OpenShift without git commit & push to GitHub:

    git clone https://github.com/fabric8io-images/s2i.git
    cd s2i/java/images/jboss
    eval $(minishift docker-env)
    docker build -t fabric8/s2i-java .
    oc tag --source=docker fabric8/s2i-java:latest s2i-java:latest


### How to clean up

    oc delete imagestream s2i-java
    oc delete imagestream s2i-java-example
    oc delete build s2i-java-example-1
    oc delete buildconfig s2i-java-example
    oc delete deploymentconfig s2i-java-example
    oc delete service s2i-java-example


## TODO points

* Why isn't it incremental?  Keep re-downloading Maven basics, every time..
* Support Gradle!
* Monitoring..
* Sources should not be runtime container?!


## More background

* https://github.com/fabric8io-images/s2i
* https://github.com/fabric8io-images/s2i/tree/master/java/images/jboss
* https://github.com/fabric8io-images/s2i/issues/112
* https://github.com/openshift/source-to-image

