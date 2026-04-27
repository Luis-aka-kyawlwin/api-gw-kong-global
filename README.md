# Multi-Controller Kong Deployment Runbook

This runbook deploys 4 separate Kong Ingress Controller (KIC) and Gateway API data planes:

1. Global KIC/gateway:
`Release=kong-global`, `Namespace=kong`, `GatewayClass=kong-global`
2. Retail KIC/gateway:
`Release=kong-retail`, `Namespace=retail-banking`, `GatewayClass=kong-retail`
3. Payments KIC/gateway:
`Release=kong-payments`, `Namespace=payments`, `GatewayClass=kong-payments`
4. GRC KIC/gateway:
`Release=kong-grc`, `Namespace=grc`, `GatewayClass=kong-grc`

## Step 0: Pre-check

```bash
kubectl config current-context
helm version
kubectl get ns
```

## Step 1: Install Gateway API CRDs

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
kubectl get crd | grep gateway.networking.k8s.io
```

## Step 2: Remove old single-controller Kong (if any)

```bash
helm ls -A
helm -n kong uninstall kong || true
kubectl delete gatewayclass kong --ignore-not-found
```

## Step 3: Add/update Kong Helm repo

```bash
helm repo add kong https://charts.konghq.com || true
helm repo update
```

## Step 4: Install KIC + gateway for each domain

### 4.1 Global KIC (public entry)

```bash
helm upgrade --install kong-global kong/ingress \
  -n kong --create-namespace \
  --set controller.ingressController.ingressClass=kong-global \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-global \
  --set controller.ingressController.env.gateway_to_reconcile=kong/global-kong-api-gateway \
  --set gateway.proxy.type=LoadBalancer
```

### 4.2 Retail KIC

```bash
helm upgrade --install kong-retail kong/ingress \
  -n retail-banking --create-namespace \
  --set controller.ingressController.ingressClass=kong-retail \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-retail \
  --set controller.ingressController.env.gateway_to_reconcile=retail-banking/retail-banking-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

### 4.3 Payments KIC

```bash
helm upgrade --install kong-payments kong/ingress \
  -n payments --create-namespace \
  --set controller.ingressController.ingressClass=kong-payments \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-payments \
  --set controller.ingressController.env.gateway_to_reconcile=payments/payments-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

### 4.4 GRC KIC

```bash
helm upgrade --install kong-grc kong/ingress \
  -n grc --create-namespace \
  --set controller.ingressController.ingressClass=kong-grc \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-grc \
  --set controller.ingressController.env.gateway_to_reconcile=grc/grc-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

### 4.5 Verify Helm releases

```bash
helm ls -A
kubectl get pods -n kong
kubectl get pods -n retail-banking
kubectl get pods -n payments
kubectl get pods -n grc
```

## Step 5: Apply manifests in this repo

```bash
kubectl apply -f global-kong-gateway/0-kong-global-gateway-class.yaml
kubectl apply -f domain-specific-kong-gateway/0-kong-gateway-class.yaml
kubectl apply -f global-kong-gateway/1-kong-global-gateway.yaml
kubectl apply -f domain-specific-kong-gateway/

kubectl apply -f apps/1-retail-banking/
kubectl apply -f apps/2-payments/
kubectl apply -f apps/3-risk-compliance/

kubectl apply -f reference-grants/
kubectl apply -f global-kong-gateway/2-global-domain-httproutes.yaml
```

## Step 6: Verify Gateway API resources

```bash
kubectl get gatewayclass
kubectl get gateway -A
kubectl get httproute -A
kubectl get referencegrant -A
```

```bash
kubectl describe gateway -n kong global-kong-api-gateway
kubectl describe gateway -n retail-banking retail-banking-kong-api-gateway
kubectl describe gateway -n payments payments-kong-api-gateway
kubectl describe gateway -n grc grc-kong-api-gateway
```

## Step 7: Verify expected proxy service names

`global-kong-gateway/2-global-domain-httproutes.yaml` expects:

1. `kong-retail-gateway-proxy` in `retail-banking`
2. `kong-payments-gateway-proxy` in `payments`
3. `kong-grc-gateway-proxy` in `grc`

```bash
kubectl get svc -n retail-banking | grep gateway-proxy
kubectl get svc -n payments | grep gateway-proxy
kubectl get svc -n grc | grep gateway-proxy
```

## Step 8: Test traffic via global gateway only

### Option A: LoadBalancer IP

```bash
kubectl get svc -n kong
export GLOBAL_PROXY_IP=$(kubectl get svc -n kong kong-global-gateway-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "$GLOBAL_PROXY_IP"
```

```bash
curl -s -H 'Host: retail-banking.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: payments.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: grc.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
```

### Option B: Port-forward

```bash
kubectl -n kong port-forward svc/kong-global-gateway-proxy 18080:80
```

In another terminal:

```bash
curl -s -H 'Host: retail-banking.hellocloudbank.io' http://127.0.0.1:18080 | jq
curl -s -H 'Host: payments.hellocloudbank.io' http://127.0.0.1:18080 | jq
curl -s -H 'Host: grc.hellocloudbank.io' http://127.0.0.1:18080 | jq
```

## Step 9: Verify app services are internal only

```bash
kubectl get svc -n retail-banking customer-profile-svc -o wide
kubectl get svc -n payments transfer-svc -o wide
kubectl get svc -n grc fraud-svc -o wide
```

Expected: `TYPE=ClusterIP`.
