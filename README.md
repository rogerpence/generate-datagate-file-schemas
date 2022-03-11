Writing business applications requires writing tons of complex, specialized code. It also requires writing tons of repetitive, boring, mechanical code. To be effective coders, you need to maximize the time you spend on custom, unique code and minimize the time you spend on repetitive, repeatable code. Let's spend a little thinking about how to generate that repeatable code.

By "repeatable code" I mean things like 

* Standard Create/Read/Update/Delete operations
* Code to populate paged lists for displaying data
* File and data structure declarations 
* Input panel creation for Web or Windows apps
* Code to populate dropdown lists

By standardizing these common processes, not only do you make them available quickly, but you also make them available reliably. Code produced with a tested, solid template reduces testing and Q/A time and ensures correctness. 

To generate these repeatable chunks of code, we need:

1. An easy way to produce reliable meta data about files/tables (ie, what are the fields/columns and their data types). This data is pulled directly from existing files and tables.

2. A way to extend this existing meta data (ie, what kind of UI element should a field use in an input panel) to create a more complete data dictionary. You can do a lot without having a way to extend existing meta data, but and extended data dictionary adds substantial possibilities. For now, we'll consider a nice to have, but unnecessary thing--but we will get back to it later.

3. A flexible template engine that lets you create templates that produce AVR, C#, SQL, HTML, Markdown, ASPX, MVC or other Web markup, or any other output type that you need.  

This repo tackles the first part of the challenge, producing meta data about DataGate files. 

### Project dependencies

This project requires 

* The DLL produced by this project https://github.com/rogerpence/ArgyBargy 
* The NewtonSoft.json DLL from this project at [Nuget.org](https://www.nuget.org/):
  * https://www.nuget.org/packages/Newtonsoft.Json/

Schemas can be produced on any Windows PC with AVR runtime installed and a DataGate database connection available. 

### Producing reliable meta data

`GenFileSchema` is an AVR program that produces a Json file (a schema file) that describes each data file in a given library. It was written with AVR 15.x but should work with versions down to about AVR 12.x and anything higher than AVR 15.x.

`GenFileSchema` is a command line application. After installing it, type `GenFileSchema` on the command line and press enter. The screen of text below is shown. (You can also see this screen using the --help or -h flag.)

```
Generate DataGate file schemas for a library

Flag                  ShortHand  Required  Description
--------------------  ---------  --------  ---------------------------------------------
--databasename           -d        True    DateGate Database Name
--library                -l        True    Library name
--outputpath             -o        False   Output path (appended to output path selected--see below)
--physicalsonly          -po       False   Process physical files only (default is false)
--help                   -h                Show this help

The default schema output path root is:
    C:\Users\roger\Documents
That default can be overridden in the 'TargetFolderRoot' of the app config file.
The full schema output path is the output path plus the --outputpath/-op path. That path must exist.path. 
```

The default schema output path is `c:\users\[user]documents` but that path can be overridden with the `TargetFolderRoot` key in the `app.config` file.

```
    <appSettings>
        <add key="TargetFolderRoot"
             value="c:\somewhere..."
        />
    </appSettings>   
```

>To make this change within Visual Studio, make the change to the `app.config` file and recompile the app. If you want to change this path without recompiling the program,  _carefully_ change the `GenFileSchema.exe.config` file (which is essentially the runtime version of `app.config`.) This file is in the same folder as the `GenFileSchema.exe.` Note that changes made to `GenFileSchema.exe.config` file are overwritten the next time you compile the app.  

### Using the program

To produce Json schema files for all of the files in the `Devo` library in the database defined by the `*Public/DG NET Local` database name: 

```
genfileschema -d "*Public/DG NET Local" -l devo -o devo 
```

By default, this produces the Json files in the `c:\users\[user]documents\devo` directory.

The files produced look like this:

```
{
    "dbname": "*public/dg net local",
    "library": "devo",
    "file": "states",
    "format": "rstates",
    "description": "",
    "type": "physical",
    "recordlength": "50",
    "keylength": "48",
    "basefile": "",
    "duplicatekeys": "not allowed",
    "sqlserveruniqueindex": "unique",
    "alias": "states",
    "keyfieldslist": "state",
    "allfieldslist": "state, abbrev",
    "fields": [
        {
        "name": "state",
        "description": "",
        "alias": "state",
        "fulltype": "Type(*char) Len(48)",
        "type": "*char",
        "length": "48",
        "decimals": "",
        "systemtype": "System.String",
        "iskey": true,
        "keyposition": 0,
        "allownull": false,
        "sqlservertype": "char",
        "sqlservernull": "NOT NULL",
        "sqlserverprimarykey": "PRIMARY KEY"
        },
        {
        "name": "abbrev",
        "description": "",
        "alias": "abbrev",
        "fulltype": "Type(*char) Len(2)",
        "type": "*char",
        "length": "2",
        "decimals": "",
        "systemtype": "System.String",
        "iskey": false,
        "keyposition": -1,
        "allownull": false,
        "sqlservertype": "char",
        "sqlservernull": "NOT NULL",
        "sqlserverprimarykey": ""
        }
    ],
    "keyfields": [
        {
        "name": "state",
        "description": "",
        "alias": "state",
        "fulltype": "Type(*char) Len(48)",
        "type": "*char",
        "length": "48",
        "decimals": "",
        "systemtype": "System.String",
        "iskey": true,
        "keyposition": 0,
        "allownull": false,
        "sqlservertype": "char",
        "sqlservernull": "NOT NULL",
        "sqlserverprimarykey": "PRIMARY KEY"
        }
    ],
    "nonkeyfields": [
        {
        "name": "abbrev",
        "description": "",
        "alias": "abbrev",
        "fulltype": "Type(*char) Len(2)",
        "type": "*char",
        "length": "2",
        "decimals": "",
        "systemtype": "System.String",
        "iskey": false,
        "keyposition": -1,
        "allownull": false,
        "sqlservertype": "char",
        "sqlservernull": "NOT NULL",
        "sqlserverprimarykey": ""
        }
    ]
}
```


### File-level tokens

```
dbname................... Database name
library.................. Library name
file..................... File name
format................... Format name
description.............. File description
type..................... File type (physical or logical)
recordlength............. Record length
keylength................ Key length
basefile................. Base file (logical files only)
duplicatekeys............ Are duplicated keys allowed
sqlserveruniqueindex..... 'unique' if index is unique
alias.................... File alias (same as 'file')
keyfieldslist............ Comma-separated list of key fields
allfieldslist............ Comma-separated list of all fields
```

### Field-level tokens
```
name..................... Field
description.............. Description 
alias.................... Field alias (initially the same as 'name')
fulltype................. Field full type
type..................... Field type
length................... Field length
decimals................. Field decimals
systemtype............... .NET system type
iskey.................... True if field is a key field
keyposition.............. Zero-based key position or -1 if field is not a key
allownull................ Allow nulls (True or False)
sqlservertype............ SQL Server type 
sqlservernull............ Nullable
sqlserverprimarykey...... 'PRIMARY KEY' if field is part of primary key
```

### What can I do with the Json schema files

The Json schema files can be used by any Json-capable tool to do whatever you can dream up. 

For example, using [Python](https://www.python.org/) with the [Jinja](https://jinja.palletsprojects.com/en/3.0.x/) template engine provides some interesting opportunities. [See this GitHub repo for more information.]()https://github.com/rogerpence/LibrettoX
