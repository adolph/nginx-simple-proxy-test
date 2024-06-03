
# nginx simple proxy test

Based on nginx documentation [Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html#conf_structure).

## Prerequisites

- Docker (or equivalent, need commands for 'docker' and 'docker-compose')
- a terminal application and basic cli knowledge
- curl
- grep

## Outcomes

Observe how nginx can proxy http from another server. 

## Instructions

1. Download this git archive and cd into it
2. Use the below command to set up the networked containers

```bash
docker-compose up -d
# Expect output similar to:
# ✔ Network nginx_my-network  Created                                                                                                                                                     0.0s 
# ✔ Container nginx-webapp-1  Started                                                                                                                                                     0.0s 
# ✔ Container nginx-proxy-1   Started                                                                                                                                                     0.0s 
```

3. Confirm the container names:
```bash
docker container ps
# Expect output similar to
# CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
# xxxxxxxxxxxx   nginx:1.27.0   "/docker-entrypoint.…"   3 seconds ago   Up 3 seconds   0.0.0.0:8070->80/tcp   nginx-proxy-1
# xxxxxxxxxxxx   nginx:1.27.0   "/docker-entrypoint.…"   3 seconds ago   Up 3 seconds   0.0.0.0:8090->80/tcp   nginx-webapp-1
```

4. Confirm the containers have the correct local directores mounted:
```bash
curl -s localhost:8070 | grep title
# Expect: 
#<title>Welcome to proxy host!</title>
curl -s localhost:8090 | grep title
# Expect: 
#<title>Welcome to webapp host!</title>
```

5. Get the web application's IP Address using the name from above (in this case "nginx-webapp-1"):
```bash
docker container inspect nginx-webapp-1 | grep IPAddress
# Expect:
#  "SecondaryIPAddresses": null,
#  "IPAddress": "",
#    "IPAddress": "172.21.0.2",
```

6. Confirm the proxy container can see the web app container (in this case the IP address is 172.21.0.2, the address will be different on your system)
```bash
docker exec nginx-proxy-1 sh -c 'curl -s 172.21.0.2:80 | grep title'
# Expect:
# <title>Welcome to webapp host!</title>
```

7. Edit the proxy container nginx config to perform proxying of the webapp container
  a. Add the "#" character to the beginning of lines 15 to 18
  b. Remove the leading "#" character from the beginning of lines 24 to 26
  c. Reload the configuration in the proxy container and test it
```bash
docker exec nginx-proxy-1 sh -c 'nginx -s reload'                   
# Expect: 
#2024/06/01 12:00:00 [notice] 50#50: signal process started
nginx % curl -s localhost:8090 | grep title                                 
# Expect:
#<title>Welcome to webapp host!</title>
```

If the result of the last command says "proxy" instead of "webapp" then recheck the above steps.

8. Clean up
```bash
docker-compose down 
# Expect:
#✔ Container nginx-webapp-1  Removed                                                                                                                                                     0.1s 
#✔ Container nginx-proxy-1   Removed                                                                                                                                                     0.2s 
#✔ Network nginx_my-network  Removed    
```
