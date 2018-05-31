# capture-lookups
A Symfony Console command. Searches for configuration file that lists URLs of Google Sheets, grabs the Sheets and stores their data locally as CSV files.

Designed be used in the context of the Symfony Console application at https://github.com/forikal-uk/xml-authoring-tools which, in turn, is used in the context of a known directory structure which is based on [xml-authoring-project](https://github.com/forikal-uk/xml-authoring-project).


# Usage instructions


## Specifying the Lookup tables to collect

We assume this command is run in the context of an [xml-authoring-project](https://github.com/forikal-uk/xml-authoring-project). ie. the key aspects of the structure of the directory is known.

Use the `LookUpTablesConfig.yaml` configuration file which defines the locations of the Google Sheets we must collect.

## An example 

I have placed some example files in this project:

https://github.com/forikal-uk/capture-lookups/tree/master/tests/example

In that example, the console command will look in this example [`LookUpTablesConfig.yaml` file](https://github.com/forikal-uk/capture-lookups/blob/master/tests/example/LookUpTablesConfig.yaml).
(See [Yaml Spec > Example 2.6. Mapping of Mappings](http://yaml.org/spec/1.2/spec.html#id2759963) )


This will tell the command to go and find [the 'ItemDetailsLookupTable' Google Sheet specified](https://docs.google.com/spreadsheets/d/1ShazUnvBjWMe1OwJhpoegtx6QfqO8JITtkGlnLgEZrU/edit#gid=0) and, for each tab (that is not skipped), it will write out [this CSV file](https://github.com/forikal-uk/capture-lookups/blob/master/tests/example/ItemDetailsLookupTable-Sheet1.csv).

Note the column headers _may_ start on a row which is not the first row. Hence, `StartingFromRow` value in the configuration.
If there are NUMROWS_SIGNIFY_END_OF_DATA (a constant in the command) consecutive blank rows we assume we are at the end of the sheet's data. So, if someone accidentally adds one blank row we continue, but say 10 blank rows is definately the end of the data rows and we can stop.

## Skipped Tabs - Naming convention

By _Google Sheet tab_ I mean one of the sheets _within_ a workbook. 

Any Google Sheet tab which has a trailing underscore will be considered to be skipped. 

* `foo_` *is* skipped.
* `foo` is not skipped.
* `_foo` is *not* skipped either. 

## Connecting to GSuite

The file that Google Api uses to authenticate access to GSuite should be in the root of the [xml-authoring-project](https://github.com/forikal-uk/xml-authoring-project).

The [ping-drive project explains how to get set up to connect to GSuite](https://github.com/forikal-uk/ping-drive#usage).


## Run the command

When the command is run, it will:

* Search for the scapesettings.yaml in the current working directory, if not found it will look in the parent recursively until a file named scapesettings.yaml is found.
* Determine the `DestinationDirectory` to write-to:
  * If `DestinationDirectory` option is passed to command, use that.
  * If no `DestinationDirectory` option is passed to command, set it to the default `DestinationDirectory` (see below). 
    * The default `DestinationDirectory` is the working directory in which the command was invoked. 
* For each Lookup table specified in the configuration file:
  * Go to the Google Sheet on GSuite
  * Determine and note the name of the Google Sheet
  * For each tab in that sheet:
    * If the tab's name indicates it should be ignored (has a trailing underscore), ignore that tab, skip and move on to the next tab.
    * Else, note the tab name
    * Combine the Google Sheet name with the tab name to set the resulting CSV file's name: `<GoogleSheetName>-<TabName>.csv`. 
    * Check the name to ensure it is made of only alphanumeric characters, dot, hyphen or underscore. (i.e the name is less likely to cause issues if used as a filename on Windows or MacOS)  
    * If the name contains invalid characters, write a meaningful error message to STD_OUT and STD_ERR and exit with an error code.  
    * Check to see if a CSV file matching that name is already stored in the destination directory
    * If it is already present and the `-f` (--force) flag  is NOT set, ask user "Permission to overwrite the file y/n?". With the suggested default prompt being no, `[n]`.
    * If it is already present and the -f (--force) flag  is set, overwrite the existing file without prompting the user.
    * Else, create a CSV file with the chosen name. 
    * Write the contents of the Google Sheet Tab as a CSV file. (comma delimeter, double quotes used to encapsulate strings)  



