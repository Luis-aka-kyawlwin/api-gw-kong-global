# Multi-Controller Kong Deployment Plan

This guide deploys 4 Kong controller/data-plane releases:

1. Global: `kong-global` in namespace `kong` (`GatewayClass`: `kong-global`)
2. Retail: `kong-retail` in namespace `retail-banking` (`GatewayClass`: `kong-retail`)
3. Payments: `kong-payments` in namespace `payments` (`GatewayClass`: `kong-payments`)
4. GRC: `kong-grc` in namespace `grc` (`GatewayClass`: `kong-grc`)

## Prerequisites

1. Gateway API CRDs are installed.
2. Helm is installed.
3. You are in `/workspace/apex/12-apigateway/06-kong-global-api-gateway`.

## Step 1: Install Gateway API CRDs

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

## Step 2: Add Kong Helm repo

```bash
helm repo add kong https://charts.konghq.com
helm repo update
```

## Step 3: Install 4 Kong releases

### 3.1 Global Kong (public entry)

```bash
helm upgrade --install kong-global kong/ingress \
  -n kong --create-namespace \
  --set controller.ingressController.ingressClass=kong-global \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-global \
  --set controller.ingressController.env.gateway_to_reconcile=kong/global-kong-api-gateway \
  --set gateway.proxy.type=LoadBalancer
```

### 3.2 Retail Kong (domain-specific)

```bash
helm upgrade --install kong-retail kong/ingress \
  -n retail-banking --create-namespace \
  --set controller.ingressController.ingressClass=kong-retail \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-retail \
  --set controller.ingressController.env.gateway_to_reconcile=retail-banking/retail-banking-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

### 3.3 Payments Kong (domain-specific)

```bash
helm upgrade --install kong-payments kong/ingress \
  -n payments --create-namespace \
  --set controller.ingressController.ingressClass=kong-payments \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-payments \
  --set controller.ingressController.env.gateway_to_reconcile=payments/payments-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

### 3.4 GRC Kong (domain-specific)

```bash
helm upgrade --install kong-grc kong/ingress \
  -n grc --create-namespace \
  --set controller.ingressController.ingressClass=kong-grc \
  --set controller.ingressController.env.gateway_api_controller_name=konghq.com/kic-gateway-controller-grc \
  --set controller.ingressController.env.gateway_to_reconcile=grc/grc-kong-api-gateway \
  --set gateway.proxy.type=ClusterIP
```

## Step 4: Apply manifests from this repo

```bash
kubectl apply -f distributed-gateway/0-kong-gateway-class.yaml
kubectl apply -f distributed-gateway/
kubectl apply -f apps/1-retail-banking/5-customer-profile-httproute.yaml
kubectl apply -f apps/2-payments/5-transfer-httproute.yaml
kubectl apply -f apps/3-risk-compliance/5-fraud-httproute.yaml
kubectl apply -f reference-grants/
kubectl apply -f global-kong-gateway/
```

## Step 5: Verify resources

```bash
kubectl get gatewayclass
kubectl get gateway -A
kubectl get httproute -A
kubectl get referencegrant -A
```

Check domain proxy Services expected by `global-kong-gateway/2-global-domain-httproutes.yaml`:

```bash
kubectl get svc -n retail-banking | grep gateway-proxy
kubectl get svc -n payments | grep gateway-proxy
kubectl get svc -n grc | grep gateway-proxy
```

Expected service names:

1. `kong-retail-gateway-proxy`
2. `kong-payments-gateway-proxy`
3. `kong-grc-gateway-proxy`

If your release generated different names, update `global-kong-gateway/2-global-domain-httproutes.yaml`.

## Step 6: Test traffic through global gateway only

Get global proxy IP:

```bash
export GLOBAL_PROXY_IP=$(kubectl get svc -n kong kong-global-gateway-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "$GLOBAL_PROXY_IP"
```

Run curl tests:

```bash
curl -s -H 'Host: retail-banking.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: payments.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: grc.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
```

Optional local test via port-forward:

```bash
kubectl -n kong port-forward svc/kong-global-gateway-proxy 18080:80
curl -s -H 'Host: retail-banking.hellocloudbank.io' http://127.0.0.1:18080 | jq
curl -s -H 'Host: payments.hellocloudbank.io' http://127.0.0.1:18080 | jq
curl -s -H 'Host: grc.hellocloudbank.io' http://127.0.0.1:18080 | jq
```
