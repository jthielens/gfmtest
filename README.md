# README #


## Overview [&GreaterGreater;](#-getting-started-) ##

This connector implements a batch API for manipulating the Harmony API using
a file request/response model.
It is packaged as a Harmony connector, suitable
for use as a custom _Other Folder_ for portal users, allowing users to upload
batch request files and download corresponding result files.
It is also packaged as a command line utility in the form of an executable
java jar.

In both packages, the utility uses the REST API and must be configured with
a server url, username and password.
The connector stores this information securely in the connection profile.
The command line utility maintains a set of connection profiles in the
`$HOME/.cic/profiles` file.

Request files maybe either in YAML/JSON format or in CSV format. The YAML/JSON
format consists of a collection of requests based on the native Harmony
[REST API](https://developer.cleo.com/api/getting-started/overview.html) with
extensions to indicate the operation (`add`, `update`, `list`, `delete`).

There are five separate CSV templates, one for each of the following object types:

* Authenticators (user groups)
* Users
* AS2 connections
* SFTP connections
* FTP connections

The CSV templates use specific column headings to map to object properties using
a simpified, flattened approach compared with the full JSON object schema.
The CSV formats can be easier to work with for bulk imports, but for full access
to all object properties the YAML/JSON format is required.

## [&LessLess;](#overview-) Getting Started [&GreaterGreater;](#-password-generation-) ##

### Harmony Connector [&gt;](#-command-line-)

To use the Harmony Connector you must install the connector package, restart Harmony, and then configure the connector for use.

1. Obtain the connector package `connector-batchapi-5.6-SNAPSHOT-distribution.zip`.
2. Install the connector in your Harmony:<br/>&bull; `Harmonyc -i connector-batchapi-5.6-SNAPSHOT-distribution.zip`.
3. Restart Harmony.
4. Log back into the Harmony admin. From the Hosts tab add a new host:<br/>&bull; select `Connections`&rarr;`Generic`&rarr;`Generic BatchAPI`<br/>&bull; right-click `Clone and Activate`<br/>&bull; Click `Ok` and `Done`.
5. Setup your new `BatchAPI` host:<br/>&bull; On the `BatchAPI` tab set the url (e.g. `https://localhost:6080`), User, Password and Working Directory. The Working Directory is where result files are stored, and in a clustered setup must be shared between the Harmony servers in the cluster.
6. Test your new connection:<br/>&bull; In the `send` action use the single command `PUT test.yaml`<br/>&bull; create a request file `test.yaml` as shown [below](#getting-started-test-file) in your outbox directory.<br/>&bull; Run the action. You should see a `test.results.yaml` file in your working directory.


### [&lt;](#harmony-connector-) Command Line [&gt;](#-getting-started-test-file-)

The command line package does not require installation, but simply installation
of the prerequisites (Java 8&mdash;openjdk8 is fine on [Windows](https://access.redhat.com/documentation/en-us/openjdk/8/html/openjdk_8_for_windows_getting_started_guide/getting_started_with_openjdk_for_windows) or [Linux](https://openjdk.java.net/install/) or [Mac OS](https://github.com/AdoptOpenJDK/homebrew-openjdk)) and the executable jar
`connector-batchapi-5.6-SNAPSHOT-commandline.jar`.

If Java and the command line jar are properly installed, the following will work:

```
$ java -jar connector-batchapi-5.6-SNAPSHOT-commandline.jar --help
usage: com.cleo.labs.connector.batchapi.processor.Main
    --help
    --url <URL>                VersaLex url
 -k,--insecure                 Disable https security checks
 -u,--username <USERNAME>      Username
 -p,--password <PASSWORD>      Password
 -i,--input <FILE>             input file YAML, JSON or CSV
    --generate-pass            Generate Passwords for users
    --export-pass <PASSWORD>   Password to encrypt generated passwords
    --operation <OPERATION>    default operation: list, add, update, delete or preview
    --include-defaults         include all default values when listing connections
    --template <TEMPLATE>      load CSV file using provided template
    --profile <PROFILE>        Connection profile to use
    --save                     Save/update profile
    --remove                   Remove profile
```

You can provide all the connection parameters (`url`, `username` and `password`) on the command line, but this is both inconvenient and insecure. Instead it is preferable to setup a profile for each Harmony server you need to work with.

To create a profile, use:

```
java -jar connector-batchapi-5.6-SNAPSHOT-commandline.jar --url https://192.168.7.22 -u administrator -p Admin --save
```

This will create a default profile in `$HOME/.cic/profiles` that looks like:

```
---
default:
  url: "http://192.168.7.22"
  insecure: false
  username: "administrator"
  password: "Admin"
  exportPassword: null
```

Add the `--profile name` option to select a profile name other than `default`. Using named profiles you can save as many profiles as you need. You can also edit the `profiles` file directly, taking care to preserve its simple YAML format. In fact, since using passwords in command lines is insecure, it is recommended to edit the passwords manually in `profiles`. If you use the command line to create the profiles initially, it is better to use dummy passwords for subsequent replacement through manual edits.

When running the utility, the `default` profile will be loaded by default, unless an alternate profile is specified with `--profile name`. Any additional connection options supersede the profile values.

To verify your profile, process a simple [test file](#getting-started-test-file) with the `-i` option:

```
java -jar connector-batchapi-5.6-SNAPSHOT-commandline.jar -i test.yaml
```

The results will be written to the standard output.

The special input filename `-` represents the standard input (use `-i ./-` if you really have a file named `-`). This can be used for a kind of shorthand query syntax like

```
java -jar connector-batchapi-5.6-SNAPSHOT-commandline.jar -i - <<< '{"operation":"list","username":"bob"}'
```

or even more compactly

```
java -jar connector-batchapi-5.6-SNAPSHOT-commandline.jar --operation list -i - <<< 'username:bob'
```

You may use `-i` multiple times to supply a sequence of input files to be processed. Keep in mind that while a single input file can only be in one of the six supported formats (YAML/JSON or CSV for one of the five CSV object types), you can mix and match input file formats when using multiple  `-i`.

### [&lt;](#-command-line-) Getting Started Test File [&GreaterGreater;](#-password-generation-)

Use this test file, edited as instructed, to verify your installation as described above. The expected test result file follows.

#### test.yaml ####

Use your own username or some other known username.

```
---
operation: list
type: user
username: you
```

#### test.results.yaml ####

```
---
- result:
    status: success
    message: found user you
  id: PMHTT7QjTg6EMyoxIQ15DA
  username: you
  email: you@yours.com
  authenticator: Your User Group
```

The result attributes will vary depending on the user details recorded. Note that default values for a user are suppressed from the results file.

## [&LessLess;](#-getting-started-) Password Generation [&GreaterGreater;](#-configuration-reference-) ##

When creating users, an initial value for the password must be provided (whether this initial password must be reset when the user logs in for the first time is controlled by the `accept.security.passwordRules.requirePasswordResetBeforeFirstUse` property of the authenticator, identified by the `alias` in the user creation request&mdash;see the API documentation for [Authenticators (Native User)](https://developer.cleo.com/api/api-reference/post-authenticators-native-user.html)).

The batch API utility includes an option for generating random passwords instead of having them supplied in the input file. Generated passwords have the following structure:

* 5 randomly selected upper case letters
* 1 randomly selected separator (from a set of 8 possible separators)
* 5 randomly selected digits
* 1 randomly selected separator
* 5 randomly selected lower case letters
* 1 randomly selected separato
* 5 randomly selected digits

An example generated password might be `FEWSH_77121|denco+13057`. This format ensures that most length and complexity requirements can be met, while also providing over 89 bits of entropy.

Passwords are not included in the result record for generated users. Instead, an additional result block is appended to the results file:

```
- result:
    status: success
    message: generated passwords
    passwords:
    - authenticator: Users
      username: testUser
      email: testUser@test.com
      password: FEWSH_77121|denco+13057
    - authenticator: Users
      username: testUser2
      email: testUser2@test.com
      password: LJBXI_99080-orpug-12738
```

The passwords can then be communicated to the users out of band. For an additional layer of security in handling these credentials, the generated passwords may be encrypted by an _export password_, in which case the passwords are encrypted and encoded in base64 in the results file:

```
- result:
    status: success
    message: generated passwords
    passwords:
    - authenticator: Users
      username: testUser
      email: testUser@test.com
      password: U2FsdGVkX18KWfsFAdh3rQzFNE6d5noCbBd3cqiJu1Yw8oUgvPBCXomRne+ZqbAl
    - authenticator: Users
      username: testUser2
      email: testUser2@test.com
      password: U2FsdGVkX18yY07hCqwg+xn3st+KwKDqFr3BWRYE9NulzBirPgRHK4TFE4XENcc+
```

The encryption format is suitable for decryption using [openssl](https://wiki.openssl.org/index.php/Binaries) as follows:

```
$ openssl aes-256-cbc -d -a <<< "U2FsdGVkX18KWfsFAdh3rQzFNE6d5noCbBd3cqiJu1Yw8oUgvPBCXomRne+ZqbAl"
enter aes-256-cbc decryption password:
LJBXI_99080-orpug-12738
```

## [&LessLess;](#-password-generation-) Configuration Reference [&GreaterGreater;](#-request-processing-) ##

This table describes the options that control the Batch API utility, both in its command line and Harmony connector packagings. Note that the profile management options are available for the command line only&mdash;in Harmony you create separate connections of type `BatchAPI` to achive the same effect.

Command Line                    | Connector         | Description
--------------------------------|-------------------|------------
--url <URL>                     | Url               | The Harmony URL, e.g. `https://localhost:6080`
-u, --username &lt;USERNAME&gt; | User              | The user authorized to use the Harmony API
-p, --password &lt;PASSWORD&gt; | Password          | The user's password
-i, --input &lt;FILE&gt;        | `PUT` file        | input file YAML, JSON or CSV
--generate-pass                 | Generate Password | Select to enable password generation for created users
--export-pass &lt;PASSWORD&gt;  | Export Password   | Password used to encrypt generated passwords in the results file
--operation &lt;OPERATION&gt;   | Default Operation | The default operation for entries lacking an explicit "operation"
-k, --insecure                  | Ignore TLS Checks | Select to bypass TLS hostname and trusted issuer checks
--profile &lt;PROFILE&gt;       | &nbsp;            | The named profile to load instead of "default"
--include-defaults              | &nbsp;            | Include all default values when listing connections
--template &lt;TEMPLATE&gt;     | &nbsp;            | load CSV file using provided template
--save                          | &nbsp;            | Select to create/update named profile (or "default")
--remove                        | &nbsp;            | Select to remove named profile (or "default")


## [&LessLess;](#-configuration-reference-) Request Processing [&GreaterGreater;](#-csv-files-and-templates) ##

### Requests [&gt;](#-results-)

Regardless of the input format (YAML/JSON or CSV), the input file is processed as a sequence of requests.
Each request has an _operation_ and an _object type_ to operate on.
Operations are typically performed on single objects referenced by the object name.
The underlying API uses `alias` to represent the object name, except for users who are identified by `username`&mdash;
the batch utility uses a type-specific _Identifier_ to name objects of different types.
Operations supporting sets of objects (list and delete) may specify a filter string in place of a specific object name&mdash;the filter expressions are passed directly to the underlying API, so make sure to use the appropriate `alias` or `username` attribute in filters.

Object Type | Description | Identifier | Meta Type
------------|-------------|------------|-----
Authenticator | A container for users that defines many properties, including folder structure and security properties (see the [API reference](https://developer.cleo.com/api/api-reference/post-authenticators-native-user.html)) | `authenticator` | `authenticator`
User        | A user who logs in to Harmony using FTP, SFTP, or through the https Portal | `username` | `user`
Connection  | A connection to a server over FTP, SFTP, or AS2 | `connection` | `connection`

Each object type is managed with its own endpoint in the underlying Harmony API, so the batch utility must be able to determine which endpoint to use for a given request. To do this it uses a combination of the object name and object type, depending on the request operation.

Each object type corresponds to a _meta type_ in the API (`meta.type`). Objects of type `authenticator` and `connection` also have a specific `type`:

Object Type   | Meta Type       | Type            | Description
--------------|-----------------|-----------------|------------
Authenticator | `authenticator` | `nativeUser`    | Users defined natively in the Harmony user table
&nbsp;        | &nbsp;          | `systemLdap`    | Users defined in the System LDAP directory
&nbsp;        | &nbsp;          | `authConnector` | Users defined through an authentication connector
Connection    | `connection`    | `as2`           | A connection to an AS2 partner
&nbsp;        | &nbsp;          | `sftp`          | A connection to an SFTP server
&nbsp;        | &nbsp;          | `ftp`           | A connection to an FTP server
&nbsp;        | &nbsp;          | _many others_   | See [here](https://developer.cleo.com/api/api-reference/resource-connections.html) for a list of many other supported connection types
User          | `user`          | `user`          | Users don't have a type, only a meta type

The batch utility uses a meta-type-specific tag for the object name, e.g. `username: name` for users, `connection: name` for connections, and `authenticator: name` for authenticators. For operations involving existing objects (i.e. anything other than `add`), the specific type does not need to be provided. For `add` operations, both the name of the new object and its specific type are required. 'add' and 'update' operations must also supply an _Entity Body_, the details of the object to be created or updated. For `list` and `delete` operations the entity body is ignored.

Operation | Description               | Filter Supported | Entity Body
----------|---------------------------|:----------------:|:-----------:
list      | List existing object(s)   | &check;          | &nbsp;
add       | Create a new object       | &nbsp;           | &check;
update    | Update an existing object | &check;          | &check;
delete    | Delete existing object(s) | &check;          | &nbsp;

The default operation is `add`, unless this is overridden with the `--operation <OPERATION>` argument for the command line utility. In any case, if an operation is specified in a request it is honored over the default.

The `list` and `delete` operations may be applied to sets of objects identified by a [filter](https://developer.cleo.com/api/getting-started/overview.html#filter). If a `filter` is provided in a request, the `type` is also required and the meta-type-specific name tag is prohibited.

For example, to list a specific SFTP connection, use (note that `type`, while permitted, is not required, but if supplied, it must match the type of the named object):

```
---
- operation: list
  connection: "mysftp"
```

and to query for all SFTP connections, use (note that `connection` is prohibited and `type` is required):

```
---
- operation: list
  type: connection
  filter: type eq "sftp"
```

Again, remember to use `alias` in filter expressions if needed for a `connection` or `authenticator` filter. The following two requests are equivalent:

```
---
- operation: list
  type: connection
  filter: alias eq "mysftp"
- operation: list
  connection: "mysftp"
```

### [&lt;](#requests-) Results [&gt;](#-action-handling-)

Each request produces one or more results.
The results are formatted into a YAML collection where each entry has a `result` object:

```
---
- result:
    status: success or error
    message: description of the success or error
    optional additional result information...
  additional object information...
```

#### `list` operations

Single object requests (without `filter`) generate results whose _additional object information_ describes the object found:

```
---
- result:
    status: success
    message: found user edie
  id: 9ybMFuRpRjqRVJmE5HE5aQ
  username: edie
  email: edie@cleo.demo
  authenticator: Users
```

`filter` requests generate one result per object found with the count "m of n" in the message, also filling in _additional object information_:

```
- result:
    status: success
    message: found connection loopback sftp (1 of 2)
  id: JOe0WtAxTFS7q3fd0mdg2Q
  type: sftp
  connect:
    host: mysftp.cleo.demo
    port: 10022
    username: user1
  outgoing:
    storage:
      outbox: outbox/
    partnerPackaging: false
  incoming:
    storage:
      inbox: inbox/
    partnerPackaging: false
  connection: mysftp1
- result:
    status: success
    message: found connection vagrant sftp (2 of 2)
  id: WFZUCRdQSp-eVXuT-C7b0w
  type: sftp
  connect:
    host: mysftp.cleo.demo
    port: 22
    username: user2
  outgoing:
    storage:
      outbox: outbox/
    partnerPackaging: false
  incoming:
    storage:
      inbox: inbox/
    partnerPackaging: false
  connection: mysftp2
```

`list` operations for authenticators implicitly include all the user objects grouped under that authenticator, resulting in one additional result for each user found. These users are identified with result messages like "found authenticator Users: user m of n".

`list` operations, with or without `filter`, generate an appropriate error result if no matching object(s) is/are found.

#### `add` operations

Add requests generate results whose _additional object information_ describes the object created.

```
---
- result:
    status: success
    message: created test
  id: f9dz4JRZQcOpHbm-OvCwig
  username: test
  email: test@cleo.demo
  authenticator: Users
```

'add' operations generate an appropriate error result if an object is not created. If the batch utility does not encounter an error preparing the request, the error message is generated by the Harmony API itself.

#### `update` operations

Successful updates produce two separate results.
The first result, identified with a message like "updating user alice", includes a representation of the object before any updates were applied, exactly as if it had been produced by a `list` operation.
The second result, identified with a message like "user alice updated", includes a representation of the updated object.

Note that unlike `list` operations, `update` operations on authenticators do not affect users, so the additional results produced by `list` for these nested users are not included for `update`.

Bulk update requests may be applied to sets of objects using a `filter` in the request instead of naming a specific object. Any attributes provided in the request are merged/overlaid with the existing object attributes. The reported results appear in before/after pairs as described above.

#### `delete` operations

Results for `delete` requests are very similar to those for `list` requests, except that the objects listed are deleted with a result message like "deleted user alice". The results can be replayed as `add` requests to restore the deleted objects.

> Note that the batch utility is currently unable to list passwords, whether password hashes for users or encrypted passwords for connections, due to limitations of the underlying Harmony REST API. So while the `delete` results can be replayed as `add` requests, new passwords will have to be generated or the old passwords will need to be added from a another source.

Bulk delete requests may be applied to sets of objects using a `filter` in the request instead of naming a specific object. One result it reported for each object deleted, with a result message like "deleted user alice (m of n)".

### [&lt;](#-results-) Action Handling [&gt;](#-certificate-handling)

In the native Harmony API, actions are a separate resource type, linked to connections, authenticators, and users through `_links`. The batch utility simplifies this processing by treating the set of actions for an object as a separate object nested within the parent object itself:

```
- username: testUser
  email: testUser@test.com
  authenticator: Users
  home:
    dir:
      override: local/root/run/
  actions:
    connectTest:
      alias: connectTest
      commands:
      - GET -DEL *
      - LCOPY -REC %inbox% %inbox%/in
    other:
      alias: other
      commands:
      - # other commands here
```

In the embedded `actions` object, each action is represented as a sub-object whose attribute name is the same as the action's `alias` (if the attribute name and `alias` disagree, the `alias` is used). A `list` operation for any object will render any linked actions as an embedded `actions` property as illustrated above.

On update, actions in the request are matched up against existing actions on the object by `alias`:

* any request actions not appearing in the existing object are added
* any request actions matching `alias` with existing actions are updated
* any existing actions with no matching `alias` in the request are left unchanged

In order to delete an existing action, create an action with the matching `alias` in the request, adding the property `operation: delete`:

```
- username: testUser
  email: testUser@test.com
  authenticator: Users
  home:
    dir:
      override: local/root/run/
  actions:
    other:
      alias: other
      opreation: delete
      commands:
      - # other commands here
```

### [&lt;](#-action-handling-) Certificate Handling [&gt;](#-csv-files-and-templates)

Like actions, certificates in the native Harmony API are handled as a separate linked resource.

The batch utility will be updated to handle certificates as nested objects, but today certificates must be handled outside of the utility.

## [&LessLess;](#-request-processing-) CSV Files and Templates

In many cases involving batch operations, most parts of each request, or at least the request skeleton, are the same.
The detais for each request can then conveniently be represented in tabular form.

The batch utility supports this mode of operation using CSV files for the tabular data and templates in YAML format.
The CSV file must have a header, which defines replacement token names for the columns of the table.
The YAML template encodes requests, much as illustrated above, but with replacement tokens that fill in values from the CSV file, using the replacement token names from the CSV header.

For example, a simple `add` user request:

```
---
- username: alice
  password: password
  email: alice@cleo.demo
  authenticator: Users
```

could be generalized with replacement tokens for username, password, and email with a CSV file:

```
user,pass,email
alice,password,alice@cleo.demo
bob,password,bob@cleo.demo
```

against the template:

```
---
- username: ${user}
  password: ${pass}
  email: ${email}
  authenticator: Users
```

resulting in the same effect as the following explicit YAML request file:

```
---
- username: alice
  password: password
  email: alice@cleo.demo
  authenticator: Users
- username: bob
  password: password
  email: bob@cleo.demo
  authenticator: Users
```

Provide a template using the `--template` command line option and `--input` for the CSV file.

### Built-in templates

If you provide a CSV file for `--input` and do not provide an explicit `--template`, the batch utility will attempt to use one of its built-in templates based on an analysis of the content:

* files with a `UserAlias` column in the header use the `authenticator` template
* files without a `type` column in the header use the `user` template
* files with a `type` column whose data values are all `as2` use the `as2` template
* files with a `type` column whose data values are all `sftp` use the `sftp` template
* files with a `type` column whose data values are all `ftp` use the `ftp` template
* files with a `type` column whose data values are not all the same cause an error

The built-in templates support the following header columns. You may include them in your CSV file in any order, but keep in mind that the `UserAlias` and `type` columns are essential to the template selection process:

authenticator   | user              | as2               | sftp              | ftp
----------------|-------------------|-------------------|-------------------|----
&nbsp;          | &nbsp;            | type              | type              | type
UserAlias       | Host              | alias             | alias             | alias
&nbsp;          | &nbsp;            | url               | host              | host
&nbsp;          | &nbsp;            | &nbsp;            | port              | port
&nbsp;          | UserID            | AS2From           | username          | username
&nbsp;          | Password          | AS2To             | password          | password
FTP             | WhitelistIP       | Subject           | &nbsp;            | channelmode
SSHFTP          | &nbsp;            | encrypted         | &nbsp;            | activelowport
HTTP            | &nbsp;            | signed            | &nbsp;            | activehighport
Access          | Email             | receipt           | &nbsp;            | &nbsp;
FolderPath      | DefaultHomeDir    | receipt_sign      | &nbsp;            | &nbsp;
HomeDir         | CustomHomeDir     | receipt_type      | &nbsp;            | &nbsp;
DownloadFolder  | &nbsp;            | inbox             | inbox             | inbox
UploadFolder    | &nbsp;            | outbox            | outbox            | outbox
OtherFolder     | OtherFolder       | &nbsp;            | &nbsp;            | &nbsp;
ArchiveSent     | &nbsp;            | sentbox           | sentbox           | sentbox
ArchiveReceived | &nbsp;            | receivedbox       | receivedbox       | receivebox
&nbsp;          | CreateCollectName | CreateSendName    | CreateSendName    | CreateSendName
&nbsp;          | ActionCollect     | ActionSend        | ActionSend        | ActionSend
&nbsp;          | Schedule_Send     | Schedule_Send     | Schedule_Send     | Schedule_Send 
&nbsp;          | CreateReceiveName | CreateReceiveName | CreateReceiveName | CreateReceiveName
&nbsp;          | ActionReceive     | ActionReceive     | ActionReceive     | ActionReceive
&nbsp;          | Schedule_Receive  | Schedule_Receive  | Schedule_Receive  | Schedule_Receive
&nbsp;          | action&lowbar;<i>alias</i>&lowbar;name     |action&lowbar;<i>alias</i>&lowbar;name | action&lowbar;<i>alias</i>&lowbar;name | action&lowbar;<i>alias</i>&lowbar;name
&nbsp;          | action&lowbar;<i>alias</i>&lowbar;commands | action&lowbar;<i>alias</i>&lowbar;commands | action&lowbar;<i>alias</i>&lowbar;commands | action&lowbar;<i>alias</i>&lowbar;commands
&nbsp;          | action&lowbar;<i>alias</i>&lowbar;schedule |action&lowbar;<i>alias</i>&lowbar;schedule |action&lowbar;<i>alias</i>&lowbar;schedule | action&lowbar;<i>alias</i>&lowbar;schedule
&nbsp;          | HostNotes         | &nbsp;            | &nbsp;            | &nbsp;

The user and connection templates provide fixed slots for two actions. The action&lowbar;<i>alias</i>&lowbar;xxx columns allow for an arbitrary number of additional actions to be defined. The action alias is taken from the `name` column: the _alias_ portion in the column name can be the same alias, but is really used only to match up the `name`, `commands` and `schedule` columns for the action (and to keep the sets of columns for additional actions distinct from each other&mdash;you may not have multiple columns with the same header name).

A few columns can be multi-valued:

* `WhitelistIP`: multiple IP addresses separated by `;`
* `OtherFolder`: multiple custom folder paths separated by `;`
* action&lowbar;<i>alias</i>&lowbar;commands and other action command script columns: multiple commands separated by `;` or `|`

As a convenience action&lowbar;<i>alias</i>&lowbar;schedule and other action schedule columns accept the shorthand `polling` for the official API schedule `on file continuously`.

Built-in templates can be used for only a single object type per file and a single request per row. You can construct your own templates that can use conditionals and token expressions to create multiple object types from a single CSV, or can create multiple requests per row.

### Advanced template features

#### JSON and YAML

Keep in mind that the Harmony API operates on JSON data. JSON builds three simple concepts into arbitrarily rich data structures. You can think of a JSON data structure as a tree-structured collection of nodes:

* scalar nodes&mdash;strings, numbers (integers and floating point), booleans (true/false) and `null`:<br/>`'string'` or `"string"`, `25`, `3.14`, `true`, `false`, `null`, ...
* array nodes&mdash;lists of other nodes, including scalars, objects, and other arrays:<br/>`[node, node, node, ...]`
* object nodes&mdash;sets of named fields, where the names are strings and the values are other nodes, including scalars, arrays, and other objects:<br/>`{"name":node,"name":node,"name":node,...}`

YAML provides an optional alternate representation for arrays and objects, which replaces the `[]` and `{}` syntax with indentation (it also makes the `'` and `"` enclosing strings optional is most cases). Arrays are represented as nodes introduced by `-` at the same indentation level. Objects are represented by `name:` at the same indentation level. Nested arrays and objects are indicated by increasing indenting (by spaces&mdash;tabs are prohibited in YAML) instead of potentially confusing groups of e.g. `{{[{}]}}`.

YAML arrays `[1,2,3]` and `[{"user":"bob","age":42},{"user":"amy","special":true,"details":{"reason":"happy"}}]`:

```
- 1
- 2
- 3
```

```
- user: bob
  age: 42
- user: amy
  special: true
  details:
    reason: happy
```

#### Token replacement

The most basic template feature is token replacement. Any field name or scalar value in the tree is scanned for `${token}` blocks, and these are replaced by the corresponding token (or nothing, if the token is not defined or its cell is empty in the CSV file). Multiple `${token}` may appear in a single field name or value. So if `a` is `lions` and `b` is `lambs`:

> `This example shows ${a} and ${b} together` &rarr; `This example shows lions and lambs together`

Whenever a `${token}` appears with other text or tokens, it is converted into a string. If a `${token}` appears all by itself in a value as a _singleton_, in some cases it is necessary to make sure it is rendered as a scalar of a specific type. This can be achieved by appending `:int`, `:boolean` or `:string` to force the appropriate interpretation.

#### JavaScript expressions

In fact the `token` is more accurately described as a JavaScript expression. Every column value is converted into a JavaScript variable (whose name is taken from the column header), so `${a}` is really just evaluating the `a` variable in the JavaScript engine. But other expressions may be used as needed:

> `${a.toUpperCase()}` &rarr; `LIONS`
> 
> `${a.length}` &rarr; `5`

#### Manipulating tree structure

##### Conditionals: `${if:`

The template expander supports _conditional_ expressions, evaluating a JavaScript expression before expanding a portion of the template. For conditional expansion use the special `${if:expression}` singleton as a field name in the template. If the expression evaluates to a value that is considered true-ish, the value attached to that special field is merged into the parent context.

For `${if` the following are considered true-ish:

* a boolean value of true
* any non-empty string other than `no`, `none`, `na` or `false` (case insensitive)
* any non-zero integer (or non-0.0 floating point number)
* JavaScript `null` is considered `false`

If the condition is satisfied, the value could be any of scalar, object, or array. How this value is merged depends on the parent context of the `${if` field:

* if the `${if` was an element of a parent array, a scalar or object value is added to the parent array, and all elements of any array value are added as (possibly) multiple entries in the parent array (a nested array is _not_ created&mdash;if this is what you need you must wrap the result array in another array).
* if the `${if` was a field of an object, the value _must_ be an object, and that objects fields are merged into the parent object. Array and scalar values lack any kind of field name under which to merge the values into the parent.

An array example:

```
---
- ${if:true}:
  - 1
  - 2
- ${if:true}: 3
- ${if:false}: 4
- ${if:true}:
    result: 5
- 6
```

produces

```
---
- 1
- 2
- 3
- result: 5
- 6
```

An object example:

```
---
${if:true}:
  field1: 1
  field2: 2
${if:true&&true}:
  field3: 3
fixed4: 4
${if:false}:
  result: 5
fixed6: 6
```

produces (note the `${if:true&&true}` since a second `${if:true}` would overwrite the first one):

```
---
field1: 1
field2: 2
field3: 3
fixed4: 4
fixed6: 6
```

##### Loops: `${for:`

The template expander supports looping over template fragments based on either:

* splitting a column value, typically on some separator (see `WhitelistIP` or action commands)
* matching multiple column headers, for example the action&lowbar;<i>alias</i>&lowbar;xxx columns

Like a conditional, a loop is indicated by a special singleton expression in a field name:

* `${for identifier:expression}` to loop over an expression using column values
* `${for column identifier:regex}` to loop over column names matching `regex`

Processing is similar to a conditional, except that the value node of the `${for:` field is evaluated once for every value of the `expression` or every match of the `regex`. In these evaluations, a new JavaScript variable is injected into the JavaScript engine using the selected `identifier` (so make sure the chosen `identifier` does not mask one of your column headings). Merging of the results follows the same rules as for conditional values.

The `expression` should be an array expression: the most usual case is something like `${for id:value.split(/;/)}` to separate a multi-valued column value into its parts. Constant arrays can also be used like `${for id:[1,2,3]}`.

The `regex` may have a capture group to map a match subportion of the column name to the bound `identifier`. If no capture groups are defined, the whole column name is used. Only the first capture group is used. For example, the action&lowbar;<i>alias</i>&lowbar;xxx columns are processed in the built-in templates as follows:

```
- connection: ${alias}
  ... lots of template goes here ...
  actions:
    ${for column action:action_([^_]+)_alias}:
      ${eval('action_'+action+'_alias')}:
        alias: ${eval('action_'+action+'_alias')}
        commands:
        - ${for command:eval('action_'+action+'_commands').split(/;\|/)}: ${command}
        schedule: ${s=eval('action_'+action+'_schedule');s=='polling'?'on file continuously':s}
```

##### Null pruning

As with conditionals, any object field whose value evaluates to `null` is omitted from the template expansion. Any array that then ends up empty or object that ends up with no fields is also omitted. These omissions can propagate up the template tree to omit entire branches if the leaf nodes are not expanded.

For example, in this fragment from the built-in user template:

```
---
- username: ${username}
  ...
  home:
    subfolders:
      default:
      - ${for other:OtherFolder.split(';')}:
          usage: other
          path: ${other}
```

the entire `subfolders` branch will be pruned if `OtherFolder` is blank or null. But in this example from the authenticator template:

```
---
type: nativeUser
...
  home:
  subfolders:
    default:
    - ${if:DownloadFolder}:
        usage: download
        path: ${DownloadFolder}
```

the `usage: download` field will prevent the entry in the `default:` array from being pruned, which will then prevent the entire `default:` array and possibly the `subfolders:` field from being pruned. So the `${if:DownloadFolder}:` conditional is needed to force the needed pruning.

#### Testing your template

Use `--operation preview` to test the effects of your template on your CSV data. No API calls will be made to the Harmony API, but the requests will be converted to YAML and displayed directly in the result output with the message "request preview".
