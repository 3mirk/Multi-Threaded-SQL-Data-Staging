# Multi-Threaded-SQL-Data-Staging

Typically, SQL connections are handled via built-in data connectors (or ODBC's) in Data Visualization tools like Tableau or PowerBI. 
However, connecting simultaneously to a larger-and-larger number of such connections has several downsides. 
These drawbacks can include less-than-ideal parallel loading, delays due to datatype inferences, and data-handling bottlenecks (storing data in memory first, then transfering to data model).

Additionally, setting up a significantly large number of data sources in Data Viz tools like Tableau or PowerBI can be quite cumbersome or error-prone. 
One will usually have to either manually load each data source via whatever "Data Source Wizards" are availabe in the software, or add the sources via M-Code or Parametrization which has its own significant limitations.

This Python script serves as a data staging tool that addresses both category of issues by exporting results into a single CSV file. 
The script is also modular, allowing for easy modification and scalability; i.e. results can easily be exported to a SQL Database as opoosed to a CSV File.

Foremost, this script accomodates the querying of any number of SQL databases specified by the user. 
This is handled by a user-defined configuration list which allows for the rapid addition or removal of SQL connections as needed. 
Additionally, the script dynamically handles varying SQL Database types via pre-defined queries (currently accomodating Postgres and MySQL). 
Thus, the PowerBI needs to be only connected to a single CSV file.

Secondarily, the script utilizes ThreadPoolExecutor to facilitate simultaneous SQL pulls from multiple sources, which is significantly superior to PowerBI parallel loading. 
This data flow is generally much more secure and handles potential latency or timeout issues much better. 
Too many simultaneous quiries via PowerBI's internal ODBC's can easily crash or timeout, especially as the row count increases. 
Connecting via Python reduces network handshakes that PowerBI may engage for query folding which can quickly add overhead.

I have tested my dashboard in 2 model flows.

**PowerBI:**
* PowerBI directly connected to 19 SQL Databases (3 MySQL, 17 Postgres)
* All Data Refreshes is handled directly inside of PowerBI and Power Query
* 50,000 row limit for each Database * 19 DB's = 950,000 rows
* Average PowerBI Refresh Time (10 Refreshes): 224 seconds

**Python Data-Staging + PowerBI CSV Refresh**
* All 19 SQL connections are handled inside Python Script
* All rows are exported to a single CSV File.
* PowerBI dashboard has only one data source, being this CSV Export.
* Operator has to execute Python Script first, then refresh PowerBI dashboard once the script is completed
* Average Python Script Time (10 Executions): 78 seconds
* Average PowerBI Refresh Time (10 Executions): 38 seconds
* Average Total Refresh Time (10 Runs): 116 seconds
  
**This represents a 108-second (48%) reduction in cycle time.**

Admittedly, direct internal refreshes in PowerBI are slightly more convenient for end users.
However, the data-staging significantly proves its merit by providing a much more stable and faster refresh cycle-times, avoiding crashes and timeouts.

The benefits of this external data-staging is even more evident the report builder.
This method allows them to optimize parallel data-loads, easily scale for additional SQL data sources, and quickly modify the modular queries and export components per their use case.
