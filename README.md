Benthos
=======

Benthos is a service for piping large volumes of data from one source to another with low latency and file based persistence in case of service outages or back pressure further on the pipeline.

This is currently an experimental project and is not recommended for production systems.

## Design

Benthos uses memory mapped files for storing messages in transit with a low latency impact. If benthos crashes with a buffered queue of messages pending then those messages should be preserved and the buffer is resumed where it left off.

```
+---------------------------------+    +--------+    +---------------------------------+
|         Input Streams           |--->| Buffer |--->|         Output Streams          |
| ( ZMQ, stdin, websockets, etc ) |    +--------+    | ( ZMQ, stdin, websockets, etc ) |
+---------------------------------+        |         +---------------------------------+
                                           v
                                     +------------+
                                     | Filesystem |
                                     +------------+
```

Benthos supports various input and output strategies. It is possible to combine multiple inputs and multiple outputs in one service instance. Currently combined outputs will each receive every message, where any blocked output will block all outputs. If there is demand then other output paradigms such as publish, round-robin etc could be supported.

## Speed and Benchmarks

Obviously the main goal of Benthos is to be stable, low-latency and high throughput. Benthos comes with two benchmarking tools using ZMQ4 `benthos_zmq_producer` and `benthos_zmq_consumer`. The producer pushes an endless stream of fixed size byte messages with a configured time interval between each message. The consumer will read an endless stream of messages and print a running average of latency, throughput (per second) and total messages received.

Using these tools on my laptop shows promising results, but it would be useful to get some more meaningful third party benchmarks to publish.

A snippet of the results thus far from my laptop using the ZMQ4 input, file persisted buffer and ZMQ4 output configuration:

| Message size | Stream interval | Avg. Latency |
|-------------:|----------------:|-------------:|
|         5000 |            10ms |        800us |

## Install

```shell
go get github.com/jeffail/benthos
```

## ZMQ4 Support

Benthos supports ZMQ4 for both data input and output. To add this you need to install libzmq4 and use the compile time flag when building benthos:

```shell
go install -tags "ZMQ4"
```

## Run

```shell
benthos -conf ./config.yaml
```

## Config

Create a default configuration file:

```shell
benthos --print-yaml > config.yaml
```

The configuration file should contain a section for inputs, outputs and a buffer. The inputs and outputs sections may contain one or more entries, where each entry will define a unique input or output.

For example, if we wanted to output to both a ZMQ4 push socket as well as stdout our outputs section in a YAML config might look like this:

```yaml
outputs:
- type: zmq4
  zmq4:
    addresses:
      - tcp://*:1234
    socket_type: PUSH
- type: stdout
```

Or, in a JSON config file it might look like this:

```json
"outputs": [
	{
		"type": "zmq4",
		"zmq4": {
			"addresses": [
				"tcp://*:1235"
			],
			"socket_type": "PUSH"
		}
	},
	{
		"type": "stdout"
	}
]
```
