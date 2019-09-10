# Node-exporter for Docker Swarm
Adding the ability to get the hostname in containers. 

The problem is well described in [swarmprom](https://github.com/stefanprodan/swarmprom#prometheus-service-discovery) README:
>When Prometheus runs the DNS lookup, Docker Swarm will return a list of IPs for each task. Using these IPs, Prometheus will bypass the Swarm load-balancer and will be able to scrape each exporter instance.

>The problem with this approach is that you will not be able to tell which exporter runs on which node. Your Swarm nodes' real IPs are different from the exporters IPs since exporters IPs are dynamically assigned by Docker and are part of the overlay network. Swarm doesn't provide any records for the tasks DNS, besides the overlay IP. If Swarm provides SRV records with the nodes hostname or IP, you can re-label the source and overwrite the overlay IP with the real IP.

## Usage
in your Docker Swarm file:
```yml
 node-exporter:
    image: gsix/node-exporter-swarm:latest
    networks:
      - net
    environment:
      - NODE_ID={{.Node.ID}}
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
      - /etc/hostname:/etc/nodename
    command:
      - '--path.sysfs=/host/sys'
      - '--path.procfs=/host/proc'
      - '--collector.textfile.directory=/etc/node-exporter/'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
      - '--no-collector.ipvs'
    deploy:
      mode: global
      resources:
        limits:
          memory: 128M
        reservations:
          memory: 64M
```        
Or run: `docker-compose up`  

## Reference
1. [swarmprom](https://github.com/stefanprodan/swarmprom#prometheus-service-discovery)
