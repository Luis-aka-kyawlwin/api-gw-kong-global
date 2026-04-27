NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kong-global     kong            1               2026-04-27 18:21:12.552556057 +0000 UTC deployed        ingress-0.24.0  3.9        
kong-grc        grc             1               2026-04-27 18:21:59.067802148 +0000 UTC deployed        ingress-0.24.0  3.9        
kong-payments   payments        1               2026-04-27 18:21:45.871196043 +0000 UTC deployed        ingress-0.24.0  3.9        
kong-retail     retail-banking  1               2026-04-27 18:21:29.5739782 +0000 UTC   deployed        ingress-0.24.0  3.9        
NAME                                     READY   STATUS    RESTARTS   AGE
kong-global-controller-856557d49-qghv4   1/1     Running   0          2m57s
kong-global-gateway-798f478556-qc6pk     1/1     Running   0          2m57s
NAME                                      READY   STATUS    RESTARTS   AGE
kong-retail-controller-68fdbc9d95-grxvl   1/1     Running   0          2m41s
kong-retail-gateway-586cb6b4d8-5ndmr      1/1     Running   0          2m41s
NAME                                        READY   STATUS    RESTARTS   AGE
kong-payments-controller-7bb64c6ff5-7shqt   1/1     Running   0          2m25s
kong-payments-gateway-6c97d9cb64-jb764      1/1     Running   0          2m25s
NAME                                   READY   STATUS    RESTARTS   AGE
kong-grc-controller-77cb8847cd-z9pf9   1/1     Running   0          2m11s
kong-grc-gateway-6f8d77b97b-98t7j      1/1     Running   0          2m11s

###
vagrant@apex-vagrant-box:~$ kubectl get gatewayclass
kubectl get gateway -A
kubectl get httproute -A
kubectl get referencegrant -A
NAME            CONTROLLER                                   ACCEPTED   AGE
kong-global     konghq.com/kic-gateway-controller-global     True       60s
kong-grc        konghq.com/kic-gateway-controller-grc        True       60s
kong-payments   konghq.com/kic-gateway-controller-payments   True       60s
kong-retail     konghq.com/kic-gateway-controller-retail     True       60s
NAMESPACE        NAME                              CLASS           ADDRESS          PROGRAMMED   AGE
grc              grc-kong-api-gateway              kong-grc                         True         59s
kong             global-kong-api-gateway           kong-global     172.18.255.180   True         61s
payments         payments-kong-api-gateway         kong-payments                    True         59s
retail-banking   retail-banking-kong-api-gateway   kong-retail                      True         59s
NAMESPACE        NAME                              HOSTNAMES                              AGE
grc              fraud-httproute                   ["grc.hellocloudbank.io"]              2m22s
kong             grc-global-httproute              ["grc.hellocloudbank.io"]              53s
kong             payments-global-httproute         ["payments.hellocloudbank.io"]         53s
kong             retail-banking-global-httproute   ["retail-banking.hellocloudbank.io"]   53s
payments         transfer-httproute                ["payments.hellocloudbank.io"]         2m37s
retail-banking   customer-profile-httproute        ["retail-banking.hellocloudbank.io"]   2m52s
NAMESPACE        NAME                                        AGE
grc              allow-kong-global-route-to-grc-proxy        55s
payments         allow-kong-global-route-to-payments-proxy   55s
retail-banking   allow-kong-global-route-to-retail-proxy     55s

###
vagrant@apex-vagrant-box:~$ kubectl get svc -n retail-banking | grep gateway-proxy
kubectl get svc -n payments | grep gateway-proxy
kubectl get svc -n grc | grep gateway-proxy
kong-retail-gateway-proxy                   ClusterIP   10.132.242.128   <none>        80/TCP,443/TCP                  8m39s
kong-payments-gateway-proxy                   ClusterIP   10.132.219.15    <none>        80/TCP,443/TCP                  8m23s
kong-grc-gateway-proxy                   ClusterIP   10.132.6.173     <none>        80/TCP,443/TCP                  8m10s
vagrant@apex-vagrant-box:~$ 



###
vagrant@apex-vagrant-box:~$ curl -s -H 'Host: retail-banking.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: payments.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
curl -s -H 'Host: grc.hellocloudbank.io' "http://$GLOBAL_PROXY_IP" | jq
{
  "name": "customer-profile-svc",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.252.2.5"
  ],
  "start_time": "2026-04-27T18:31:10.369523",
  "end_time": "2026-04-27T18:31:10.384123",
  "duration": "14.600229ms",
  "body": "HelloCloudBank | Retail Banking | customer-profile-svc",
  "upstream_calls": {
    "http://account-svc.retail-banking.svc.cluster.local:8082": {
      "name": "account-svc",
      "uri": "http://account-svc.retail-banking.svc.cluster.local:8082",
      "type": "HTTP",
      "ip_addresses": [
        "10.252.1.6"
      ],
      "start_time": "2026-04-27T18:31:10.375155",
      "end_time": "2026-04-27T18:31:10.382185",
      "duration": "7.029525ms",
      "headers": {
        "Content-Length": "952",
        "Content-Type": "text/plain; charset=utf-8",
        "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
      },
      "body": "HelloCloudBank | Retail Banking | account-svc",
      "upstream_calls": {
        "http://statement-svc.retail-banking.svc.cluster.local:8083": {
          "name": "statement-svc",
          "uri": "http://statement-svc.retail-banking.svc.cluster.local:8083",
          "type": "HTTP",
          "ip_addresses": [
            "10.252.3.6"
          ],
          "start_time": "2026-04-27T18:31:10.380002",
          "end_time": "2026-04-27T18:31:10.381434",
          "duration": "1.432386ms",
          "headers": {
            "Content-Length": "298",
            "Content-Type": "text/plain; charset=utf-8",
            "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
          },
          "body": "HelloCloudBank | Retail Banking | statement-svc",
          "code": 200
        }
      },
      "code": 200
    }
  },
  "code": 200
}
{
  "name": "transfer-svc",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.252.3.7"
  ],
  "start_time": "2026-04-27T18:31:10.411119",
  "end_time": "2026-04-27T18:31:10.429166",
  "duration": "18.046517ms",
  "body": "HelloCloudBank | Payments | transfer-svc",
  "upstream_calls": {
    "http://payment-gateway-svc.payments.svc.cluster.local:7072": {
      "name": "payment-gateway-svc",
      "uri": "http://payment-gateway-svc.payments.svc.cluster.local:7072",
      "type": "HTTP",
      "ip_addresses": [
        "10.252.1.7"
      ],
      "start_time": "2026-04-27T18:31:10.420061",
      "end_time": "2026-04-27T18:31:10.428252",
      "duration": "8.190707ms",
      "headers": {
        "Content-Length": "916",
        "Content-Type": "text/plain; charset=utf-8",
        "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
      },
      "body": "HelloCloudBank | Payments | payment-gateway-svc",
      "upstream_calls": {
        "http://fx-svc.payments.svc.cluster.local:7073": {
          "name": "fx-svc",
          "uri": "http://fx-svc.payments.svc.cluster.local:7073",
          "type": "HTTP",
          "ip_addresses": [
            "10.252.2.6"
          ],
          "start_time": "2026-04-27T18:31:10.426068",
          "end_time": "2026-04-27T18:31:10.426581",
          "duration": "512.975µs",
          "headers": {
            "Content-Length": "278",
            "Content-Type": "text/plain; charset=utf-8",
            "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
          },
          "body": "HelloCloudBank | Payments | fx-svc",
          "code": 200
        }
      },
      "code": 200
    }
  },
  "code": 200
}
{
  "name": "fraud-svc",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.252.1.8"
  ],
  "start_time": "2026-04-27T18:31:10.488471",
  "end_time": "2026-04-27T18:31:10.506585",
  "duration": "18.114332ms",
  "body": "HelloCloudBank | GRC | fraud-svc",
  "upstream_calls": {
    "http://audit-svc.grc.svc.cluster.local:6062": {
      "name": "audit-svc",
      "uri": "http://audit-svc.grc.svc.cluster.local:6062",
      "type": "HTTP",
      "ip_addresses": [
        "10.252.3.8"
      ],
      "start_time": "2026-04-27T18:31:10.496547",
      "end_time": "2026-04-27T18:31:10.505790",
      "duration": "9.242761ms",
      "headers": {
        "Content-Length": "900",
        "Content-Type": "text/plain; charset=utf-8",
        "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
      },
      "body": "HelloCloudBank | GRC | audit-svc",
      "upstream_calls": {
        "http://sanction-svc.grc.svc.cluster.local:6063": {
          "name": "sanction-svc",
          "uri": "http://sanction-svc.grc.svc.cluster.local:6063",
          "type": "HTTP",
          "ip_addresses": [
            "10.252.2.7"
          ],
          "start_time": "2026-04-27T18:31:10.503289",
          "end_time": "2026-04-27T18:31:10.504129",
          "duration": "840.346µs",
          "headers": {
            "Content-Length": "285",
            "Content-Type": "text/plain; charset=utf-8",
            "Date": "Mon, 27 Apr 2026 18:31:10 GMT"
          },
          "body": "HelloCloudBank | GRC | sanction-svc",
          "code": 200
        }
      },
      "code": 200
    }
  },
  "code": 200
}
vagrant@apex-vagrant-box:~$ 

###

