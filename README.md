Benchmarks for [Bref](https://github.com/brefphp/bref) running on AWS Lambda.

## Scenarios

- PHP function: a simple PHP function, see `function/index.php`
- HTTP application: a simple PHP web page that returns `Hello world`, see `fpm/index.php`

## Median (warm) execution time

Average execution time for a lambda that doesn't do anything.

Number of samples: 900

### Bref 1.x (PHP 8.1)

| Memory                       |   128 |  512 | 1024 | 1769 |
|------------------------------|------:|-----:|-----:|-----:|
| PHP function                 | 145ms | 27ms | 15ms | 14ms |
| PHP function (BREF_LOOP_MAP) |       |      |  1ms |  1ms |
| HTTP application             |   1ms |  1ms |  1ms |  1ms |
| Laravel                      |       |      |  9ms |      |

([`BREF_LOOP_MAP` docs](https://bref.sh/docs/environment/performances.html#bref-for-event-driven-functions))

### Bref 2.x (PHP 8.1)

| Memory                       |   128 |  512 | 1024 | 1769 |
|------------------------------|------:|-----:|-----:|-----:|
| PHP function                 | 250ms | 46ms | 24ms | 21ms |
| PHP function (BREF_LOOP_MAP) |       |      |  1ms |  1ms |
| HTTP application             |   1ms |  1ms |  1ms |  1ms |
| Laravel                      |       |      |  8ms |      |

### Bref 2.x ARM (PHP 8.0)

| Memory                       |   128 |  512 | 1024 | 1769 |
|------------------------------|------:|-----:|-----:|-----:|
| PHP function                 | 240ms | 41ms | 21ms | 20ms |
| PHP function (BREF_LOOP_MAP) |       |      |  1ms |  1ms |
| HTTP application             |   1ms |  1ms |  1ms |  1ms |
| Laravel                      |       |      | 11ms |      |

For comparison on a 512M Digital Ocean droplet we get 1ms for "HTTP application" and 6ms for Symfony.

## CPU performance

The more RAM, the more CPU power is allocated to the lambda. This is clearly visible when running [PHP's official `bench.php` script](https://github.com/php/php-src/blob/master/Zend/bench.php).

| Memory      |  128 |  512 |  1024 |  1769 |
|-------------|-----:|-----:|------:|------:|
| `bench.php` | 5.7s | 1.4s | 0.65s | 0.33s |

For comparison  `bench.php` runs in 1.3s on a 512M Digital Ocean droplet, in 0.8s on a 2.8Ghz i7 and in 0.6s on a 3.2Ghz i5.

## Cold starts

Number of samples: 20

### Bref 1.x (PHP 8.1)

| Memory           |   128 |   512 |  1024 |  1769 |
|------------------|------:|------:|------:|------:|
| PHP function     | 420ms | 250ms | 230ms | 228ms |
| HTTP application | 420ms | 330ms | 310ms | 310ms |
| Laravel          |       |       | 920ms |       |

### Bref 2.x (PHP 8.1)

| Memory           |   128 |   512 |  1024 |  1769 |
|------------------|------:|------:|------:|------:|
| PHP function     | 465ms | 235ms | 210ms | 205ms |
| HTTP application | 370ms | 280ms | 266ms | 266ms |
| Laravel          |       |       | 885ms |       |

### Bref 2.x ARM (PHP 8.0)

| Memory           |   128 |   512 |  1024 |  1769 |
|------------------|------:|------:|------:|------:|
| PHP function     | 440ms | 205ms | 175ms | 165ms |
| HTTP application | 325ms | 230ms | 220ms | 220ms |
| Laravel          |       |       | 820ms |       |

Measuring cold starts in CloudWatch Logs Insights:

```
filter @type = “REPORT” and @initDuration
| stats
 count(@type) as count,
 min(@billedDuration) as min,
 avg(@billedDuration) as avg,
 pct(@billedDuration, 50) as p50,
 max(@billedDuration) as max
by @log
| sort @log
```

## Reproducing

You will need [to install dependencies of Bref](https://bref.sh/docs/installation.html). Then:

- clone the repository
- `make setup`
- `make deploy`

Then run the `make bench-*` commands.
