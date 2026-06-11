# oascli

oascli is a CLI tool for OpenAPI schema based Service, acting as a universal client to manage web services complying with OpenAPI specification. Besides being used standalone, it can run as MCP in both stdio and http modes to support AI agents working with web services.

<!-- more -->

### Get Started

Download latest oascli from [https://github.com/ITimothyTrue/oascli/releases/tag/v1.0.3](https://github.com/ITimothyTrue/oascli/releases/tag/v1.0.3)

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
schema func-ls schema1
```

2. add service and authentication

```
svc add svc1 <server-addr> schema1

svc auth add svc1 basic <user> <passwd>
```

3. operate the remote service

```
func ls svc1

func call <function>
```

4. run as MCP server

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
# start oascli in http mode
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
schema                       Manage openapi schemas
    add                      Add a new openapi schema. Arguments: <schema-id> <URL/file> [method]
    get                      Get schema details. Arguments: <schema-id> [schema-id ...]
    ls                       List schemas. Arguments: [regex]
    rm                       Remove a schema. Arguments: <schema-id> [schema-id ...]
    prefix                   Update a schema prefix. Arguments: <schema-id> <prefix>
    disable                  Disable or show disabled schemas for MCP modes. Arguments: [schema-id ...]
    enable                   Enable or show enabled schemas for MCP modes. Arguments: [schema-id ...]
    auth                     Manage authentication for a schema
    func                     Manage schema functions

svc                          Manage web services
    add                      Add a new web service. Arguments: <svc-id> <address> <schema-id> [prefix]
    get                      Get service details. Arguments: <svc-id> [svc-id ...]
    ls                       List services. Arguments: [regex]
    rm                       Remove a service. Arguments: <svc-id> [svc-id ...]
    prefix                   Update a service prefix. Arguments: <svc-id> <prefix>
    disable                  Disable or show disabled services for MCP modes. Arguments: [svc-id ...]
    enable                   Enable or show enabled services for MCP modes. Arguments: [svc-id ...]
    param                    Manage common parameters
    auth                     Manage authentication for a service
    func                     Manage service functions

auth                         Manage authentications
    add                      Add a new authentication. Arguments: <auth-id> <type> <properties>
    get                      Get authentication details. Arguments: <auth-id> [auth-id ...]
    ls                       List authentications. Arguments: [regex]
    rm                       Remove an authentication. Arguments: <auth-id> [auth-id ...]

mcp                          MCP helper operations
    load                     Load functions. Arguments: <schema/svc> [id ...]
    unload                   Load functions. Arguments: <schema/svc> [id ...]
    ls                       List loaded functions. Arguments: <schema/svc> [id ...]
    get                      Get function details. Arguments: <function> [function ...]

Local file operations
cat                          Display the contents of a file on local machine. Arguments: <file>
cd                           Change working directory on local machine. Arguments: [dir]
info                         Get file details on local machine. Arguments: <dir/file>
ls                           List file, directory or regex matched items. Arguments: [dir/file/regex]
mkdir                        Make a directory on local machine. Arguments: <dir>
pwd                          Show the current directory on local machine
rm                           Remove a file, directory or regex matched items. Arguments: <dir/file/regex>

quit                         Exit this program. Same as bye, exit
help                         Show help

```

> Arguments inside <> are required.

> Arguments inside [] are optional.

> When typing a command, tips will show up for easy usage. Use &lt;tab&gt; key to select from the tip list.

### Features

#### Authentication

4 types of authentications are supported: basic, oauth, mtls and token. Authentication is mainly required for service access; schema may also require authentication if it's hosted remotely.

```
svc auth add <svc1> basic <username> <password>

svc auth add <svc1> oauth <cert-url> <client-id> <client-secret>

svc auth add <svc1> mtls <key-pem> <cert-pem> [trust-pem]

svc auth add <svc1> token <key> <token> <header/query>
```

#### Schema

A schema refers to an OpenAPI schema definition, either from a remote URL or a local file.

```
schema add <schema-id> <url/file> [method] [auth]

// schema from a local file
schema add schema-1 /path-local-file

// schema from GET a remote URL with no auth
schema add schema-2 https://host...

// schema from a remote URL requiring auth
schema add schema-3 https://host... post
schema auth add oauth <cert-url> <client-id> <client-secret>
```

#### Service

A web service provides functionalities by following an OpenAPI schema. Each endpoint or method is defined as a function.

> A service can have its own prefix to identify its functions in case different services provide same functions.

```
svc add <svc-id> <address> <schema-id> [prefix]
```

#### Function

A function is implicitly an endpoint (method), from either a schema or service.

```
// list functions of a service
svc func ls <schema-id>

// get a function signature
svc func get <schema-id> function-name

// generate a sample 
svc func sample <schema-id> function-name

// invoke remote method
svc func call <schema-id> function-name 'json-payload'
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
