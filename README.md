# Diff for Elasticsearch

**Warning: This is a work-in-progress. Things might break without warning.**

The `esdiff` tool iterates over two indices in Elasticsearch 5.x or 6.x
and performs a diff between the documents in those indices.

It does so by scrolling over the indices. To allow for a stable sort
order, it uses `_id` by default (`_uid` in ES 5.x). You can provide
a different sort field by using the `-sort` option.

## Example usage

First, we need to setup two Elasticsearch clusters for testing,
then seed a few documents.

```
# Create an Elasticsearch 5.x and 6.x cluster, available at
# http://localhost:19200 and http://localhost:19201
$ mkdir -p data
$ docker-compose -f docker-compose.infra.yml up -d

# Add some documents
$ ./seed/01.sh

# Compile
$ go build
```

Let's make a simple diff:

```
$ ./esdiff 'http://localhost:19200/index01/tweet' 'http://localhost:19201/index01/_doc'
Deleted	2
Updated	3	{*diff.Document}.Source["message"]:
	-: "Playing the piano is fun as well"
	+: "Playing the guitar is fun as well"
```

Use JSON as output format instead. Together with
[`jq`]()
and
[`jiq`]()
this is quite powerful.

```
$ ./esdiff -o json 'http://localhost:19200/index01/tweet' 'http://localhost:19201/index01/_doc' | jq 'select(.mode | contains("deleted"))'
{
  "mode": "deleted",
  "_id": "2",
  "src": {
    "_id": "2",
    "_source": {
      "message": "Running is fun",
      "user": "olivere"
    }
  },
  "dst": null
}
```

You can also pass a query to filter the source and/or the destination,
using the `-sf` and `-df` args respectively:

```
$ ./esdiff -o json -sf='{"term":{"user":"olivere"}}' 'http://localhost:19200/index01/tweet' 'http://localhost:19201/index01/_doc' | jq .
{
  "mode": "unchanged",
  "_id": "1",
  "src": {
    "_id": "1",
    "_source": {
      "message": "Welcome to Golang",
      "user": "olivere"
    }
  },
  "dst": {
    "_id": "1",
    "_source": {
      "message": "Welcome to Golang",
      "user": "olivere"
    }
  }
}
{
  "mode": "deleted",
  "_id": "2",
  "src": {
    "_id": "2",
    "_source": {
      "message": "Running is fun",
      "user": "olivere"
    }
  },
  "dst": null
}
```

Use `-h` to display all options:

```
$ esdiff -h
General usage:

	esdiff [flags] <source-url> <destination-url>

General flags:
  -df string
    	Raw query for filtering the destination, e.g. {"term":{"name.keyword":"Oliver"}}
  -o string
    	Output format, e.g. json
  -sf string
    	Raw query for filtering the source, e.g. {"term":{"user":"olivere"}}
  -size int
    	Batch size (default 100)
  -sort string
    	Sort field, e.g. _id or name.keyword
```

## License

MIT. See [LICENSE](https://github.com/olivere/esdiff/blob/master/LICENSE).