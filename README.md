# Sportradar consumer

A small example to consume sportradar push feeds based on [Benthos](https://www.benthos.dev)

## Features

- High availability
- Multiple push feed comsumption
- Message deduplication
- Message filtering (e.g. heartbeats)
- Configurable output methods (http, sql, redis stream, etc...)
- Metrics (e.g. Prometheus)
## Run Locally

Clone the project

```bash
  git clone https://github.com/IAmBod/sportradar-consumer.git sportradar_consumer
```

Go to the project directory

```bash
  cd sportradar_consumer
```

Configure your environment in .env file

Start the server

```bash
  docker compose up
```

## Environment Variables

To run this project, you will need to add the following environment variables to your .env/.env.local file

`SPORTRADAR_API_KEY` API key fron https://console.sportradar.com/

`SPORTRADAR_ACCESS_LEVEL` Access level for api key

## Usage/Examples

### Multiple inputs

When you want to process multiple feeds (football, basketball, etc.)

```yaml
input:
    broker:
        inputs:
            -   http_client:
                    url: https://api.sportradar.com/soccer-extended/${SPORTRADAR_ACCESS_LEVEL}/v4/stream/events/subscribe?api_key=${SPORTRADAR_API_KEY}&format=json
                    verb: GET
                    stream:
                        enabled: true
                        reconnect: true
                        scanner:
                            lines: { }
            -   http_client:
                    url: https://api.sportradar.com/soccer-extended/${SPORTRADAR_ACCESS_LEVEL}/v4/stream/events/subscribe?api_key=${SPORTRADAR_API_KEY}&format=json
                    verb: GET
                    stream:
                        enabled: true
                        reconnect: true
                        scanner:
                            lines: { }
```

### HTTP output
```yaml
output:
    http_client:
        url: http://noop/post
        verb: POST
```

### Postgres output
```yaml
output:
    sql_insert:
        driver: postgres
        dsn: postgres://sportradar:password@postgres:5432/sportradar?sslmode=disable
        table: sportradar_messages
        columns: [ payload ]
        args_mapping: |
            root = [
              this.format_json(no_indent: true)
            ]
        init_statement: |
            CREATE TABLE IF NOT EXISTS sportradar_messages (
                id SERIAL PRIMARY KEY,
                payload TEXT
            );
```

### Redis Stream output
```yaml
output:
    redis_streams:
        url: redis://redis:6379
        stream: sportradar_messages
```

## License

This software is distributed under [MIT](LICENSE)
