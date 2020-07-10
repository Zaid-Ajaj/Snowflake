# Snowflaqe [![Build status](https://ci.appveyor.com/api/projects/status/ulq0vfun1ij7ix58?svg=true)](https://ci.appveyor.com/project/Zaid-Ajaj/snowflaqe) [![Nuget](https://img.shields.io/nuget/v/Snowflaqe.svg?colorB=green)](https://www.nuget.org/packages/Snowflaqe)

A dotnet CLI tool to work with GraphQL queries. It allows for static query *verification* and advanced *type checking* against a remote or local schema as well as generating *type-safe* clients for F# and [Fable](https://fable.io/) (more generation targets in the future).

## Installation
Install as a global dotnet CLI tool
```
dotnet install snowflaqe -g
```
## Using The Tool
Create a JSON file called `snowflaqe.json` with the following shape:
```
{
    "schema": "<schema>",
    "queries: "<queries>",
    "project": "<project>",
    "output": "<output>"
    ["errorType"]: <custom error type>,
    ["target"]: <target>
}
```
Where
 - `<schema>` can be either the URL to the GraphQL backend; a relative path to another JSON file containing the output of the [standard Introspection](https://github.com/Zaid-Ajaj/Snowflaqe/blob/master/src/Introspection.gql) query, which you can execute against the backend yourself (this allows for offline verification and type-checking); or a file with a `.gql` or `.graphql` extension containing a schema. 
 - `<queries>` is an absolute or relative path to a directory that contains `*.gql` files that contain individual GraphQL queries that `snowflaqe` will run the verification against.
 - `<project>` is the name of the project will be generated.
 - `<output>` is an absolute or relative path to a directory where the project will be generated
 - `<errorType>` optional custom error type to be generated. See below to learn more.
 - `<target>` optional the code-generation target which can either be `fable` (default), `fsharp` or `shared`.

> Using `shared` as a code generation target actually builds 3 projects! One contains just the types and can be *shared* across platforms. The other two reference this shared projects and implmenent Fable specific client and dotnet specific client, respectively.

After creating the configuration file. You can `cd` your way to where you have the config file and run:
```
snowflaqe
```
which will by default only do static query verification and static type-checking against the `<schema>`. You can also *reference* the configuration file in another directory via a relative path:
```
snowflaqe --config ./src/snowflaqe.json
```
> In this case, the file doesn't necessarily have to be called `snowflaqe.json`.

## Generate Client Project
```
snowflaqe --generate

snowflaqe --config ./path/to/snowflaqe.json --generate
```
Will generate a full project in the designated `output` directory.

## Custom Error Type

By default, the error type that is generated in the global types looks like this:
```fs
type ErrorType = { message: string }
```
This type is important because every request you make to the GraphQL backend returns `Result<Query, ErrorType list>` but the errors that come back are usually determined by the backend and not exposed through the schema. That is why you can customize this error type using the `errorType` configuration element:
```
{
    "schema": "<schema>",
    "queries: "<queries>",
    "project": "<project>",
    "output": "<output>",
    "errorType": {
        "CustomErrorType": {
            "Message": "string"
            "Path": "string list"
            "RequestId": "string"
        }
    }
}
```
which will generate:
```fs
type CustomErrorType = {
    Message: string
    Path: string list
    RequestId: string
}
```
## Not Supported

There are a couple of features of the GraphQL specs which `snowflaqe` doesn't (yet) know how to work with:
 - [ ] GraphQL Union Types
 - [ ] Subscriptions (non-goal of the tool, maybe in the future)
