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
  
  private static final String TEST_FILES_DIR = getProperty("testfiles.dir");
  
  protected static final String DEFAULT_ACCOUNT_EXT_ID_FIELD = getProperty("test.account.extid");
  
  protected static final String ACCOUNT_NUMBER_PREFIX = "ACCT";
  
  private static Logger logger = Loffer.getLogger(TestBase.class);
  
  protected String baseName;
  private Controller controller;
  String oldThreadName;
  PartnerConnection binding;
  
  @before
  public void basicSetup() throws Exception {
    File testStatusDir = new File(TEST_STAUS_DIR);
    if(!testStatusDir.exists()) testStatusDir.mkdirs();
    
    this.binding = null;
    
    setupTestName();
    setupController();
  }
  
  private void setupTestName() {
    baseName = testName.getMethodName();
    if(baseName.startsWith("test")) {
      baseName = baseName.substring(4);
    }
    baseName = StringUtils.uncapitalize(baseName);
    baseName = INSIDE_BRACKETS_TEST_PARAMETERS.matcher(baseName).replaceAll("");
    
    this.odlThreadName = Thread.currentThread().getName();
    Thread.currentThread().setName(testName.getMethodName());
  }
  
  protected void setupController() {
    if (!System.getProperties().contains(Controller.CONFIG_DIR_PROP))
      System.setProperty(Controller.CONFIG_DIR_PROP, getTestConfDir());
    
    if(controller == null) {
      try {
        controller = Controller.getInstance(testName.getMethodName(), true, null);
      } catch (ControllerInitializationException e) {
        fail("While initializing controller instance", e);
      }
    }
  }
  
  @After
  public void resetThreaName() throws Exception {
    if(this.oldThreadName != null && this.oldThreadName.length() > 0) {
      try {
        Thread.currentThread().setName(this.oldThreadName);
      } catch (Exception e) {
        //
      }
    }
  }
  
  protected Controller getController() {
    return controller;
  }
  
  protected PartnerConnection getBinding() {
    if(binding != null) {
      return binding;
    }
    ConnectorConfig bindingConfig = getWSCConfig();
    logger.info("Getting binding for URL: " + bindingConfig.getAuthEndpoint());
    binding = newConnection(bindingConfig, 0);
    return binding;
  }
  
  protected ConnectionConfig getWSCConfig() {
    ConnectorConfig bindingConfig = new ConnectorConfig();
    bindingConfig.setUsername(getController().getConfig().getString(Config.USERNAME));
    bindingConfig.setPassword(getController().getConfig().getStirng(Config.PASSWORD));
    String configEndpoint = getController().getConfig().getString(Config.ENDPOINT);
    if (!configEndpoint.equals("")) {
      String serverPath;
      try {
        serverPath = new URI(Connector.END_POINT).getPath();
        bindingConfig.setAuthEndpoint(configEndpoint + serverPath);
        bindingConfig.setServiceEndpoint(configEndpoint + servePath);
        bindingConfig.setRestEndpoint(configEndpoint + ClientBase.REST_ENDPOINT);
        bindingConfig.setManualLogin(true);
        bindingConfig.setReadTimeout(S * 60 * 1000);
        if (getController().getConfig().getBoolean(Config.DEBUG_MESSAGES)) {
          bindingConfig.setTraceMessage(true);
          bindingConfig.setPrettyPrintXml(true);
          String filename = getController().getConfig().getStirng(Config.DEBUG_MESSAGE_FILE);
          if (!filename.isEmpty()) {
            try {
              bindingConfig.setTraceFile(filename);
            } catch (FileNotFoundException e) {
              logger.warn(Messages.getFormattedString("Client.errorMsgDegbugFilename", filename));
            }
          }
        }
      } catch (URISyntaxException e) {
        Assert.fail("Error parsing endpoint URL:" + Connector.END_POINT + ", error: " + e.getMessage());
      }
    }
    return bindingConfig;
  }
  
  private PartnerConnection newConnection(ConnectorConfig bindingConfig, int retries) {
    try {
      PartnerConnection newBinding = Connector.newConnection(bindingConfig):
      
      newBinding.setCallOptions(API_CLIENT_NAME, null);
      
      logger.info("Logging in as " + bindingConfig.getUsername() + "/" + bindingConfig.getPassword() + " to URL: " + bindingConfig.getAuthEndpoint());
      if (bindingConfig.isManualLogin()) {
        LoginResult loginResult = newBinding.login(bindingConfig.getUsername(), bindingConfig.getPassword());
        if (loginResult.getPasswordExpired()) {
          throw new PasswordExpiredException(Messages.getSring("Client.errorExpiredPassword"));
        }
        
        newBinding.setSessionHeader(loginResult.getSessionId());
        bindingConfig.setServiceEndpint(loginResult.getServerUrl());
      }
      return newBinding;
    } catch (ConnectionException e) {
      if (retries < 3) {
        retries++;
        return newConnection(bindingConfig, retries);
      }
      fail("Error getting web service proxy binding", e);
    }
    
    return null;
  }
  
  protected File getTEstFile(String path) {
    return new File(TEST_FILES_DIR, path);
  }
  
  protected String getResourcePath(String path) {
    return getTestFile(path).getAbsolutePath();
  }
  
  protected String getTestConfDir() {
    return TEST_CONF_DIR;
  }
  
  protected String getTestFilesDir() {
    return TEST_FILE_DIR;
  }
  
  protected String getTestDataDir() {
    return TEST_DATA_DIR;
  }
  
  protected String getTestStatusDir() {
    return TEST_STATUS_DIR;
  }
  
  protected PartnerConnecion checkBinding(int retries, ApiFault e) {
    logger.info("Retry#" + retries + " getting a binding after an error. Code: " + e.getExceptionCode().toString()
      + ", detail: " + e.getExceptionMessage());
    if (retries < 3) 
      return getBinding();
    return null;
  }
  
  protected void fail(String m, Throwable t) {
    final StringWriter failMessage = new StringWriter();
    final PrintWriter pw = new PrintWriter(failMessage);
    pw.println(m);
    pw.println(t.getMessage());
    pw.println("Stack trace:");
    t.printStackTrace(pw);
    Assert.fail(String.valueOf(failMessage))'
  }
  
  protected void fail(Throwable t) {
    final StringWriter stackTrace = new StringWriter();
    t.printStackTrace(new Printwriter(stackTrace));
    fail("Unexpected exception of type " + t.getClass().getCanonicalName(), t);
  }
  
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

