## What is it

CLI tool to patch Hasura metadata `json` file with needed objects or with another Hasura metadata file. You can use it to deploy complex CI/CD flows for applications, which are using Hasura on a backend.

## Why is it useful

If you use different environments, you likely have different webhooks in each environment. With this tool you can describe needed metadata for each webhook separately and finally merge them all when you need to deploy your release. Such approach gives an ability to several people to develop different webhooks independently.

## Requirements

- Python
- `requirements.txt`

## How does it work

### Syntax 

```shell
python main.py -r remote_schemas -r actions -r custom_types -r event_triggers -s dev_metadata.json -m prod_metadata.json -o out.json
```
Find all the syntax by `python main.py --help`

### Supported operations

#### Merge
Default mode. Use it to merge needed Hasura objects from a mixin file to a source metadata file.

#### Replace `-r`

Use it to define, which objects should be replaced in a source metadata file by objects from a mixin. 

For example, if a mixin file is another Hasura metadata file and you call

```shell
python main.py -r event_triggers -s dev_metadata.json -m prod_metadata.json -o out.json
```

then:
1. All the event triggers for all the tables in `dev_metadata.json` will be removed.
2. New event triggers from `prod_metadata.json` will be inserted instead.
3. All the other metadata from `dev_metadata.json` remains untouched and the result metadata goes to `out.json`.

### Typical Hasura metadata release flow

1. Export metadata in `json` format from Hasura in dev environment, which should be deployed to production.
2. Export metadata in `json` format from Hasura in current production environment.
3. Run patcher with source `json` file from dev and mixin file from production.
4. Run patcher with output `json` file from previous step and mixin files with new Hasura objects to deploy.
5. The last output file is your new Hasura metadata for production environment. 

Points 1-3 are needed to migrate new tables and permissions from dev environment, since they are not supported by this patcher for now.

## Supported Hasura objects

* [x] Event triggers
* [x] Remote schemas
* [x] Actions
* [x] Custom types
* [ ] Tables
* [ ] Table permissions

## Hasura objects format to patch

### Event trigger

```json
{
    "type": "event_trigger",
    "object": {
        "name": "myEventTrigger",
        "table": {
            "schema": "core",
            "name": "some_table"
        },
        "definition": {
            "enable_manual": false,
            "insert": {
                "columns": "*"
            },
            "update": {
                "columns": []
            }
        },
        "retry_conf": {
            "num_retries": 0,
            "interval_sec": 10,
            "timeout_sec": 60
        },
        "webhook": "https://mywebhook.url",
        "config": "1.2.3",
        "headers": [{
            "name": "token",
            "value": "%TOKEN_ENV%"
        }],
        "comment": "0.1.0"
    }
}
```

### Action

```json
{
  "type": "action",
  "object": {
      "name": "myAction",
      "definition": {
        "handler": "https://mywebhook.url",
        "output_type": "myOutputType",
        "headers": [
          {
            "name": "Authorization",
            "value_from_env": "AUTHORIZATION_HEADER"
          }
        ],
        "arguments": [
          {
            "name": "value",
            "type": "String!"
          }
        ],
        "type": "mutation",
        "kind": "synchronous"
      },
      "permissions": [
        {
          "role": "user"
        }
      ],
      "comment": "0.1.0"
  }
}
```

### Custom type

```json
{
    "type": "custom_type",
    "object": {
      "name": "myCustomType",
      "fields": [
        {
          "name": "affected_rows",
          "type": "Int!"
        }
      ]
    }
}
```

### Remote schema

```json
{
    "type": "remote_schema",
    "object": {
      "name": "myRemoteSchema",
      "definition": {
        "url": "https://myschema.url/graphql",
        "timeout_seconds": 60,
        "forward_client_headers": false,
        "headers": [
        {
          "name": "Authorization",
          "value_from_env": "AUTHORIZATION_HEADER"
        }
      ]
    },
      "comment": "0.1.0"
    }
}
```
