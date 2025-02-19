# pipelinewise-tap-s3-csv

[![PyPI version](https://badge.fury.io/py/pipelinewise-tap-s3-csv.svg)](https://badge.fury.io/py/pipelinewise-tap-s3-csv)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/pipelinewise-tap-s3-csv.svg)](https://pypi.org/project/pipelinewise-tap-s3-csv/)
[![License: MIT](https://img.shields.io/badge/License-GPLv3-yellow.svg)](https://opensource.org/licenses/GPL-3.0)

This is a [Singer](https://singer.io) tap that reads data from files located inside a given S3 bucket and produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

This is a [PipelineWise](https://transferwise.github.io/pipelinewise) compatible tap connector.

## How to use it

The recommended method of running this tap is to use it from [PipelineWise](https://transferwise.github.io/pipelinewise). When running it from PipelineWise you don't need to configure this tap with JSON files and most of things are automated. Please check the related documentation at [Tap S3 CSV](https://transferwise.github.io/pipelinewise/connectors/taps/s3_csv.html)

If you want to run this [Singer Tap](https://singer.io) independently please read further.

### Install and Run

First, make sure Python 3 is installed on your system or follow these
installation instructions for [Mac](http://docs.python-guide.org/en/latest/starting/install3/osx/) or
[Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-ubuntu-16-04).

It's recommended to use a virtualenv:

```bash
  python3 -m venv venv
  pip install pipelinewise-tap-s3-csv
```

or

```bash
  python3 -m venv venv
  . venv/bin/activate
  pip install --upgrade pip
  pip install .
```

### Configuration

Here is an example of basic config, and a bit of a run down on each of the properties:

    ```json
    {
        "aws_access_key_id": "ACCESS_KEY",
        "aws_secret_access_key": "SECRET_ACCESS_KEY",
        "start_date": "2000-01-01T00:00:00Z",
        "bucket": "tradesignals-crawler",
        "tables": [{
            "search_prefix": "feeds",
            "search_pattern": ".csv",
            "table_name": "my_table",
            "key_properties": ["id"],
            "delimiter": ","
        }]
    }
    ```

- **start_date**: This is the datetime that the tap will use to look for newly updated or created files, based on the modified timestamp of the file.
- **account_id**: This is your AWS account id
- **role_name**: In order to access a bucket, the tap uses boto3 to assume a role in your AWS account. If you have your AWS account credentials set up locally, you can specify this as a role which your local user has access to assume, and boto3 should by default pick up your AWS keys from the local environment.
- **bucket**: The name of the bucket to search for files under.
- **external_id**: (potentially optional) Running this locally, you should be able to omit this property, it is provided to allow the tap to access buckets in accounts where the user doesn't have access to the account itself, but is able to assume a role in that account, through a shared secret. This is that secret, in that case.
- **tables**: An escaped JSON string that the tap will use to search for files, and emit records as "tables" from those files. Will be used by a [`voluptuous`](https://github.com/alecthomas/voluptuous)-based configuration checker.

The `table` field consists of one or more objects, JSON encoded as an array and escaped using backslashes (e.g., `\"` for `"` and `\\` for `\`), that describe how to find files and emit records. A more detailed (and unescaped) example below:

```
[
    {
        "search_prefix": "exports"
        "search_pattern": "my_table\\/.*\\.csv",
        "table_name": "my_table",
        "key_properties": "id",
        "date_overrides": "created_at",
        "delimiter": ","
    },
    ...
]
```

- **search_prefix**: This is a prefix to apply after the bucket, but before the file search pattern, to allow you to find files in "directories" below the bucket.
- **search_pattern**: This is an escaped regular expression that the tap will use to find files in the bucket + prefix. It's a bit strange, since this is an escaped string inside of an escaped string, any backslashes in the RegEx will need to be double-escaped.
- **table_name**: This value is a string of your choosing, and will be used to name the stream that records are emitted under for files matching content.
- **key_properties**: These are the "primary keys" of the CSV files, to be used by the target for deduplication and primary key definitions downstream in the destination.
- **date_overrides**: Specifies field names in the files that are supposed to be parsed as a datetime. The tap doesn't attempt to automatically determine if a field is a datetime, so this will make it explicit in the discovered schema.
- **delimiter**: This allows you to specify a custom delimiter, such as `\t` or `|`, if that applies to your files.

A sample configuration is available inside [config.sample.json](config.sample.json)

---

Based on Stitch documentation

