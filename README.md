# oascli

oascli is a CLI tool for OpenAPI schema based Service, acting as a universal client to manage web services complying with OpenAPI specification. Besides being used standalone, it can run as MCP in both stdio and http modes to integrate web services with AI agents.

<!-- more -->

### Get Started

Download latest oascli from [https://github.com/ITimothyTrue/oascli/releases/tag/v1.0.1](https://github.com/ITimothyTrue/oascli/releases/tag/v1.0.1)

Note: if your box is Mac OS, you may meet with oascli not being trusted when opening it. Go to 'System Settings | Privacy & Security', scroll down and trust it from the 'Security' section, then re-open oascli.

```
# oascli -h
usage: oascli [-h|--help] [-c|--config "<value>"] [-m|--mode "<value>"]
              [-l|--address "<value>"] [-s|--statement "<value>"
              [-s|--statement "<value>" ...]] [-v|--version]

              OpenAPI schema based Service CLI

Arguments:

  -h  --help       Print help information
  -c  --config     Config file path. Default: ~/.oascli.json
  -m  --mode       mode to work as MCP: stdio, http
  -l  --address    address for http mode. Default: 0.0.0.0:7908
  -s  --statement  Statement to execute. Multiple statements are executed
                   sequentially
  -v  --version    Print version details

```

#### Normal Setup

1. add OpenAPI schema and check if the schema is parsed successfully

```
schema add schema1 /path-to/local-openapi-schema.json

schema parse schema1
```

2. add auth if the web service requires authentication

```
auth add auth1 ...
```

3. add service and check the functions are parsed successfully

```
svc add svc1 schema1 auth1

svc functions svc1
svc parse svc1
```

4. operate the remote service

```
func sample function1
func call function1
```

5. work as MCP

```stdio
{
    "mcpServers": {
        "oascli": {
            "command": "/path-to/oascli",
            "args": [
                "-m",
                "stdio"
            ]
        }
    }
}
```

```http
// start oascli in http mode
oascli -m http

{
    "mcpServers": {
        "oascli": {
            "type": "http",
            "url": "http://localhost:7908/mcp"
        }
    }
}
```

### Commands

oascli comes with an intuitive help command:

```
>> help
func                         Function operations
    get                      Get function details. Arguments: <function> [function ...]
    sample                   Generate a json payload for a function, or save it locally. Arguments: <function> [file]
    call                     Invoke a function. Arguments: <function> [json/file]
    curl                     Generate a curl command. Arguments: <function> [json/file]

auth                         Manage authentications
    add                      Add a new authentication. Arguments: <auth-id> <type> <properties>
    get                      Get authentication details. Arguments: <auth-id> [auth-id ...]
    ls                       List authentications. Arguments: [regex]
    rm                       Remove an authentication. Arguments: <auth-id> [auth-id ...]

schema                       Manage openapi schemas
    add                      Add a new openapi schema. Arguments: <schema-id> <URL/file> [method] [auth-id]
    get                      Get schema details. Arguments: <schema-id> [schema-id ...]
    ls                       List schemas. Arguments: [regex]
    rm                       Remove a schema. Arguments: <schema-id> [schema-id ...]
    parse                    Parse schemas. Arguments: [schema-id ...]

svc                          Manage web services
    add                      Add a new web service. Arguments: <svc-id> <address> <schema-id> [auth-id] [prefix]
    get                      Get service details. Arguments: <svc-id> [svc-id ...]
    ls                       List services. Arguments: [regex]
    rm                       Remove a service. Arguments: <svc-id> [svc-id ...]
    functions                List functions of services. Arguments: [svc-id ...]
    parse                    Parse services. Arguments: [svc-id ...]
    prefix                   Update a service prefix. Arguments: <svc-id> <prefix>
    disable                  Disable services for MCP modes. Arguments: <svc-id> [svc-id ...]
    enable                   Enable services for MCP modes. Arguments: <svc-id> [svc-id ...]

Local file operations
cat                          Display the contents of a file on local machine. Arguments: <file>
cd                           Change working directory on local machine. Arguments: [dir]
info                         Get file details on local machine. Arguments: <dir/file>
ls                           List file, directory or regex matched items. Arguments: [dir/file/regex]
mkdir                        Make a directory on local machine. Arguments: <dir>
pwd                          Show the current directory on local machine
rm                           Remove a file, directory or regex matched items. Arguments: <dir/file/regex>

prop                         Manage properties
    ls                       List properties. Arguments: [key-pattern]
    keys                     Show property keys. Arguments: [key-pattern]
    count                    Show count of properties. Arguments: [key-pattern]
    get                      Get value of a property. Arguments: <key>
    set                      Set value for a property. Arguments: <key> <value>
    unset                    Unset a property value. Arguments: <key>

quit                         Exit this program. Same as bye, exit
help                         Show help

```

> Arguments inside <> are required.

> Arguments inside [] are optional.

> When typing a command, tips will show up for easy usage. Use &lt;tab&gt; key to select from the tip list.

### Properties

Properties are used to tune performance. 

```
>> help prop
prop                         Manage properties
    ls                       List properties. Arguments: [key-pattern]
    keys                     Show property keys. Arguments: [key-pattern]
    count                    Show count of properties. Arguments: [key-pattern]
    get                      Get value of a property. Arguments: <key>
    set                      Set value for a property. Arguments: <key> <value>
    unset                    Unset a property value. Arguments: <key>
```

#### Cache

Cache can be enanbled to improve processing performance by not parsing schemas per request.

| Property | Description | Example |
|:---------|:------------|:---------|
| cache-schemas | enable cache for schemas | true or false, default value is true |

### Features

#### Authentication

4 types of authentications are supported: basic, oauth, mtls and token.

```
auth add <auth-id> basic <username> <password>

auth add <auth-id> oauth <cert-url> <client-id> <client-secret>

auth add <auth-id> mtls <key-pem> <cert-pem> [trust-pem]

auth add <auth-id> token <key> <token> <header/query>
```

#### Schema

A schema refers to an OpenAPI schema content, either from a remote URL or a local file.

```
schema add <schema-id> <url/file> [method] [auth]

// schema from a local file
schema add schema-1 /path-local-file

// schema from GET a remote URL with no auth
schema add schema-2 https://host...

// schema from a remote URL requiring auth
schema add schema-3 https://host... post auth-1
```

#### Service

A web service provides functionalities by following an OpenAPI schema. Each endpoint or method is defined as a function.

> A service can have its own prefix to identify its functions in case different services provide same functions.

```
svc add <svc-id> <address> <schema-id> [auth-id] [prefix]
```

#### Function

A function is implicitly an endpoint (method) from a web service.

```
// get a function signature
func get function-name

// generate a sample 
func sample function-name

// invoke remote method
func call function-name 'json-payload'
```

#### Regular Expression

Some commands support regular expressions. Following are some examples:

```
^A          Names starting with A
c$          Names ending with c
\.txt       Files ending with .txt
group.*     Same as .*group.*
\d          Match a digit 0-9
\s          Match a whitespace
(?i)        Prefix for case insensitive, like (?i)Abc
```
