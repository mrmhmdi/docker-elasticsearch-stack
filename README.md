

<h1 align="center">ELK Stack with Dcoker-Compose (for develepment)</h1>

<p align="center" width="100%">
    <img width="10%" src="https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/blt8b679e63f2b49b27/5d082d93877575d0584761c0/logo-logstash-32-color.svg">
    <img width="10%" src="https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/blt36f2da8d650732a0/5d0823c3d8ff351753cbc99f/logo-elasticsearch-32-color.svg">
    <img width="10%" src="https://static-www.elastic.co/v3/assets/bltefdd0b53724fa2ce/blt4466841eed0bf232/5d082a5e97f2babb5af907ee/logo-kibana-32-color.svg">
</p><br>

> <p align="center">for local development with basic configurations.</p>

## installation
> Before getting started, ensure that you have installed Docker and have access to Docker Compose on your host machine. You can refer to <https://docs.docker.com/get-docker/> for installation instructions.

Run these commands to create the network and volume:

```
docker network create elastic
docker volume create my_elasticsearch_data
```
*If you modify the name of the volume or network, be sure to update it in the `docker-compose.yaml` file as well, or you may encounter an error*

Before bringing up our container, it's necessary to modify the path of the logs folder to the desired location that should be monitored. You can include multiple folders for monitoring. To do this, update or add the path within the `docker-compose.yaml` file, specifically in the Logstash section. Map it from your host folder to the desired folder in the container, as illustrated below:
```yaml
volumes:
    - ./logstash/pipeline:/usr/share/logstash/pipeline
    # for example:
    # - your/host/folder : your/container/folder
    - D:/common_projects/elasticsearch_with_docker/logs:/var/log/host
```

Now, we are ready to bring up our Elastic Stack using the following command in the shell:

```
docker compose up -d
```

## configuration

1.  <details>
    <summary>Elasticsearch</summary>
    
    - Change the `node.name` to your desired value.

    - By default, `network.host` `and http.host` are set to `0.0.0.0`.
    
    - By default, security is disabled. However, if you need to secure the configuration, navigate to `elasticsearch/elasticsearch.yml` and uncomment the relevant configurations. It's advisable to refer to the Elasticsearch documentation for using [Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html).<br>
        *you can find this config at the end of the file to uncomment it.*
        ```yml
        xpack.security.enabled: false

        xpack.security.enrollment.enabled: true

        # Enable encryption for HTTP API client connections, such as Kibana Logstash, and Agents
        xpack.security.http.ssl:
        enabled: true
        keystore.path: path/to/certs/http.p12

        # Enable encryption and mutual authentication between cluster nodes
        xpack.security.transport.ssl:
        enabled: true
        verification_mode: certificate
        keystore.path: path/to/certs/transport.p12
        truststore.path: path/to/certs/transport.p12
        # Create a new cluster with the current node only
        # Additional nodes can still join the cluster later
        cluster.initial_master_nodes: ["change-the-name"]
        ```
    </details>

2.  <details>
    <summary>Logstash</summary>

    - By default, we monitor the `/var/log/host` directory, and every file with a `.log` is continuously checked by Logstash. Whenever there is a change, it sends the data to Elasticsearch

    - if you are using file beat you can use this code and replace it in `logstash/pipeline/log_stream.conf` directory:
        ```
        input {
            beats {
                port => 5044
                codec => json
            }
        }
        ```

    </details>

3.  <details>
    <summary>Kibana</summary>

    - For development purposes, there's no need to concern ourselves with configuring Kibana. However, if it becomes necessary, you can add the following YAML code to the Kibana configuration in the `docker-compose.yaml` file.:<br>
        ```yaml
        volumes:
            - ./kibana/kibana.yml:/usr/share/kibana/kibana.yml
        ```
    </details>