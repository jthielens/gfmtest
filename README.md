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
    --export-pass <PASSWORD>   Password to encrypt generated passwords
    --generate-pass            Generate Passwords for users
    --help
 -i,--input <FILE>             input file YAML, JSON or CSV
 -k,--insecure                 Disable https security checks
 -p,--password <PASSWORD>      Password
    --profile <PROFILE>        Connection profile to use
    --remove                   Remove profile
    --save                     Save/update profile
 -u,--username <USERNAME>      Username
    --update                   Updates existing objects
    --url <URL>                VersaLex url
$
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
    status: "success"
    message: "found user you"
  id: "PMHTT7QjTg6EMyoxIQ15DA"
  username: "you"
  email: "you@yours.com"
  alias: "Your User Group"
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
    status: "success"
    message: "generated passwords"
    passwords:
    - alias: "Users"
      username: "testUser"
      email: "testUser@test.com"
      password: "FEWSH_77121|denco+13057"
    - alias: "Users"
      username: "testUser2"
      email: "testUser2@test.com"
      password: "LJBXI_99080-orpug-12738"
```

The passwords can then be communicated to the users out of band. For an additional layer of security in handling these credentials, the generated passwords may be encrypted by an _export password_, in which case the passwords are encrypted and encoded in base64 in the results file:

```
- result:
    status: "success"
    message: "generated passwords"
    passwords:
    - alias: "Users"
      username: "testUser"
      email: "testUser@test.com"
      password: "U2FsdGVkX18KWfsFAdh3rQzFNE6d5noCbBd3cqiJu1Yw8oUgvPBCXomRne+ZqbAl"
    - alias: "Users"
      username: "testUser2"
      email: "testUser2@test.com"
      password: "U2FsdGVkX18yY07hCqwg+xn3st+KwKDqFr3BWRYE9NulzBirPgRHK4TFE4XENcc+"
```

The encryption format is suitable for decryption using [openssl](https://wiki.openssl.org/index.php/Binaries) as follows:

```
$ openssl aes-256-cbc -d -a <<< "U2FsdGVkX18KWfsFAdh3rQzFNE6d5noCbBd3cqiJu1Yw8oUgvPBCXomRne+ZqbAl"
enter aes-256-cbc decryption password:
LJBXI_99080-orpug-12738
```

## [&LessLess;](#-password-generation-) Configuration Reference [&GreaterGreater;](#-next-) ##

Command Line              | Connector         | Description
--------------------------|-------------------|------------
--url <URL>               | Url               | The Harmony URL, e.g. `https://localhost:6080`
-u, --username <USERNAME> | User              | The user authorized to use the Harmony API
-p, --password <PASSWORD> | Password          | The user's password
--generate-pass           | Generate Password | Select to enable password generation for created users
--export-pass <PASSWORD>  | Export Password   | Password used to encrypt generated passwords in the results file
-k, --insecure            | Ignore TLS Checks | Select to bypass TLS hostname and trusted issuer checks
--profile <PROFILE>       |                   | The named profile to load instead of "default"
--save                    |                   | Select to create/update named profile (or "default")
--remove                  |                   | Select to remove named profile (or "default")


## [&LessLess;](#-configuration-reference-) Next [&GreaterGreater;](#-more-) ##

