---
title: kafka
url: configuration/plugins/messagebus_consumers/kafka
---

## Description

This plugin allow hermes-client to receive events from an Apache Kafka server.

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
            # MANDATORY : hermes-client's cert file that will be used for
            # authentication
            certfile: /path/to/.hermes/dummy.crt
            # MANDATORY : hermes-client's cert file's private key
            keyfile: /path/to/.hermes/dummy.pem
            # MANDATORY : The PKI CA cert
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt

          # MANDATORY : the topic to send events to
          topic: hermes
          # MANDATORY : the group_id to assign client to. Set what you want here.
          group_id: hermes-grp
```
