# File Sideload

File Sideload adds two media ingesters: `sideload` and `sideload_dir`.

`sideload` is used for ingesting files one at a time. It requires one property: `ingest_filename`, which is the name of a file within the configured sideload directory to ingest. In the REST API, using the sideload ingester would look like:

```json
{
    "o:ingester": "sideload",
    "ingest_filename": "example.jpg"
}
```

`sideload_dir` is used for ingesting an entire folder at once. It requires one property: `ingest_directory`, which is the name of a directory to ingest. You can also set `ingest_directory_recursively` to true to recursively ingest all the contents of the chosen directory and its subdirectories. In the REST API, using the sideload_dir ingester would look like:

```json
{
    "o:ingester": "sideload_dir",
    "ingest_directory": "example_dir",
    "ingest_directory_recursively": true
}
```
