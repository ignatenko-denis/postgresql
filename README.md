Run manually - [Vacuum](https://postgrespro.ru/docs/postgrespro/12/sql-vacuum) optimize all schemas, all tables in PostgreSQL
```postgresql
VACUUM (VERBOSE, ANALYZE);
```

***

To see all installed extensions
```postgresql
select * from pg_available_extensions;
```

Create schema "cron" with permissions to user "sample"
```postgresql
CREATE SCHEMA IF NOT EXISTS cron AUTHORIZATION "sample";
SET search_path = cron;
GRANT ALL ON ALL TABLES IN SCHEMA cron TO "sample";
ALTER DEFAULT PRIVILEGES IN SCHEMA cron GRANT ALL ON TABLES TO "sample";
GRANT SELECT ON ALL TABLES IN SCHEMA cron TO "sample";
ALTER DEFAULT PRIVILEGES IN SCHEMA cron GRANT SELECT ON TABLES TO "sample";
```

Run as superuser
```postgresql
CREATE EXTENSION IF NOT EXISTS pg_cron WITH SCHEMA cron VERSION "1.2" CASCADE;
```

Schedule by [pg_cron](https://github.com/citusdata/pg_cron) run Vacuum every day at 06:00am (GMT)
```postgresql
SELECT cron.schedule('0 6 * * *', 'VACUUM (VERBOSE, ANALYZE)');
```

Schedule deleting old data on Saturday at 7:00am (GMT)
```postgresql
SELECT cron.schedule('0 7 * * 6', $$DELETE FROM sample.sample_rq rq WHERE rq.request_received < now() - interval '1 year'$$);
```

Custom function
```postgresql
CREATE OR REPLACE FUNCTION timestamp_to_char(TIMESTAMP WITH TIME ZONE) RETURNS TEXT AS
$$
SELECT to_char($1, 'yyyy_mm');
$$
    LANGUAGE sql IMMUTABLE
                 RETURNS NULL ON NULL INPUT;
```

Custom function usage
```postgresql
SELECT timestamp_to_char(CURRENT_TIMESTAMP);
```

All table names in DB
```postgresql
select c.relname from pg_class c;
```

All tables have system column tableoid - table id
```postgresql
SET search_path = sample;

DROP TABLE IF EXISTS cities CASCADE;

CREATE TABLE IF NOT EXISTS cities
(
    id   BIGSERIAL PRIMARY KEY NOT NULL,
    name VARCHAR(40)           NOT NULL
);

INSERT INTO cities (name) VALUES ('Moscow');
INSERT INTO cities (name) VALUES ('Paris');
INSERT INTO cities (name) VALUES ('London');

SELECT rq.tableoid, p.relname, rq.name
FROM cities rq, pg_class p
WHERE rq.tableoid = p.oid;

-- tableoid::regclass - print table name
SELECT rq.tableoid::regclass, rq.name FROM cities rq;
```
