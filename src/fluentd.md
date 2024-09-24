[source](https://www.youtube.com/watch?v=sIVGsQgMHIo&t=44s)

# What is Fluentd

It is a event/data collection tool. We can have multiple applications with tons
of different logs formats. Fluentd tries to unify them.

It is extensible, the core program is small, and does only the main stuff.

- Divide & Conquer
- Buffering & Retries
- Error Handling
- Message Routing
- Parallelism

It delegate the rest to the users via plugins, reading data, parsing data,
buffering data, write data.

It is reliable, it is important to not loose data. Fluentd tries to move the
data in small pieces and checks if the data passed successfully if not it
retries.

Fluentd can tag the data and route it according to the label.

# Use Cases

### Simple Forwarding

You have some simple files that and logs from a port and
want to forward them to mongodb. Here is a simple configuration example.

logs from a file
```
<source>
    type tail
    path /var/log/httpd.log
    format apache2
    tag web.access
</source>
```

logs from client libraries
```
<source>
    type forward
    port 24224
</source>
```

store logs to ES and HDFS
```
<source backend.* >
    type mongo
    database fluent
    collection test
</source>
```

# Lambda Architecture

