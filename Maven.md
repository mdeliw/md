# Maven

[TOC]

---

## Maven variables

```
project.base.*
project.build.*
```

## Generating project

```bash
$ mvn archetype:generate -B -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.1 -DgroupId=org.deliwala -DartifactId=sample-project -DVersion=0.1-SNAPSHOT

# or shorter
$ mvn archetype:generate -B -DgroupId=org.deliwala -DartifactId=sample-project -Dversion=0.0.1 -DarchetypeArtifactId=maven-archetype-quickstart

# quiet clean
$ mvn -q clean package

# verbose clean
$ mvn -X clean package
```

## Compiler option

```bash
$ mvn compile
```

```xml
<!-- We specify the Maven compiler plugin as we need to set it to Java 1.8 -->
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.1</version>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

## Skipping test

Using surefire plugin:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.19.1</version>
  <configuration>
    <skipTests>true</skipTests>
  </configuration>
</plugin>
<!-- or -->
<properties>
  <maven.test.skip>true</maven.test.skip>
</properties>
```

```bash
$ mvn -q clean package -DskipTests
$ mvn -q clean package -Dmaven.test.skip=true 
```

## Executing program in mvn

### `exec:java`

Class provided in command line

```xml
<!-- exec:java -D<args> -->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <!-- no configuration, class injected from command line -->
</plugin>
```

Class provided in pom

```bash
mvn exec:exec
```

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <configuration>
    <mainClass>org.some.package.MainClass</mainClass>
  </configuration>
</plugin>
```

### `exec:java@App1`

Wire multiple main classes

```bash
mvn exec:java@App1
mvn exec:java@App2
```

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <executions>
    <execution>
      <id>App1</id>
      <configuration>
	    <mainClass>org.some.package.MainClass1</mainClass>
	  </configuration>
    </execution>
    <execution>
      <id>App2</id>
      <configuration>
	    <mainClass>org.some.package.MainClass2</mainClass>
	  </configuration>
    </execution>
  </executions>
</plugin>
```

Alternatively 

```bash
mvn exec:exec@run-app
```

```xml
<!-- or 3. using exec:java or exec:exec@run-app -->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <executions>
    <execution>
	  <id>run-app</id>
	  <goals>
	    <goal>exec</goal>
      </goals>
	  <configuration>
	    <executable>java</executable>
		<arguments>
		  <argument>-jar</argument>
		  <argument>target/${project.artifactId}-${project.version}-fat.jar</argument>
		</arguments>
      </configuration>
	</execution>  
  </executions>
</plugin>
```

### exec:exec

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <configuration>
    <executable>java</executable>
		<arguments>
      <argument>-classpath</argument>
      <classpath />
      <argument>${mainClass}</argument>
      <argument>${port}</argument>
      <argument>-Xms512m</argument>
      <argument>-Xmx512m</argument>
      <argument>-XX:NewRatio=3</argument>
      <argument>-XX:+PrintGCTimeStamps</argument>
      <argument>-XX:+PrintGCDetails</argument>
      <argument>-Xloggc:gc.log</argument>
      <argument>${args}</argument>
		</arguments>
  </configuration>
</plugin>

<properties>
	<mainClass>some.package.Class</mainClass>
  <port>8080</port>
  <args>...</args>
</properties>
```

exec:exec@App

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>?</version>
  <executions>
    <execution>
      <id>App1</id>
      <configuration>
	    <executable>java</executable>
     	<arguments>
	  	  <argument>-Xms512m</argument>
	  	  <argument>-Xmx512m</argument>
	  	  <argument>-XX:NewRatio=3</argument>
	  	  <argument>-XX:+PrintGCTimeStamps</argument>
	  	  <argument>-XX:+PrintGCDetails</argument>
	  	  <argument>-Xloggc:gc.log</argument>
	  	  <argument>-classpath</argument>
	  	  <classpath />
	  	  <argument>org.some.package.App1</argument>
	  	  <argument>${args}</argument>
		</arguments>
	  </configuration>
    </execution>
    <execution>
      <id>App2</id>
      <configuration>
	    <executable>java</executable>
     	<arguments>
	  	  <argument>-Xms512m</argument>
	  	  <argument>-Xmx512m</argument>
	  	  <argument>-XX:NewRatio=3</argument>
	  	  <argument>-XX:+PrintGCTimeStamps</argument>
	  	  <argument>-XX:+PrintGCDetails</argument>
	  	  <argument>-Xloggc:gc.log</argument>
	  	  <argument>-classpath</argument>
	  	  <classpath />
	  	  <argument>org.some.package.App2</argument>
	  	  <argument>${args}</argument>
		</arguments>
	  </configuration>
    </execution>
  </executions>
</plugin>
```

## Examples

```bash
# some examples run the program in the same jvm that loaded mvn.

# Example 1
$ mvn exec:java -Dexec.mainClass="org.some.package.MainClass" -Dexec.args="arg1 arg2"

# Example 2
$ mvn exec:java -Dexec.args="arg1 arg2"
$ mvn exec:java@App1
$ mvn exec:java@App2

# Example 3
$ mvn exec:java -Dexec.args="arg1 arg2"
$ mvn exec:exec@run-app
$ mvn exec:exec
$ mvn exec:exec@App1
```

## Generating fat-jar

```bash
$ mvn package
$ java -jar <outputfolder>/fat.jar
```

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>?</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>          
                <outputDirectory>
                    ${project.build.directory}/{$project.build.finalName}.lib
                </outputDirectory>
                <excludeArtfiacts>junit</excludeArtfiacts>
                <overWriteIfNew>true</overWriteIfNew>
            </configuration>configuration>
        </execution>
    </executions>
</plugin>
```

Or..

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <archive>
            <manifest>
                <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                <addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
                <addClasspath>true</addClasspath>
                <classpathPrefix>${project.build.finalName}.lib</classpathPrefix>
                <mainClass>some.package.App</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

Or using `assembly`

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>
                            some.package.Class
                        </mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>

```

Or using Shade

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <!-- default is <project-name>-<version>-shaded.jar -->
                <finalName>user-${artifactId}-${version}</finalName>
                <transformers>
                    <transformer implementation=                        "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>some.package.Class</mainClass>
                        <build-Number>123</buildNumber>
                    </transformer>
                </transformers>
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## Using profile

```bash
$ mvn -P<some-profile-name>
```

```xml
<profiles>
  <profile>
    <id>some-profile-name</id>
    <build>
      <plugins>
        <plugin>...</plugin>
      </plugins>
    </build>
    <properties>...</properties>
  </profile>
</profiles>
```

## Using auto clean

```xml
<plugin>
  <artifactId>maven-clean-plugin</artifactId>
  <configuration>
    <skip>false</skip> <!-- true to skip -->
  </configuration>
  <executions>
    <execution>
      <id>auto-clean</id>
      <phase>initialize</phase>
      <goals>     
        <goal>clean</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

---

## Using Vertx embedded

> Vertx is embedded inside the program rather than used via verticle.

```xml
<properties>
    <!-- the main class -->
    <exec.mainClass>io.vertx.example.HelloWorldEmbedded</exec.mainClass>
</properties>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
              <Main-Class>${exec.mainClass}</Main-Class>
            </manifestEntries>
          </transformer>
          <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
            <resource>META-INF/services/io.vertx.core.spi.VerticleFactory</resource>
          </transformer>
        </transformers>
        <artifactSet>
        </artifactSet>
        <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar</outputFile>
      </configuration>
    </execution>
  </executions>
</plugin>
```

## Using Vertx verticle

```xml
<properties>
  <!-- the main verticle class name -->
  <main.verticle>io.vertx.example.HelloWorldVerticle</main.verticle>
</properties>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
              <Main-Class>io.vertx.core.Launcher</Main-Class>
              <Main-Verticle>${main.verticle}</Main-Verticle>
            </manifestEntries>
          </transformer>
          <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
            <resource>META-INF/services/io.vertx.core.spi.VerticleFactory</resource>
          </transformer>
        </transformers>
        <artifactSet>
        </artifactSet>
        <outputFile>
          ${project.build.directory}/${project.artifactId}-${project.version}-fat.jar
        </outputFile>
      </configuration>
    </execution>
  </executions>
</plugin>

<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>1.4.0</version>
  <executions>
    <execution>
      <id>run</id>
      <goals>
        <goal>java</goal>
      </goals>
      <configuration>
        <mainClass>io.vertx.core.Launcher</mainClass>
        <arguments>
          <argument>run</argument>
          <argument>${main.verticle}</argument>
        </arguments>
      </configuration>
    </execution>
    <execution>
      <id>run-app</id>
      <goals>
        <goal>exec</goal>
      </goals>
      <configuration>
        <executable>java</executable>
        <arguments>
          <argument>-jar</argument>
          <argument>target/${project.artifactId}-${project.version}-fat.jar</argument>
        </arguments>
      </configuration>
    </execution>
  </executions>
</plugin>
```

## Using Vertx maven service factory

> May not be recommended, as code depends on maven service factory.

```bash
# running the verticle
# pom for my-verticle has minimal settings
$ vertx run maven:io.vertx:maven-service-factory-verticle:3.8.1::my-verticle

# running the API to deploy the verticle
$ vertx run io.vertx.examples.MyDeployingVerticle -cp target/maven-service-factory-api-3.8.1.jar
```

```xml
<!-- for the API project -->
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-maven-service-factory</artifactId>
  <version>${project.version}</version>
  <classifier>shaded</classifier>
</dependency>
```

## Plugins and Purpose

- maven-resource-plugin
- jaxws-maven-plugin
- maven-checkstyle-plugin
- maven-compiler-plugin
- maven-jar-plugin
- maven-assembly-plugin
- maven-release-plugin
- maven-dependency-plugin
- maven-failsafe-plugin
- maven-surefile-plugin
- maven-pmd-plugin
- cobertura-maven-plugin
- findbugs-maven-plugin
- jaxb2-maven-plugin
- jacoco-maven-plugin
- maven-clean-plugin
- maven-resources-plugin
- maven-site-plugin
- maven-project-info-reports-plugin
- maven-deploy-plugin
- maven-install-plugin
- hibernate-ehcache
- hibernate-annotations
- net.sf.ehcache
- org.jasypt
- openpojo
- commons-codec
- google-http-client
- mockito-core
- mockito-junit-jupiter

## Reference

- https://maven.apache.org/index.html
- https://mincong-h.github.io/series/maven-plugins/
- https://www.baeldung.com/executable-jar-with-maven
- https://github.com/vert-x3/vertx-stack
- https://reactiverse.io/vertx-maven-plugin/
- https://github.com/reactiverse/vertx-maven-plugin
- https://github.com/vert-x3/vertx-examples

## Pending

- [ ] Nothing

