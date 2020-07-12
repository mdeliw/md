## 	Maven

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-core</artifactId>
	<version>2.11.0</version>
</dependency>
```

## Testing

```java
String jsonString = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(object);
```

## Marshalling

### @JsonAnyGetter

Returns a map as json without the map name as the parent node.

```java
private Map properties<String, String> = new HashMap<>();
@JsonAnyGetter
public Map getProperties() {
    return properties;
}
```

Without annotation:

```json
{
  "properties" : {
    "RollNo" : "1",
    "Name" : "Mark"
  }
}
```

With annotation

```json
{
    "RollNo": "1",
    "Name": "Mark"
}
```

### @JsonGetter

Allows specific methods to be marked as getters

```java
private String name;

@JsonGetter
public String getStudentName() {
    return name;
}
```

Without annotation

```json
{
    "studentName": "Mark"
}
```

With annotation

```json
{
    "name": "Mark"
}
```

### @JsonPropertyOrder

Allows a specific order to be preserved while serializing a JSON object.

```java
@JsonPropertyOrder({ "rollNo", "name" })
class Student {
    private String name;
    private int rollNo;
    ...
```

### @JsonRawValue

​	Allows to serialize a text without escaping or without any decoration.