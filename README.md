# Alma DV Time In Work Order Status
This is a repository with the Oracle DV code used to create an Alma Data Visualization showing the amount of time items in Alma spend in a specific work order status over time.
## Structure
Each folder represents a DV file (workbook, data flow, etc), and each file within that includes the code for a specific step.
Alma DV automatically translates fields to a hyperlinked text string that appears simply as "Item Location Code" but includes a full XSA path hidden behind the link (eg "XSA('HE_12345678920004107_4107_D_na04.alma.exlibrisgroup.com'.'__CLIENT_FLOW_DATASET__')."Columns"."Item Location Code""). I recommend typing in each field as needed and selecting it from the results that pop up in the editor.
![Screenshot showing what it looks like to select a field from the popup when you start typing in DV](/images/selecting-dv-fields.png)
## Process
### Create a Join Dataset to combine two Analytics subject areas
#### Create a Dataset
In Alma DV, choose Create>Dataset.
Choose Local Subject Area
#### Add the "Requests" subject area.
This is required to get the work order start date and work order completion date.
Select at least the following fields:
- Request Details>Request Id
- Request Date>Request Date
- Request Completion Date>Request Completion Date
#### Add the "Physical Items Historical Events" subject area.
This is required to get the status start date and status end date.
Select at least the following fields:
- Item Event Details>Request id
- Event Start Date>Event Start Date
- Event End Date>Event End Date
- Item Work Order Department Status Change>Work Order Status
- Physical Item Details (Current)>Physical Item Id
You might also need the following fields if you have multiple work order departments or types with the same status:
- Work Order Type
- Work Order Department
Optionally, select additional fields you want to use in your report, such as "Item Library Code" and "Item Location Code" if you want to be able to filter by location, or "Language Code" if you want to be able to filter by language.
#### Create an Inner join on Request Id
Draw a line between the subject areas in the Join Diagram and select Request Id as the column to match on.
Save the dataset.
### Create a Data Flow
Select the Join dataset you saved in the last step.
Select all columns from the dataset.
#### Filter the dataset to the Work Order Status
Add a Data Flow step to filter the dataset to the desired Work Order Status (and any other filters you may need).
#### Add Columns
Add a Data Flow step to Add Columns for the following, using the code in this repository as a base:
- Days in Cataloging
Optionally, add columns for things like location if some level of binning is needed.
#### Aggregate to remove extra events
Because we wanted to get a sense of how long it takes us to catalog resources, we wanted to exclude cases where an item was placed in the cataloging status quickly while trying to resolve issues while scanning items "done," etc. We decided that the best approach for our needs was to use only the longest or earliest cataloging status for each Physical Item Id.
Add a Data Flow step to Aggregate.
Add maximum days in cataloging:
- Aggregate: "Days in Cataloging"
- Function: "Maximum"
- New Column name: "Days in Cataloging Maximum"
Add minimum event start date
- Aggregate: "Event Start Date"
- Function: "Minimum"
- New column name: "Earliest Event Start Date"
Group by all the fields you want to use in your report and all the fields required to aggregate, including at minimum:
- Physical Item Id
#### Save/Export
Add a Data Flow step to Save Data.
Create a name for the dataset that is created by this Data Flow
Treat all the columns as "Attribute"
#### Save the Data Flow
### Running the Data Flow
The data flow needs to be run manually and creates a CSV dataset within Alma DV that you will use in your workbook. To keep this CSV file up to date you have to manually run the data flow or set up a schedule for the data flow to run automatically.
### Create a DV Workbook
Create a workbook and select the CSV export from the data flow.
#### Create calculation fields
Create new calculation fields in the Data tab for at least the following:
- Number of items
- Days in Cataloging bins
#### Create a Date Filter
To view just the stats from the past year, add a filter to the visualization for Earliest Event Start Date:
- Relative Time
- Type: Last
- Increment: 12
- Time Level: Months
- Relative To: Today
#### Create a Trellis Chart
- Type: Horizontal Stacked
- Trellis Columns: Earliest Event Start Date (Month)
- Values (X-Axis): Number of Items
- Category (Y-Axis): Days in Cataloging bins
#### Create a Stacked Bar Chart
- Type: 100% Stacked Bar
- Values (Y-Axis): Number of Items
- Category (X-Axis): Earliest Event Start Date (Month)
- Color: Days in Cataloging bins
#### Create an Interactive Filter
Drag a field you want to filter by into the canvas. Choose the visualization type "Dashboard filters" or "List."
If using "List," click the "Use as Filter" option to the left of the visualization title.
## Questions
For questions or more information, email <bright@gwu.edu>