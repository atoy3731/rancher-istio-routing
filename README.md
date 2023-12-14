# Rancher Istio Routing

## LoadBalancer Handling

This assumes you have a mechanism to provision an IP/ELB for services of type `LoadBalancer`. If you're in AWS, use the AWS cloud provider. If you're on bare metal or a hypervisor, consider using MetalLB or KubeVIP.

If you're unsure, after your Istio controlplane is created, look at `kubectl get svc -n istio-system`. If they are stuck in Pending, there is an issue.

## Cert-Manager

For instance to route with Rancher, you need to install Rancher and create your certificate. For this example, it'll use a self-signed cert:

1. Install cert-manager:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
```

2. Create `cattle-system` namespace:
```bash
kubectl create ns cattle-system
```

3. Create cluster-issuer:
```bash
kubectl apply -f cert-manager/cluster-issuer.yaml
```

3. Update `cert-manager/certificate.yaml` with your Rancher hostname.

4. Create certificate:
```bash
kubectl apply -f cert-manager/certificate.yaml
```

## Istio

Now, you'll need to install Istio and install the istio operator. You'll need `istioctl`, which you can download (here)[https://github.com/istio/istio/releases] (NOTE: You might need expand the Assets list. Make sure you download istioctl and not istio.)

Once you have downloaded istioctl and added to your path, do the following:

1. Install istio operator:
```bash
istioctl operator init
```

2. Update the `istio/gateway.yaml`/`istio/virtual-service.yaml` with your Rancher hostname.

2. Create the Istio controlplane:
```bash
kubectl apply -f istio/controlplane.yaml
```

3. Wait for Istio pods to come online (wait for ingressgateway and istiod):
```bash
watch kubectl get pods -n istio-system
```

4. Create gateway:
```bash
kubectl apply -f istio/gateway.yaml
```

5. Create virtual service:
```bash
kubectl apply -f istio/virtual-service.yaml
```

## Rancher

When you install Rancher, since the TLS certificate should already exist, use the flag `--set ingress.tls.source=secret`.