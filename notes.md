## 1. Introducing Splunk
### What is Splunk
- Index data -> search & investigate -> add knowledge -> monitor & alert -> report & analyze
- aggregate, analyze, and get answers from your machine data
- application management, operations management, security & compliance, etc.

### What Data
- ANY data from ANY source
- sources: computers, network devices, virtual machines, internet devices, communication devices, sensors, databases
- data: logs, configurations, messages, call detail records, clickstream, alerts, metrics, scripts, changes, tickets

### How does Splunk Work?
search head -> splunk indexer <- splunk forwarders

### How is Splunk Deployed?
- Enterprise: components installed and administered on prem
- Cloud: Enterprise as scalable service (no infra required)
- Light: solution for small IT envs  

### Spunk Apps
- address wide variety of use cases and extend power of Splunk
- collections of files containing data inputs, UI elements, and/or knowledge objects
- allows multiple workspaces for different use cases/user roles to co-exist on a signle Splunk instance
- 1000+ read-made apps available on Splunkbase or admins can build their own

### Splunk Enhanced Solutions
- Splunk IT Service Intelligence (ITSI)
  - next gen monitoring and analytics solution for IT Ops
  - uses machine learning and event analytics to simplify operations and prioritize problem resolution
- Splunk Enterprise Security (ES)
  - comprehensive security information and event management (SIEM) solution
  - quicly detect and respond to internal and external attacks
- Splunk User Behavior Analytics (UBA)
  - finds unknown, known, and hidden threats by analyzing user behavior and flagging unusual activity

### Users & Roles
- users assigned roles -> determine their compatibilities and data access
- admin, power, user
- admins can create additional roles

- apps allow different workspaces for specific use cases or user roles to co-exist on a single Splunk instance

### Search & Reporting App
- provides a default interface for searching and analyzing data
- enables you to create knowledge objects, reports, and dashboards
- click Data Summary to see hosts, sources, or sourcetypes on separate tabs
- host = unique identifier of where the events originated (host name, IP address, etc.)
- source = name of file, stream, and other input
- sourcetype = specific data type or data format

## 2. Splunk Components
### Indexer
- process machine data, storing result in indexes as events, enabling fast search and analysis
- creates a number of files oganized in sets of directories by age
  - contains raw data (compressed) and indexes (points to the raw data)

### Search Heads
- allows users to use the search language to search the indexed data
- distributes user search requests to indexers
- consolidates results and extracts field value pairs from the events to the user
- knowledge objects on the search heads can be created to extract additional fields and transform the data without changing the underlying index data
- tools to enhance visualization

### Forwarders
- consume and send data to index
- require minimal resources and have little impact on performance
- reside on machines where data originates
- primary way data is supplied for indexing

### Other Components
- deployment server, cluster master, license master

### Deployment
- standalone
  - single server: all functions in a single instance
  - for testing, proof of concept, personal use, and learning
  - recommend: have at least one test/dev setup at site
- basic
  - similar to standalone
  - manage deployment of forwarder configs
  - forwarders collect data and sent to servers (install forwarders at data source - prod servers)
- multi instance
  - increases indexing and searching capacity
  - search management and index functions are split across multiple machines
- increasing capacity
  - adding search head cluster
    - services more users for increased search capacity
    - allows users and searchers to share resources
    - coordinate activities to handle search requests and distribute the requests across the set of indexers
  - require min 3 search heads
  - deployer is used to manage and distribute apps to members of search head cluster
- index cluster
  - traditional
    - configured to replicate data
    - prevent data loss
    - promote availability
    - manage multiple indexers
  - non replicating
    - offer simplified management
    - do not provide availability or data recovery

## 3. Installing Splunk
### Common Splunk Commands
- `splunk help` display usage summary
- `splunk [start | stop | restart]` manage splunk processes
- `splunk start --accept-license` automatically accept license w/o prompt
- `splunk status` display Splunk process status
- `splunk show splunkd-port` show the port that the splunkd listens on
- `splunk show web-port` show the port that splunk web listens on
- `splunk show servername` show servername of instance
- `splunk show default-hostname` show default host name used for all data inputs
- `splunk enable boot-start -user` init script to run splunk enterprise at system startup

## 4. Getting Data In
### Index Time Process  
1. input phase: handled at the source (usually forwarder)
  - data sources are being opened and read
  - data is handled as streams and any config settings are applied to the entire stream
2. parsing phase: handled by indexers (or heavy forwarders)
  - data is broken up into events and advanced processing can be performed
3. indexing phase
  - license meter runs as data and is initially written to disk, prior to compression
  - after data is written to disk, it *cannot* be changed

### Data Input Types
- files & directories: monitoring text files and/or directory structures containing text files
- network data: listening on a port for network data
- script output: executing a script and using the output from the script as the input
- windows log: monitoring windows events logs, AD, etc.
- HTTP: using HTTP event collector
- add data inputs via: apps & add ons, splunk web, CLI, edit inputs.conf

### Metadata
- when index data source, Splunk assigns metadata values
- applied to entire source
- can override defaults at input time or later
- source = path of input file
- host = hostname for inputting instances
- sourcetype = source filename
- index = defaults to main

### Add Data
- how to get add data: click on add data icon OR settings > data inputs > add new
- source options: upload, monitor, forward

### Set Source Type
- index once = for one time indexing or testing
- Splunk automatically determines source type for major data types where there is enough data
- can choose different source type or create new source type name
- data preview displays how processed events will be indexed
- move forward *ONLY IF* events are correctly separated & right timestamps are highlighted

### Pretrained Source Types
- default settings for many types of data
- docs contain list of source types that Splunk automatically recognizes
- apps can define additional source types

### Input Settings
- select index where input should be stored
- to store in a new index, first create the new index

## 5. Basic Search
### Search Assistant
- provides selections for how to complete search string
- before first pipe, it looks for matching terms
- enabled by default in SPL editor user preferences
- compact is selected by default (choose full for more info)
- full mode
  - click more or less accordingly
  - deselect auto open to toggle full mode off
- helps match parentheses
- when ) is typed, corresponding ( is auto highlighted
  - if no (, *nothing* is highlighted

### Viewing Search Results
- results returned immediately in reverse chronological order (newest first)
- matching terms highlighted
- each event has: timestamp, host, source, sourcetype, index
- click on item in search results -> you can: add to/exclude from search or new search including only that item

### Time Range
- to specify beginning and ending for a time range, use `earliest` and `lastest`
- `earliest=-h` looks back one hour
- `earliest=-2d@d lastest=@d` looks back from two days ago, up to beginning of today
- `earliest=6/5/2017:12:30:00` looks back to specified time

### Viewing Subset of Results w/ Timeline
- click & drag across a series of bars (filters current search results, not re-execute search)
- filters events & displayed in reverse chronological order

### Other Timeline Controls
- format timeline: hides/shows timeline in different views
- zoom out: expands the time focus and re-executes search
- zoom to selection: narrows time range and re-executes search
- deselect: in drilldown, returns original results set; otherwise, grayed out/unavailable

### Controlling & Saving Search Jobs
- each search = job
- pause = toggles to resume the search
- stop = finalize search in progress
- jobs available for 10 mins (default)
- link to results from *Job* menu

### Setting Permissions
- private (default): only creator can access
- everyone: all app users can access search results
- lifetime: default is 10 mins; schedule report to keep search results longer

### Sharing Search Jobs
- share button
  - gives everyone read permissions
  - extend results retention to 7 days
  - sharable link
- multiple users working on same issue to see same data
  - more efficient
  - less load on server & disk space used
- click printer icon to print results or save as PDF

### Exporting Search Results
- export results -> raw events (text file), CVS, XML, JSON

### View Saved Jobs
- saved searches from *Activity* menu (Activity > Jobs)
- search jobs view displays jobs: in past 10 mins & extended for 7 days

### View Search History
- display most recent ad-hoc searches
- set time filter to narrow results
- > icon (leftmost column) expands to display full text

## 6. Using Fields
- between search terms, *AND* is implied unless otherwise specified

### Field Discovery
- automatically discovers fields based on sourcetype and key/value pairs in data
- prior to search, some fields already stored w/ even in the index:
  - meta fields: host, source, sourcetype, index
  - internal fields: time and raw
- some fields in overall data may not appear within results of a particular search
- data-specific fields come from data within event (i.e. `action=purchase` or `status=200`)

### Fields Sidebar
- selected fields - set of configurable fields displayed for each event (default: host, source, sourcetype)
  - appear listed under every event that includes those fields
- interesting fields - occur in at least 20% of resulting events
- all fields - link to view all fields (including non-interesting fields)

### Using Fields in Searches
- field names *ARE* case sensitive
- field values *ARE NOT*
- for IP fields, Splunk is subnet/CIDR aware
- use wildcards to match range of field values
- use relational operators (w/ numeric and alphanumeric fields)

### != vs NOT
- != field exists and value in field does not equal whatever is specified
- NOT like !=, but also all events where field *DNE*
- could return the same results (if know field always exists)

### Search Modes
- fast: emphasizes speed over completeness
- smart: balances speed and completeness (*default*)
- verbose: emphasizes completeness over speed & allows access to underlying events when using reporting or statistical commands (in addition to totals and stats)

## 7. Best Practices
### Search
- time is most efficient filter
- specify at least one index value at beginning of search  
- more search terms = good
- specific search terms
- inclusion > exclusion
- filter early (i.e remove dups then sort)
- avoid wildcards at beginning/middle of string
  - scan all events within timeframe & return inconsistent results
- OR > *

### Working w/ Indexes
- can specify multiple index values in a search
- can use * in index values
- can search w/o index (*not recommended* bc inefficient)

### Viewing Index Field
- index always appears as a field
- specify at least one index in search
- in example, no index specified so data from 2 indexes: web & sales (*BAD*)

## 8. Splunk's Search Language
### Syntax Components
1. search terms - keywords, phrases, bools, etc.
2. commands - create chart, compute stats, eval & format, etc.
3. functions - get sum, get avg, transform vals, etc.
4. arguments - calculate avg for specific field, convert ms to s, etc.
5. clauses - give field another name or group vals by or over

- top defaults to top 10
- color based on search syntax; rest of search string remains black
- bools & command modifiers = orange
- commands = blue
- command args = green
- functions = purple

### Creating a Table
- `table` -> table formed by only fields in the arg list
- cols displayed in order given by command
  - col headers are field names
  - each row = event & contains field values for that event

### Renaming Fields in Table
- `rename` gives fields more meaningful names
- when including spaces/special chars in field names, use double quotes
- after rename, can't access it w/ original name

### fields Command
- field extraction is costly
- `field` allows you to include or exclude specific fields
- `fields + ` (default) for inclusion
  - before field extraction
  - improves performance
- `fields - ` for exclusion
  - after field extraction
  - no performance benefit
  - makes table/display easier to read

### dedup Command
- remove duplicates from results

### sort Command
- order results in `+` ascending (default) or `-` descending
- `limit` option limits results
- `sort -/+<fieldname>` sign followed by fieldname sorts results in sign's order
- `sort -/+ <fieldname>` sign followed by *space* and then fieldname applies sort order to *all* following fields w/p different explicit sort order

## 9. Transforming Commands
### top Command
- finds most common values of a given field in the result set
- by default, displayed in tabled format & returns top 10 results
- automatically returns count and percent cols
- common constraints: limit, countfield, showperc
- top command w/ limit=20 automatically added when Top values in field window is clicked
- control # of results using `limit`
- `limit=#` returns this number of results
- `limit=0` returns unlimited results
- if `showperc` not included (or included & set to `t`), percent col is displayed
- if `showperc=f`, percent col is NOT displayed
- by default, display name of countfield is count
- `countfield=string` renames field for display purposes

### rare Command
- returns least common field vals of a given field in results
- options identical to `top` command

### stats Command
- calculate stats on data
- common functions:
  - count - returns # of events that match search criteria
  - distinct_count, dc - returns count of unique values for given field
  - sum - returns sum of numeric vals
  - avg - returns avg of numeric vals
  - list - lists all vals of a given field
  - values - lists unique vals of a given field
- count
  - use `as` clause to rename the count field
  - adding `field` as arg to count returns number of events where a value is present for specified field
    i.e. `stats count(vendor_action) as ActionEvents, count as TotalEvents`
- by fields
  - by clause returns count for each val of named field or set of fields
  - can use any number of fields in the by field list
- disctint_count(field)
  - i.e. `stats dc(s_hostname) as "websites visited:"`
- sum(field)
  - for fields w/ numeric val, can sum actual vals of that field
- avg(field)
  - an event is not considered in calculation if it doesnt have the field OR has an invalid val for the field
- list(field)
  - use `values` function to return list of unique field vals
  - i.e. `stats list(s_hostname) as "Websites visited:" by cs_username`

### Formatting stats Tables   
- color code data in each col, based on rules you define
- add number formatting (i.e. currency symbols, thousands separators)
- format data on per-col basis by clicking edit icon above that col   

## 10. Creating Reports and Dashboards
- timechart = report that shows avg over time
- for alphanumeric char fields, only 3 available reports
- heat map highlights outstanding vals
- high & low vals highlights max & min of non-zero vals
- most of this section was clicking on buttons

## 11. Pivot and Datasets
### Datasets tab
- displays list of available lookup table files ("lookups") and data models
- each lookup and data model represents a specific category of data
- prebuilt lookups and data models make it easier to interact with your data

- pivot automatically populates with a count of events for the selected object
- instant pivot allows you to utilize the pivot tool without a preexisting data model
  - instant pivot creates an underlying data model utilizing the search criteria entered during the initial search

## 12. Creating and Using Lookups
### What is a Lookup
- sometimes static (or relatively unchanging) data is required for searches, but isn't available in the index
- lookups pull such data from standalone files at search time and add it to search results
- lookups allow you to add more fields to your events
  - descriptions for HTTP status codes
  - sale prices for products
  - user names, IP addresses, and workstation IDs associated with RFIDs
- after looking is configured, use lookup fields in searches
- lookup fields also appear in fields sidebar
- lookup field values are case sensitive by default

### inputlookup Command
- used to load the results from a specified static lookup
- useful to:
  - review data in .csv file
  - validate the lookup

### Advanced Lookup Options
- min/max # of matches for each input lookup value
- default value to output (when fewer than min # of matches present for a given input)
- case sensitivity match on/off
- batch index query: improves performance for large lookup files
- match type: supplies format for non-exact matching
- filter lookup: filters results before returning data

### lookup Command
- used to lookup fields (if lookup is not configured to run automatically)
- OUTPUT arg is optional
  - if OUTPUT not specified, lookup returns all fields from lookup table except match fields
  - if OUTPUT specified, fields overwrite existing fields
- output lookup fields exist only for the current search
- use `OUTPUTNEW` when you don't want to overwrite existing fields

- to use automatic lookup, specify output fields in your search
- if field in lookup table represents timestamp, can create time-based lookup

## 13. Creating Scheduled Reports and Alerts
### Why Scheduled Reports?
- useful for
  - monthly, weekly, daily executive/managerial roll up reports
  - dashboard performance
  - automatically sending reports via email

### Creating Scheduled Report
- schedule - frequency to run the report
- time range - by default, search time range used
  - can select time range from presets, relative, or advanced
  - typically, time range relative to schedule
- schedule window - determins time frame to run report
  - if there are other reports scheduled to run at the same time, can provide window in which to run report
  - provides efficiency when scheduling several reports to run

### Add Actions
- log event - creates an indexed, searchable log event
- output results to lookup - sends results of search to CSV lookup file
- output results to telemetry endpoint - sends usage metrics back to Splunk
- run a script - runs a previously created script
- send email - sends an email with results to specified recipients
- webhook - sends an HTTP POST request to specified URL

### Edit Permissions
- owner - all data accessible by the owner appears in the report
- user - only data allowed to be accessed by the user role appears

- before a report can be embedded, it must be scheduled

### What are Alerts?
- based on searches that can run either
  - on a regular scheduled interval
  - in real time
- alerts are triggered when results of the search meet a specific condition that you define
- alerts can:
  - create entry in triggered alerts
  - log an event
  - output results to lookup file
  - send emails
  - use a webhook
  - perform a custom action

### Alerts
- private - only you can access, edit and view triggered alerts
- shared in app
  - all users of app can view triggered alerts
  - by default, everyone has read access and power has write access
- scheduled alerts
  - search runs at a defined interval
  - evaluates trigger condition when search completes
- real time alerts
  - search runs constantly in the background
  - evaluates trigger conditions within a window of time based on conditions you define
- alert examines complete results set after search is run

### Setting Trigger Conditions
- capture a larger data set, then apply more stringent criteria to results before executing alert
- per-result - triggers when a result is returned
- number of results - define how many results are returned before the alert triggers
- number of hosts - define how many unique hosts are returned before the alert triggers
- number of sources - define how many unique sources are returned before alert triggers
- custom - define customer condition using search language

### Alert Actions
- **once** executes actions one time for all matching events within the scheduled time and conditions
  - select throttle option to suppress actions for results within a specified time range
- **for each result** - executes alert actions once for each result that matches the conditions
  - select throttle option to suppress actions for results that have same field value within a specified time range
  - certain situations can cause a flood of alerts, when really you only want one
- all actions available for scheduled reports are also available for alerts
- if have admin privileges, can use a log event action
- customize contents of email alerts

### Editing Alert Permissions
- owner - only you can access, edit and view triggered alerts
- app - users of app can access, edit and view triggered alerts

