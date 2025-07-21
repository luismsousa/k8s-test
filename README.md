# k8s-test

## Provisioning the Cluster

In this particular example, I'm going to run this in my home cluster (set up using rancher and k3s) but the "Live" environment would likely be run in a PAAS like AWS EKS.

### Cluster Security

* This cluster would be private and the control API would only be accessible with a tool like tailscale or wireguard VPN.
* The authentication to the cluster would be based on IAM users and roles (Source: <https://docs.aws.amazon.com/eks/latest/userguide/cluster-auth.html>)
* I'm setting up a traefik instance in front of the service to take care of the TLS termination. In this particular case it's using LetsEncrypt certificates using a DNS challenge and picking up the annotations of the ingress object.
  * In a production environment, there would be a load balancer controller provisioned ALB that would handle this requirement.
  * If there's an additional requirement for end to end encryption then we'd be looking at provisioning some sort of service mesh like istio or api gateway like ambassador.

## Provisioning the workload

Because the provided image <https://hub.docker.com/r/kennethreitz/httpbin> does not have an arm build, I've forked the underlying repo, set up a github actions pipeline to do a multi-arch build of the image and upload it to dockerHub
Using the built-in Metrics server, we'll be setting up a horizontal pod autoscaller to monitor the `requests-per-second` metric of our ingress and scale on demand.

## Testing and Monitoring

```bash
docker run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- https://httpbin.turing.nuvai.cloud; done"
```
