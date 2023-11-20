---
title: kafka
url: configuration/plugins/messagebus_producers/kafka
---

## Description

This plugin allow hermes-server to send produced events over an Apache Kafka server.

## Configuration

It is possible to connect to Kafka server without authentication, or with [SSL (TLS) authentication](https://kafka.apache.org/documentation/#security_ssl).

```yaml
hermes:
  plugins:
    messagebus:
      kafka:
        settings:
          # MANDATORY : the Kafka server or servers list that can be used
          servers:
            - dummy.example.com:9093

          # Facultative : enables SSL authentication. If set, the 3 options below
          # must be defined
          ssl:
            # MANDATORY : hermes-server's cert file that will be used for
            # authentication
            certfile: /path/to/.hermes/dummy.crt
            # MANDATORY : hermes-server's cert file's private key
            keyfile: /path/to/.hermes/dummy.pem
            # MANDATORY : The PKI CA cert
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt

          # MANDATORY : the topic to send events to
          topic: hermes
```
