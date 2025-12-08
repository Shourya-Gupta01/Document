# CAP_INCIDENT_MNG

a basic cap application for incident management

# Getting Started

Welcome to your new project incident management.

It contains these folders and files, following our recommended project layout:

File or Folder | Purpose
---------|----------
`app/` | content for UI frontends goes here
`db/` | your domain models and data go here
`srv/` | your service models and code go here
`package.json` | project metadata and configuration
`readme.md` | this getting started guide

## Next Steps

- Open a new terminal and run `cds watch`
- (in VS Code simply choose _**Terminal** > Run Task > cds watch_)
- Start adding content, for example, a [db/schema.cds](db/schema.cds).

## Learn More

Learn more at [capire](https://cap.cloud.sap/docs/get-started/).

# Tutorials Hands-On 

- [Full-Stack Application with CAP](https://developers.sap.com/group.cap-application-full-stack.html)

---
## 1. Initialize Project

To initialize your project, run the following command:

```shell
cds init incident-management
```

This command creates a folder incident-management with your newly created CAP project.

While you’re in the incident-management folder, choose the burger menu and then choose Terminal → New Terminal.
Install dependencies with the following command:

```shell
npm install
```
The CAP server serves all the CAP sources from your project. It also “watches” all the files in your projects and conveniently restarts whenever you save a file. Changes you’ve made are immediately served without you having to run the command again. In this newly created project the CAP server tells you that there are no models or service definitions yet that it can serve.


---

## 2. Add a domain model

### For Node.js
1. In the db folder, create a new schema.cds file.

2. Paste the following code snippet in the schema.cds file.

```shell
using { cuid, managed, sap.common.CodeList } from '@sap/cds/common';
namespace sap.capire.incidents; 

/**
* Incidents created by Customers.
*/
entity Incidents : cuid, managed {  
customer     : Association to Customers;
title        : String  @title : 'Title';
urgency        : Association to Urgency default 'M';
status         : Association to Status default 'N';
conversation  : Composition of many {
    key ID    : UUID;
    timestamp : type of managed:createdAt;
    author    : type of managed:createdBy;
    message   : String;
};
}

/**
* Customers entitled to create support Incidents.
*/
entity Customers : managed { 
key ID        : String;
firstName     : String;
lastName      : String;
name          : String = trim(firstName ||' '|| lastName);
email         : EMailAddress;
phone         : PhoneNumber;
incidents     : Association to many Incidents on incidents.customer = $self;
creditCardNo  : String(16) @assert.format: '^[1-9]\d{15}$';
addresses     : Composition of many Addresses on addresses.customer = $self;
}

entity Addresses : cuid, managed {
customer      : Association to Customers;
city          : String;
postCode      : String;
streetAddress : String;
}

entity Status : CodeList {
key code: String enum {
    new = 'N';
    assigned = 'A'; 
    in_process = 'I'; 
    on_hold = 'H'; 
    resolved = 'R'; 
    closed = 'C'; 
};
criticality : Integer;
}

entity Urgency : CodeList {
key code: String enum {
    high = 'H';
    medium = 'M'; 
    low = 'L'; 
};
}

type EMailAddress : String;
type PhoneNumber : String;

```
What happens here?

The provided CDS code snippet defines several entities and their relationships.

The Incidents entity represents incidents created by customers, with fields for customer, title, urgency, status, and a composition of many conversations. Each conversation has an ID, timestamp, author, and message.

The Customers entity represents customers who can create support incidents. It includes fields for ID, first name, last name, e-mail, phone, credit card number, and a composition of many addresses. The name field is calculated from the firstName and lastName fields. Elements can be specified with a calculation expression, in which you can refer to other elements of the same entity. These calculated elements are used for convenience. For more information, see Calculated Elements.

The Addresses entity represents the addresses of customers, with fields for customer, city, postcode, and street address.

The Status and Urgency entities represent the status and urgency of incidents, respectively. Both are of type CodeList and include a code field with a set of predefined values.

The EMailAddress and PhoneNumber are defined as types of String.

The code also includes the use of cuid and managed from @sap/cds/common, which are common features for defining entities in CDS. The cuid feature provides a unique identifier for an entity, while managed adds common administrative fields such as createdAt and createdBy.

Now to start the application use `cds watch` is recommended as it detects `cds-tsx` under the hood after detecting `tsconfig.json`. While `cds-ts` can also be used, `cds-tsx` is generally faster.

Run the following command:

```shell
cds watch
```

---

## 3. Create services
### For Node.js
It’s a good practice in CAP to create single-purposed services. Therefore, let’s define a ProcessorService that support engineers use to process incidents created by customers. Let’s also define an AdminService that administrators use to perform admin activities such as analyzing audit logs.

To create the services’ definition:

1. In the srv folder, create a new services.cds file.

2. Paste the following code snippet in the services.cds file:

```shell
using { sap.capire.incidents as my } from '../db/schema';

/**
 * Service used by support personell, i.e. the incidents' 'processors'.
 */
service ProcessorService { 
    entity Incidents as projection on my.Incidents;

    @readonly
    entity Customers as projection on my.Customers;
}

/**
 * Service used by administrators to manage customers and incidents.
 */
service AdminService {
    entity Customers as projection on my.Customers;
    entity Incidents as projection on my.Incidents;
    }

```

This time, the CAP server reacted with additional output in the terminal

As you can see in the log output, the new file created two generic service providers: ProcessorService that serves requests on the /odata/v4/processor endpoint and AdminService that serves requests on the /odata/v4/admin endpoint. If you open the link <http://localhost:4004> from SAP Business Application Studio in your browser, you see the generic index.html page:


You have to stop the CAP server with ```Ctrl + C``` and start it again with the ```cds watch``` command.

---

## 4. Generate comma-separated values (CSV) templates
Since we already have an SQLite in-memory database that was automatically created in the previous step, let’s now fill it with some test data.

1. Run the following command in the incident-management root folder of your project:

 ```shell
cds add data
```
2. Check the output similar as below.
Adding feature 'data'...
Creating db/data/sap.capire.incidents-Addresses.csv
Creating db/data/sap.capire.incidents-Customers.csv
Creating db/data/sap.capire.incidents-Incidents.csv
Creating db/data/sap.capire.incidents-Incidents.conversation.csv
Creating db/data/sap.capire.incidents-Status.csv
Creating db/data/sap.capire.incidents-Status.texts.csv
Creating db/data/sap.capire.incidents-Urgency.csv
Creating db/data/sap.capire.incidents-Urgency.texts.csv

Successfully added features to your project.


You can find the generated CSV templates within the db folder, in a newly created data folder.


---

## 5. Create services
### For Node.js
Important consideration for test data

In the previous step, you added several CSV files with test data. These files are required to pre-fill the SQLite memory with data for local testing. Test data is most suitable for development environments, where schema changes are frequent and broad.

When you redeploy your database, it deletes all tables and views and create them again. This behavior is known as drop-create.

- Test files should never be deployed to an SAP HANA production database as table data.
In such cases, changing a data file can cause the deletion of all files of affected database tables, even if the data files for the affected tables have been removed before. SAP HANA remembers all data files that have ever been deployed to the tables and might restore them.

- Only master data files are be delivered in this way. Master data files are files, which are defined by the application developer and can’t be changed by the application. Examples for master data include country codes, status codes and criticality, and urgency codes and descriptions.

- Delivering files for tables with customer data will cause data loss in productive scenarios! See section [Providing Initial Data](https://cap.cloud.sap/docs/guides/databases#providing-initial-data) in the CAP documentation for more details.

- Drop-create is most appropriate for development. However, drop-create isn’t suitable for database upgrades in production, as all customer data is lost. To avoid data loss, cds deploy also supports automatic schema evolution. See section [Schema Evolution](https://cap.cloud.sap/docs/guides/databases-sqlite#schema-evolution).


Replace the respective generated CSV templates with the following content:
- db/data/sap.capire.incidents-Addresses.csv:
```shell
ID,customer_ID,city,postCode,streetAddress
17e00347-dc7e-4ca9-9c5d-06ccef69f064,1004155,Rome,00164,Piazza Adriana
d8e797d9-6507-4aaa-b43f-5d2301df5135,1004161,Munich,80809,Olympia Park
ff13d2fa-e00f-4ee5-951c-3303f490777b,1004100,Walldorf,69190,Dietmar-Hopp-Allee
```
- db/data/sap.capire.incidents-Customers.csv:
 ```shell
ID,firstName,lastName,email,phone
1004155,Daniel,Watts,daniel.watts@demo.com,+39-555-123
1004161,Stormy,Weathers,stormy.weathers@demo.com,+49-020-022
1004100,Sunny,Sunshine,sunny.sunshine@demo.com,+49-555-789
```
- db/data/sap.capire.incidents-Incidents.conversation.csv:
```shell
ID,up__ID,timestamp,author,message
2b23bb4b-4ac7-4a24-ac02-aa10cabd842c,3b23bb4b-4ac7-4a24-ac02-aa10cabd842c,1995-12-17T03:24:00Z,Harry John,Can you please check if battery connections are fine?
2b23bb4b-4ac7-4a24-ac02-aa10cabd843c,3a4ede72-244a-4f5f-8efa-b17e032d01ee,1995-12-18T04:24:00Z,Emily Elizabeth,Can you please check if there are any loose connections?
9583f982-d7df-4aad-ab26-301d4a157cd7,3583f982-d7df-4aad-ab26-301d4a157cd7,2022-09-04T12:00:00Z,Sunny Sunshine,Please check why the solar panel is broken.
9583f982-d7df-4aad-ab26-301d4a158cd7,3ccf474c-3881-44b7-99fb-59a2a4668418,2022-09-04T13:00:00Z,Bradley Flowers,What exactly is wrong?
```
- db/data/sap.capire.incidents-Incidents.csv:
```shell
ID,customer_ID,title,urgency_code,status_code
3b23bb4b-4ac7-4a24-ac02-aa10cabd842c,1004155,Inverter not functional,H,C
3a4ede72-244a-4f5f-8efa-b17e032d01ee,1004161,No current on a sunny day,H,N
3ccf474c-3881-44b7-99fb-59a2a4668418,1004161,Strange noise when switching off Inverter,M,N
3583f982-d7df-4aad-ab26-301d4a157cd7,1004100,Solar panel broken,H,I
```
- db/data/sap.capire.incidents-Status.csv:
```shell
code,descr,criticality
N,New,3
A,Assigned,2
I,In Process,2
H,On Hold,3
R,Resolved,2
C,Closed,4
```
- db/data/sap.capire.incidents-Urgency.csv:
```shell
code,descr
H,High
M,Medium
L,Low
```

Notice that ```cds add data``` created eight files, while we’re adding data to just six of them. We’re leaving the files ```sap.capire.incidents-Status.texts.csv``` and ```sap.capire.incidents-Urgency.texts.csv``` empty because they hold translated text that are filled once the application is localized and translations are created.


Upon detecting these new files, the CAP server prints a message stating that the content of the files has been filled into the database automatically:
[cds] - connect to db > sqlite { database: ':memory:' }
  > init from db\data\sap.capire.incidents-Addresses.csv 
  > init from db\data\sap.capire.incidents-Customers.csv 
  > init from db\data\sap.capire.incidents-Incidents.csv 
  > init from db\data\sap.capire.incidents-Incidents.conversation.csv
  > init from db\data\sap.capire.incidents-Status.csv 
  > init from db\data\sap.capire.incidents-Status.texts.csv 
  > init from db\data\sap.capire.incidents-Urgency.csv 
  > init from db\data\sap.capire.incidents-Urgency.texts.csv 
/> successfully deployed to in-memory database.


Make sure that your CAP server is still running. You can start it with ```cds watch```.


Now that the database is filled with some initial data, you can send complex OData queries served by the built-in generic service providers. With the generic index.html page opened in your browser, paste the following queries at the end of the current URL and check the result:
- /odata/v4/processor/Incidents
- /odata/v4/processor/Customers?$select=firstName&$expand=incidents

---
# Add SAP Fiori Elements UIs
## 1. Generate the UI with an SAP Fiori Elements template
1. Go to your IncidentManagement space.
2. Choose the burger menu and then choose View → Command Palette.
   You can also invoke the Command Palette quickly using the following key combination:

- For macOS: Command + Shift + P
- For Windows: Ctrl + Shift + P
3. Type Fiori: Open Application Generator in the field and select this entry from the list.

4. In the Template Selection step:

- Choose the List Report Page template tile.

- Choose Next.
5. In the Data Source and Service Selection step:

- In the Data Source dropdown menu, select Use a Local CAP Project.

- In the Choose a CAP project dropdown menu, select the incident-management project.

- In the OData Service dropdown menu, select the ProcessorService (Node.js) (or ProcessorService (Java) if you’re developing a CAP Java application).

- Choose Next.

6. In the Entity Selection step:

- In the Main Entity dropdown menu, select Incidents.

- Leave the Navigation Entity value as None, and then select Yes to add table columns automatically.

- In the Table Type dropdown menu, select Responsive.

- Choose Next.
7. In the Project Attributes step:

- In the Module Name field, enter incidents.

- In the Application Title field, enter Incident-Management.

- In the Application Namespace field, enter ns.

- Leave the default values for all the other settings and choose Finish.
 
The application is now generated and in a few seconds you can see the application’s incidents folder in the app folder of your incident-management project. It contains a webapp folder with a Component.js file that’s typical for an SAPUI5 application. However, the source code of this application is minimal. It inherits its logic from the sap/fe/core/AppComponent class. This class is managed centrally by SAP Fiori elements and provides all the services that are required (routing, edit flow) so that the building blocks and the templates work properly.

---
## 2. Start the Incident-Management application
### Node.js
You can create a CAP project in either Node.js or Java. You have to choose one way or the other and follow through. The tabs Node.js and Java provide detailed steps for each alternative way.

Instead of using ```cds watch``` command in the terminal to start the service, you use the ```watch-incidents``` script that has been added to the package.json file by the application generator. The script contains an additional ```sap-ui-xx-viewCache=false``` parameter added to the application’s start URL. This parameter ensures that if custom extensions are implemented, changes to the extension artifacts get properly updated when refreshing the UI.

If the ```cds watch``` command is already running in a terminal, end it with the ```Ctrl + C``` key combination. Otherwise, the default port 4004 is already in use by the running CAP server’s process.

1. In the Application Info - incidents tab, choose the Preview Application option.
If you get an error SyntaxError: Unexpected token / in JSON at position 4, open the file .vscode/launch.json, delete any comments that you have there, and try again.

This option opens a dropdown menu at the top offering all scripts maintained in the scripts section of the ```package.json``` file that is based on the ```cds run``` and ```cds watch``` commands.

In case the Application Info - incidents tab is closed:

1.1. Invoke the Command Palette - View → Command Palette or Command + Shift + P for macOS / Ctrl + Shift + P for Windows. <br>
1.2. Choose Fiori: Open Application Info.

2. Select the watch-incidents npm script.
This script runs the service in a terminal session of the application modeler and automatically starts the SAP Fiori application in a new browser session.

3. You can now see the application with the generated columns. Choose Go.

---
## 3. Configure the List View Page
In this section, you modify the List View Page of the UI with the SAP Fiori page editor.

Add filter fields
1. In the Application Info - incidents tab, choose the Open Page Map option.
   The page map of the incidents application opens in a new tab within SAP Business Application Studio. You see the properties on the right side of the page map. You can edit these properties to update the UI of the application.
   In case the Application Info - incidents tab is closed:

1.1. Invoke the Command Palette - View → Command Palette or Command + Shift + P for macOS / Ctrl + Shift + P for Windows. <br>
1.2. Choose Fiori: Open Application Info.

2. In the List Report tile, choose the Pencil icon next to the title. The page editor is opened.
3. In the Filter Bar section of the page editor, choose Filter Fields and then choose the Plus icon to add filter fields. Then, select Add Filter Fields in the dropdown menu.
4. In the Add Filter Fields popup:

- Select the status_code and urgency_code checkboxes in the Filter Field dropdown menu.
- Choose Add. Your application is updated to show the new filters.
  This way you define which properties are exposed as search fields in the header bar above the list.
  
## Edit filter fields
The filter labels are text strings. It’s a good idea to update them so they’re compliant with internationalization standards (i18n).

1. Change the urgency_code filter label. In the Label field, change the value to Urgency. Press Enter to confirm the change.
2. Choose the Globe icon to generate a translatable text key and choose Substitute.
3. Choose the status_code filter. In the Label field, change the value to Status. Press Enter to confirm the change.

4. Choose the Globe icon to generate a translatable text key and choose Substitute.

5. For both the Urgency and Status filters, in the Display Type dropdown menu, select Value Help. A popup shows up.
6. In the Define Value Help Properties for Urgency/Status popup:

- Choose the dropdown menu in the Value Description Property field.
- Select descr.
- Choose Apply.
  
## Edit columns
1. Expand the Columns section under Table and delete the columns customer_ID, urgency_code and status_code.
2. In Table → Columns, choose the Plus icon to add columns. Choose Add Basic Columns.
3. In the Add Basic Columns popup, choose the dropdown menu in the Columns field and:
   - Select the status → descr checkbox.
- Select the urgency → descr checkbox.
- Select the customer → name checkbox and add the columns.
4. Move the name column just under the Title column.
5. Choose the Title column, choose the Globe icon in the Label field to generate a translatable text key.

The filter labels are text strings. It’s a good idea to update them so they’re compliant with internationalization standards (i18n).

Learn more about how internationalization works for the back end part in Where to Place Text Bundles? in the CAP documentation.

6. For each of the name, Description (status/descr), and Description (urgency/descr) columns:

- In the Label field, change the value to Customer, Status, and Urgency, respectively.
- Press Enter to confirm the change.
- Choose the Globe icon in the Label field to generate a translatable text key.

## Configure tables
1. Choose Table and, in the Initial Load dropdown menu, select Enabled to load the data automatically.
2. In the Type dropdown menu, select ResponsiveTable to make the table responsive.
3. Navigate to Table → Columns → Status and in the Criticality dropdown menu, select status/criticality.
   
## Check the result
The list page of the Incident Management application is ready and running.
Navigate back to the page editor and choose Page Map in the top left. This action takes you back to the overview of the incidents application.

---
## 4. Configure the Incident Object Page
In this section, you modify the Incident Object Page of the UI with the SAP Fiori page editor.

## Edit header
1. Make sure that the SAP Fiori page editor is open. If you closed it, choose the Open Page Map option in the Application Info - incidents tab.

To open the Application Info - incidents tab:

1.1. Invoke the Command Palette - View → Command Palette or Command + Shift + P for macOS / Ctrl + Shift + P for Windows. <br>
1.2. Choose Fiori: Open Application Info.

2. In the Incident Object Page tile, choose the Pencil icon next to the title.
3. Choose Header and in the Title dropdown menu, select title.
4. In the Description Type dropdown menu, select Property. A popup opens.
5. In the Define Property popup, choose the dropdown menu in the Description field and:

- Select customer → name.
- Choose Apply.
In the Icon URL field, enter ``sap-icon://alert``.

## Add Overview section
1. Choose Sections and then choose the Plus icon to add more sections. Choose Add Group Section.
2. In the Add Group Section popup:

- Enter Overview in the Label field.
- Choose the Globe icon to generate a translatable text key and choose Substitute.
- Choose Add.
## Add Details subsection
1. Navigate to Sections → Overview → Subsections.

2. Choose the Plus icon to add more sections and choose Add Form Section.
3. In the Add Form Section popup:

- Enter Details in the Label field.
- Choose the Globe icon to generate a translatable text key and choose Substitute.
- Choose Add.
- 
## Configure fields
1. Choose Sections → General Information and choose the Globe icon in the Label field for General Information to generate a translatable text key.
2. Navigate to Sections → General Information → Form → Fields and delete the urgency_code and status_code fields.
3. For the customer_ID field:

- Move the field down just under the Title field.
- In the Label field, change the value to Customer.
- Press Enter to confirm the change.
- Choose the Globe icon in the Label field to generate a translatable text key.
4. For the Customer field, select customer/name in the Text dropdown menu, select Text Only in the Text Arrangement dropdown menu, and then select the hyperlink Edit properties for Value Help under Display Type. A popup opens.
5. In the Define Value Help Properties for Customer popup:

- Select None in the Value Description Property dropdown.
- Switch off Display as Dropdown.
- Under Result List, choose Add Column and add columns name and email.
- Choose Apply.
  If there are already some columns in the result list, then delete them and keep only the columns name and email.

6. Navigate to Sections, drag the General Information, and drop it in the Overview → Subsections node.
7. Navigate to Sections → Overview → Subsections → Details → Form → Fields, choose the Plus icon to add more fields, and then choose Add Basic Fields.
8. In the Add Basic Fields popup

- From the dropdown menu in the Fields field, select status_code and urgency_code.
- Choose Add.
9. For the Status field, select status/descr in the Text dropdown menu and then select Value Help in the Display Type dropdown menu. A popup opens.
10. In the Define Value Help Properties for Status popup:

- Select Status in the Value Source Entity dropdown menu.
- Select code in the Value Source Property dropdown menu.
- Select descr in the Value Description Property dropdown menu.
- Leave the default values for the rest of the properties and choose Apply.
11. For the Urgency field, select urgency/descr in the Text dropdown menu and then select Value Help in the Display Type dropdown menu. A popup opens.
12. In the Define Value Help Properties for Urgency popup:

- Select Urgency in the Value Source Entity dropdown menu.
- Select code in the Value Source Property dropdown menu.
- Select descr in the Value Description Property dropdown menu.
- Leave the rest of the properties with the default values and choose Apply.
## Add Conversation section
1. Navigate to Sections and then choose the Plus icon to add more sections. Select Add Table Section in the dropdown menu.

2. Choose Add Table Section. A popup appears.
3. In the Add Table Section popup:

- Enter Conversation in the Label field.
- Choose the Globe icon to generate a translatable text key.
- Select conversation in the Value Source dropdown menu and choose Add.
## Configure columns
1. Navigate to Conversation → Table → Columns and choose the Plus icon to add columns.

2. Choose Add Basic Columns. A popup appears.
3. In the Add Basic Columns popup:

- In the Columns dropdown menu, select author, message, and timestamp.
- Choose Add.
 4. For each of the CreatedBy, message, and CreatedOn columns:

- In the Label field, change the value to Author, Message, and Date, respectively.
- Press Enter to confirm the change.
- Choose the Globe icon in the Label field to generate a translatable text key.
## Configure the table and check the result
1. For Table, in the Type dropdown menu, select ResponsiveTable to make the table responsive.
2. In the Creation Mode: Name dropdown menu, select Inline.
3. Preview the app.

---
## 5. Enable draft with `@odata.draft.enabled`

SAP Fiori supports editing business entities with draft states stored on the server, so users can interrupt editing and continue later on, possibly from different places and devices. CAP, as well as SAP Fiori elements, provides out-of-the-box support for drafts. We recommend that you always use draft when your SAP Fiori application needs data input by users.

- For more details, see the SAP Fiori Design Guidelines for Draft Handling.
- Read more about Draft Support in the CAP documentation.
  
Enabling a draft for an entity allows the users to edit the entities. To enable a draft for an entity exposed by a service, follow these steps:

1. Open the srv/services.cds file.

2. Annotate the file with @odata.draft.enabled like this:
```shell
service ProcessorService { 
...
}
...
annotate ProcessorService.Incidents with @odata.draft.enabled; 
```
3. Start creating a new incident but leave the Customer, Status, and Urgency fields empty.
4. Go back to the list view page without creating the incident. You see the incident draft there with the empty fields.
5. Try to access it again to continue the creation from where you stopped.

---
## Add Custom Logic
## 1. Add Custom Logic
### For Node.js
You can create a CAP project in either Node.js or Java. You have to choose one way or the other and follow through. The tabs Node.js and Java provide detailed steps for each alternative way.

In this tutorial, you add some custom code to the CAP application. Depending on the contents of the title property, the custom code changes the value of the urgency property.

1. In SAP Business Application Studio, go to your IncidentManagement dev space.

Make sure the IncidentManagement dev space is in status RUNNING.

2. Create a new services.js file in the srv folder of the INCIDENT-MANAGEMENT application.

3. Add the following code (the actual business logic) to the services.js file:
```shell
const cds = require('@sap/cds')

class ProcessorService extends cds.ApplicationService {
  /** Registering custom event handlers */
  init() {
    this.before("UPDATE", "Incidents", (req) => this.onUpdate(req));
    this.before("CREATE", "Incidents", (req) => this.changeUrgencyDueToSubject(req.data));

    return super.init();
  }

  changeUrgencyDueToSubject(data) {
    let urgent = data.title?.match(/urgent/i)
    if (urgent) data.urgency_code = 'H'
  }

  /** Custom Validation */
  async onUpdate (req) {
    let closed = await SELECT.one(1) .from (req.subject) .where `status.code = 'C'`
    if (closed) req.reject `Can't modify a closed incident!`
  }
}
module.exports = { ProcessorService }
```
4. Make sure that the SAP Fiori application is running. If you closed it, choose the Preview Application option in the Application Info - incidents tab and select the watch-incidents npm script.

To open the Application Info - incidents tab:

4.1. Invoke the Command Palette - View → Command Palette or Command + Shift + P for macOS / Ctrl + Shift + P for Windows.
4.2. Choose Fiori: Open Application Info.
5. Create a new incident with the word urgent in its title and with the urgency set to Medium.
You see that the value in the Urgency field is automatically set to high.

## 2. Understanding the custom code
### For Node.js
Because your file is called services.js and has the same name as your application definition file services.cds, CAP automatically treats it as a handler file for the application defined in there. CAP exposes several [events](https://cap.cloud.sap/docs/node.js/events) and you can easily write handlers like the one mentioned earlier.

In this case, the event is triggered after a READ was carried out for the Incidents entity. In your custom handler, you get all the data (in this case, all the incidents) that was read according to the query. You can loop over each of them and, if needed, adjust the data of the response. In this case, you change the value of the urgency property when the title contains the word urgent. The new values for Urgency are then part of the response to the READ request.
