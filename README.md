# Launching Tutorial Console (SAP Commerce 123 Interactive)
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris123"
java -jar target/hybris123.war

# open the console
http://localhost:8080
```

# Using SAP Commerce Server
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"
./hybrisserver start
./hybrisserver stop
./hybrisserver debug
```

# Data Models
Specified in `*-items.xml` files.

## Generating Java classes specified
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"
ant clean all
```

## Updating database
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"
ant updatesystem
```

### Listing all tables in the database
#### Using the HAC
1. Start the SAP Commerce server
2. Go to Monitoring > Database > Table Size > Calculate Table Sizes
3. Search for the table name.

#### Using the default hsqldb DatabaseManager
⚠️ NOTE: This didn't work for me as the GUI's navigation list didn't work.

```bash
# in working directory "$HYBRIS_HOME_DIR/hybris"

# opening database manager
java -cp ./bin/platform/lib/dbdriver/hsqldb*.jar org.hsqldb.util.DatabaseManager --url jdbc:hsqldb:file:./data/hsqldb/mydb &

# closing database manager
pkill -f "DatabaseManager"
```

# Importing Data
## Through HAC
1. Start the SAP Commerce server
2. Go to Console > ImpEx Import
3. Paste the Impex statements into the Import content text box
4. Click on Import Content

## By Convention
Store impex files in `hybris/bin/custom/<EXTENSION_NAME>/resources/impex`

### Naming convention:
To ensure that impex files are loaded during SAP Commerce initialization or update phase, the impex files have to follow a specific naming convention:
* 2 types of impex files:
	* always imported when the platform is initialized with the extension: `essentialdata*.impex`
	* only imported when explicitly specified in the HAC during initialization: `projectdata*.impex`

## By Code
* Ensure that impex files DO NOT follow the naming conventions specified in "Naming convention" section.
* Add ServiceLayer class to `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/setup/`
	* naming format: `<EXTENSION_NAME>CustomSetup.java`
		* for consistency only; naming of file has no impact on functionality
* Register this class as a Spring bean in `hybris/bin/custom/<EXTENSION_NAME>/resources/<EXTENSION_NAME>-spring.xml`

### Mechanism
Use the `@SystemSetup` annotation in any ServiceLayer class to hook ServiceLayer code into the SAP Commerce initialization and update life-cycle events. In this way, you can provide a means for creating essential and project data for any extension.
* By using annotations, no interfaces have to be implemented or parent classes extended, keeping the classes loosely coupled.
* Methods are called in an order that respects the extension build order.
	* You can define when a method should be executed, and pass context information to it.
* Official Docs: [Hooks for Initialization and Update Process](https://help.sap.com/docs/SAP_COMMERCE/d0224eca81e249cb821f2cdf45a82ace/8bcb3edb86691014af58b7162c06e1d5.html?locale=en-US&version=2105)

# Service Layer
* Interfaces specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/service`
	* naming format: `*Service.java` 
* Implementations specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/service/impl`
	* naming format: `Default*Service.java`

Official Docs: [Service Layer](https://help.sap.com/docs/SAP_COMMERCE/d0224eca81e249cb821f2cdf45a82ace/df85e82ae7dd4956b28c266222fcc693.html?locale=en-US)

# Data Access Objects (DAO)
* Interfaces specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/daos`
	* naming format: `*DAO.java`
* Implementations specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/daos/impl`
	* naming format: `Default*DAO.java`

# Integration Tests
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"

# close SAP Commerce instance
./hybrisserver stop

# compile integration test classes
ant clean all

# initialize test tenant
ant yunitinit

# run integration tests for a specific package
ant integrationtests -Dtestclasses.packages="<PACKAGE_NAME>.*"

# view test results in this file:
# "$HYBRIS_HOME_DIR/hybris/log/junit/index.html"
```

## Integration Tests for Service Layer
Specified in `hybris/bin/custom/<EXTENSION_NAME>/testsrc/<EXTENSION_NAME>/service/impl`
* naming format: `Default*ServiceIntegrationTest.java`

## Integration Tests for DAO
Specified in `hybris/bin/custom/<EXTENSION_NAME>/testsrc/<EXTENSION_NAME>/daos/impl`
* naming format: `Default*DAOIntegrationTest.java`

# Unit Tests
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"

# close SAP Commerce instance
./hybrisserver stop

# compile integration test classes
ant clean all

# ℹ️ NO NEED to initialize test tenant

# run integration tests for a specific package
ant unittests -Dtestclasses.packages="<PACKAGE_NAME>.*"

# view test results in this file:
# "$HYBRIS_HOME_DIR/hybris/log/junit/index.html"
```
## Unit Tests for Service Layer
Specified in `hybris/bin/custom/<EXTENSION_NAME>/testsrc/<EXTENSION_NAME>/service/impl`
* naming format: `Default*ServiceUnitTest.java`

## Unit Tests for DAO
Specified in `hybris/bin/custom/<EXTENSION_NAME>/testsrc/<EXTENSION_NAME>/daos/impl`
* naming format: `Default*DAOUnitTest.java`
