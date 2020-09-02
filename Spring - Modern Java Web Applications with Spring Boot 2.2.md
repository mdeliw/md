# Modern Java Web Applications with Spring Boot 2.2

Video: https://subscription.packtpub.com/video/programming/9781788993241

Code: https://github.com/PacktPublishing/Modern-Java-Web-Applications-with-Spring-Boot-2.x

## OpenJDK 12

```bash
# installed JDK version
java -version

# jdk location
which java

# installed jre version
/Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java -version 

# remove previous java runtimes.
# List installed java
cd /Library/Java/JavaVirtualMachines/
ls -l /Library/Java/JavaVirtualMachines/

# Remove Java 10
sudo rm -rf java-10.jre
sudo rm -rf /Library/Java/JavaVirtualMachines/*-10.jre

# tap a brew repo.
brew tap AdoptOpenJDK/openjdk

# install OpenJDK using brew.
brew cask install adoptopenjdk12
```

> Do not attempt to uninstall Java by removing the Java tools from `/usr/bin`. This directory is part of the system software and any changes will be reset by Apple the next time you perform an update of the OS.

## Setup Environment

Following video instructions, and setup Run configuration on Intellij:

<img src="Spring - Modern Java Web Applications with Spring Boot 2.2.assets/image-20200830115457126.png" alt="image-20200830115457126" />

<img src="Spring - Modern Java Web Applications with Spring Boot 2.2.assets/image-20200830115637014.png" alt="image-20200830115637014" style="zoom:25%;" />

```bash
# cli
./gradlew bootWar
./gradlew bootRun
```

