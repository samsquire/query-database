# Query Database

Working with databases, we spend a lot of time marshalling data around to get the data structure we can actually process against. What if whenever you queried your database, you got the data structure you want?

Is this even possible to?

 * automatically transform documents from structure A to structure B
 * is it possible to optimise a data structure based on the intended functions to run against it?

# Backstory

You're talking to some API, it gives you data back in the following format:

```
{
    "head": {
        "vars": [
            "a",
            "b",
            "c"
        ]
    },
    "results": {
        "bindings": [
            {
                "a": {
                    "type": "uri",
                    "value": "http://samsquire.com/CLIAutoComplete"
                },
                "b": {
                    "type": "uri",
                    "value": "http://samsquire.com/card"
                },
                "c": {
                    "type": "uri",
                    "value": "http://samsquire.com/IdeaHome"
                }
            },
            {
                "a": {
                    "type": "uri",
                    "value": "http://samsquire.com/ContentManagementSystem"
                },
                "b": {
                    "type": "uri",
                    "value": "http://samsquire.com/card"
                },
                "c": {
                    "type": "uri",
                    "value": "http://samsquire.com/IdeaHome"
                }
            }
        ]
    }
}
```

You would rather the following data structure:

```
{
	a: [],
	b: [],
	c: []
}

```

I can write the transformation code for this myself but should I want to? Is it possible to handle this transformation automatically, when I query the database?

I would like to push this transformation to the database layer and have it handle the transformation for me. The database then can consider my requested data structure and give me the data I need in the most efficient way it can create it.

```
var database = new QueryDatabase();

var query = new Query(data) { 
	return data.reduce(function (pullout, current) {
      if (current.a)
      pullout.a.push(current.a.value);
      if (current.b)
      pullout.b.push(current.b.value);
      if (current.c)
      pullout.c.push(current.c.value);
      return pullout;
    }, {a: [], b: [], c: []} ));
}

database.add(query);
database.addJson(data);
```

# Optimising Data Structures

Imagine you have the following data structure:

```
data = [
    {
        "name":  "Friend",
        "age":  25,
        "locations": [
            {
                "name": "Home",
                "lat": 54.33353535,
                "lon": 23.133333337
            }],
        "shared": [
            {
                "type": "file",
                "path": "/friends file",
                "key": "E2B06046-F4DB-47BD-A8BF-9B425C016EC5"
            },
            {
                "type": "file",
                "path": "/workerjs",
                "key": "B77045A5-CCCB-49D9-9962-5C5DC96B1698"
            },
            {
                "type": "file",
                "path": "/somethingelse",
                "key": "B12C0007-97FF-41D4-AF04-F00F5039D930"
            },
            {
                "type": "resource",
                "resourceShared": "diskspace",
                "key": "B12C0007-97FF-42D4-AF04-F00F5039D930",
                "storageShared": "500gb"
            },
            {
                "type": "resource",
                "resourceShared": "vpnaccess",
                "key": "B12C0007-97FF-42D4-AF04-F00F5039D935"
            }
        ]
    }
]
```

Most of the time, you are querying for objects with type file in the shared array. You find all your queries include the equivalent of:

```
data.map(function (item) {
	return item.shared.filter(function (shared) { 
		return shared.type === 'file';
	});
})
```

Can the structure of the functions against the data structure give us clues how the data is structured so that we can transform the data structure into one that the resulting data structure is easier and cheaper to query? For example, why can't we move the shared file objects into their own array? We wouldn't need to filter anymore.

![Automated query optimisation](https://raw.github.com/samsquire/query-database/master/transformation.png "Scrolling down a spiral")

In the above example, we could pull up the items in the shared array and put them into a files array.

# Document transformation

Can we transform two arbitrary documents, if we mark them up in a certain way?
