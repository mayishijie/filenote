# 1. hive启动失败，说未找到TABLE.VERSION

```shell
Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:244)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:158)
Caused by: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1523)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:86)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:132)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:104)
	at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:3005)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:3024)
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:503)
	... 8 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.hive.metastore.MetaStoreUtils.newInstance(MetaStoreUtils.java:1521)
	... 14 more
Caused by: javax.jdo.JDODataStoreException: Exception thrown obtaining schema column information from datastore
NestedThrowables:
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive.version' doesn't exist
	at org.datanucleus.api.jdo.NucleusJDOHelper.getJDOExceptionForNucleusException(NucleusJDOHelper.java:451)
	at org.datanucleus.api.jdo.JDOPersistenceManager.jdoMakePersistent(JDOPersistenceManager.java:732)
	at org.datanucleus.api.jdo.JDOPersistenceManager.makePersistent(JDOPersistenceManager.java:752)
	at org.apache.hadoop.hive.metastore.ObjectStore.setMetaStoreSchemaVersion(ObjectStore.java:6773)
	at org.apache.hadoop.hive.metastore.ObjectStore.checkSchema(ObjectStore.java:6670)
	at org.apache.hadoop.hive.metastore.ObjectStore.verifySchema(ObjectStore.java:6645)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.hive.metastore.RawStoreProxy.invoke(RawStoreProxy.java:114)
	at com.sun.proxy.$Proxy6.verifySchema(Unknown Source)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.getMS(HiveMetaStore.java:572)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.createDefaultDB(HiveMetaStore.java:624)
	at org.apache.hadoop.hive.metastore.HiveMetaStore$HMSHandler.init(HiveMetaStore.java:461)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.<init>(RetryingHMSHandler.java:66)
	at org.apache.hadoop.hive.metastore.RetryingHMSHandler.getProxy(RetryingHMSHandler.java:72)
	at org.apache.hadoop.hive.metastore.HiveMetaStore.newRetryingHMSHandler(HiveMetaStore.java:5768)
	at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.<init>(HiveMetaStoreClient.java:199)
	at org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient.<init>(SessionHiveMetaStoreClient.java:74)
	... 19 more
Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'hive.version' doesn't exist
	at sun.reflect.GeneratedConstructorAccessor18.newInstance(Unknown Source)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)
	at com.mysql.jdbc.Util.getInstance(Util.java:387)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:939)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3878)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3814)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2478)
	at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2625)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2547)
	at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2505)
	at com.mysql.jdbc.StatementImpl.executeQuery(StatementImpl.java:1370)
	at com.mysql.jdbc.DatabaseMetaData$2.forEach(DatabaseMetaData.java:2415)
	at com.mysql.jdbc.DatabaseMetaData$2.forEach(DatabaseMetaData.java:2313)
	at com.mysql.jdbc.IterateBlock.doForAll(IterateBlock.java:50)
	at com.mysql.jdbc.DatabaseMetaData.getColumns(DatabaseMetaData.java:2311)
	at org.datanucleus.store.rdbms.adapter.BaseDatastoreAdapter.getColumns(BaseDatastoreAdapter.java:1532)
	at org.datanucleus.store.rdbms.schema.RDBMSSchemaHandler.refreshTableData(RDBMSSchemaHandler.java:911)
	at org.datanucleus.store.rdbms.schema.RDBMSSchemaHandler.getRDBMSTableInfoForTable(RDBMSSchemaHandler.java:821)
	at org.datanucleus.store.rdbms.schema.RDBMSSchemaHandler.getRDBMSTableInfoForTable(RDBMSSchemaHandler.java:770)
	at org.datanucleus.store.rdbms.schema.RDBMSSchemaHandler.getSchemaData(RDBMSSchemaHandler.java:206)
	at org.datanucleus.store.rdbms.RDBMSStoreManager.getColumnInfoForTable(RDBMSStoreManager.java:2378)
	at org.datanucleus.store.rdbms.table.TableImpl.validateColumns(TableImpl.java:215)
	at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.performTablesValidation(RDBMSStoreManager.java:3393)
	at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.addClassTablesAndValidate(RDBMSStoreManager.java:3190)
	at org.datanucleus.store.rdbms.RDBMSStoreManager$ClassAdder.run(RDBMSStoreManager.java:2841)
	at org.datanucleus.store.rdbms.AbstractSchemaTransaction.execute(AbstractSchemaTransaction.java:122)
	at org.datanucleus.store.rdbms.RDBMSStoreManager.addClasses(RDBMSStoreManager.java:1605)
	at org.datanucleus.store.AbstractStoreManager.addClass(AbstractStoreManager.java:954)
	at org.datanucleus.store.rdbms.RDBMSStoreManager.getDatastoreClass(RDBMSStoreManager.java:679)
	at org.datanucleus.store.rdbms.RDBMSStoreManager.getPropertiesForGenerator(RDBMSStoreManager.java:2045)
	at org.datanucleus.store.AbstractStoreManager.getStrategyValue(AbstractStoreManager.java:1365)
	at org.datanucleus.ExecutionContextImpl.newObjectId(ExecutionContextImpl.java:3827)
	at org.datanucleus.state.JDOStateManager.setIdentity(JDOStateManager.java:2571)
	at org.datanucleus.state.JDOStateManager.initialiseForPersistentNew(JDOStateManager.java:513)
	at org.datanucleus.state.ObjectProviderFactoryImpl.newForPersistentNew(ObjectProviderFactoryImpl.java:232)
	at org.datanucleus.ExecutionContextImpl.newObjectProviderForPersistentNew(ExecutionContextImpl.java:1414)
	at org.datanucleus.ExecutionContextImpl.persistObjectInternal(ExecutionContextImpl.java:2218)
	at org.datanucleus.ExecutionContextImpl.persistObjectWork(ExecutionContextImpl.java:2065)
	at org.datanucleus.ExecutionContextImpl.persistObject(ExecutionContextImpl.java:1913)
	at org.datanucleus.ExecutionContextThreadedImpl.persistObject(ExecutionContextThreadedImpl.java:217)
	at org.datanucleus.api.jdo.JDOPersistenceManager.jdoMakePersistent(JDOPersistenceManager.java:727)
	... 37 more
```

解决并发现问题？

> 刚开始以为是MySQL中hive的元信息缺失导致的，后面去MySQL上查看，show tables，发现那些表都是在的，说明实际上应该是存在的。
>
> 后面自己在MySQL上直接查询这些表，也显示表不存在，现在就很奇怪了，明明表存在，却说表不存在，那么很有可能是就是MySQL的一些问题了，后面想想查看MySQL的配置文件，发现了一个属性：lower_case_table_names=1，这个属性是让系统查询忽略表大小的，不过具体为啥这个能影响hive，目前还没去深究，但是就是这个导致表明明存在但是却显示不存在了。将该属性注掉或者改为默认值0就行了。