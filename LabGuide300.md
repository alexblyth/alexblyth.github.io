![](./images/300/TITLE300.png)

Updated: Feb 14 2019

# ADWC Lab 4: Data Loading

## Introduction

In this lab, you will be uploading files to the Oracle Cloud Infrastructure (OCI) Object Storage, creating sample tables, loading data into them from files on the OCI Object Storage, and troubleshooting data loads with errors.

You can load data into Autonomous DW Cloud using Oracle Database tools, and Oracle and 3rd party data integration tools. You can load data:

+ from files local to your client computer, or
+ from files stored in a cloud-based object store

For the fastest data loading experience Oracle recommends uploading the source files to a cloud-based object store, such as Oracle Cloud Infrastructure Object Storage, before loading the data into your Autonomous DW Cloud.

To load data from files in the cloud into your Autonomous DW Cloud database, use the new PL/SQL DBMS_CLOUD package. The DBMS_CLOUD package supports loading data files from the following Cloud sources: Oracle Cloud Infrastructure Object Storage, Oracle Cloud Infrastructure Object Storage Classic, and Amazon AWS S3.

This lab shows how to load data from Oracle Cloud Infrastructure Object Storage using two of the procedures in the DBMS_CLOUD package:

+ create_credential: Stores the object store credentials in your Autonomous DW Cloud schema.
    + You will use this procedure to create object store credentials in your Autonomous DW Cloud admin schema.
+ copy_data: Loads the specified source file to a table. The table must already exist in Autonomous DW Cloud.
    + You will use this procedure to load tables in your admin schema with data from data files staged in the Oracle Cloud Infrastructure Object Storage cloud service.
        
For more information about loading data, see the documentation <a href="https://www.oracle.com/pls/topic/lookup?ctx=en/cloud/paas/autonomous-data-warehouse-cloud&id=CSWHU-GUID-07900054-CB65-490A-AF3C-39EF45505802">Loading Data from Files in the Cloud</a>.

If you are interesting is using impdp to copy data from an existing Oracle database, there is some good example code at the ORACLE-BASE website: <a href="https://oracle-base.com/articles/vm/oracle-cloud-autonomous-data-warehouse-adw-import-data-from-object-store">https ://oracle-base.com/articles/vm/oracle-cloud-autonomous-data-warehouse-adw-import-data-from-object-store</a>.


To **log issues**, click [here](https://github.com/alexblyth/alexblyth.github.io/issues/new) to go to the github oracle repository issue submission form.

## Objectives

-   Learn how to use the SQL Developer Data Import Wizard

-   Learn how to upload files to the OCI Object Storage

-   Learn how to define object store credentials for ADWC

-   Learn how to create tables in your database

-   Learn how to load data from the Object Store

-   Learn how to troubleshoot data loads

## Required Artifacts

-   The following lab requires an Oracle Public Cloud account. You may your own cloud account, a cloud account that you obtained through a trial, or a training account whose details were given to you by an Oracle instructor.

-   Oracle SQL Developer (see Lab100 for more specifics on the version of SQL Developer and how to install and configure it). 

# Download Sample Data
## Steps
### STEP 1: Download the Sample Data Files to Your Local Computer

-   For this lab, you will need a handful of data files.  Click <a href="./files/datafiles.zip" target="_blank">here</a> to download a zipfile of the 5 sample data files for you to upload to the object store. Unzip it to a directory on your local computer. You will see:-

    -   Sales data: **sales.csv.gz**
    -   Customer data: **customers.csv**
    -   Channel data (with intentional errors): **channels\_error.csv**
    -   Channel data: **channels.csv**
    -   Products data: **products.txt**

# Create the tables
## Steps
### STEP 2: Create Target Tables for Data Loading

-   Connected as your user in SQL Developer, copy and paste <a href="./scripts/300/create_tables.txt" target="_blank">this code snippet</a> to SQL Developer worksheet. Take a moment to examine the script. Then click the **Run Script** button to run it.

    ![](./images/300/Picture300-2.png)

-   If you want, you can compare your output to <a href="./scripts/300/create_tables_output.txt" target="_blank">this expected output.</a>  It is expected that you may get ORA-00942 errors during the DROP TABLE commands, but you should not see any other errors. 

Note that you do not need to specify anything other than the list of columns when creating tables in the SQL scripts. You can use primary keys and foreign keys if you want, but they are not required.

# Load data from the local file system
## Steps
### STEP 3: Loading Data Using the Data Import Wizard in SQL Developer

-   Expand ‘**Tables**’ in your user schema object tree. If you don't see any tables, click on the refresh icon (two curved arrows) to refresh the table list.  You will see all the tables you have created previously. Select table **CHANNELS**. Clicking the right mouse button opens the context-sensitive menu in SQL Developer; select ‘**Import Data**’:

    ![](./images/300/Picture300-13b.png)

    ![](./images/300/Picture300-14b.png)

-   The Data Import Wizard is started. Enter the following information:

    -   Select **Local File** as source for the data load

    -   Click the browse button and navigate to the channels.csv file (you extracted this file from the zip file you downloaded at the start of this lab).
    
   

After entering this information, you can preview the data and select the appropriate file formats. You will see that the data preview is
interactive and changes according to your selection.

When you are satisfied with the file content view, click **NEXT**.

![](./images/300/snap0014654.jpg)

-   On Step 2 of the Import Wizard, you control the import method and parameters. Leave the Import Method as Insert. Click **NEXT**.

![](./images/300/snap0014655.jpg)

-   The Choose Columns screen lets you select the columns you want to import.  Leave the defaults and click **NEXT**.

-   The column definition screen shows you whether the sample data violates any of the existing column definitions of table CHANNELS (for a load into a new table you would select the column names and data types for the new table). Click **NEXT**.


![](./images/300/snap0014656.jpg)

-   The final screen reflects all your choices made in the Wizard. Click **FINISH** to load the data into table CHANNELS.

![](./images/300/snap0014657.jpg)


# Load data from the Object Store using the Data Import Wizard
## Steps
### STEP 4: Create a Database Credential for Your User

-   In order to access data in the Object Store you have to enable your database user to authenticate itself with the Object Store using your OCI object store account and Auth token. We have created this for you in a commonly used account.

-   Connected as your user in SQL Developer, copy and paste <a href="./scripts/300/create_credential.txt" target="_blank">this code snippet</a> to SQL Developer worksheet.

    Specify the credentials for your Oracle Cloud Infrastructure Object Storage service: The username will be the **OCI username** (which is not the same as your database username) and the OCI object store **Auth Token** listed below.  In this example, the crediential object named **OBJ\_STORE\_CRED** is created. You reference this credential name in the following steps.

    Use the following username and "password" (also know as authorization token)

    username - **cloud.admin**

    password - **CRx{8JcdL6Dw6_wuG0zQ**

    ![](./images/300/snap0015312.jpg)

<!-- -->



You will be using the following URLs to load data into the ADW database


https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/gse00014543/ADWHOL/customers.csv
https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/gse00014543/ADWHOL/channels_error.csv
https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/gse00014543/ADWHOL/products.txt
https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/gse00014543/ADWHOL/sales.csv.gz


-   Run the script. 

    ![](./images/300/Picture300-12.png)

-   Now you are ready to load data from the Object Store.


### STEP 5 : Loading Data Using the Data Import Wizard in SQL Developer

-   Expand ‘**Tables**’ in your user schema object tree. If you don't see any tables, click on the refresh icon (two curved arrows) to refresh the table list.  You will see all the tables you have created previously. Select table **CUSTOMERS**. Clicking the right mouse button opens the context-sensitive menu in SQL Developer; select ‘**Import Data**’:

    ![](./images/300/snap0014659.jpg)

    ![](./images/300/snap0014660.jpg)

-   The Data Import Wizard is started. Enter the following information:

    -   Select **Oracle Cloud Storage** as source for the data load

    -   Enter the URL of **customers.csv** as the file to load. Remember to use the URLs listed above. The customer URL is:

        https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/gse00014543/ADWHOL/customers.csv

    -   Select the Credential you previously created for authentication with the Object Store, **OBJ\_STORE\_CRED**

    -   Click the **Preview** button

After clicking preview, you will likely get a warning about the format which we will fix next.

![](./images/300/snap0014661.jpg)

  - Now we will fix the file format settings.

     - Click OK to close the Invalid format window if you have not already.

     - Deselect the Header checkbox as this file does not have a header.  When you deselect the checkbox, you will get another Field Format message.  Click OK to acknowledge the message window.

     - Change the delimiter to the vertical bar |

The data should now look much better.

![](./images/300/snap0014662.jpg)


When you are satisfied with the data preview, click **NEXT**.

  - On the Import Method page, you can click on Load Options to see some of the available options.  For this exercise, leave the options at their defaults.  Click **NEXT** to advance to the next page of the wizard.

![](./images/300/snap0014663.jpg)

  -   On the Column Definition page, you can control how the fields of the file map to columns in the table.  You can also adjust certain properties.

![](./images/300/snap0014664.jpg)

  - Click on the **COLUMN21** with the warning icon beside it.  This is informing us of a data problem.

  - In this case, the date format does not match our data.  Change the format to be YYYY-MM-DD-HH24-MI-SS  (you can type this in the Format field)

  - You also need to change the date format for **COLUMN22** to the same YYYY-MM-DD-HH24-MI-SS format.

  - Click **NEXT**.

![](./images/300/snap0014665.jpg)

-   The last screen before the final data load enables you to test a larger row count than the sample data of the beginning of the wizard to see whether the previously made decisions are satisfying for your data load. Note that we are not loading any data when iterating back and forth between this screen and previous ones. Select **TEST RESULTS** and look at the log, the data you would load, any mistakes and how the external table definition looks like based on your inputs.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When done with your investigation, click **NEXT**.

![](./images/300/snap0014666.jpg)

![](./images/300/snap0014667.jpg)



-   The final screen reflects all your choices made in the Wizard. Click **FINISH** to load the data into table CUSTOMERS.

# Load data from the Object Store using DBMS_CLOUD
## Steps
### STEP 6 : Loading Data Using the PL/SQL Package, DBMS_CLOUD

As an alternative to the wizard-guided data load, you can use the PL/SQL package **DBMS_CLOUD** directly. This is the preferred choice for any load automation.

-   Connected as your user in SQL Developer, copy and paste <a href="./scripts/300/load_data.txt" target="_blank">this code snippet</a> to SQL Developer worksheet. We use the **copy\_data** procedure of the **DBMS\_CLOUD** package to copy the data (**sales.csv.gz**) staged in your object store.

    -   At the top of the script, specify the Object Store base URL in the definition of the **base\_URL** variable. You have constructed and saved the URL in the step "Construct the URLs of the Files on Your OCI Object Storage". 
    ![](./images/300/snap0014550.jpg)

    -   For the **credential_name** parameter in the **copy\_data** procedure, it is the name of the credential you defined in the step "Create a Database Credential for Your User".  You probably don't need to change this.

    -  For the **format** parameter, it is a list of DBMS_CLOUD options (which are documented <a href="https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/user/dbmscloud-reference.html">here</a>.)  One of the more interesting options in this example is compression option which is set to gzip.  Loading compressed files can often be faster than loading uncompressed files due to the savings in network transmission time.



-   Run the script.

-   You have successfully loaded the sample tables. Now, you can run any sample query in the <a href="https://docs.oracle.com/database/122/DWHSG/part-relational-analytics.htm#DWHSG8493" target="_blank">relational analytics</a> section of the Oracle documentation. For example, to analyze the cumulative amount sold for specific customer IDs in quarter 2000, you could run the query in <a href="./scripts/300/query_tables.txt" target="_blank">this code snippet</a>. ( <a href="https://docs.oracle.com/database/122/DWHSG/sql-analysis-reporting-data-warehouses.htm#GUID-33B4DE75-D7F8-4AE1-9F2E-C2846F72CC1E__GUID-4CB0EE02-AA9F-42D9-8F1B-2CD477496CD9" target="_blank">link</a> to documentation).

    ![](./images/300/Picture300-20.png)

### STEP 7 : Troubleshooting DBMS_CLOUD data loads

-   Connected as your user in SQL Developer, run the following query to look at past and current data loads.
```
select * from user_load_operations;
```
Notice how this table lists the past and current load operations in your schema.  Any data copy and data validation operation will be automatically
tracked by Oracle.

-   For an example of how to troubleshoot a data load, we will attempt to load a data file with the wrong format (channels_error.csv).  Specifically, the default separator is the | character, but the channels_error.csv file uses a semicolon instead.  To attempt to load bad data, copy and paste <a href="./scripts/300/load_data_with_errors.txt" target="_blank">this code snippet</a> to a SQL Developer worksheet and run the script as your user in SQL Developer. Specify the URL that points to the **channels\_error.csv** file. You have constructed and saved the URL in the step "Construct the URLs of the Files on Your OCI Object Storage". Note that you are going to load the data with errors this time.

![](./images/300/snap0014669.jpg)

-   Run the following queries to see the load that errored out.
```
select * from user_load_operations where status='FAILED';
```
![](./images/300/Picture300-22.png)

A load or external table validation that errors out is indicated by status=FAILED in this table. Get the names of the log and bad files for the failing load operation from the column **logfile\_table** and **badfile\_table**. The logfile_table column shows the name of the table you can query to look at the log of a load operation. The column badfile_table shows the name of the table you can query to look at the rows that got errors during loading.

-   Query the log and bad tables to see detailed information about an individual load. In this example, the names are copy$15_log and copy$15_bad respectively.

    ![](./images/300/Picture300-23.png)

    ![](./images/300/Picture300-24.png)

-   To learn more about how to specify file formats, delimiters, and more, you can review DBMS_CLOUD Package Format Options: https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/user/dbmscloud-reference.html

-   Keep your SQL Deveoper open and move to the next lab - Querying External Data.

# Setup the OCI Object Store - (Optional Lab)
This lab shows __one__ way of loading files into the Oracle Object Store which can then be loaded in to the Oracle Autonomous Data Warehouse. Other options to load data include Oracle Data Intgration Cloud Platofrm, Oracle GoldenGate & Data Integrator and 3rd Party ETL tools. Oracle Autonomous Data Warehouse is also capable of querying these files directly from the Object Storage, as you have seen in earlier labs.
## Steps    
### STEP 8 : Navigate to the Storage Tab, then Object Storage 
To learn more about the OCI Object Storage, check out this <a href="https://docs.us-phoenix-1.oraclecloud.com/Content/GSG/Tasks/addingbuckets.htm" target="_blank">documentation</a> .

-   In the OCI Compute Console, click on the **Storage** tab, then click on **Object Storage** on the left-hand menu:
    ![](images/300/snap0014297.jpg)

-   Choose the root compartment in the **COMPARTMENT** drop-down if it is not already choosen. The name of your root compartment might be different.
    ![](images/300/snap0014298.jpg)

### STEP 9 : Create a Bucket for the Object Storage
In OCI Object Storage, a bucket is the terminology for a container of multiple files. 

-   Click the **Create Bucket** button:
    ![](images/300/snap0014299.jpg)

-   **Name your bucket** and click **Create Bucket** button. 
    ![](images/300/snap0014300.jpg)

### STEP 10 : Upload Files to Your OCI Object Store Bucket

-   Click on your **bucket name** to open it:
    ![](images/300/snap0014301.jpg)

-   Click on the **Upload Object** button:
    ![](images/300/snap0014302.jpg)

-   Using the browse button or drag-and-drop select the **sales.csv.gz** file you downloaded earlier and click Upload Object:
    ![](images/300/snap0014303.jpg)

-   Repeat this for the **customers.csv**, **products.txt**, and **channels_error.csv** files.

-   The end result should look like this with all 4 files listed under Objects:
    ![](images/300/snap0014304.jpg)

Congratulations - you have now uploaded files into Oracle Object Storage! These could then be used to load data into the ADW as you did in Step 5.