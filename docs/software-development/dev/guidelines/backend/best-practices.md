# Best Practices


## Database Connection Pool Monitoring

The `sql.DB` object represents a pool of both in-use and idle connections to the database. Proper and secure pool settings are crucial and should be tailored to the workload based on continuous monitoring.

Each service should have a Prometheus collector and export the state of the [DBStats structure](https://golang.org/pkg/database/sql/#DBStats). The pool settings should be adapted to the current workload with room for potential growth. It is important to limit the size of the pool.

Additional information:

1. [Configuring sql.DB](https://www.alexedwards.net/blog/configuring-sqldb) — general information.
2. [`dbstats_collector.go`](https://github.com/prometheus/client_golang/blob/master/prometheus/collectors/dbstats_collector.go)— example of implementation of a collector and the library used for monitoring.


## Circuit Breaker Pattern

To prevent cascading failures due to synchronous calls between services, all external communications (HTTP and gRPC calls) should be wrapped in a **circuit breaker**. It allows the service not to degrade if a third-party dependency is temporarily unavailable or does not fit into the expected SLO.

There is [a fairly clear and simple explanation with an example](https://dev.to/he110/circuitbreaker-pattern-in-go-43cn).
We use the second version of this package ([`sony/gobreaker/v2`](https://github.com/sony/gobreaker/v2)), but these are minor things.


## Exponential Backoff Pattern

To reduce the excessive load on an external dependency that is already behaving unstably, so as not to further worsen its condition, **Exponential Backoff Pattern** should be applied.

There is [a pretty good article](https://readmedium.com/code-smarter-exponential-backoff-in-go-made-easy-ba5224e18805) describing both exponential backoff and related concepts like jitter and maxcap.

In our Go services we recommend to use [`cenkalti/backoff`](https://github.com/cenkalti/backoff/v4) package.
