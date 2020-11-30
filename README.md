# Fluentd Plugin ClickHouse Database

## How to use it?

Put out_clickhousejson.rb to /etc/td-agent/plugin

```
wget -O /etc/td-agent/plugin/out_clickhousejson.rb  https://raw.githubusercontent.com/vitalvas/fluent-plugin-clickhouse-json/master/out_clickhousejson.rb
```

There is mimimum fields in example td-agent.conf:

```
<source>
    @type http
    port 8888
</source>
<match inp>
    @type clickhousejson
    table <table>
</match>
```

Additional fields:

```
<match inp>
    @type clickhousejson
    host 127.0.0.1
    port 8123
    user <user>, default user is "default"
    password <password>, default password is "
    database <database>
    table <table>
    datetime_name <field name>, field with DateTime value
    tz_offset <minutes>, timezone offset in minutes

    buffer_type file
    buffer_path /var/log/td-agent/buffer/inp.logs
    buffer_chunk_limit 2g
    buffer_queue_limit 256
    flush_at_shutdown true
    flush_interval 30s
</match>
```

Before launching td-agent, create table into ClickHouse:

```sql
CREATE TABLE FLUENT (
    Date Date MATERIALIZED toDate(DateTime),
    DateTime DateTime,
    Str String,
    Num Int32
) ENGINE = MergeTree(Date, Date, DateTime, 8192);
```

Start td-agent and send a few events to fluentd:

```bash
curl -X POST -d 'json={"Num":1}' http://localhost:8888/inp
curl -X POST -d 'json={"Num":2}' http://localhost:8888/inp
curl -X POST -d 'json={"Num":3}' http://localhost:8888/inp
```

After a few seconds, when buffer flushes, in ClickHouse you could see this:

```
┌───────Date─┬────────────DateTime─┬─Str─┬─Num─┐
│ 2017-11-06 │ 2017-11-06 14:42:03 │ inp │   1 │
│ 2017-11-06 │ 2017-11-06 14:42:06 │ inp │   2 │
│ 2017-11-06 │ 2017-11-06 14:42:09 │ inp │   3 │
└────────────┴─────────────────────┴─────┴─────┘
```
