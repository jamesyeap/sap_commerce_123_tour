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

# CronJobs
A cron job consists of the job class containing the business logic, and a trigger to start the job at regular intervals.

The first step in setting up a new cron job is to notify SAP Commerce of your new class by creating your essential data.
* During the creation of essential data, a ServicelayerJob item is created for every Spring definition that has a class implementing the JobPerformable interface. The code attribute of each of the new job item is set to the name of the relevant Spring bean.
* To get a list of all ServicelayerJob items, run the following FlexibleSearch query in HAC
```
# get all service layer jobs
SELECT {code} FROM {servicelayerjob}

# get a specific service layer job (by the Java class name)
# this query looks for a servicelayerJob item that has been created for the job "sendNewsJob.java"
SELECT {code} FROM {servicelayerjob} WHERE {code} = 'sendNewsJob'
```

Once a ServicelayerJob item is created, you can create a cron job to wrap the ServicelayerJob, and define a trigger to that starts it.
* A cron expression is a string comprised of 6 or 7 fields separated by white space. Fields can contain any of the allowed values, along with various combinations of allowed special characters for that field.

## ServicelayerJob
### Writing Jobs (with Java)
1. Write job in `hybris/bin/custom/<EXTENSION_NAME>/src/<EXTENSION_NAME>/jobs`
```java
package concerttours.jobs;

import de.hybris.platform.cronjob.enums.CronJobResult;
import de.hybris.platform.cronjob.enums.CronJobStatus;
import de.hybris.platform.cronjob.model.CronJobModel;
import de.hybris.platform.servicelayer.config.ConfigurationService;
import de.hybris.platform.servicelayer.cronjob.AbstractJobPerformable;
import de.hybris.platform.servicelayer.cronjob.PerformResult;
import de.hybris.platform.util.mail.MailUtils;
import java.util.Date;
import java.util.List;
import org.apache.commons.mail.Email;
import org.apache.commons.mail.EmailException;
import org.apache.commons.configuration.Configuration;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Required;
import concerttours.model.NewsModel;
import concerttours.service.NewsService;

public class SendNewsJob extends AbstractJobPerformable<CronJobModel>
{
    private static final Logger LOG = Logger.getLogger(SendNewsJob.class);
    private NewsService newsService;
    private ConfigurationService configurationService;

    @Required
    public NewsService getNewsService()
    {
        return newsService;
    }

    @Required
    public ConfigurationService getConfigurationService()
    {
        return configurationService;
    }

    @Required
    public void setNewsService(final NewsService newsService)
    {
        this.newsService = newsService;
    }

    @Required
    public void setConfigurationService(final ConfigurationService configurationService)
    {
        this.configurationService = configurationService;
    }
    @Override
    public PerformResult perform(final CronJobModel cronJob)
    {
        LOG.info("Sending news mails. Note that org.apache.commons.mail.send() can block if behind a firewall/proxy.");
        final List<NewsModel> newsItems = getNewsService().getNewsOfTheDay(new Date());
        if (newsItems.isEmpty())
        {
            LOG.info("No news items for today, skipping send of ranking mails");
            return new PerformResult(CronJobResult.SUCCESS, CronJobStatus.FINISHED);
        }
        final StringBuilder mailContentBuilder = new StringBuilder(2000);
        int index = 1;
        mailContentBuilder.append("Todays news summary:\n\n");
        for (final NewsModel news : newsItems)
        {
            mailContentBuilder.append(index++);
            mailContentBuilder.append(". ");
            mailContentBuilder.append(news.getHeadline());
            mailContentBuilder.append("\n");
            mailContentBuilder.append(news.getContent());
            mailContentBuilder.append("\n\n");
        }
        try
        {
            sendEmail(mailContentBuilder.toString());
        }
        catch (final EmailException e)
        {
            LOG.error("Problem sending new email. Note that org.apache.commons.mail.send() can block if behind a firewall/proxy.)");
            LOG.error("Problem sending new email.", e);
            return new PerformResult(CronJobResult.FAILURE, CronJobStatus.FINISHED);
        }
        return new PerformResult(CronJobResult.SUCCESS, CronJobStatus.FINISHED);
    }
    private void sendEmail(final String message) throws EmailException
    {
        final String subject = "Daily News Summary";
        // get mail service configuration
        final Email email = MailUtils.getPreConfiguredEmail();
        //send message
        Configuration config = getConfigurationService().getConfiguration();
        String recipient = config.getString("news_summary_mailing_address", null);
        email.addTo(recipient);
        email.setSubject(subject);
        email.setMsg(message);
        email.setSSL(true);
        email.send();
        LOG.info(subject);
        LOG.info(message);
    }
}
```
2. Register the job in `*-spring.xml`
```xml
<bean id="sendNewsJob" class="concerttours.jobs.SendNewsJob" parent="abstractJobPerformable">
    <property name="newsService" ref="newsService" />
    <property name="configurationService" ref="configurationService" />
</bean>
```

## Writing Jobs (with Groovy)
1. In HAC, go to the Console tab, select the Scripting Languages option, and select Groovy in the Script type drop-down menu.
2. Enter the name of script in the **code** field above the input box.
3. Write the script in the input box.
```
import de.hybris.platform.cronjob.enums.*
import de.hybris.platform.servicelayer.cronjob.PerformResult
import de.hybris.platform.servicelayer.search.*
import de.hybris.platform.servicelayer.model.*
import de.hybris.platform.catalog.enums.ArticleApprovalStatus 
import concerttours.model.ConcertModel
  
searchService = spring.getBean("flexibleSearchService")
modelService = spring.getBean("modelService")
query = new FlexibleSearchQuery("Select {pk} from {Concert}");
searchService.search(query).getResult().each {
  if (it.daysUntil < 1) 
  { 
    it.approvalStatus = ArticleApprovalStatus.CHECK
  }
  modelService.saveAll()
}
```
4. Click on the **Save** button.

## Triggers
### Creating triggers using Impex (when Job is written in Java)
In `/hybris/bin/custom/<EXTENSION_NAME>/resources/impex`, add a file named `essentialdata-Jobs.impex`
* must include the prefix `essentialdata` to specify that this Impex file is run on initialization
* note that if running this Impex script in HAC, the "Enable code execution" option must be checked in the "Settings" area
```
# ===== EXAMPLE =====

# Define the cron job and the job that it wraps
# ===== In this example, the cronjob is named "sendNewsCronJob" and the Job class it uses is "sendNewsJob.java" =====
INSERT_UPDATE CronJob; code[unique=true];job(code);singleExecutable;sessionLanguage(isocode)
;sendNewsCronJob;sendNewsJob;false;de
 
# Define the trigger that periodically invokes the cron job
# ===== In this example, the trigger is invoked once every 10 seconds =====
INSERT_UPDATE Trigger;cronjob(code)[unique=true];cronExpression
#% afterEach: impex.getLastImportedItem().setActivationTime(new Date());
; sendNewsCronJob; 0/10 * * * * ?
```

### Creating triggers using Impex (when Job is written in Groovy)
```
# ===== EXAMPLE =====

# Define the ScriptingJob
INSERT_UPDATE ScriptingJob; code[unique= true ];scriptURI
;clearoldconcertsJob;model://clearoldconcerts

# Define the CronJob
INSERT_UPDATE CronJob; code[unique= true ];job(code);sessionLanguage(isocode)
;clearoldconcertsCronJob;clearoldconcertsJob;en

# Define the trigger
INSERT_UPDATE Trigger;cronjob(code)[unique=true];cronExpression
#% afterEach: impex.getLastImportedItem().setActivationTime(new Date());
; clearoldconcertsCronJob; 0/10 * * * * ?
```

# Backoffice
## Adding Backoffice to SAP Commerce
1. Add the extension to `hybris/config/localextension.xml`
```xml
<extension name="platformbackoffice"/> 
```
2. Rebuild the system
```bash
ant build
```
3. Reinitialize the system
```bash
ant initialize;

# then, go to HAC
# Platform > Initialization > (MAKE SURE "platformbackoffice" IS CHECKED) > Click on "Initialize"
```

# Localization
You can declare attributes and relations of item types as localized in an extension's `*-items.xml` file, and the system automatically provides for multiple values for different locales.
* Even the type system names themselves can be localized so that when item type and attribute names appear in user interfaces, they can be in the user's chosen language.

When the SAP build system encounters the localized keyword, instead of a single valued attribute, it creates a map of values keyed by language for that attribute, and generates an equivalent construct in the database.

Impex has a built-in syntax for specifying values for localized values.

## Enabling localization for attributes
In `*-items.xml` file,
```xml
<!-- Append "localized:" to the type specification -->
<attribute qualifier="hashtag" type="localized:java.lang.String">
  <description>hashtag of concert tour for social media</description>
  <persistence type="property" />
</attribute>
```
## Localizing Attribute Values with ImpEx
```
# Insert Products
INSERT_UPDATE Product;code[unique=true];name;band(code);hashtag[lang=en];hashtag[lang=de];$supercategories;manufacturerName;manufacturerAID;unit(code);ean;variantType(code);$catalogVersion;$approved
;201701;The Grand Little Tour;A001;GrandLittle;GrossWenig;;x;y;pieces;;Concert
```

## Localizing Attribute Values in HAC
1. In `/hybris/bin/custom/<EXTENSION_NAME>/resources/localization`, for each locale, add a file named `<EXTENSION_NAME>-locales_<LOCALE_ISOCODE>.properties`
* each entry is in the form `key=localized value`
```
# ===== ENTRY FORMATS =====

# type.<code of type>.name=XY
# type.<code of type>.<qualifier of attribute>.name=XY
# type.<code of type>.description=XY
# type.<code of type>.<qualifier of attribute>.description=XY
```
2. Go to HAC > Platform > Update
3. Select only the **"Localize types"** option
4. Click on the **"Update"** button

### Examples
* in `concerttours/resources/localization/concerttours-locales_en.properties`:
```
type.Band.name=Band
type.Band.description=a group or band of musicians of singers
type.Band.code.name=code
type.Band.code.description=unique short id for the band
type.Band.name.name=name
type.Band.name.description=name of the band
type.Band.history.name=history
type.Band.history.description=short story about the band
type.Band.albumSales.name=album sales
type.Band.albumSales.description=number of albums sold
```
* in `concerttours/resources/localization/concerttours-locales_de.properties`:
```
type.Band.name=Bande
type.Band.description=Eine Gruppe oder eine Band von Musikern oder Sängern
type.Band.code.name=code
type.Band.code.description=Einzigartige kurze ID für die Bande
type.Band.name.name=Name
type.Band.name.description=Name der Bande
type.Band.history.name=Geschichte
type.Band.history.description=Kurzgeschichte über die Bande
type.Band.albumSales.name=Albumverkauf
type.Band.albumSales.description=Anzahl der verkauften Alben
```

# Validation Constraints
## Validation constraints with `items.xml` file

## Validation constraints with Backoffice

## Validation constraints with ImpEx
In `hybris/bin/custom/<EXTENSION_NAME>/resources/impex`, create a file named `essentialdata-constraint.impex`
* which contains statements to insert constraints into the "MinConstraint" table

### Example
```
INSERT_UPDATE MinConstraint;id[unique=true];severity(code,itemtype(code));active;annotation;descriptor(enclosingType(code),qualifier);message[lang=de];message[lang=en];value
;AlbumSalesMustNotBeNegative;ERROR:Severity;true;javax.validation.constraints.Min;Band:albumSales;Albumverkäufe dürfen nicht negativ sein;Album sales must not be negative;0
```
