# Aerospike CDT Examples

This repository contains examples of how Aerospike's complex [data types](https://www.aerospike.com/docs/guide/data-types.html),
in particular [Map](https://www.aerospike.com/docs/guide/cdt-map.html)
and [List](https://www.aerospike.com/docs/guide/cdt-list.html) can be used to
implement common patterns.

## Using Maps to Capture and Query Events

In this example, a KV-ordered Map is used to collect a user's events.
The events are keyed on the millisecond timestamp of the event, with the value
being a list of the following structure:
```
timestamp: [ event-type, { map-of-all-other-event-attributes } ]
```
For example:
```python
event = {
    1523474236006: ['viewed', {'foo':'bar', 'sku':3, 'zz':'top'}] # represents a single event
}
```

This enables several types of searches using the Map API methods:
 * Get the event in a timestamp range (time slicing)
 * Get all the events of a specific type
 * Get all the events matching a list of specified types

The [map return type](https://www.aerospike.com/apidocs/python/aerospike.html#map-return-types)
can be key-value pairs or the count of the elements matching the specified
'query' criteria, or something else that matters to the applicaiton developer.
```
$ python event_capture_and_query.py -h "172.16.60.131"
```

See: [event_capture_and_query.py](blob/master/event_capture_and_query.py)

## Maps as Event Containers with Timestamp Values

In this example, a KV-ordered Map is used to track messages in a conversation
between two users. Each message has a UUID, which will act as the key. The
message is logged to a point in time, with a timestamp as the first index of
a list value:

```
UUID: [ timestamp, { map-of-all-other-event-attributes } ]
```
For example:
```python
message = {
    '319fa1a6-0640-4354-a426-10c4d3459f0a': [1523474316003, "Hee-hee!"]
}
```

This enables searches for messages by:
 * Get messages in a timestamp range (time slicing)
 * Get messages by their rank (most recent N messages)

Because the timestamp is not the key we can use a less fine grain resolution
(seconds rather than milliseconds) and have multiple messages with the same
timestamp.

```
$ python event_query_by_value_interval.py -h "172.16.60.131"
```

See: [event_query_by_value_interval.py](blob/master/event_query_by_value_interval.py)

## Capped Collection of Events
Expanding on the previous examples of capturing and querying events in a map, we
will see how a collection (map or list) can be capped to a specified size.

In this example we're tracking high scores for video games over time. The
KV-sorted map has a milisecond timestamp for a key, and list of the following
structure for a value:
```
[score,  {'name': name, 'dt': YYYY-MM-DD HH:mm:ss}]
```
For example:
```python
score = {
    1512435671573: [9800,  {'name': 'CPU', 'dt': '2017-12-05 01:01:11'}]
}
```

In the example the following happens:
 * Initialize a collection of 6 Pac-Man high scores
 * Get the scores, sorted by date, then sorted by rank
 * Expland the top scores list to 100 with 94 new randomly generated elements
 * Show the highest and lowest of the top-100 high scores
 * Inject a new top score, keeping the map capped to 100 elements, **in a single operation** (using [operate()](https://www.aerospike.com/apidocs/python/client.html#aerospike.Client.operate))
 * Show that the number of elements remains at 100, same highest top score, and the new score at the bottom of the top-100

```
$ python capped_events.py -h "172.16.60.131"
```

See: [capped_events.py](blob/master/capped_events.py)

