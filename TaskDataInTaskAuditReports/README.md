# Task Data in Task Audit Trail Reports

This guide provides step-by-step instructions for displaying task data in task audit reports. The process involves creating a custom dataset, setting up API authentication, configuring an integration, creating an import workflow, and establishing custom joins between datasets.

## Table of Contents

- [Step 1: Creating a Custom Dataset](#step-1-creating-a-custom-dataset)
- [Step 2: Creating an API Key](#step-2-creating-an-api-key)
- [Step 3: Creating an Integration](#step-3-creating-an-integration)
- [Step 4: Creating an Import Workflow](#step-4-creating-an-import-workflow)
- [Step 5: Creating a Custom Join](#step-5-creating-a-custom-join)
- [Step 6: View the end result](#step-6-view-the-end-result)

---

## Step 1: Creating a Custom Dataset

### 1.1 Navigate to Custom Datasets

- Navigate to the **Datasets** section from the main menu

<p align="center">
  <img src="images/CreatingCustomDataset/1.png" alt="Step 1" style="max-width: 600px; max-height: 600px;">
</p>

### 1.2 Configure Dataset Settings

- Click on the New Dataset button

<p align="center">
  <img src="images/CreatingCustomDataset/2.png" alt="Step 2" style="max-width: 600px; max-height: 600px;">
</p>

### 1.3 Review Dataset Configuration

- Create a table with columns that match the data in the tasks you want to match with the task audit trail. In this example we are using the Connection profiling list report
- To help with performance, only include the columns you wish to include in your reports. We don't recommend including columns like the SQL for the same reason
- Be sure to include a task number column - that is what will be used to join on
- Choose column lengths longer than the task data. If data is attempted to be imported that is longer than the custom table column length, the import process will fail
- Click Save. This new table will be the destination for the task data

<p align="center">
  <img src="images/CreatingCustomDataset/3.png" alt="Step 3" style="max-width: 600px; max-height: 600px;">
</p>

---

## Step 2: Creating an API Key

### 2.1 Access API Key Management

- Navigate to the **API Keys** page

<p align="center">
  <img src="images/CreateAPIKey/1.png" alt="Step 1" style="max-width: 600px; max-height: 600px;">
</p>

### 2.2 Initiate API Key Creation

- Click on **Create API key** button

<p align="center">
  <img src="images/CreateAPIKey/2.png" alt="Step 2" style="max-width: 600px; max-height: 600px;">
</p>

### 2.3 Configure API Key Details

- Provide an API key name - something descriptive like "Task Import API Key" makes sense

<p align="center">
  <img src="images/CreateAPIKey/3.png" alt="Step 3" style="max-width: 600px; max-height: 600px;">
</p>

### 2.4 Save and Copy API Key

- Save the API key configuration
- Copy and securely store the **Encoded token**. We will be using this value later and it will not be
able to be retrieved again after you navigate away (create a new key if you lose this key later)

<p align="center">
  <img src="images/CreateAPIKey/4.png" alt="Step 4" style="max-width: 600px; max-height: 600px;">
</p>

---

## Step 3: Creating an Integration

### 3.1 Navigate to Integrations

- Go to the **Integrations** page
- Click to create a new integration

<p align="center">
  <img src="images/CreateIntegration/1.png" alt="Step 1" style="max-width: 600px; max-height: 600px;">
</p>

### 3.2 Select Integration Type

- Switch to the "Discover" tab
- Choose "Inbound application programming interface (API)"

<p align="center">
  <img src="images/CreateIntegration/2.png" alt="Step 2" style="max-width: 600px; max-height: 600px;">
</p>

### 3.3 Choose the data set
- Choose the data set created in step 1 above

<p align="center">
  <img src="images/CreateIntegration/3.png" alt="Step 3" style="max-width: 600px; max-height: 600px;">
</p>

### 3.4 Configure API endpoint

- Give the Integration a name. We used "Task data import"
- For the URL, specify the following audit task endpoint: https://guardium.security.ibm.com/api/v3/cases/*/tasks
- For the certificate, use the text in the included [guardium_prod_cert.txt](guardium_prod_cert.txt) file
- For the authentication type choose "Authentication header". Specify "authorization" as the header name and include the API key copied in step 2.4 above
- Keep the defaults for the rest of the parameters and scroll down to the bottom of the page.
Click "Test Connection". This should populate the API endpoint data on the right similar to what is shown in the screenshot below.


<p align="center">
  <img src="images/CreateIntegration/4.png" alt="Step 4" style="max-width: 600px; max-height: 600px;">
</p>

### 3.5 Configure data mapping
This is where a JMESPath to translate the data into an array of JSON objects is specified. Once
done a mapping of the object fields to the JSON object keys is defined.
- Select "Overwrite" for the Data integration type
- Specify the following aggregation query, adjusting it:
  - Change the title to the report name that generates the target audit tasks
  - Change the fields (eg: ClientIP, DBUserName, etc.) to reflect the data in your audit tasks
```
tasks[?title == 'Connection profiling list(1)'].{ number: join('-', ['TASK', number]), ClientIP: report_result.rows[0].row.ClientID, DBUserName: report_result.rows[0].row.DBUserName, DatabaseName: report_result.rows[0].row.DatabaseName, OSUser: report_result.rows[0].row.OSUser, ServerHostName: report_result.rows[0].row.ServerHostName, ServerIP: report_result.rows[0].row.ServerIP, ServerType: report_result.rows[0].row.ServerType, ServiceName: report_result.rows[0].row.ServiceName, SourceProgram: report_result.rows[0].row.SourceProgram, DatabaseType: report_result.rows[0].row.DatabaseType}
```
- Click "Run Query". You should see a JSON array with the task numbers and the fields you wish to
import



<p align="center">
  <img src="images/CreateIntegration/5.png" alt="Step 5" style="max-width: 600px; max-height: 600px;">
</p>

### 3.6 Perform mapping
- Scroll down and map the fields that result from the aggregation in the previous step to the
custom dataset columns

<p align="center">
  <img src="images/CreateIntegration/6.png" alt="Step 6" style="max-width: 600px; max-height: 600px;">
</p>

### 3.7 Save Integration

- Click Next and Finish

<p align="center">
  <img src="images/CreateIntegration/7.png" alt="Step 7" style="max-width: 600px; max-height: 600px;">
</p>

---

## Step 4: Creating an Import Workflow

### 4.1 Access Workflow Management

- Navigate to the **Workflows** page
- Click to create a new import workflow

<p align="center">
  <img src="images/CreateImportWorkflow/1.png" alt="Step 1" style="max-width: 600px; max-height: 600px;">
</p>

### 4.2 Create the workflow
- Click Manage workflow then Create workflow

<p align="center">
  <img src="images/CreateImportWorkflow/2.png" alt="Step 2" style="max-width: 600px; max-height: 600px;">
</p>

### 4.3 Select Workflow type
- Choose the Scheduled workflow type

<p align="center">
  <img src="images/CreateImportWorkflow/3.png" alt="Step 3" style="max-width: 600px; max-height: 600px;">
</p>

### 4.4 Add action
- Add an "Import data" action from the drop down

<p align="center">
  <img src="images/CreateImportWorkflow/4.png" alt="Step 4" style="max-width: 600px; max-height: 600px;">
</p>

### 4.5 Select the integration
- Choose the integration we created in step 3
<p align="center">
  <img src="images/CreateImportWorkflow/5.png" alt="Step 5" style="max-width: 600px; max-height: 600px;">
</p>

### 4.6 Proceed with scheduling
- This workflow will only have the single action - the data import

<p align="center">
  <img src="images/CreateImportWorkflow/6.png" alt="Step 6" style="max-width: 600px; max-height: 600px;">
</p>

### 4.7 Select notification mechanism
- For the notification select "IBM Guardium"

<p align="center">
  <img src="images/CreateImportWorkflow/7.png" alt="Step 7" style="max-width: 600px; max-height: 600px;">
</p>

### 4.8 Schedule the job
- Create a daily import schedule. It makes sense to do this at a time after the workflow that creates
the tasks runs

<p align="center">
  <img src="images/CreateImportWorkflow/8.png" alt="Step 8" style="max-width: 600px; max-height: 600px;">
</p>

### 4.9 Set the name
- Name the workflow or keep the default and click next

<p align="center">
  <img src="images/CreateImportWorkflow/9.png" alt="Step 9" style="max-width: 600px; max-height: 600px;">
</p>

### 4.10 Set the permissions
- Set the permissions or keep the default and click next

<p align="center">
  <img src="images/CreateImportWorkflow/10.png" alt="Step 10" style="max-width: 600px; max-height: 600px;">
</p>

### 4.11 Run the workflow to test it out
- Click on the three-dot menu for the workflow and click "Run Once Now"

<p align="center">
  <img src="images/CreateImportWorkflow/11.png" alt="Step 11" style="max-width: 600px; max-height: 600px;">
</p>

### 4.12 Monitor execution
- You can monitor the execution of the workflow by clicking on the workflow link and selecting the
"Run history" tab

<p align="center">
  <img src="images/CreateImportWorkflow/12.png" alt="Step 12" style="max-width: 600px; max-height: 600px;">
</p>

### 4.13 Confirm successful execution
- Wait for the workflow to complete. If the workflow fails with the error in the second
screenshot below, check the following:
  - The column widths for the target data set are appropriately set and
  - The filter in the JMESPath is only getting data related to the report you are interested in (null values for a column will result in this error)

<p align="center">
  <img src="images/CreateImportWorkflow/13.png" alt="Step 13" style="max-width: 600px; max-height: 600px;">
</p>

<p align="center">
  <img src="images/CreateImportWorkflow/14.png" alt="Step 14" style="max-width: 600px; max-height: 600px;">
</p>

### 4.14 Confirm data import
- Once a successful import has occurred, you can view the data imported by going back to the
dataset page and clicking of the target dataset. The imported data will be in the "Records" tab.
If you like you can skip this step as a data preview will also be shown in the next section
where a custom join is created.

<p align="center">
  <img src="images/CreateImportWorkflow/15.png" alt="Step 15" style="max-width: 600px; max-height: 600px;">
</p>

---

## Step 5: Creating a Custom Join

### 5.1 Access reports interface

- Access the reports page using the main menu

<p align="center">
  <img src="images/CreateCustomJoin/1.png" alt="Step 1" style="max-width: 600px; max-height: 600px;">
</p>

### 5.2 Open a report that uses the task audit data
- Open the report that you wish to include the task details with the task audit data
In this example we're using the built-in "Audit results log" report. Adding the join to
any report in the report category will make the joined columns available to all reports
in the category

<p align="center">
  <img src="images/CreateCustomJoin/2.png" alt="Step 2" style="max-width: 600px; max-height: 600px;">
</p>

### 5.3 Edit the columns
- In the main data table area, click the "Customize columns" button

<p align="center">
  <img src="images/CreateCustomJoin/3.png" alt="Step 3" style="max-width: 600px; max-height: 600px;">
</p>

### 5.4 Click the add columns button

- Select the "Add columns from custom dataset" button at the top of the column menu

<p align="center">
  <img src="images/CreateCustomJoin/4.png" alt="Step 4" style="max-width: 600px; max-height: 600px;">
</p>

### 5.5 Select the data set
- Select the custom data set we created earlier. You should see the imported data in 
the preview pane
<p align="center">
  <img src="images/CreateCustomJoin/5.png" alt="Step 5" style="max-width: 600px; max-height: 600px;">
</p>

### 5.6 Create the join condition
- Here we specify what columns we'll join on. In this case we want to join the imported
TaskNumber column to the "Entry title" column in the Audit log category

<p align="center">
  <img src="images/CreateCustomJoin/6.png" alt="Step 6" style="max-width: 600px; max-height: 600px;">
</p>

### 5.7 Select columns to display
- Select the columns you would like to include in your reports by selecting the check
boxes at the top of the screen
- Click Next

<p align="center">
  <img src="images/CreateCustomJoin/7.png" alt="Step 7" style="max-width: 600px; max-height: 600px;">
</p>


### 5.8 Confirm Join Configuration
- Click Submit to create the join and add the columns

<p align="center">
  <img src="images/CreateCustomJoin/9.png" alt="Step 9" style="max-width: 600px; max-height: 600px;">
</p>

### 5.9 Review the columns
- The columns are now shown in the column editor. By default, the columns you added
all get selected and added to the report
- Click Apply

<p align="center">
  <img src="images/CreateCustomJoin/8.png" alt="Step 8" style="max-width: 600px; max-height: 600px;">
</p>


## Step 6: View the end result
We have now added the audit task details to our audit log report. You can use these new
fields the same way as any report field. The data will be refreshed according to the 
schedule created in Step 4. If a task cannot be found for the Task ID shown in a row,
NULL values are displayed (an outer join).

<p align="center">
  <img src="images/CreateCustomJoin/11.png" alt="Step 11" style="max-width: 600px; max-height: 600px;">
</p>
