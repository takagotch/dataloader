### dataloader
---
https://github.com/forcedotcom/dataloader

```
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


