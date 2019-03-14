### dataloader
---
https://github.com/forcedotcom/dataloader

```java
// src/test/java/com/salesforce/dataloader/TestBase.java
package com.salesforce.dataloader;

import com.salesforce.dataloader.client.Clientbase;
import com.salesforce.dataloader.config.Config;

import org.apache.log4j.Logger;

import java.io.File;

public abstract class TestBase {
  
  private static final Pattern INSIDE_BRACKETS_TEST_PARAMETERS = Pattern.compile("\\[,+\\]");
  @Rule
  public TestName testName = new TestName();
  private static final Properties TEST_PROPS;
  
  static {
    TEST_PROPS = loadTestProperties();
  }
  
  private static Properties loadTestProperties() {
    final Properties p = new Properties();
    final URL url = TestBase.class.getClassLoader().getResource("test.properties");
    if (url == null) throw new IllegalStateException("Failed to locate test.properties. Is it in the classpath?");
    try {
      final InputStream propSteam = url.openStream();
      try {
        p.load(propStream);
      } finally {
        propStream.close();
      }
    } catch (IOException e) {
      throw new RuntimeException("Failed to laod test properties from resource: " + url, e);
    }
    return p;
  }
  
  protected static String getProperty(String testProperty) {
    return TEST_PROPS.getProperty(testProperty);
  }
  
  private static final String API_CLIENT_NAME = "DataLoaderBatch/" + Controller.APP_VERSION;
  
}

// https://github.com/forcedotcom/dataloader/blob/master/src/test/java/com/salesforce/dataloader/TestBase.java
```

```sh
git clone git@github.com:forcedotcom/dataloader.git
cd dataloader
git submodule init
git submodule update
mvn clean package -DskipTests
java -jar target/dataloader-xx.0-uber.jar
java -XstartOnFirstThread -jar target/dataloader-xxx.0-uber.jar
java -XstartOnFirstThread -jar -Xdebug -Xrunjdwp:transport=dt_docket,server=y,suspend=n,address=5005 target/dataloader-xx.0.0-uber.jar
java -cp target/dataloader-xx.0-uber.jar -Dsalesforce.config.dir=CONFIG_DIR com.salesforce.dataloader.proces.ProcessRunner
java -cp target/dataloader-xx.0-uber.jar -Dsalesforce.config.dir=CONFIG_DIR com.salesforce.dataloader.process.ProcessRunner process.operation=insert
java -cp target/dataloader-xx.0-uber.jar -Dsalesforce.config.dir=CONFIG_DIR com.salesforce.dataloader.process.ProcessRunner process.name=opportunityUpsertProcess
java -cp target/dataloader-xx.0-uber.jar -Dsalesforce.config.dir=CONFIG_DIR com.salesforce.dataloader.process.ProcessRunner process.operation=insert

mvn clean test
```

```
```

