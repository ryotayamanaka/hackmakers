# Create and Enable a Database User in SQL Developer Web

## Introduction

This lab walks you through the steps to get started with SQL Developer Web. You will learn how to create a user in SQL Developer Web and provide that user the access to SQL Developer Web.

Estimated time: 3 minutes

### Objectives

- Learn how to create a database user in SQL Developer Web.

### Prerequisites

* Oracle cloud account
* Provisioned Autonomous Database Shared Free Tier Instance

## **STEP 1:** Create a database user

1. Login as the Admin user in SQL Developer Web of the newly created ADB Free Tier instance.

  Go to your Cloud Console, click **Autonomous Transaction Processing**. Select the ADB instance **ATP Graph** you created in Lab 2.

  ![](images/select_ATP.png " ")

  In Autonomous Database Details page, click **Service Console**. Make sure your brower allow pop-up windows.

  ![](images/ADB_console.png " ")

  Choose Development from the list on the left, then click the **SQL Developer Web**.

  ![ADB Console Development Page](images/ADB_ConsoleDevTab.png " ")

  Enter `ADMIN` as Username, and the enter the password you set up at Lab 2 Step 2, Section 7.

  ![](images/login.png " ")
  Login as the ADMIN user. 

  ![Login as Admin](images/ADB_SQLDevWebHome.png)

2. Now create the `hackmakers` user. Enter the following commands into the SQL Worksheet and run it while connected as the Admin user.

Note: Replace **<specify_a_password>** with a valid password string after copying and pasting the text below but **before executing** it in SQL Developer Web.

    CREATE USER hackmakers
    IDENTIFIED BY <specify_a_password> 
    DEFAULT TABLESPACE data 
    TEMPORARY TABLESPACE temp 
    QUOTA UNLIMITED ON data;

    CREATE ROLE graph_developer;
    CREATE ROLE graph_administrator;

    GRANT connect, resource, graph_developer TO hackmakers;

  ![](images/06.png " ")

## **STEP 2:** Enable SQL Developer Web for the new user

Now provide SQL Developer Web access for this user. See the [documentation](https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/user/sql-developer-web.html#GUID-4B404CE3-C832-4089-B37A-ADE1036C7EEA) for details.

First clear the previous text in the SQL Worksheet.

Copy and paste the following text into the SQL Worksheet and run it.

    BEGIN
      ORDS_ADMIN.ENABLE_SCHEMA(
        p_enabled => TRUE,
        p_schema => 'hackmakers',
        p_url_mapping_type => 'BASE_PATH',
        p_url_mapping_pattern => 'hackmakers',
        p_auto_rest_auth => TRUE
      );
      COMMIT;
    END;
    /

The URL for SQL Developer Web for this user will have `hackmakers` in place of `admin` in it.

Save the URL for the next step.

For details, see the ["Provide SQL Developer Web Access to Database Users"](https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/user/sql-developer-web.html#GUID-4B404CE3-C832-4089-B37A-ADE1036C7EEA) section in the documentation.

You may now proceed to [the next lab](../04_create-and-populate-tables/create-and-populate-tables.md).
