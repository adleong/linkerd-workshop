# linkerd-workshop

In this workshop we'll get started with linkerd by deploying it locally and
using it to send traffic to some local python services.

## Create Some Python Services

We'll start up two simple python services, one that serves the word "cat" on
port 7777 and one that serves the word "dog" on port 7778

```bash
mkdir cat dog
cd cat
echo cat > index.html
python3 -m http.server 7777 &
cd ../dog
echo dog > index.html
python3 -m http.server 7778 &
```

## Download Linkerd

Now we'll download the latest version of linkerd from Github.

```bash
curl -LO https://github.com/linkerd/linkerd/releases/download/1.0.2/linkerd-1.0.2-exec
chmod +x linkerd-1.0.2-exec
```

## Write a Simple Linkerd Config

Create a file called `linkerd.yml` that will configure Linkerd's behavior.

```yaml
namers:
- kind: io.l5d.fs
  rootDir: disco

routers:
- protocol: http
  dtab: /svc => /#/io.l5d.fs ;
  servers:
  - port: 4140

telemetry:
- kind: io.l5d.recentRequests
  sampleRate: 1.0
```

## Create a Service Discovery Directory

We'll create a service discovery directory that Linkerd will use to discover
where services are running.

```bash
mkdir disco
cat << EOF > /disco/animal
> 127.1 7777
> 127.1 7778
> EOF
```

## Start Linkerd!

```bash
./linkerd-1.0.2-exec linkerd.yml
```

## Send Some Requests!

```bash
curl localhost:4140 -H "Host: animal"
```

Or

```bash
http_proxy=localhost:4140 curl animal
```

## Take a Look at the Admin Dashboard

```bash
open localhost:9990
```
