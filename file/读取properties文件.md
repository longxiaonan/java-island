读取项目resources/import.properties文件

```java
@Override
public Map<String, String> getProperties() throws IOException {
    Properties properties = new Properties();
    properties.load(new BufferedReader(new InputStreamReader(this.getClass().getResourceAsStream("/import.properties"))));
    Map<String, String> map = (Map) properties;
    return map;
}
```

