avop.sh
=======

The avop.sh project aims to provides a Shell script that enables to interact with AirVantage M2M Cloud API in command-line.

Prerequisites
-------------

### AirVantage account

Depending on your location, you need an account on one of AirVantage M2M Cloud datacenters:
* https://na.airvantage.net (North America)
* https://eu.airvantage.net (EMEA)

### jq command-line JSON processor

This project requires the  *jq* command-line JSON processor, which is available at http://stedolan.github.io/jq/

To install it, you only need to:

1. Download the appropriate binaries for your system from http://stedolan.github.io/jq/download/
2. Put these binaries in your *PATH*.

Installing the **avop** script
------------------------------

The installation procedure is simple as downloading the script, and include it in your *PATH*.

This can be done with the following command lines:

``` sh
git clone https://github.com/bmiegemolle/avop.sh.git
export PATH=`pwd`/avop.sh/scripts:${PATH}
```
You're now ready to use the **avop** script! Enjoy!    

Get an access token
-------------------

You can get an access token to AirVantage M2M Cloud API with the _avop ssh_ command:

> avop ssh _host_

Example:
``` sh
avop ssh na.airvantage.net
```

You'll be asked for your username and password, and you'll get an access token to AirVantage M2M Cloud API. This token will be stored in your file system (in the _/tmp/avop-access-token_ file), and it will be automatically used by any further **avop** commands.

Who am I?
---------

You can retrieve the details of the currently loggued user with the _avop whoami_ command:

> avop whoami

Example:
``` sh
avop whoami
```

List entities
-------------

The _avop ls_ command will enable you to list some entities of the AirVantage M2M Cloud portal:

> avop ls [/entities] [--uid-only]

* _/entities_ is optional. If not specified, then _systems_ will be listed.
* _--uid-only_ argument enables to return only a list of entities uid as plain text, instead of entities as JSON objects.

Examples:
``` sh
# List all systems, and display them as a list of JSON objects
avop ls
avop ls /systems

# List all users, and display them as a list of plain text UIDs
avop ls /users --uid-only

# List all operations, and display them as a list of JSON objects
avop ls /operations
```

Find entities
-------------

The _avop find_ command will enable you to search some entities of the AirVantage M2M Cloud portal:

> avop find _-field_ _value_ [/entities] [--uid-only]

* _-field value_ enables you to specify a search criteria.
* _/entities_ is optional. If not specified, then _systems_ will be searched.
* _--uid-only_ argument enables to return only a list of entities uid as plain text, instead of entities as JSON objects.

Examples:
``` sh
# Find systems which name contains "test", and display them as a list of JSON objects
avop find -name test

# Find systems with a commStatus equals to "ERROR", and display them as a list of plain text UIDs
avop find -commStatus ERROR --uid-only

# Find users whose name contains "admin", and display them as a list of JSON objects
avop find -name admin /users

# Find operations created by the user "administrator@m2mop.net", and display them as a list of JSON objects
avop find -user administrator@m2mop.net /operations
```

Get entities details
--------------------

Using the _avop cat_ command, you can retrieve the details of a specific entity of the portal:

> avop cat [_arg_]

_arg_ can be:
* /_entities_/_uid_: display the details of the specified entity
* /_uid_: idem, but assumes that the entity is a system
* /_entities_: entities UIDs are retrived from the standard input
* *empty*: entities UIDs are also retrived from the standard input, but assumes that those entities are systems

Examples:
``` sh
# Displays the details of the system with UID "c996897fa60d4882a720292171debb5a"
avop cat c996897fa60d4882a720292171debb5a
avop cat /systems/c996897fa60d4882a720292171debb5a

# Displays the details of the operation with UID "650db7d9ab4547d4b2e7f8e8cb70f7e2"
avop cat /operations/650db7d9ab4547d4b2e7f8e8cb70f7e2

# Displays the details of all systems with a commStatus equals to "ERROR"
avop find -commStatus ERROR --uid-only | avop cat

# Displays the details of all users
avop ls /users --uid-only | avop cat /users
```

The two last commands show how to use the _find_ (or _ls_) and _cat_ commands in pipeline. The standard output produced by the _find_ (or _ls_) commands can be directly used by the _cat_ command as long you have used the _--uid-only_ argument to produced only plain text UIDs.

Known limitations
-----------------

1. OAuth API client informations (client ID and secret key) are both hard-coded in the **avop** script. This API client only exists on _qa-trunk_ platform. Those informations should actually be asked by the _avop ssh_ command.
2. All avop commands only work with API that return a list of items identified by a UID (unique identifier). It does not work with other entities, such as labels.
3. Pagination is not handled yet by the **avop** script. Default pagination will thus be used when accessing to any API (i.e. no offset, and up to 100 items returned).

What's coming next?
-------------------

Two main topics should be addressed soon:

1. Remove the limitations described above.
2. Implement commands to create or edit systems, to create operations, etc.
