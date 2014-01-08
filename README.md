AirVantage API access from Shell command-line
=============================================

The **airvantage-api-shell** project aims to provides a light and flexible Shell script that enables to interact with AirVantage M2M Cloud API in command-line.

Prerequisites
-------------

### AirVantage account

Depending on your location, you need an account on one of AirVantage M2M Cloud datacenters:
* https://na.airvantage.net (North America)
* https://eu.airvantage.net (EMEA)

### jq command-line JSON processor

This project requires the  `jq` command-line JSON processor, which is available at http://stedolan.github.io/jq/

To install it, you only need to:

1. Download the appropriate binaries for your system from http://stedolan.github.io/jq/download/
2. Put these binaries in your `PATH`.

Installing the `av` script
------------------------------

The installation procedure is simple as downloading the script, and include it in your `PATH`.

This can be done with the following command lines:

``` sh
git clone https://github.com/bmiegemolle/airvantage-api-shell.git
export PATH=`pwd`/airvantage-api-shell/scripts:${PATH}
```
You're now ready to use the `av` script! Enjoy!

Get an access token
-------------------

You can get an access token to AirVantage M2M Cloud API with the `av ssh` command:

> av ssh _host_

Example:
``` sh
av ssh na.airvantage.net
```

You'll be asked for your username and password, and you'll get an access token to AirVantage M2M Cloud API. This token will be stored in your file system (in the _/tmp/av-access-token_ file), and it will be automatically used by any further `av` commands.

Who am I?
---------

You can retrieve the details of the currently loggued user with the `av whoami` command:

> av whoami

Example:
``` sh
av whoami
```

List entities
-------------

The `av ls` command will enable you to list some entities of the AirVantage M2M Cloud portal:

> av ls [/entities] [--uid-only]

* _/entities_ is optional. If not specified, then _systems_ will be listed.
* _--uid-only_ argument enables to return only a list of entities uid as plain text, instead of entities as JSON objects.

Examples:
``` sh
# List all systems, and display them as a list of JSON objects
av ls
av ls /systems

# List all users, and display them as a list of plain text UIDs
av ls /users --uid-only

# List all operations, and display them as a list of JSON objects
av ls /operations
```

Find entities
-------------

The `av find` command will enable you to search some entities of the AirVantage M2M Cloud portal:

> av find _-field_ _value_ [/entities] [--uid-only]

* _-field value_ enables you to specify a search criteria.
* _/entities_ is optional. If not specified, then _systems_ will be searched.
* _--uid-only_ argument enables to return only a list of entities uid as plain text, instead of entities as JSON objects.

Examples:
``` sh
# Find systems which name contains "test", and display them as a list of JSON objects
av find -name test

# Find systems with a commStatus equals to "ERROR", and display them as a list of plain text UIDs
av find -commStatus ERROR --uid-only

# Find users whose name contains "admin", and display them as a list of JSON objects
av find -name admin /users

# Find operations created by the user "administrator@m2mop.net", and display them as a list of JSON objects
av find -user administrator@m2mop.net /operations
```

Get entities details
--------------------

Using the `av cat` command, you can retrieve the details of a specific entity of the portal:

> av cat [_arg_]

_arg_ can be:
* /_entities_/_uid_: display the details of the specified entity
* /_uid_: idem, but assumes that the entity is a system
* /_entities_: entities UIDs are retrived from the standard input
* *empty*: entities UIDs are also retrived from the standard input, but assumes that those entities are systems

Examples:
``` sh
# Displays the details of the system with UID "c996897fa60d4882a720292171debb5a"
av cat c996897fa60d4882a720292171debb5a
av cat /systems/c996897fa60d4882a720292171debb5a

# Displays the details of the operation with UID "650db7d9ab4547d4b2e7f8e8cb70f7e2"
av cat /operations/650db7d9ab4547d4b2e7f8e8cb70f7e2

# Displays the details of all systems with a commStatus equals to "ERROR"
av find -commStatus ERROR --uid-only | av cat

# Displays the details of all users
av ls /users --uid-only | av cat /users
```

The two last commands show how to use the _find_ (or _ls_) and _cat_ commands in pipeline. The standard output produced by the _find_ (or _ls_) commands can be directly used by the _cat_ command as long you have used the _--uid-only_ argument to produced only plain text UIDs.

Edit an entity
--------------

You can edit entities with the `av vi` command. In a nutshell, this command retrieves the entity details and opens the `vi` editor to edit them. When exiting the editor, the data will automatically be sent to the server in order to update the entity.

> av vi _arg_

_arg_ can be:
* /_entities_/_uid_: edit the details of the specified entity
* /_uid_: idem, but assumes that the entity is a system

This command outputs the entity details _after_ the update, or the error message if something bad happened on server-side.

Examples:
``` sh
# Edits the system with UID "c996897fa60d4882a720292171debb5a"
av vi c996897fa60d4882a720292171debb5a
av vi /systems/c996897fa60d4882a720292171debb5a

# Edits the gateway with UID "590fea92135a46e9acc7b59952843ec9"
av vi /gateways/590fea92135a46e9acc7b59952843ec9
```

Delete an entity
--------------

It possible to delete entities from the AirVantage M2M Cloud portal by using the `av rm` command.

> av rm _arg_

_arg_ can be:
* /_entities_/_uid_: delete the specified entity
* /_uid_: idem, but assumes that the entity is a system

**Warning:** No confirmation will be prompted before actually deleting the entity. You should be sure of what you're doing when invoking this command!

Examples:
``` sh
# Deletes the system with UID "c996897fa60d4882a720292171debb5a"
av rm c996897fa60d4882a720292171debb5a
av rm /systems/c996897fa60d4882a720292171debb5a

# Removes the gateway with UID "590fea92135a46e9acc7b59952843ec9"
av rm /gateways/590fea92135a46e9acc7b59952843ec9
```

Users CSV export example
------------------------

By mixing `av` commands with standard **Shell** ones, you have a powerful way to manipulate AirVantage M2M Cloud API.

For example, if you want to export a list of AirVantage users in a CSV file, the following script will do the trick:
``` sh
# CSV header
echo "UID;EMAIL;NAME;PHONE_NUMBER;COMPANY_UID;COMPANY_NAME" > out.csv

# CSV content
av ls /users --uid-only  \
    | av cat /users  \
    | sed '/^$/d'  \
    | while read line;  \
      do  \
          csv_line="`echo $line | jq '.uid, .email, .name, .phoneNumber, .company.uid, .company.name' | tr '\n' ';'`";  \
          echo "${csv_line%?}";  \
      done >> out.csv

# Have a look at the wonderful generated CSV file
cat out.csv
```

And that's all! Simple as that! With the `av` script, you can do whatever you want. The only limit is your imagination (and the known limitations listed below  ;) ).

Known limitations
-----------------

1. OAuth API client informations (client ID and secret key) are both hard-coded in the `av` script. This API client only exists on _qa-trunk_ platform. Those informations should actually be asked by the _av ssh_ command.
2. All `av` commands only work with API that return a list of items identified by a UID (unique identifier). It does not work with other entities, such as labels.
3. Pagination is not handled yet by the `av` script. Default pagination will thus be used when accessing to any API (i.e. no offset, and up to 100 items returned).

What's coming next?
-------------------

Two main topics should be addressed soon:

1. Remove the limitations described above.
2. Implement commands to create or remove systems, to create operations, etc.
