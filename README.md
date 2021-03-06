# JSON Structures To Define Siddhi Apps

## Contents
1. [General Types](#general-types)
    1. [Key-Value Pair JSON](#key-value-pair-json)
    2. [Attributes](#attributes)
    3. [Annotations](#annotations)
    4. [Store](#store)    
2. [Stream Definition](#stream-definition)
3. [Table Definition](#table-definition)
4. [Window Definition](#window-definition)
5. [Trigger Definition](#trigger-definition)
6. [Aggregation Definition](#aggregation-definition)
7. [Source Definition](#source-definition)
8. [Sink Definition](#sink-definition)
9. [Query Definition](#query-definition)
    1. [Query Input](#query-input)
        1. [Window-Filter-Projection Query](#window-filter-projection)
        2. [Join Query](#join)
        3. [Pattern & Sequence Query](#pattern-sequence)
    2. [Query Select](#query-select)
    3. [Query Output](#query-output)
        1. [Insert](#insert)
        2. [Delete](#delete)
        3. [Update](#update)
        4. [Update Or Insert](#update-or-insert)
10. [Partition Definition](#partition-definition)

## General Types
These are general structures that are used in most of the siddhi element definitions, they are not stand 
alone elements (that is why they do not have an 'id'). 

### Key-Value Pair JSON
This is used to define a map of key-value pair and is named as `{Key-Value Pair JSON}` when the structure 
is referred.
```
{
    `key`: `value`,
    ...
}
```

### Attributes
Define's attributes of a Stream, Table etc.. that have the same format. So they all use the same structure 
which is named `attributeList` where ever they are used in a element definition.
```
[
    {
        name*: ‘’,
        type*: ‘string|int|long|double|float|bool|object’
    },
    ...
]
```
_**NOTE - The JSON attributes that end with a `*` symbol means that the attribute cannot be left null/empty**_

_**Example:** To define the attributes `(name string, age int)` the JSON structure would look like this,_
```
[
    {
        name: 'name',
        type: 'string'
    },
    {
        name: 'age',
        type: 'int'
    }
]
```

### Annotations
Defines the annotations of any Siddhi element, as all annotations have somewhat the same structure.
They are defined in as a part of a Siddhi element using the name `annotationList`, 
and have the following JSON structure.
```
[
    {
        name*: ‘’,
        type*: ‘list’,
        values*: [
            {
                value: '',
                isString: true|false
            },
            ...
        ]
    },
    << and|or >>
    {
        name*: ‘’
        type*: ‘map’,
        values*: {
            'option1': {
                value: '',
                isString: true|false
            },
            ...
        }
    },
    ...
]

```

_**Example:** To define the annotations `@Async(buffer.size='1024') @PrimaryKey('name','age')` the JSON 
structure would look like this,_
```
[
    {
        name: 'Async',
        type: 'map',
        value: {
            'buffer.size': '1024'
        }
    },
    {
        name: 'PrimaryKey',
        type: 'list',
        values: ['name', 'age']
    }
]
```

**Note - Sources, Sinks & Stores do not come under the general annotation struct as they have a different 
structure and are shown separately**

### Store
**The store annotation is only used for tables and aggregations**. So they are only defined in both table and 
aggregation JSON structs as the name `store` in the following format:
```
{
    type*: ‘’,
    options*: {Key-Value Pair JSON}
}
```

_**Example:**_
```
@Store(type="rdbms",
    jdbc.url="jdbc:mysql://localhost:3306/production",
    username="wso2",
    password="123" ,
    jdbc.driver.name="com.mysql.jdbc.Driver")
```
_The JSON for the above store definition is,_
```
{
    type: 'rdbms',
    options: {
        'jdbc.url': 'jdbc:mysql://localhost:3306/production',
        'username': 'wso2',
        'password': '123',
        'jdbc.driver.name': 'com.mysql.jdbc.Driver'
    }
}
```                  
                  

## Stream Definition
```
{
    id*: '',
    name*: '',
    isInnerStream*: true|false,
    attributeList*: {Attributes JSON Array},
    annotationList: {Annotations JSON Array}
}
```

_**Example:**_
```
@Async(buffer.size="1024")
define stream InStream (name string, age int);
```
_The JSON for the above stream definition is,_
```
{
    id: 'InStream',
    name: 'InStream',
    isInnerStream: false,
    attributeList: [
        {
            name: 'name',
            type: 'string'
        },
        {
            name: 'age',
            type: 'int'
        }
    ],
    annotationList: [
        {
            name: 'Async',
            type: 'map',
            values: {
                'buffer.size': '1024'
            }
        }
    ]
}
```
**Note that if a stream is an inner stream, then it's attributes cannot be defined and it cannot have any annotations. 
Have to handle this in another way.**

## Table Definition
```
{
    id*: ‘’,
    name*: ‘’,
    attributeList*: {Attributes JSON Array},
    store: {Store JSON},
    annotationList: {Annotations JSON Array}
}
```

_**Example:**_
```
define table InTable (name string, age int);
```
_The JSON for the above table definition is,_
```
{
    id: 'InTable',
    name: 'InTable',
    attributeList: [
        {
            name: 'name',
            type: 'string'
        },
        {
            name: 'age',
            type: 'int'
        }
    ],
    store: {},
    annotationList: []
}
```

## Window Definition
```
{
    id*: ‘’,
    name*: ‘’,
    attributeList*: {Attributes JSON Array},
    function*: ‘time|length|timeBatch|lengthBatch...’,
    parameters*: ['value1',...],
    outputEventType: ‘current|expired|all’,
    annotationList: {Annotations JSON Array}
}
```

_**Example:**_
```
define window SensorWindow (name string, value float) timeBatch(1 second) output expired events;
```
_The JSON for the above window definition is,_
```
{
    id: 'SensorWindow',
    name: 'SensorWindow',
    attributeList: [
        {
            name: 'name',
            type: 'string'
        },
        {
            name: 'value',
            type: 'float'
        }
    ],
    function: 'timeBatch',
    parameters: ['1 second'],
    outputEventType: 'expired',
    annotationsList: []
}
```

## Trigger Definition
```
{
    id*: ‘’,
    name*: ‘’,
    at*: ‘’
    annotationList: {Annotations JSON Array}
}
```

_**Example:**_
```
define trigger FiveMinTrigger at every 5 min;
```
_The JSON for the above trigger definition is,_
```
{
    id: 'FiveMinTrigger',
    name: 'FiveMinTrigger',
    at: 'every 5 min',
    annotationList: []
}
```

## Aggregation Definition
```
{
    id*: ‘’,
    name*: ‘’,
    from*: ‘’,
    select*: {Query Select JSON},
    groupBy: ['value1',...],
    aggregateByAttribute*: '',
    aggregateByTimePeriod*: {
        minValue*: '', // Atleast one value should be added, and that will be marked as the minValue
        maxValue: '' // Max value is added if the user wants to define a range of timestamps
    },
    store: {Store JSON},
    annotationList: {Annotations JSON Array}
}
```

_**Example:**_
```
define aggregation TradeAggregation
    from TradeStream
        select symbol, avg(price) as avgPrice, sum(price) as total
        group by symbol
        aggregate by timestamp every sec...year;
```
_The JSON for the above aggregation definition is,_
```
{
    id: 'TradeAggregation',
    name: 'TradeAggregation',
    from: 'TradeStream',
    select: {
        type: 'user_defined',
        value: [
            {
                expression: 'symbol',
                as: ''
            },
            {
                expression: 'avg(price)',
                as: 'avgPrice'
            },
            {
                expression: 'sum(price)',
                as: 'total'
            }
        ]
    },
    groupBy: ['symbol'],
    aggregateByAttribute: 'timestamp',
    aggregateByTimePeriod: {
        minValue: 'sec', 
        maxValue: 'year'
    },
    store: {},
    annotationList: []
}
```

## Source Definition
```
{
    id*: ‘’,
    type*: ‘’,
    options: {Key-Value Pair JSON},
    map: {
        type*: ‘’,
        options: {Key-Value Pair JSON},
        attributes: {
            type*: ‘map’
            value*: {Key-Value Pair JSON}
        }
        << or >>
        attributes: {
            type*: ‘list’
            value*: ['value1',...]
        }
    }
}
```

_**Example:**_
```
@Source(
    type = 'http',
    receiver.url='http://localhost:8006/productionStream',
    basic.auth.enabled='false',
    @map(type='json')
)
```
_The JSON for the above source definition is,_
```
{
    id: '<UUID>',
    type: 'http',
    options: {
        'receiver.url': 'http://localhost:8006/productionStream',
        'basic.auth.enabled': 'false',
    },
    map: {
        type: 'json',
        options: {},
        attributes: {}
    },
}
```                   

## Sink Definition
```
{
    id*: ‘’,
    type*: ‘’,
    options: {Key-Value Pair JSON},
    map: {
        type*: ‘’,
        options: {Key-Value Pair JSON},
        payload: {
            type*: ‘map’,
            value*: {Key-Value Pair JSON}
        }
        << or  >>
        payload: {
            type*: 'single',
            value*: ''
        }
    }
}
```

_**Example:**_
```
@sink(
    type='http',
    publisher.url='http://localhost:8005/endpoint', 
    method='POST', 
    headers='Accept-Date:20/02/2017', 
    basic.auth.username='admin', 
    basic.auth.password='admin', 
    basic.auth.enabled='true',
    @map(type='json')
)
```
_The JSON for the above sink definition is,_
```
{
    id: '<UUID>',
    type: 'http',
    options: {
        'publisher.url': 'http://localhost:8005/endpoint',
        'method': 'POST',
        'headers': 'Accept-Date:20/02/2017',
        'basic.auth.username': 'admin',
        'basic.auth.password': 'admin',
        'basic.auth.enabled': 'true'
    },
    map: {
        type: 'json',
        options: {},
        payload: {}
    }
}
```

## Query Definition
All queries have the following JSON body structure
```
{
    id*: '',
    queryInput*: {Query Input JSON},
    select*: {Query Select JSON},
    groupBy: ['value1',...],
    orderBy: [
        {
            value*: '',
            order: 'asc|desc'
        },
        ...
    ],
    limit: <integer|long>, 
    having: '',
    outputRateLimit: ''
    queryOutput*: {Query Output JSON},
    annotationList: {Annotation JSON Array}
}
```

### Query Input
The query input can be of the following types:
* Window-Filter-Projection
* Join
* Pattern & Sequence

#### <a name="window-filter-projection">JSON Structure for `Window-Filter-Projection` query input type:</a>
Note that for this type there are a few conditions for each type even though they have the same JSON structure:
* If the `type` is `window`, then the query must have a window and can have an optional filter.
* If the `type` is `filter`, then the query must have a filter and no window.
* If the `type` is `projection`, then the query cannot have a filter or window.
```
{
    type*: 'window|filter|projection',
    from*: '',
    filter: '',
    window: {
        function*: '',
        parameters*: ['value1',...],
        filter: ''
    }
}
```

_**Example:**_
```
from InputStream[age >= 18]#window.time(1 hour)[age < 30]
select ...
```
_The JSON for the above `Window-Filter-Projection` input is,_
```
{
    type: 'window',
    from: 'InputStream',
    filter: 'age >= 18',
    window: {
        function: 'time',
        parameters: ['1 hour'],
        filter: 'age < 30'
    }
}
```

#### <a name="join">JSON Structure for `Join` query input type:</a>
A `join` query can be one of 4 types:
* Join Stream
* Join Table
* Join Aggregation
* Join Window

The way to identify a join query type is by using the `joinWith` attribute. 
However all 4 of these join types will be defined using the same JSON structure.
```
{
    type*: 'join',
    joinWith*: 'stream|table|window|aggregation',
    left*: {Join Element JSON},
    joinType*: 'join|left_outer|right_outer|full_outer',
    right*: {Join Element JSON},
    on: '',
    within: '', // If joinWith == aggregation
    per: '' // If joinWith == aggregation
}
```
The `Join Element JSON` has the following structure:
```
{
    type*: 'stream|table|window|aggregation',
    from*: '',
    filter: '', // If there is a filter, there must be a window for joins (the only exception is when type = window).
    window: {   
        function*: '',
        parameters*: ['value1',...]
    },
    as: '',
    isUnidirectional: true|false // Only one 'isUnidirectional' value can be true at a time (either left definition|right definition|none)
}
```
There are a few conditions that must be met for a join query input to be a valid one:
* Atleast one, `left` or `right` JSON value must be of stream type, or else it is not a valid join query input.
* If a `Join element JSON` is of type `window`, then that element's window attribute must be null. This is because a window definition cannot have another window within it.
* If there is a `Join Element JSON` of type `aggregation`, then the `within` and `per` attributes in the JSON structure cannot be null. If there is no aggregation definition, then those attributes have to be null.
* If a `Join Element JSON` has a `window`, then it must have a filter as well or else it is invalid (except if that element is of type `window` definition).
* Only one `Join Element JSON` can be marked as `isUnderectional: true`. It is either the left definition, right definition or neither of them.

_**Example:**_
```
from StockStream as S join TradeAggregation as T
    on S.symbol == T.symbol 
    within "2014-02-15 00:00:00 +05:30", "2014-03-16 00:00:00 +05:30" 
    per "days" 
select ...
```
_The JSON for the above `Join Aggregation` input is,_
```
{
    type: 'join',
    joinWith: 'aggregation',
    left: {
        type: 'stream',
        from: 'StockStream',
        filter: '',
        window: {},
        as: 'S',
        isUnidirectional: false
    },
    joinType: 'join',
    right: {
        type: 'aggregation',
        from: 'TradeAggregation',
        filter: '',
        window: {},
        as: 'T',
        isUnidirectional: false
    },
    on: 'S.symbol == T.symbol',
    within: '\"2014-02-15 00:00:00 +05:30\", \"2014-03-16 00:00:00 +05:30\"',
    per: '\"days\"'
}
```

#### <a name="pattern-sequence">JSON Structure for `Pattern` & `Sequence` query input types:</a>
The JSON structure for both patterns & sequences are identical:
```
{
    type*: 'pattern|sequence',
    conditionList*: [
        {
            id: '',
            streamName*: '',
            filter: ''
        },
        ...
    ],
    logic*: ''
}
```

_**Example**_
```
from every event1=InStream[age < 100]<21:234> within 10 min ->
    every event2=InStream[age > 30] and event3=InStream[age < 50]->
    every not InStream[age >= 18] for 5 sec ->
    every not InStream[age < 18] and event6=InStream[age > 30]
select ...
```
_The above pattern input is defined by the following JSON structure:_
```
{
    type: 'pattern',
    conditionList: [
        {
            id: 'event1',
            streamName: 'InStream',
            filter: 'age < 100'
        },
        {
            id: 'event2',
            streamName: 'InStream',
            filter: 'age > 30'
        },
        {
            id: 'event3',
            streamName: 'InStream',
            filter: 'age < 50'
        },
        {
            id: 'event4',
            streamName: 'InStream',
            filter: 'age >= 18'
        },
        {
            id: 'event5',
            streamName: 'InStream',
            filter: 'age >= 18'
        },
        {
            id: 'event5',
            streamName: 'InStream',
            filter: 'age >= 18'
        }
    ],
    logic: 'every event1<21:234> within 10 min -> every event2 and event3 -> every not event4 for 5 sec -> every not event5 and event6'
}
```
**Note - If their is a `not` statement before an `id` in the `logic` attribute, then that `id` will not be displayed in the _source view_.**
_For an example:_
```
{
    type: 'sequence',
    conditionList: [
        {
            id: 'e1',
            streamName*: 'InStream',
            filter: 'age >= 18'
        },
        {
            id: 'e2',
            streamName*: 'InStream',
            filter: 'age < 30'
        }
    ],
    logic: 'every e1+ within 10 min, not e2 for 10 min'
}
```
The siddhi code for the above JSON would look like this:
```
from 
    every e1=InStream[age >= 18], 
    not InStream[age < 30] for 10 min
select ...
```
Notice that the second sequence `not InStream[age < 30] for 10 min` does not have the phrase 
`"e2="` reference like `e1` does because it has a `not` statement before it.

### Query Select
```
{
    type*: 'user_defined',
    value*: [
        {
            expression*: '',
            as: ''
        },
        ...
    ]
    << or >>
    type*: 'all',
    value*: '*'
}
```

_**Example:**_
```
select Stream1.id as UID, avg(price) as avgPrice, ((TempStream.temp - 32) * 5)/9 as Celsius, isInStock ...
```
_The JSON for the above `select` function is,_
```
{
    type: 'user_defined',
    value: [
        {
            expression: 'Stream1.id',
            as: 'UID'
        },
        {
            expression: 'avg(price)',
            as: 'avgPrice'
        },
        {
            expression: '((TempStream.temp - 32) * 5)/9',
            as: 'Celsius'
        },              
        {
            expression: 'isInStock',
            as: ''
        }
    ]
}
```

### Query Output
All query outputs have the following generic structure:
```
{
    type*: 'insert|delete|update|update-or-insert-into',
    output*: {INSERT JSON|DELETE JSON|UPDATE JSON|UPDATE-OR-INSERT JSON},
    target*: ''
}
```

The `output` Attribute Can Be Of 4 JSON Structures:
* Insert
* Delete
* Update
* Update or insert into

#### <a name="insert">JSON structure for `insert` query output type:</a>
```
{
    eventType: 'current|expired|all'
}
```

_**Example:**_
```
insert all events into LogStream;
```
_The JSON for the above `insert` function is,_
```
{
    type: 'insert',
    output: {
        eventType: 'all'
    },
    target: 'LogStream'
}
```

#### <a name="delete">JSON structure for the `delete` query output type:</a>
```
{
    eventType: 'current|expired|all',
    on*: ''
}
```

_**Example:**_
```
delete RoomTypeTable
    for all events 
    on RoomTypeTable.roomNo == roomNumber;
```
_The JSON for the above `insert` function is,_
```
{
    type: 'delete',
    output: {
        eventType: 'all',
        on: 'RoomTypeTable.roomNo == roomNumber'
    },
    target: 'RoomTypeTable',
}
```

#### <a name="update">JSON structure for the `update` query output type:</a>
```
{
    eventType: 'current|expired|all',
    set*: [
        {
            attribute*: '',
            value*: ''
        },
        ...
    ],
    on*: ''
}
```

_**Example:**_
```
update RoomTypeTable
    set RoomTypeTable.people = RoomTypeTable.people + arrival - exit
    on RoomTypeTable.roomNo == roomNumber;
```
_The JSON for the above `update` function is,_
```
{
    type: 'update',
    output: {
        eventType: '',
        set: [
            {
                attribute: 'RoomTypeTable.people',
                value: 'RoomTypeTable.people + arrival - exit'
            }
        ],
        on: 'RoomTypeTable.roomNo == roomNumber'
    },
    target: 'RoomTypeTable',
}
```


#### <a name="update-or-insert">JSON structure for the `update or insert into` query output type:</a>
```
{
    eventType: 'current|expired|all',
    set*: [
        {
            attribute*: '',
            value*: ''
        },
        ...
    ],
    on*: ''
}
```

_**Example:**_
```
update or insert into RoomAssigneeTable
    set RoomAssigneeTable.assignee = assignee
    on RoomAssigneeTable.roomNo == roomNo;
```
_The JSON for the above `update or insert into` function is,_
```
{
    type: 'update_or_insert_into',
    output: {
        eventType: '',
        set: [
            {
                attribute: 'RoomAssigneeTable.assignee',
                value: 'assignee'
            }
        ],
        on: 'RoomAssigneeTable.roomNo == roomNo'
    },
    target: 'RoomAssigneeTable',
}
```

## Partition Definition
```
{
    id*: ‘’,
    members*: {
        queryIds*: ['query1',...],
        innerStreamIds: ['stream1',...] 
    },
    partitionWith*: {
        type*: ‘{value|range}’,
        expressions*: [
            {
                condition: ‘’,
                streamId: ‘’
            },
            ...
        ]
    },
    annotationList: {Annotation JSON Array}
}
```

_**Example:**_
```
@info(name='TestPartition')
partition with ( roomNo >= 1030 as 'serverRoom' or
                 roomNo < 1030 and roomNo >= 330 as 'officeRoom' or
                 roomNo < 330 as 'lobby' of TempStream)
begin
    @info(name='TestQuery1') 
    from TempStream
    select *
    insert into #OutputStream;

    @info(name='TestQuery2') 
    from TempStream#window.time(10 min)
    select roomNo, deviceID, avg(temp) as avgTemp
    insert into AreaTempStream;
end;
```
_The JSON for the above `partition` is,_
```
{
    id: ‘TestPartition’,
    members: {
        queryIds: ['TestQuery1','TestQuery2'],
        innerStreamIds: ['OutputStream'] 
    },
    partitionWith: {
        type: ‘range’,
        expressions: [
            {
                condition: ‘roomNo >= 1030 as \'serverRoom\'
                    or roomNo < 1030 and roomNo >= 330 as \'officeRoom\'
                    or roomNo < 330 as \'lobby\'’,
                streamId: ‘TempStream’
            }
        ]
    },
    annotationList: [
        {
            name: ‘info’
            type: ‘map’,
            values: {
                'name': 'TestPartition'  
            }
        }
    ]
}
```
_The JSON for `TestQuery1` is,_
```
{
    id: 'TestQuery1',
    queryInput: {
        type: 'window_filter_projection',
        from: 'TempStream',
        filter: '',
        window: {}
    },
    select: {
        type: 'all',
        value: '*'
    },
    groupBy: [],
    orderBy: [],
    limit: '', 
    having: '',
    output: ''
    queryOutput: {
        type: 'insert',
        eventType: '',
        target: '#OutputStream'
    },
    annotationList: [
        {
            name: ‘info’
            type: ‘map’,
            values: {
                'name': 'TestQuery1'  
            }
        }
    ]
}
```
_The JSON for `TestQuery2` is,_
```
{
    id: 'TestQuery2',
    queryInput: {
        type: 'window_filter_projection',
        from: 'TempStream',
        filter: '',
        window: {
            function: 'time',
            parameters: ['10 min'],
            filter: ''
        }
    },
    select: {
        type: 'user_defined',
        value: [
            {
                expression: 'roomNo',
                as: ''
            },
            {
                expression: 'deviceID',
                as: ''
            },
            {
                expression: 'avg(Temp)',
                as: 'avgTemp'
            }
        ]
    },
    groupBy: [],
    orderBy: [],
    limit: '',
    having: '',
    output: ''
    queryOutput: {
        type: 'insert',
        eventType: '',
        target: 'AreaTempStream'
    },
    annotationList: [
        {
            name: ‘info’
            type: ‘map’,
            values: {
                'name': 'TestPartition'  
            }
        }
    ]
}
```
