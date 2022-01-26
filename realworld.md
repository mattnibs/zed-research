# JQ

Here we are compiling a list of finding real world problems that perhaps zq 
can solve.

Sources:
- List of most popular jq tags on stackoverflow https://stackoverflow.com/questions/tagged/jq?tab=Votes
- https://codefaster.substack.com/p/jq-features-considered-harmful
- https://www.moschetti.org/rants/jqvspy.html

### Converting an js object into an array

https://stackoverflow.com/questions/70771975/converting-a-list-of-objects-to-an-array

*input:*

```json
{
  "alpha": {
    "bytes": {
      "value": 4789440
    },
    "doc_count": 7723
  },
  "beta": {
    "bytes": {
      "value": 4416862639
      },
    "doc_count": 1296
  }
}
```

*output:*

```
[
  {
    "key": "alpha",
    "bytes": {
      "value": 4789440
    },
    "doc_count": 7723
  },
  {
    "key": "beta",
    "bytes": {
      "value": 4416862639
    },
    "doc_count": 1296
  }
]
```

jq:

```
jq '[to_entries[] | {key} + .value]' input.json
```

ZQ needs to add over for records to get this working.

### Filter an object array based of values in inner array

https://stackoverflow.com/questions/26701538/how-to-filter-an-array-of-objects-based-on-values-in-an-inner-array-with-jq

357 Votes

Filter on objects where Names does not have a value containing "data" in it.

*input:*
```
[
  {
    "Id": "cb94e7a42732b598ad18a8f27454a886c1aa8bbba6167646d8f064cd86191e2b",
    "Names": [
      "condescending_jones",
      "loving_hoover"
    ]
  },
  {
    "Id": "186db739b7509eb0114a09e14bcd16bf637019860d23c4fc20e98cbe068b55aa",
    "Names": [
      "foo_data"
    ]
  }
]
```

*output:*
```
cb94e7a42732b598ad18a8f27454a886c1aa8bbba6167646d8f064cd86191e2b
```

jq solution:

```
. - map(select(.Names[] | contains ("data"))) | .[] .Id
```

zq:

```
let id = Id over Names => ( *data* | yield id )
```

### Merging 2 objects from two files

https://stackoverflow.com/questions/19529688/how-to-merge-2-json-objects-from-2-files-using-jq

173 Votes

An interesting question in the comments:

> "If the individual files are sorted by a key, is it possible to preserve the
order in the resulting file?"

file1:
```
{
    "value1": 200,
    "timestamp": 1382461861,
    "value": {
        "aaa": {
            "value1": "v1",
            "value2": "v2"
        },
        "bbb": {
            "value1": "v1",
            "value2": "v2"
        },
        "ccc": {
            "value1": "v1",
            "value2": "v2"
        }
    }
}
```

file2:
```
{
    "status": 200,
    "timestamp": 1382461861,
    "value": {
        "aaa": {
            "value3": "v3",
            "value4": 4
        },
        "bbb": {
            "value3": "v3"
        },      
        "ddd": {
            "value3": "v3",
            "value4": 4
        }
    }
}
```

output:
```
{
    "value": {
        "aaa": {
            "value1": "v1",
            "value2": "v2",
            "value3": "v3",
            "value4": 4
        },
        "bbb": {
            "value1": "v1",
            "value2": "v2",
            "value3": "v3"
        },
        "ccc": {
            "value1": "v1",
            "value2": "v2"
        },
        "ddd": {
            "value3": "v3",
            "value4": 4
        }
    }
}
```

The accepted answer is: `jq -s '.[0] * .[1]' file1 file2`. Note the `-s` argument
is needed to convert the inputs into a single array.

This allows the use of an arbitrary amount of objects:

```
echo '{"A": {"a": 1}}' '{"A": {"b": 2}}' '{"B": 3}' |\
  jq --slurp 'reduce .[] as $item ({}; . * $item)'
```

### first, last, nth

`echo [2,4,6] | jq 'last'` returns as you would expect `6`, but `echo [2,4,6] | jq last(.)`
returns the entire array `[2,4,6]`.

Normally `first` operates on arrays but with an expression they operate on the
json values that come out of the expression. Weird.

```
echo '{"posts": [{"title": "Frist psot", "author": "anon"}, {"title": "A well-written article", "author": "person1"}], "realnames": {"anon": "Anonymous Coward", "person1": "Person McPherson"}}' |
jq '.realnames as $names | .posts | map({title, author: $names[.author]})'
```

The author doesn't like this because it's hard to unittest, and says you should
use python for this reason. Not sure I buy it.

### Googling

> Typically I use jq to filter and process the JSON output into submission until I get what I want. But if you’re anything like me, you spend a lot of time googling how to do what you want in jq because the syntax can get a little out of hand. In fact, I keep notes with example jq queries I’ve used before in case I need those techniques again.

https://blog.kellybrazil.com/2020/03/25/jello-the-jq-alternative-for-pythonistas/

### Groupby users in ndjson list of tweets

This is from a jq tutorial by Matthew Lincoln https://programminghistorian.org/en/lessons/json-and-jq#extracting-user-data. The author attempting to use jq to extract a csv table of twitter
users from a ndjson list of tweets: https://programminghistorian.org/assets/jq_twitter.json

The jq for doing this:

```
group_by(.user)
| .[] 
| {user_id: .[0].user.id, user_name: .[0].user.screen_name, user_followers: .[0].user.followers_count, tweet_ids: [.[].id | tostring] | join(";")}
```

Note the gross use of `.[0]` everywhere to select each json object.

The zq for this:

```
tweet_ids:=collect(id) by user_id:=user.id, user_name:=user.screen_name, user_followers:=user.followers_count
```
