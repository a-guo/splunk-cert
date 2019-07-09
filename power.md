## 1. Course Intro
- Buttercup Games
- web = online transactions
- security = badge reader data, AD/DNS data, web login data
- sales = retail sales data, BI data
- network = firewall data, email data, web appliance data
- games = game logs

## 2. Beyond Search Fundamentals
### Basic Search Review
- keywords -> search for single word or group of words
- booleans (NOT, OR, AND)
  - AND implied
  - uppercase
  - () force precedence
- phrases
  - "web error" != web AND error
- field searches
  - status=404, user=admin
- wildcard
  - status=40* matches 40, 40a, 404, etc.
  - starting keywords with wildcard = inefficient
- comparisons
  - =, !=, <, <=, >=, >
- table: returns table containing only specified fields in result set
- rename: renames a field in results
- fields: includes or excludes specified fields
- dedup: removes duplicates from results
- sort: sorts results by specified field
- lookup: adds field values from external source (i.e. csv files)

### Case Sensitive
- boolean operators (uppercase)
- field names
- field values **from lookup**
- regex
- eval and where commands (i.e. eval, where, stats)
- tags

### Case Insensitive
- command names
- command clauses (i.e. AS, BY, WITH)
- search terms
- statistical functions
- field values

### Buckets
- new events -> hot bucket (only writable bucket)
- hot -> warm -> cold
- each bucket has its own raw data, metadata and index files
- metadata files track source, sourcetype and host
- admins can add more

### Searching
- time range to choose which bucket to search -> use bucket indexes to find qualifying events

### General Search Practices
- time is most efficient filter
- most powerful keywords are host, source, sourcetype
- use fields command to extract only fields needed
- only searches for whole words, but wildcards allowed
- *trailing* wildcards = efficient
  - wildcards in *middle* -> inconsistent results
- wildcards tested after all other terms
- inclusion > exclusion
- filter as early as possible (remove dups then sort)
- use appropriate search mode
  - fast - performance > completeness
  - smart - default
  - verbose - completeness > performance

### Tranforming Search Commands
- massages raw data into data table
- transforms specified cell values for each event into numerical values that you can use for statistical purposes
- is required to transform search results into visualizations
- transforming commands include:
  - top
  - rare
  - chart
  - timechart
  - stats
  - geostats

### Fast Mode
- emphasizes performance -> returns only essential and required data
- for non-transforming searches
  - events - fields sidebar displays only those fields required for the search
  - patterns
  - no stats or visualizations
- contents of interesting fields sidebar are lost
- for transforming searches
  - no events or patterns
  - stats or visualizations

### Smart Mode
- best results for search
- non-transforming search:
  - events & patterns
  - no stats or visualizations
- transforming search:
  - no events or patterns
  - stats & visualizations

### Verbose Mode
- emphasizes completeness by returning all possible field and event data
- non-transforming search:
  - events & patterns
  - no stats or visualizations
- transforming search:
  - events, patterns & stats and visualizations

### Types of Searches
- dense: large percentage of data matches the search
  - use case: computing stats, reporting
  - up to 50k matching EPS (CPU bound)
- sparse: small percentage of data matches search
  - use case: troubleshooting, error analysis
  - up to 5k matching EPS (CPU bound)
- super sparse: returns small number of results from each index bucket marching search
  - IO intensive (indexer looks through all of an index's buckets)
  - lot of data -> lot of buckets -> long time
  - up to 2 seconds/index bucket (IO bound)
- rare: indexer checks all buckets to find results, but bloom filters eliminate those buckets that don't include search results
  - use case: user behavior tracking
  - up to 10-50 index buckets/second (IO bound)

### Search Job Inspector
- examine:
  - overall stats of search
  - how search was processed
  - where Splunk spent its time
- troubleshoot search's performance & understand impact of knowledge objects on processing
- any existing search job can be inspected
- 3 components:
  - head
  - execution costs
  - search job properties

### Header
- top of search job inspector provides basic info, including time to run and # of events scanned

### Execution Costs
- provides details on cost to retrieve results
- `command.search.index` - time to search index for location to read in rawdata files
- `command.search.filter` - time to filter out events that do not match
- `command.search.rawdata` - time to read events from rawdata files

### Search Job Properties
- calculate performance:
  - do **not** use `resultCount/time`
  - **USE** `scanCount/time`

## 3. Commands for Visualizations
### Visualization Type
- when search returns statistical results, results can be viewed as visualization:
  - stats table
  - charts: line, column, pie, etc.
  - single value, gauges
  - maps
- not all searches can be visually represented
- data series is a sequence of related data points that are plotted in a visualization
  - can generate any statistical or visualization results

### Single Series
- structured as tables with at least 2 cols
- leftmost col = x-axis vals
- subsequent cols = numeric y-axis vals for each series in the chart

### Multi-Series
- set up underlying search with reporting search commands (i.e. *chart* or *timechart*)

### Time Series
- displays stats trends **over time**
- can be single or multi series

### Charts
- line
  - `timechart count` or `timechart count by action`
- area
  - `timechart count` or `timechart count by action`
- column
  - `chart count over action` or `chart count over action by host`
- bar
  - `chart count over action` or `chart count over action by host`
- pie
  - `chart count over action`
- bubble
  - visualize 3D series
  - plots against 2D on x and y axis
  - **size** of bubble = val for 3rd dimension
- scatter
  - shows trend in relationships between discrete data values
  - shows discrete values that don't occur at regular intervals/belong to a series
- for line, area, and col charts, x-axis is *horizontal*
- for bar chart, x-axis is *vertical*

### chart Command
- display any series of data that you want to plot
- decide which field to plot on x-axis
  - first field after `over` clause = x-axis
  - function defines val of y-axis -> should be numeric
  - `by` clause divides data into sub-groupings
- over *field*
  - `count` func tallies no. of events for each val in the result set
- over *field* by *field*
  - split results
  - can use 2 **by** clauses
  - only split chart restults over 2D (unlike **stats**)

### Misc.
- `chart` and `timechart` automatically filter results to include top 10 highest vals
  - surplus grouped into OTHER
  - NULL & OTHER could skew results
  - `useother=f` and `usenull=f` to remove NULL & OTHER
- use `limit` arg to limit no. of plotted series
- `limit=0` = unlimited

### timechart Command
- performs statistical aggregations against time
- plots and trends data over time
- `_time` is always x-axis
- optional: split data via `by` clause for one other field
- best represented as line or area charts
- split by *usage* field, each line = unique field val
  - vs. stats -> only **one** field after by
- y-axis = count for each field val
- multi-series = no -> all fields share y-axis
- multi-series = yes -> y-axis split for each field val
  - split into sections, each spanning same max & min count
- "buckets" values of `_time` field
  - dynamic sampling intervals based on time range of search
  - default: last 60 mins = 1 min span; last 24 hrs = 30 mins span
- adjust interval via `span` arg
- can apply statistical functions

### Trellis Layout
- display multiple charts based on one result set
- allows visual comparison between different categories
- data only fetched once

### Transforming Command Summary
- `stats`
  - many multi-level breakdown
- `chart`
  - 2 multi-level breakdown
  - `useother=f` & `usenull=f`
  - `limit=n`; `default=10`
- `timechart`
  - 1 multi-level breakdown
  - `useother=f` & `usenull=f`
  - `limit=n`; `default=10`
  - span -> set time val on x-axis
- use `top/rare` to count freq of fields
- use `stats` to calculate stats for 2+ by field (non-time based)
- use `chart` to calculate stats when x-axis is not `_time`
- use `timechart` to calculate stats with `_time` as x-axis
- each col = distinct val of split-by field

## 4. Advanced Visualizations
### trendline Command
- allows you to overlay a computed moving average on a chart
- trendline computes moving avg of a field
- trendtype:
  - sma - simple moving avg
  - ema - exponential moving avg
  - wma - weighted moving avg
- **must** define *period* over which to compute trend
- period must be int between 2 & 10000

### iplocation Command
- lookup & add location info to an event
  - includes city, country, region, latitude, longitude
- not all info available for all ip address ranges
- auto define default `lat` and `lon` fields required by `geostats`

### geostats Command
- compute statistical functions and render cluster map
- data must include latitude and longitude vals
- define latfield and longfield only if they differ from default lat and lon fields
- control col count:
  - global -> `globallimit`
  - local -> `locallimit`

### Choropleth Map
- use shading to show relative metrics (sales, netowrk intruders, etc.) for predefined geographic regions
- define regional boundaries:
  - KML (keyhole markup language) file
  - KMZ (compressed keyhold markup language) file
- Splunk ships with:
  - geo_us_states = US
  - geo_countries = countries of the world

### geom Command
- `geom geo_countries featureIdField=VendorCountry`

### Single Value Visualizations: Formatting
- set color via UI or `gauge` command
- number format info in *Number Format* tab
- with `timechart` can add sparkline and trend
- **sparkline** = inline chart
  - designed to display time-based trends associated with primary key
- **trend** shows direction in which values are moving
  - appears to the right of single val

### Adding Totals via Format Options
- auto total every col using format options
- cannot indicate which col to total; all cols are always totaled
- cannot add labels
- via **Summary** tab, can add % row to the end of stats table
- all cols used to compute percentage

### addtotals Command
- compute sum of all *or selected* numeric fields for each col and place the total in the last row
- compute sum of all *or selected* numeric fields for each row & place total in last col
- `addtotals [row=bool] [fieldname=field] [col=bool] [labelfield=field] [label=string] field-list`
- row options:
  - row=t/f (default=true): col is created that contains numeric totals for each row
  - fieldname=field (default=Total): defines string used to create field name for totals col
- col options:
  - col=t/f (default=false): row created that contains numeric totals for each col
  - label=string (default=Total): defines string used to name totals row
  - labelfield=fieldname : defines where label string is placed (generally first col)
- general options:
  - field-list=one or more numeric fields (default=all numeric fields): defines numeric fields to be totaled
- `addtotals row=f col=t label=totalBytes labelfield=host Bytes`
  - don't total rows; total cols
  - add label totalBytes & place label under host col
  - only total Bytes col

## 5. Filtering and Formatting Data
### eval Command
- calculate & manipulate field vals in report
- `eval fieldname1=expression1 [,fieldname2=expression2...]`
- supports a variety of functions
- results of eval written to either new or existing field you specify
  - if dest field exists, val of fields replaced by results of eval
  - indexed data not modified, and no new data is written into index
  - field vals treated in case-sensitive manner
- calculate expressions; place results in field; use field in searches/other expressions
- arithmetic: + - * / %
- concatenation: + .
- boolean: AND OR NOT XOR
- comparison: < > <= >= != = == LIKE
- `round(field/number, decimals)` sets val of field to no. of decimals specified
- if no. of decimals unspecified, result if whole no.
- `tostring` converts numeric field val to string
  - `tostring(field, "option")`
  - options:
    - "commas": applies commas
      - if includes decimals, round to 2 decimals
    - "duration": formats number as "hh:mm:ss"
    - "hex": formats number as hexadecimal
- `eval` w/ added chars converts vals to string -> first sort then eval
- multiple expressions can be combined
- each subsequent expr references results of previous expr
- exprs separated by commas
- to count no. of events that contain a specific field val, use `count` and `eval` functions
  - use within transforming command, i.e `stats`
  - requires `as` clause
  - double quotes for char field vals
  - field vals are **case sensitive**

### if Function  
- `if(X, Y, Z)`
- takes 3 args
- first arg X is bool expr
  - if X=true -> eval second arg Y
  - if X=false -> eval third arg Z
- non-numeric vals enclosed in double quotes
- field vals case-sensitive

### case Function
- `case(X1, Y1, X2, Y2, ...)`
- first arg X1 is bool expr
- if X1=true -> eval Y1
- if X1=false -> eval next bool expr X2, etc.
- if want "otherwise" clause, test for cond known to be true (i.e. 0=0)

### Search vs. Where
- both filter results
- search:   
  - easier if familiar with basic search syntax
  - treats field vals in **case-insensitive** manner
  - allows searching on keyword
  - can be used at any point in search pipeline
- where:
  - can compare vals from two different fields
  - functions are available, i.e. `isnotnull()`
  - treats field vals in **case-sensitive** manner
  - can't appear before first pipe in search pipeline

### search Command
- use at any point in search pipeline
- like search strings before first pipe
  - use * wildcard
  - treats field vals in case-insensitive manner

### where Command
- `where eval-expression`
- same expression syntax as `eval`
- bool exprs to filter search results & only keeps results that are TRUE
- double quote strings = field vals
  - field vals case-sensitive
- unquoted/single-quoted strings = fields
- can wildcard search
- use _ for one char and % for multiple chars
- must use `like` operator w/ wildcards

### fillnull Command
- replace null vals in fields
- use `value=string` to specify string you want displayed instead
- if no `value=` clause, default replacement val is 0
- optional: restruct fields that `fillnull` apply to by listing them at end of command
  - i.e. `fillnull VALUE="N/A" discount refund`

## 6. Correlating Events
### What is a Transaction
- any group of related events that span time
- can come from multiple applications or hosts

### transaction Command
- creates single event from group of events
  - must share same value in a specified field
- `transaction field-list`
  - `field-list` can be one field name or a list of field names
  - events are grouped into transactions based on the values of these fields
  - if multiple fields & relationship exists between them, grouped into single transaction
- common constraints:
  - maxspan, maxpause, startswith, endswith
- transactions can cross multiple tiers, i.e. web servers or application servers
- produces additional fields:
  - duration - difference between timestamps for first & last event in transaction
  - eventcount - number of events in transaction
- maxspan/maxpause:
  - define max overall time span & max gap between events
  - maxspan - max total time between earliest and latest events (defailt is -1 = no limit)
  - maxpause - max total time between events (default is -1 = no limit)
- startswith/endswith:
  - form transactions based on terms, field vals, or evals
- can use statistics and reporting commands with transactions

### Investigating w/ Transactions
- useful when single event doesn't provide enough info
- can search and see additional related events
- mid - message ID
- dcid - delivery connection ID
- icid - incoming connection ID

### transaction vs. stats
- by default, use `stats` - faster & more efficient in large envs
- use `transaction` when:
  - need to see events correlated together
  - must define event grouping based on start/end vals or segment on time
- use `stats` when:
  - want to see results of calculation
  - can group events based on field val (i.e. src_ip)
- limit of 1000 events/transaction
- no limit for `stats`
- admins can change limit by configuring `max_events_per_bucket` in `limits.conf`

## 7. Introduction to Knowledge Objects
### What are Knowledge Objects
- tools used to discover and analyze various aspects of data
- data interpretation - fields and field extractions
- data classification - event types
- data enrichment - lookups and workflow actions
- normalization - tags and field aliases
- datasets - data models
- sharable - can be shared between users
- reusable - persistent objects that can be used by multiple people or apps, such as macros and reports
- searchable - since objects are persistent, they can be used in search

### Knowledge Manager
- oversees knowledge object creation and usage for a group or deployment
- normalizes event data
- creates data models for Pivot users

### Naming Conventions
- **group**: corresponds to the working groups of the user saving the objects (i.e. NOC)
- **object type**: indicates the type of object (alert, report, summary-index-populating)
- **description**: meaningful description of the context and intent of the search, limited to one or two words if possible; ensure search name is unique
- i.e SEG_Alert_WinEventLogFailures

### Permissions
- private: only person who created object can use/edit it
  - create: user, power, admin
  - read: person who created it, admin
  - write: person who created it, admin
- this app only: objects persists in the context of a specific app
  - create: power, admin
  - read: user*, power*, admin
  - write: user*, power*, admin
- all apps: objects persists globally across all apps
  - create: admin
  - read: user*, power*, admin
  - write: user*, power*, admin
- * = permission to read/write if creator gives permission to that role
- when object is created, display for set to owner by default
- when objects permissions set to app/all apps, all roles given read permissions
  - write for admin and creator unless creator edits permissions
- only admin can promote object to all apps

## 8. Creating and Managing Fields
### Field Auto-Extraction
- auto-discovers fields based on sourcetype and key/value pairs
- prior to search time, some fields already stores with event in index
  - meta fields: host, source, sourcetype
  - internal fields: `_time`, `_raw`
- field discovery finds fields directly related to search results
- may also extract fields from raw event data not directly related to search

### Performing Field Extractions
- can extract fields with Field Extractor (FX)
- use FX to extract static fields & often used in searches:
  - graphical UI
  - extract fields from events using regex or delimiter
  - extracted fields persist as knowledge objects
  - can be shared and reused in multiple searches
- access FX: settings, fields sidebar, event actions menu

### Field Extraction Methods
- regex
  - when event contains **unstructured** data like system log file
  - FX attempts to extract fields using regex that matches simliar events
- delimiter
  - when event contains **structured** data like .csv file
  - data doesn't have headers and fields must be separated by delimiters (space, comma, pipe, tab, etc.)

## 9. Creating Field Aliases and Calculated Fields
### Field Aliases
- way to normalize data over any default field (host, source or sourcetype)
- multiple aliases can be applied to one field
- applied after field extractions, before lookups
- can apply field aliases to lookups
- settings > fields > field aliases > new fifeld alias
- new field alias required for each sourcetype
- after define alias, can references them in lookup table

### Field Alias and Original Fields
- original field not affected
- both fields appear in all fields and interesting fields list (if appear in at least 20% of events)

### Calculated Field
- shortcut for performing repetitive, long or complex transformations using `eval` command
- must be based on extracted field
- settings > fields > calculated fields > new calculated field
- can use it in search like any other extracted field

## 10. Creating Tags and Event Types
### Describing Tags
- nicknames for related field/value pairs
- make data more understandable & less ambiguous
- create one or more tags for any field/value combination
- **case sensitive**
- tags appear:
  - in results as tags
  - in parentheses next to associated field/value pairs

### Searching for Tags
- tag associated w/ val: `tag=<tag name>`
- associated w/ val on specific field: `tag::<field>=<tag name>`
- partial field val: `tag=p*` (use wildcard)

### Describing Event Types
- method of categorizing events based on a search
- useful for institutional knowledge capturing and sharing
- can be tagged to group similar types of events

### Using Event Types
- to verify the event type, search for `eventtype=web_error`
- `eventtype` displays in the fields sidebar and can be added as a selected field
- Splunk evaluates teh events and applies appropriate even types at search time

### Tagging Event Types
- settings > event types
- event details > actions

### Event Types vs. Saved Reports
- event types
  - categorize events based on search string
  - tag event types to organize data into categories
  - `eventtype` field can be included in search string
  - does not include time range
- saved reports
  - search criteria will not change
  - includes time range and formatting results
  - can be shared with Splunk users and added to dashboards

## 11. Creating and Using Macros
### Macros Overview
- useful when frequently running searches or reports with similar search syntax
- time range selected at search time
- can be full search string or portion of a search that can be reused in multiple places
- can define 1+ args within search segment
  - pass param vals to macro at execution time
  - macro uses vals to resolve search string
- settings > advanced search > search macros

### Basic Macro
- surround macroname with **backtick**
  - **NOT** single quotes

### Adding Args
- include no. of args in () after macro name (i.e. `monthly_sales(3)`)
- within search def use $arg$
  - i.e `currency=$currency$`, `symbol=$symbol$`, `rate=$rate$`
- in args field, enter name of args
- provide one or more vars of macro at search time

### Using Args
- pass in args in same order as defined

### Validating Macros
- can validate arg vals in macro
- validation expressions: enter expression for each arg
  - arg must be enclosed in dollar signs
- validation error message: message that appears when you run the macro

## 12. Creating and Using Workflow Actions
### Workflow Actions
- execute workflow actions from event in search results to interact with external resources or run another search
  - GET - retrieve info from external resource
  - POST - send field vals to external resource
  - Search - use field vals to perform a secondary search
- settings > fields > workflow actions > new workflow action

## 13. Creating Data Models
### Pivot
- used for creating reports and dashboards

### Data Model Overview
- heirarchically structured datasets that generate searches and drive Pivot
- Pivot reports are created based on datasets
- each event, search or transaction saved as separate dataset
- consist of 3 types of datasets
  - events
  - searches
  - transactions

### Data Model Events
- **event datasets** contain constraints and fields
- **constraints** = search broken down into hierachy
- **fields** = properties associated w/ events

### Dataset Fields
- select fields to be included in dataset
- like constraints, fields are inherited from parent objects

### Adding Fields
- **auto-extracted** - can be default fields or manually extracted fields
- **eval expression** - new field based on an expression you define
- **lookup** - leverage an existing lookup table
  - configured same way as automatic lookup
  - use preview to test lookup settings
  - use events & values tab to verify results
- **regex** - extract a new field based on regex
- **geo ip** - add geo fields such as lat/long, country, etc.
  - map visualizations require lat/long fields
  - to use geo ip lookup, at least one IP field must be configured as IPv4
  - map not available in Pivot, data model can be called using `| pivot` and `<map>` element in dashboard population search

### Field Types
- string: alphanumeric
- number: numeric
- boolean: t/f or 1/0
- IPV4: IP address
  - at least one IPV4 attribute type must be present in data model to add Geo IP attribute

### Field Flags
- optional
- required
- hidden: not displayed to Pivot users when they select the dataset in Pivot
  - used for fields that are only being used to define another field, i.e. eval expr
- hidden & required: only events that contain this field are returned & fields hidden from use in Pivot

### Child Datasets
- when creating, give it 1+ additional constraints
- inherit all fields from parent events
- can add more fields

### Data Model Search Datasets
- arbitrary searches that include transforming commands to define the dataset that they represent
- search datasets can also have fields, which are added via **add field** button

### Data Model Transaction Datasets
- enable creation of datasets that represent transactions
- use fields that have already been added to the model using event or search datasets

### Adding a Transaction
- can add to data model
- can add eval expr or any other field to transaction to further define results

### Search and Transaction Dataset Considerations
- must be at least 1 event or search dataset before adding transaction dataset
- datasets cannot benefit from persistent data model acceleration

### Download & Upload Data Models
- backup important data models
- collaborate with other Splunk users to create/modify/test data models
- move data models from a test env to prod instance

### Data Model Acceleration
- use auto create summaries to speed completion times for pivots
- takes form of inverted time-series index files (tsidx) that have been optimized for speed
- all fields in model become "indexed" fields
- need admin permissions or `accelerate_datamodel` capability to accelerate a data model
- private data models cannot be accelerated
- accelerated data models cannot be edited

## 14. Using the Common Information Model (CIM) Add-On
### Common Information Model (CIM)
- methodology to normalize data
- ensure:
  - multiple apps can co-exist on single Splunk deployment
  - object permissions can be set to global for the use of multiple apps
  - easier and more efficient correlation of data from different sources and source types

### Email Data
- action: action taken by reporting device
  - data type: string
  - possible vals: delivered, blocked, quarantined, deleted, unknown
- duration: amount of time for completion of messaging event, in seconds
  - data type: number
  - possible vals: email
- src: system that sent message (could be aliased from more specific fields)
  - data type: string

### Network Traffic
- action: action taken by network device
  - data type: string
  - possible vals: allowed, blocked, dropped, unknown
- bytes: total count of bytes handled by this device/interface (in + out)
  - data type: number
- bytes_in: how many bytes this device/interface received
  - data type: number
- bytes_out: how many bytes this device/interface transmitted
  - data type: number
- src: source of network traffic (client requesting connection)
  - data type: string

### Web Data
- action: action taken by server or proxy
  - data type: string
- duration: time taken by proxy event, in milliseconds
  - data type: number
- http_method: HTTP method used in the request
  - data type: string
  - possible vals: GET, PUT, POST, DELETE, etc.
- src: source of network traffice (client requesting connection)
  - data type: string
- status: HTTP response code indicating status of proxy request
  - data type: string
  - possible vals: 404, 302, 500, etc.

### Splunk CIM Add-On
- set of 22 pre-configured data models
  - fields and event category tags
  - least common denominator of a domain of interest
- leverage CIM so that knowledge objects in multiple apps can co-exist on single Splunk deployment
- settings > data models

### datamodel Command
- search against specified data model object
- return description of all or a specified data model and its objects
- is a generating command -> 1st command in pipeline
- syntax: command, data model name, data model dataset name, command, pipe, etc.

### from Command
- retrieves data from data model or named dataset
- must be first command in search
- different than `datamodel`
  - `datamodel` returns **all** fields prepended with data model name
  - `from datamodel` returns specified fields only
- `from` can also retrieve data from saved searches, reports, or lookup files

