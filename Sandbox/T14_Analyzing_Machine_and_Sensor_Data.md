## Tutorial 14: Analyzing Machine and Sensor Data

**This tutorial is for HDP version 2.3 of the [Hortonworks Sandbox](http://hortonworks.com/products/sandbox) - a single-node Hadoop cluster running in a virtual machine. [Download](http://hortonworks.com/products/sandbox) the Hortonworks Sandbox to run this and other tutorials in the series.**

### Introduction

This tutorial describes how to refine data from heating, ventilation,
and air conditioning (HVAC) systems using the Hortonworks Data Platform,
and how to analyze the refined sensor data to maintain optimal building
temperatures.

Demo: Here is the video of [Enable Predictive Analytics with
Hadoop](http://www.youtube.com/watch?v=Op_5MmG7hIw) as a demo of what
you'll be doing in this tutorial.

### Sensor Data

A sensor is a device that measures a physical quantity and transforms it
into a digital signal. Sensors are always on, capturing data at a low
cost, and powering the "Internet of Things."

### Potential Uses of Sensor Data

Sensors can be used to collect data from many sources, such as:

-   To monitor machines or infrastructure such as ventilation equipment,
    bridges, energy meters, or airplane engines. This data can be used
    for predictive analytics, to repair or replace these items before
    they break.
-   To monitor natural phenomena such as meteorological patterns,
    underground pressure during oil extraction, or patient vital
    statistics during recovery from a medical procedure.

In this tutorial, we will focus on sensor data from building operations.
Specifically, we will refine and analyze the data from Heating,
Ventilation, Air Conditioning (HVAC) systems in 20 large buildings
around the world.

### Prerequisites:

-   Hortonworks Sandbox (installed and running)
-   Hortonworks ODBC driver installed and configured (if using Microsoft Excel for reporting analysis)

Refer to

-   [Tutorial 7: Installing and Configuring the Hortonworks ODBC driver
    on Windows 7](http://hortonworks.com/hadoop-tutorial/how-to-install-and-configure-the-hortonworks-odbc-driver-on-windows-7/)
-   [Tutorial 11: Installing and Configuring the Hortonworks ODBC driver
    on Mac OS X](http://hortonworks.com/hadoop-tutorial/how-to-install-and-configure-the-hortonworks-odbc-driver-on-mac-os-x/)
-   Microsoft Excel 2013 Professional Plus (optional)

**Notes:**

-   In this tutorial, the Hortonworks Sandbox is installed on an Oracle
    VirtualBox virtual machine (VM) – your screens may be different.
-   If you plan on using the Microsoft Excel for the analysis and reporting section install the ODBC driver that matches the version of Excel you are
    using (32-bit or 64-bit).
-   If choosing to use Excelm you will use the Power View feature in Microsoft
    Excel 2013 to visualize the sensor data. Power View is currently
    only available in Microsoft Office Professional Plus and Microsoft
    Office 365 Professional Plus.
-   Note, other versions of Excel will work, but the visualizations will
    be limited to charts. You can connect to any other visualization
    tool you like.
-	If not using Excel, you will be able to use Apache Zeppelin to analyze and report on the data from this tutorial.

### Overview

To refine and analyze HVAC sensor data, we will:

-   Download and extract the sensor data files.
-   Load the sensor data into the Hortonworks Sandbox.
-   Run two Hive scripts to refine the sensor data.
-   Access the refined sensor data with Microsoft Excel or Apache Zeppelin.
-   Visualize the sensor data using Excel Power View or Apache Zeppelin.

### Step 1: Download and Extract the Sensor Data Files

-   You can download the sample sensor data contained in a compressed
    (.zip) folder here:

    [SensorFiles.zip](http://s3.amazonaws.com/hw-sandbox/tutorial14/SensorFiles.zip)

-   Save the SensorFiles.zip file to your computer, then extract the
    files. You should see a SensorFiles folder that contains the
    following files:

-   HVAC.csv – contains the targeted building temperatures, along with
    the actual (measured) building temperatures. The building
    temperature data was obtained using Apache Flume. Flume can be used
    as a log aggregator, collecting log data from many diverse sources
    and moving it to a centralized data store. In this case, Flume was
    used to capture the sensor log data, which we can now load into the
    Hadoop Distributed File System (HFDS).  For more details on Flume,
    refer to Tutorial 13: Refining and Visualizing Sentiment Data

-   building.csv – contains the "building" database table. Apache Sqoop
    can be used to transfer this type of data from a structured database
    into HFDS.

### Step 2: Load the Sensor Data into the Hortonworks Sandbox

-   Navigate to the ambari login by going to the web address [`http://localhost:8080`](http://localhost:8080) 
-   Login with the username `admin` and password `admin`.

Once logged in to the sandbox, navigate to the icon at the top right of the header and view the dropdown menu. 

-	Select `HDFS Files`

   ![](./images/tutorial-14/01_hdfs_files_view.png?raw=true)

-   Note the view you are greeted with once you open the `HDFS Files` Ambari View.
-	You can view the path that you're currently in
-	You can create new directories and upload files using the button on the top.
-	You can also download whole directories as `.zip` files, move directories, and delete them as well. These options are highlighted in blue for each folder or file in a directory.

    ![](./images/tutorial-14/02_hdfs_files_ex_1.png?raw=true)


-   Navigate to the `/tmp` directory.
-	Create the directory `data` by using **New Directory** at the top of the page.
-	Once inside the directory, click **Upload** then **Browse**. Navigate to the `HVAC.csv` file that is part of the .zip file that was downloaded earlier.
-	Click **Upload** to upload the CSV file to HDFS.
-	Follow the same procedure to upload the `building.csv` file.

    ![](./images/tutorial-14/03_hdfs_files_upload.png?raw=true)

-   After uploading both files, the page should look similar to the following image:

    ![](./images/tutorial-14/04_hdfs_files_completed.png?raw=true)

-	Now user the menu on the header again to access the dropdown menu containing the link to the [`Ambari Hive View`](http://localhost:8080/#/main/views/HIVE/1.0.0/Hive)

    ![](./images/tutorial-14/05_hive_view_dropdown.png?raw=true)

-   Here we're going to define two tables that will contain data from our `HVAC.csv` and `building.csv` files.
-	Copy and paste the following query into the worksheet and click **Execute** to create the table **hvac_stage**.

```
create table hvac_stage (
recordDate string,
Time string,
TargetTemp int,
ActualTemp int,
System int,
SystemAge int,
BuildingID int) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

   ![](./images/tutorial-14/06_hive_query_stage_1.png?raw=true)

-   After the previous query has executed, **Execute** this next next query to create the table **buildings_stage**.

```
create table buildings_stage
(BuildingID int,
 BuildingMgr string,
 BuildingAge string,
 HVACproduct string,
 Country string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

   ![](./images/tutorial-14/07_hive_query_stage_2.png?raw=true)

-   After both queries have succeeded you should see `hvac_stage` and `buildings_stage` in the **Database Explorer** on the left hand side of the screen.
-	Now use the following query to load our tables with data from our data files that we previously uploaded to HDFS.

```
LOAD DATA INPATH '/tmp/data/HVAC.csv' OVERWRITE INTO TABLE HVAC_stage;
```

   ![](./images/tutorial-14/08_hive_query_load_1.png?raw=true)

-   Repeat the previous step with the following query to load the table `buildings_stage` with data.

```
LOAD DATA INPATH '/tmp/data/building.csv' OVERWRITE INTO TABLE buildings_stage;
```

   ![](./images/tutorial-14/09_hive_query_load_2.png?raw=true)

-   Now refresh the **Database Explorer**. You should see all of the tables appear on the left hand side after clicking `default`.
-	If you click the small square icon to the right hand side of any table it will immediately create a query that gives you a small sample of data from the table you selected.
- Do this for the table `hvac_stage` and `buildings_stage`. Make sure both tables have all columns populated and that it is possible to execute and see the results of both queries.

Lastly, we want to remove the header rows that are present when we query the tables in Hive. To do this we simply need to write to commands. One for each table.

```
ALTER TABLE buildings_stage SET TBLPROPERTIES ("skip.header.line.count"="1");
```

```
ALTER TABLE hvac_stage SET TBLPROPERTIES ("skip.header.line.count"="1");
```

    ![](./images/tutorial-14/10_hive_example_data.png?raw=true)

-	Now that we have both tables loaded in, we want to get better performance in Hive, so we're going to create new tables that utilize the highly efficient [**ORC** file format](http://hortonworks.com/blog/apache-orc-launches-as-a-top-level-project/). This will allow for faster queries when our datasets are much much larger.
-	Execute the following query to create a new table `hvac` that is stored as an ORC file.

```
CREATE TABLE hvac STORED AS ORC AS SELECT * FROM HVAC_stage;
```

   ![](./images/tutorial-14/11_hive_orc_1.png?raw=true)

-	Repeat the previous step, except this time we will make a table for `buildings`.

```
CREATE TABLE buildings STORED AS ORC AS SELECT * FROM buildings_stage;
```
   
   ![](./images/tutorial-14/12_hive_orc_2.png?raw=true)

### Step 3: Run Two Hive Scripts to Refine the Sensor Data

We will now use two Hive scripts to refine the sensor data. We hope to
accomplish three goals with this data:

-   Reduce heating and cooling expenses.
-   Keep indoor temperatures in a comfortable range between 65-70
    degrees.
-   Identify which HVAC products are reliable, and replace unreliable
    equipment with those models.

-   First, we will identify whether the actual temperature was more than
    five degrees different from the target temperature.

-	Create a new worksheet in the Hive view and paste the following Hive query into your window.

```
CREATE TABLE hvac_temperatures as 
select *, targettemp - actualtemp as temp_diff, 
IF((targettemp - actualtemp) > 5, 'COLD', 
IF((targettemp - actualtemp) < -5, 'HOT', 'NORMAL')) 
AS temprange, 
IF((targettemp - actualtemp) > 5, '1', 
IF((targettemp - actualtemp) < -5, '1', 0)) 
AS extremetemp from hvac;
```

-	This query creates a new table `hvac_temperatures` and copies data from the `hvac` table

-   After you paste the query use **Execute** to create the new table.
-	Then

    ![](./images/tutorial-14/13_hive_hvac_temperatures.png?raw=true)

-   On the Query Results page, use the slider to scroll to the right. You
    will notice that two new attributes appear in the `hvac_temperatures` table.

    The data in the **temprange** column indicates whether the actual
    temperature was:

    -   **NORMAL** **–** within 5 degrees of the target temperature.
    -   **COLD** **–** more than five degrees colder than the target
        temperature.
    -   **HOT** **–** more than 5 degrees warmer than the target
        temperature.

    If the temperature is outside of the normal range, `extremetemp` is
    assigned a value of 1; otherwise its value is 0.

    ![](./images/tutorial-14/14_hive_hvac_temps_example.png?raw=true)

-   Next we will combine the **hvac** and **hvac_temperatures** data sets.
 
Create a new worksheet in the hive view and use the following query to create a new table `hvac_building` that contains data from the `hvac_temperatures` table and the `buildings` table.
 
```
create table if not exists hvac_building 
as select h.*, b.country, b.hvacproduct, b.buildingage, b.buildingmgr 
from buildings b join hvac_temperatures h on b.buildingid = h.buildingid;
```

-	Use **Execute** to run the query that will produce the table with the intended data.

    ![](./images/tutorial-14/15_hive_hvac_building_query.png?raw=true)

-   After you've successfully executed the query, use the database explorer to load a sample of the data from the new `hvac_building` table.

    ![](./images/tutorial-14/16_hive_examine_hvac_building.png?raw=true)

Now that we've constructued the data into a useful format, we can use different reporting tools to analyze the results.

* * * * *

# Data Reporting 

In this tutorial you can choose to report with

-	[Microsoft Excel](#step-4a-access-the-refined-sensor-data-with-microsoft-excel)
-	[Apache Zeppelin](#step-4b-access-the-refined-sensor-data-with-apache-zeppelin)

* * * * *

## Step 4a: Access the Refined Sensor Data with Microsoft Excel
--------------------------------------------------------------

In this section, we will use Microsoft Excel Professional Plus 2013 to
access the refined sentiment data.

-   In Windows, open a new Excel workbook, then select **Data > From
    Other Sources > From Microsoft Query**.

    ![](./images/tutorial-14/18_open_query.jpg?raw=true)

-   On the Choose Data Source pop-up, select the Hortonworks ODBC data
    source you installed previously, then click **OK**.

    The Hortonworks ODBC driver enables you to access Hortonworks data
    with Excel and other Business Intelligence (BI) applications that
    support ODBC.

    ![](./images/tutorial-14/19_choose_data_source.jpg?raw=true)

-   After the connection to the Sandbox is established, the Query Wizard
    appears. Select the "hvac_building" table in the Available tables
    and columns box, then click the right arrow button to add the entire
    "hvac_building" table to the query. Click **Next** to continue.

    ![](./images/tutorial-14/20_query_wizard1_choose_columns.jpg?raw=true)

-   On the Filter Data screen, click **Next** to continue without
    filtering the data.

    ![](./images/tutorial-14/21_query_wizard2_filter_data.jpg?raw=true)

-   On the Sort Order screen, click **Next** to continue without setting
    a sort order.

    ![](./images/tutorial-14/22_query_wizard3_sort_order.jpg?raw=true)

-   Click **Finish** on the Query Wizard Finish screen to retrieve the
    query data from the Sandbox and import it into Excel.

    ![](./images/tutorial-14/23_query_wizard4_finish.jpg?raw=true)

-   On the Import Data dialog box, click **OK** to accept the default
    settings and import the data as a table.

    ![](./images/tutorial-14/24_import_data.jpg?raw=true)

-   The imported query data appears in the Excel workbook.

    ![](./images/tutorial-14/25_data_imported.jpg?raw=true)

Now that we have successfully imported the refined sensor data into
Microsoft Excel, we can use the Excel Power View feature to analyze and
visualize the data.

### Step 5: Visualize the Sensor Data Using Excel Power View

We will begin the data visualization by mapping the buildings that are
most frequently outside of the optimal temperature range.

-   In the Excel worksheet with the imported "hvac_building" table,
    select **Insert > Power View** to open a new Power View report.

    ![](./images/tutorial-14/26_open_powerview_hvac_building.jpg?raw=true)

-   The Power View Fields area appears on the right side of the window,
    with the data table displayed on the left. Drag the handles or click
    the Pop Out icon to maximize the size of the data table.

    ![](./images/tutorial-14/27_powerview_hvac_building.jpg?raw=true)

-   In the Power View Fields area, select the checkboxes next to the
    **country** and **extremetemp** fields, and clear all of the other
    checkboxes. You may need to scroll down to see all of the check
    boxes.

    ![](./images/tutorial-14/28_select_country_extremetemp.jpg?raw=true)

-   In the FIELDS box, click the down-arrow at the right of the
    **extremetemp** field, then select **Count (Not Blank)**.

    ![](./images/tutorial-14/29_extremetemp_count_not_blank.jpg?raw=true)

-   Click **Map** on the Design tab in the top menu.

    ![](./images/tutorial-14/30_open_map.jpg?raw=true)

-   The map view displays a global view of the data. We can see that the
    office in Finland had 814 sensor readings where the temperature was
    more than five degrees higher or lower than the target temperature.
    In contrast, the German office is doing a better job maintaining
    ideal office temperatures, with only 363 readings outside of the
    ideal range.

    ![](./images/tutorial-14/31_extremetemp_map.jpg?raw=true)

-   Hot offices can lead to employee complaints and reduced
    productivity. Let's see which offices run hot.

    In the Power View Fields area, clear the **extremetemp** checkbox
    and select the **temprange** checkbox. Click the down-arrow at the
    right of the **temprange** field, then select **Add as Size**.

    ![](./images/tutorial-14/32_add_temprange_as_size.jpg?raw=true)

-   Drag **temprange** from the Power View Fields area to the Filters
    box, then select the **HOT** checkbox. We can see that the buildings
    in Finland and France run hot most often.

    ![](./images/tutorial-14/33_filter_by_temprange_hot.jpg?raw=true)

-   Cold offices cause elevated energy expenditures and employee
    discomfort.

    In the Filters box, clear the **HOT** checkbox and select the
    **COLD** checkbox. We can see that the buildings in Finland and
    Indonesia run cold most often.

    ![](./images/tutorial-14/34_filter_by_temprange_cold.jpg?raw=true)

-   Our data set includes information about the performance of five
    brands of HVAC equipment, distributed across many types of buildings
    in a wide variety of climates. We can use this data to assess the
    relative reliability of the different HVAC models.

-   Open a new Excel worksheet, then select **Data > From Other Sources
    > From Microsoft Query** to access the hvac_building table. Follow
    the same procedure as before to import the data, but this time only
    select the "hvacproduct" and "extremetemp" columns.

    ![](./images/tutorial-14/35_import_hvacproduct_extremetemp.jpg?raw=true)

-   In the Excel worksheet with the imported "hvacproduct" and
    "extremetemp" columns, select **Insert > Power View** to open a new
    Power View report.

    ![](./images/tutorial-14/36_open_powerview_hvacproduct.jpg?raw=true)

-   Click the Pop Out icon to maximize the size of the data table. In
    the FIELDS box, click the down-arrow at the right of the extremetemp
    field, then select Count (Not Blank).

    ![](./images/tutorial-14/37_extremetemp_count_not_blank.jpg?raw=true)

-   Select **Column Chart > Stacked Column**in the top menu.

    ![](./images/tutorial-14/38_open_stacked_column.jpg?raw=true)

-   Click the down-arrow next to **sort by hvacproduct** in the upper
    left corner of the chart area, then select **Count of extremetemp**.

    ![](./images/tutorial-14/39_sort_by_extremetemp.jpg?raw=true)

-   We can see that the GG1919 model seems to regulate temperature most
    reliably, whereas the FN39TG failed to maintain the appropriate
    temperature range 9% more frequently than the GG1919.

    ![](./images/tutorial-14/40_chart_sorted_by_extremetemp.jpg?raw=true)

We've shown how the Hortonworks Data Platform (HDP) can store and
analyze sensor data. With real-time access to massive amounts of
temperature and other types of data on HDP, your facilities department
can initiate data-driven strategies to reduce energy expenditures and
improve employee comfort.

---------------------------------------------------------------

## Step 4b: Access the Refined Sensor Data with Apache Zeppelin
---------------------------------------------------------------

Apache Zeppelin makes data reporting easy on Hadoop. It has direct connections to Apache Spark and Hive in your cluster and allows you to create visualizations and analyze your data on the fly.

To start you're going to need to open up the [Apache Zeppelin view](http://localhost:8080/#/main/views/ZEPPELIN/1.0.0/INSTANCE_1) in Ambari.

Start by navigating back to the [Ambari Dashboard](http://localhost:8080) at `http://localhost:8080`

-	Use the dropdown menu to open the Zeppelin View.

   ![](./images/tutorial-14/41_ambari_zeppelin_view.png?raw=true)
   
-	From here we're going to need to create a new Zeppelin Notebook. 
-	Notebooks in Zeppelin is how we differentiate reports from one another.
-	Hove over **Notebook**. Use the dropdown menu and **Create a new note**.
   
   ![](./images/tutorial-14/42_create_zeppelin_note.png?raw=true)
   
-	Name the note **HVAC Analysis Report** and then **Create Note**.
   
   ![](./images/tutorial-14/43_zeppelin_naming_note.png?raw=true)
   
   
-	Head back to the Zeppelin homepage.
-	Use the **Notebook** dropdown menu to open the new notebook **HVAC Analysis Report**.

   ![](./images/tutorial-14/43_1_opening_note.png?raw=true)
   
-	Zeppelin integrates with Hadoop by using things called *interpreters*.
-	In this tutorial we'll be working with the Hive interpreter to run Hive queries in Zeppelin, then visualize the results from our Hive queries directly in Zeppelin.
-	To specify the Hive interpreter for this note, we need to put `%hive` at the top of the note. Everything afterwards will be interpreted as a Hive query.
   
   ![](./images/tutorial-14/44_blank_zeppelin_notebook.png?raw=true)
   
-	Type the following query into the note, then run it by clicking the **Run** arrow or by using the shortcut **Shift+Enter**.

```
%hive

select country, extremetemp, temprange from hvac_building
```
   
   ![](./images/tutorial-14/45_zeppelin_query.png?raw=true)
   
-	After running the previous query we can view a chart of the data by clicking the chart button located just under the query.

   ![](./images/tutorial-14/46_table_to_chart.png?raw=true)
   
-	Click **settings** to open up more advanced settings for creating the chart. Here you can experiment with different values and columns to create different types of charts.
   
   ![](./images/tutorial-14/47_changing_chart_settings.png?raw=true)
   
-	Arrange the fields according to the following image.
-	Drag the field `temprange` into the **groups** box.
-	Click **SUM** on `extremetemp` and change it to **COUNT**.
-	Make sure that `country` is the only field under **Keys**.
   
   ![](./images/tutorial-14/48_chart_setup.png?raw=true)
   
-	Awesome! You've just created your first chart using Apache Zeppelin.
-	From this chart we can see which countries have the most extreme temperature and how many **NORMAL** events there are compared to **HOT** and **COLD**.
-	From this data it could be possible to figure out which buildings might need HVAC upgrades, and which do not.
   
   ![](./images/tutorial-14/49_chart_finished.png?raw=true)
   
-	Let's try creating one more note to visualize which types of HVAC systems result in the least amount of `extremetemp` readings.
-	Paste the following query into the blank Zeppelin note following the chart we made previously.

```
%hive

select hvacproduct, extremetemp from hvac_building
```

-	Now use **Shift+Enter** to run the note.
   
   ![](./images/tutorial-14/50_second_query.png?raw=true)
   
-	Arrange the fields according to the following image so we can recreate the chart below.
-	Make sure that `hvacproduct` is in the **Keys** box.
-	Make sure that `extremetemp` is in the **Values** box and that it is set to **COUNT**.
   
   ![](./images/tutorial-14/51_chart_two.png?raw=true)
  
-	Now we can see which HVAC units result in the most `extremetemp` readings. Thus we can make a more informed decision when purchasing new HVAC systems.


Apache Zeppelin gives you the power to connect right to your Hadoop cluster to quickly obtain results from the data inside of Hadoop without having to export data to any other sources.

It's also important to note that Zeppelin contains many, many interpreters that can be utilized to obtain data in a variety of ways.

One of the default interpreters included with Zeppelin is for Apache Spark. With the popularity of Apache Spark rising, you can simply write Spark scripts to execute directly on Apache Zeppelin to obtain results from your data in a matter of seconds.


**Feedback**

We are eager to hear your feedback on this tutorial. Please let us know
what you think. [Click here to take
survey](https://www.surveymonkey.com/s/Sandbox_Machine_Sensor)
