---
layout: post
title: Getting Started Playing Around with JAX-RS 2.0 in an EE Container
---
{% include JB/setup %}

Trying out the latest JAX-RS API advancements in an EE container is a bit of a pain because pulling in the latest JAX-RS 2.0 libraries into a Java EE environment creates conflicts with the EE-shipped JAX-RS version. Ah yes, and of course you want your IDE to pick up the correct libs for code completion.

After trying and tweaking a bit the most workable solution for me was to download the latest Glassfish 4 build. It uses a Jersey version that is usually only a couple of days behind the JAX-RS API Snapshot.

Here is what I do:

First, go to the [lastest Glassfish builds](http://dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/) and download the file named glassfish-4.0-bxx.zip with xx having the highest number (as of Nov 19th this is [glassfish-4.0-b63.zip](http://dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/glassfish-4.0-b63.zip)).

This will unzip to a glassfish3/ directory, so to start the server do
    
    glassfish3/glassfish/bin/asadmin start-domain

To find the JAX-RS version included in the Glassfish you just downloaded do something like the following:
    
    jar -xf glassfish3/glassfish/modules/javax.ws.rs-api.jar META-INF/MANIFEST.MF; grep Bundle-Version META-INF/MANIFEST.MF

(This will leave a META-INF directory behind you might want to clean up afterwards)

For the b63 version this yields:
    
    Bundle-Version: 2.0.0.m12

The next step is to create a project to test things out. I am using the following archetype, but any other EE Web profile archetype should work similarly:
    
    mvn archetype:generate \
      -DarchetypeGroupId=org.codehaus.mojo.archetypes \
      -DarchetypeArtifactId=webapp-javaee6 \
      -DgroupId=org.example \
      -DartifactId=test \
      -DinteractiveMode=false

Now let's edit the POM to pull in the JAX-RS 2.0-Version used by the Glassfish. First, add the proper repository.
    
     <repository>
     <id>snapshot-repository.java.net</id>
     <name>Java.net Snapshot Repository for Maven</name>
     <url>https://maven.java.net/content/repositories/snapshots/</url>
     <layout>default</layout>
     </repository>

Then add the dependency to the specific release (note that the version is not literally the same as the bundle version you extracted above):
    
    <dependency>
      <groupId>javax.ws.rs</groupId>
      <artifactId>javax.ws.rs-api</artifactId>
      <version>2.0-m12</version>
      <scope>provided</scope>
    </dependency>

**UPDATE**: It seems you have to make sure you include this dependency **before** the dependency to the Java EE Web API. Otherwise, maven insists on trying to use the JAX-RS API version of the Java EE Web API.

You should now be able to `mvn package` and deploy to the Glassfish instance any 2.0-m12 based source code.

The final step will be to cd into your project folder and set up your IDE project. E.g. with Eclipse run 
    
    mvn eclipse:eclipse

and import as existing Maven project into Eclipse.

Code completion should now suggest you the 2.0-m12 API.

If you want to have code completion work with the SNAPSHOT, simply change the POM dependency and rerun `mvn eclipse:eclipse`. This will likely break when deployed in the container (if you use a SNAPSHOT-only API feature) but still it is nice to play around with the very latest API changes.


This focusses on JAX-RS 2.0 latest features. Keep in mind that for the rest of Java EE we referenced the EE6 Web API  profile. This is not for testing latest EE7 features in general.

**UPDATE**: You can find the releases, including the apidoc jars [here](http://repo1.maven.org/maven2/javax/ws/rs/javax.ws.rs-api/).








