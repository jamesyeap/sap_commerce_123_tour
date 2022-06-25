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
## Writing Service classes
* Interfaces specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/service`
	* naming format: `*Service.java` 
* Implementations specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/service/impl`
	* naming format: `Default*Service.java`
* Register implementations as Spring beans in `<EXTENSION_NAME>-spring.xml`

Official Docs: [Service Layer](https://help.sap.com/docs/SAP_COMMERCE/d0224eca81e249cb821f2cdf45a82ace/df85e82ae7dd4956b28c266222fcc693.html?locale=en-US)

# Data Access Objects (DAO)
## Writing DAO classes
* Interfaces specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/daos`
	* naming format: `*DAO.java`
* Implementations specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/daos/impl`
	* naming format: `Default*DAO.java`

# Data Transfer Objects (DTO)
SAP Commerce generates Java model classes that represent the different types of item that are stored in the database. These model classes are what you pass as arguments to the different Java services in the SAP Commerce service layer. Nevertheless, there are occasions when the model classes become unwieldy:
* When you need a simpler or more convenient format for some of the data to display in JSPs
* When you need a serializable set of objects to send to another system
* When you want to prevent client code from modifying attributes in a model class object directly

In these cases, you need a simpler representation of the data in the model classes. This representation is the purpose of the Data Transfer Object.

## Generating DTO classes
Unlike DAO, DTO are not written manually. Instead, do the following:
1. Specify beans in `hybris/bin/custom/<EXTENSION_NAME>/resources/<EXTENSION_NAME>-beans.xml`
2. Run `ant clean all` to generate DTO classes
3. DTO classes are generated in `hybris/bin/platform/bootstrap/gensrc`

# Facade Layer
If there is a common sequence of method calls that a client must make against a service object, it makes sense to combine the sequence into one call. You make these simplified calls with a facade object.

Facade classes help simplify the calls made to your service classes. They use simpler plain old java objects (POJOs) as argument and result objects, instead of SAP Commerce model classes. In this step, you create a new BandFacade class.

## Writing Facade classes
* Interfaces specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/facades`
	* naming format: `*Facade.java` 
* Implementations specified in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/facades/impl`
	* naming format: `Default*Facade.java`
* Register implementations as Spring beans in `<EXTENSION_NAME>-spring.xml`

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

# run both unit and integration tests for a specific package
ant alltests -Dtestclasses.packages="<PACKAGE_NAME>.*"

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

# run unit tests for a specific package
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

# Front End
## Controllers
### Writing Controller classes
Specified in `hybris/bin/custom/<EXTENSION_NAME>/web/src/<EXTENSION_NAME>/controller`
* naming format: `*Controller.java`

Important details of Controller classes:
```java
@Controller
public class ExampleController {
	// returns this page when URL contains "www.example.com/pageA"
	@RequestMapping(value = "/pageA")
	public String functionA(final Model model) {
		// maps this value to "{$attrOne}" in "pageA.jsp"
		final Integer attrOne = 1;
		model.addAttribute("attrOne", attrOne);

		// return "pageA.jsp"
		return "pageA";
	}
}
```

## JSP
### Writing JSP
Specified in `hybris/bin/custom/<EXTENSION_NAME>/web/webroot/WEB-INF/views`
* naming format: `*.jsp`

Important details of JSP files:
```jsp
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!doctype html>

<html>
	<title>Example Page</title>

	<body>
		<h1>Example of attribute mapping</h1>
		<p>Value of attrOne (as defined in ExampleController): ${attrOne}</p>
	</body>
</html>
```

### Viewing changes made to JSP files
Note: you don't have to restart the server if you create new or modify existing jsp pages. This can be very helpful in speeding up the development of your front end.

# Dynamic Attributes
* Dynamic attributes enable you to add attributes to your model, and to create custom logic for them, without touching the model class itself.
* They provide a way to generate new data, and access it without calling a separate service to do so.
	* Dynamic attributes are transient data that is not persisted to the database.

## Writing Dynamic attributes
The implementation of this new dynamic attribute has some key features:
* The persistence type is set to dynamic
* The persistence attributeHandler points to a bean that must handle the DynamicAttributeHandler interface
* The write attribute is set to false, and therefore the attribute is read-only

### Declaring attribute
Add attribute in `<EXTENSION_NAME>-items.xml`:
```xml
<attribute qualifier="daysUntil" type="java.lang.Long">
	<persistence type="dynamic" attributeHandler="concertDaysUntilAttributeHandler" />
    <modifiers read="true" write="false" />
</attribute>
```

### Defining attribute handler
1. Write handler in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/attributehandlers`
* naming format: `*AttributeHandler.java`
2. Register handler in `*-spring.xml`
* `<bean id="*AttributeHandler" class="<EXTENSION_NAME>.attributehandlers.*AttributeHandler"/>`

## Updating SAP Commerce
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"

# stop the SAP Commerce server
./hybrisserver stop

# compile the attribute handler class
ant clean all

# add the dynamic attribute to the type system
ant updatesystem
```

# Events
The Event System is a framework provided by the ServiceLayer that allows you to send and receive events within SAP Commerce.

## Predefined Events
The platform defines and publishes events for a number of predefined types of event. These include the AfterItemCreationEvent type, items of which are published after any new data item is saved to the database. To process these AfterItemCreationEvent events, you provide a listener class and register it with the event framework.

## User-defined Events
Also, you can set up components of your extension to publish events that are then received by registered listeners.

### Writing Events
1. Write event in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/events`
	* naming format: `*Event.java`
```java
package concerttours.events;
import de.hybris.platform.servicelayer.event.events.AbstractEvent;
 
public class BandAlbumSalesEvent extends AbstractEvent
{
    private final String code;
    private final String name;
    private final Long sales;

    public BandAlbumSalesEvent(final String code, final String name, final Long sales)
    {
        super();
        this.code = code;
        this.name = name;
        this.sales = sales;
    }

    public String getCode()
    {
        return code;
    }

    public String getName()
    {
        return name;
    }

    public Long getSales()
    {
        return sales;
    }

    @Override
    public String toString()
    {
        return this.name;
    }
}
```

# Listeners
Listeners are objects that are notified of events and perform business logic depending on the event that occured. Events can be published either locally or across a cluster of nodes. You can register new listeners as Spring beans in your Spring configuration XML file.

## Writing listeners
1. Write listener in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/events`
	* naming format: `*EventListener.java`
```java
/**
 * An example event listener that listens for new Band events:
 * 	- when a new Band is added, add a News instance with an announcement message
 */

package concerttours.events;

import de.hybris.platform.servicelayer.event.events.AfterItemCreationEvent;
import de.hybris.platform.servicelayer.event.impl.AbstractEventListener;
import de.hybris.platform.servicelayer.model.ModelService;
import java.util.Date;
import concerttours.model.BandModel;
import concerttours.model.NewsModel;
 
public class NewBandEventListener extends AbstractEventListener<AfterItemCreationEvent>
{
    private static final String NEW_BAND_HEADLINE = "New band, %s";
    private static final String NEW_BAND_CONTENT = "There is a new band in town called, %s. Tour news to be announced soon.";
    private ModelService modelService;

    public ModelService getModelService()
    {
        return modelService;
    }

    public void setModelService(final ModelService modelService)
    {
        this.modelService = modelService;
    }

    @Override
    protected void onEvent(final AfterItemCreationEvent event)
    {
        if (event != null && event.getSource() != null)
        {
            final Object object = modelService.get(event.getSource());
            if (object instanceof BandModel)
            {
                final BandModel band = (BandModel) object;
                final String headline = String.format(NEW_BAND_HEADLINE, band.getName());
                final String content = String.format(NEW_BAND_CONTENT, band.getName());
                final NewsModel news = modelService.create(NewsModel.class);
                news.setDate(new Date());
                news.setHeadline(headline);
                news.setContent(content);
                modelService.save(news);
            }
        }
    }
}
```
2. Register handler in `*-spring.xml`:
```xml
<bean id="*EventListener" class="<EXTENSION_NAME>.events.*EventListener" parent="abstractEventListener">
	<property name="modelService" ref="modelService" />
</bean>
```

## Updating SAP Commerce
```bash
# in working directory "$HYBRIS_HOME_DIR/hybris/bin/platform"

# stop the SAP Commerce server
./hybrisserver stop

# compile the listener class
ant clean all
```

# Interceptors
Interceptors intercept model object lifecycle transitions and, depending on the conditions of that transition, may publish an event when they do so.

An interceptor addresses a particular step in the life cycle of a model. When the life cycle reaches a certain step, you can activate a corresponding interceptor. An interceptor can:
* modify the model,
* raise an exception to interrupt the current step
* publish an event if the model matches certain criteria.

For example, you could check that an attribute contains certain values before saving the model.

## Writing Interceptors
1. Write interceptor in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/interceptors`
```java
package concerttours.interceptors;

import static de.hybris.platform.servicelayer.model.ModelContextUtils.getItemModelContext;
import de.hybris.platform.servicelayer.event.EventService;
import de.hybris.platform.servicelayer.interceptor.InterceptorContext;
import de.hybris.platform.servicelayer.interceptor.InterceptorException;
import de.hybris.platform.servicelayer.interceptor.PrepareInterceptor;
import de.hybris.platform.servicelayer.interceptor.ValidateInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import concerttours.events.BandAlbumSalesEvent;
import concerttours.model.BandModel;
 
public class BandAlbumSalesInterceptor implements ValidateInterceptor, PrepareInterceptor
{
    private static final long BIG_SALES = 50000L;
    private static final long NEGATIVE_SALES = 0L;
    @Autowired
    private EventService eventService;

    @Override
    public void onValidate(final Object model, final InterceptorContext ctx) throws InterceptorException
    {
        if (model instanceof BandModel)
        {
            final BandModel band = (BandModel) model;
            final Long sales = band.getAlbumSales();
            if (sales != null && sales.longValue() < NEGATIVE_SALES)
            {
                throw new InterceptorException("Album sales must be positive");
            }
        }
    }

    @Override
    public void onPrepare(final Object model, final InterceptorContext ctx) throws InterceptorException
    {
        if (model instanceof BandModel)
        {
            final BandModel band = (BandModel) model;
            if (hasBecomeBig(band, ctx))
            {
                eventService.publishEvent(new BandAlbumSalesEvent(band.getCode(), band.getName(), band.getAlbumSales()));
            }
        }
    }

    private boolean hasBecomeBig(final BandModel band, final InterceptorContext ctx)
    {
        final Long sales = band.getAlbumSales();
        if (sales != null && sales.longValue() >= BIG_SALES)
        {
            if (ctx.isNew(band))
            {
                return true;
            }
            else
            {
                final Long oldValue = getItemModelContext(band).getOriginalValue(BandModel.ALBUMSALES);
                if (oldValue == null || oldValue.intValue() < BIG_SALES)
                {
                    return true;
                }
            }
        }

        return false;
    }
}
```
2. Register the interceptor in `*-spring.xml`
```xml
<bean id="bandAlbumSalesInterceptor" class="concerttours.interceptors.BandAlbumSalesInterceptor" />
<bean id="BandInterceptorMapping" class="de.hybris.platform.servicelayer.interceptor.impl.InterceptorMapping">
	<property name="interceptor" ref="bandAlbumSalesInterceptor" />
	<property name="typeCode" value="Band" />
</bean>
```

