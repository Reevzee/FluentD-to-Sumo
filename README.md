# FluentD-to-Sumo
FluentD to Sumo

# Logging docker to SumoLogic

## Create SumoLogic HTTP Collector

Create a new SumoLogic HTTP Collector and note the URL generated to send logs.

## Create sidecar container

Create a new folder and create an empty plugins folder and a file fluent.conf with the following content:

```
<source>
        @type  forward
        port  24224
</source>
<match **>
        type sumologic
        flush_interval 5s
        host endpoint1.collection.eu.sumologic.com
        format json
        source_name_key app
        path /receiver/v1/http/SOME_BASE64_STRING
</match>
```

You can change format to text but be aware of a current bug in the sumologic plugin for fluent https://github.com/memorycraft/fluent-plugin-sumologic/issues/15

Create a Docker image with the following Dockerfile:

```
FROM fluent/fluentd:latest-onbuild
WORKDIR /home/fluent
ENV PATH /home/fluent/.gem/ruby/2.3.0/bin:$PATH

USER root
RUN apk --no-cache --update add sudo build-base ruby-dev && sudo -u fluent gem install fluent-plugin-sumologic-2 && rm -rf /home/fluent/.gem/ruby/2.3.0/cache/*.gem && sudo -u fluent gem sources -c && apk del sudo build-base ruby-dev && rm -rf /var/cache/apk/*

EXPOSE 24284

USER fluent
CMD exec fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT
```

Build and run this image and expose the port 24224.

You can now start your primary docker container with the following options:

```
docker run --log-driver fluentd --log-opt labels=app --label app=mysuper-server some-image:latest
```

This will log the Docker output to SumoLogic with _sourceName mysuper-server and the category you created with the HTTP collector
