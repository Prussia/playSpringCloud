#[Sleuth](https://github.com/spring-cloud/spring-cloud-sleuth#terminology)


- span id and trace id
	![alt tag](./pic/trace-id.png)
- [zipkin](https://github.com/openzipkin/zipkin)
	- [architecture](http://zipkin.io/pages/architecture.html)
		There are 4 components that make up Zipkin:  collector, storage, search, web UI
	- Data Samples
		[
		    {
		      "traceId": "bd7a977555f6b982",
		      "name": "get",
		      "id": "bd7a977555f6b982",
		      "timestamp": 1458702548467000,
		      "duration": 386000,
		      "annotations": [
			{
			  "endpoint": {
			    "serviceName": "zipkin-query",
			    "ipv4": "192.168.1.2",
			    "port": 9411
			  },
			  "timestamp": 1458702548467000,
			  "value": "sr"
			},
			{
			  "endpoint": {
			    "serviceName": "zipkin-query",
			    "ipv4": "192.168.1.2",
			    "port": 9411
			  },
			  "timestamp": 1458702548853000,
			  "value": "ss"
			}
		      ],
		      "binaryAnnotations": []
		    },	..............	
	- Example flow	
	┌─────────────┐ ┌───────────────────────┐  ┌─────────────┐  ┌──────────────────┐
	│ User Code   │ │ Trace Instrumentation │  │ Http Client │  │ Zipkin Collector │
	└─────────────┘ └───────────────────────┘  └─────────────┘  └──────────────────┘
	       │                 │                         │                 │
		   ┌─────────┐
	       │ ──┤GET /foo ├─▶ │ ────┐                   │                 │
		   └─────────┘         │ record tags
	       │                 │ ◀───┘                   │                 │
				   ────┐
	       │                 │     │ add trace headers │                 │
				   ◀───┘
	       │                 │ ────┐                   │                 │
				       │ record timestamp
	       │                 │ ◀───┘                   │                 │
				     ┌─────────────────┐
	       │                 │ ──┤GET /foo         ├─▶ │                 │
				     │X-B3-TraceId: aa │     ────┐
	       │                 │   │X-B3-SpanId: 6b  │   │     │           │
				     └─────────────────┘         │ invoke
	       │                 │                         │     │ request   │
								 │
	       │                 │                         │     │           │
					 ┌────────┐          ◀───┘
	       │                 │ ◀─────┤200 OK  ├─────── │                 │
				   ────┐ └────────┘
	       │                 │     │ record duration   │                 │
		    ┌────────┐     ◀───┘
	       │ ◀──┤200 OK  ├── │                         │                 │
		    └────────┘       ┌────────────────────────────────┐
	       │                 │ ──┤ asynchronously report span     ├────▶ │
				     │                                │
				     │{                               │
				     │  "traceId": "aa",              │
				     │  "id": "6b",                   │
				     │  "name": "get",                │
				     │  "timestamp": 1483945573944000,│
				     │  "duration": 386000,           │
				     │  "annotations": [              │
				     │--snip--                        │
				     └────────────────────────────────┘
