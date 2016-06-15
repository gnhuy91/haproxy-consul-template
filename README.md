# haproxy-consul-template
HAProxy with configuration file generated by consul-template for automatically
update nodes in Consul KV and reload HAProxy when the KV changes.

### Quick start
- Run a Consul instance (server/agent) on current machine
- Run this Docker container:
```
$ docker run -d --net=host -p 80:80 --name=haproxy gnhuy91/haproxy-consul-template
```
- Start some test services, note the `rest` tag - it makes Registrator add the tag to Consul (and `SERVICE_CHECK_*` env vars for registering health checks to Consul):
```shell
# This will create 3 containers
$ for i in $(seq 1 3); do \
  docker run -d -P \
  -e SERVICE_TAGS=rest \
  -e SERVICE_CHECK_HTTP=/ \
  -e SERVICE_CHECK_INTERVAL=5s \
  -e SERVICE_CHECK_TIMEOUT=1s \
  --name=node$i -h=node$i \
  jlordiales/python-micro-service:latest \
  ; done
```
- In another terminal, try calling the services via HAProxy:
```shell
$ while true; do curl <HAProxy IP>:80/python-micro-service/; echo -----; sleep 1; done;
```

You can stop and start those test services (`docker stop node1`) on a new terminal and see the output of above `curl` changes accordingly.

### Bonus
`curl localhost:8500/v1/health/checks/python-micro-service | jq .` for health checks status.

### References
- http://sirile.github.io/2015/05/18/using-haproxy-and-consul-for-dynamic-service-discovery-on-docker.html
- https://github.com/CiscoCloud/haproxy-consul
- https://jlordiales.me/2015/04/01/consul-template/
