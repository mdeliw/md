[toc]

# Gradle

## Java Application as CLI

```kotlin
plugins {
    java 
    application // for cli
}
repositories {
    mavenCentral()
}
dependencies {
    implementation("io.vertx:vertx-core:3.8.0")
}
java {
    sourceCompatibility = org.gradle.api.JavaVersion.VERSION_1_8
    targetCompatibility = org.gradle.api.JavaVersion.VERSION_1_8
}
application {
    mainClassName = "chapter1.firstapp.VertxEcho"
}
```

```bash
./gradlew run
```

## Java Custom Task as CLI

```kotlin
plugins {
    java
}
repositories {
    mavenCentral()
}
java {
    sourceCompatibility = org.gradle.api.JavaVersion.VERSION_1_8
    targetCompatibility = org.gradle.api.JavaVersion.VERSION_1_8
}
dependencies {
    implementation("io.vertx:vertx-core:3.8.0")
    implementation("ch.qos.logback:logback-classic:1.2.3")
    testImplementation("junit:junit:4.13")
}
tasks.create<JavaExec>("run") {
    main = project.properties.getOrDefault("mainClass", "chapter2.hello.Deployer") as String
    classpath = sourceSets["main"].runtimeClasspath
    systemProperties["vertx.logger-delegate-factory-class-name"] = "io.vertx.core.logging.SLF4JLogDelegateFactory"
}
```

```bash
./gradlew run -PmainClass=chapter2.hello.HelloVerticle
```



# Reference

- https://gradle-initializr.cleverapps.io/
- https://github.com/gradle/kotlin-dsl-samples

