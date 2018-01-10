# keepalived-vip

Technical Background : https://github.com/kweisamx/kubernetes-keepalived-vip
1. Run Cluster Role : vip-rbac.yaml (only do it one)
2. Run DaemonSet : vip-ds.yaml (only do it one)
3. Run your deployment & services
4. Run ConfigMap : vip-configmap.yaml
```

建立daemonset

```sh
$ kubectl get daemonset kube-keepalived-vip 
NAME                  DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE-SELECTOR   AGE
kube-keepalived-vip   5         5         5         5            5           
```

檢查一下配置狀態

```sh
kubectl get pod -o wide |grep keepalive
kube-keepalived-vip-c4sxw         1/1       Running            0          23h       10.87.2.6    10.87.2.6
kube-keepalived-vip-c9p7n         1/1       Running            0          23h       10.87.2.8    10.87.2.8
kube-keepalived-vip-psdp9         1/1       Running            0          23h       10.87.2.10   10.87.2.10
kube-keepalived-vip-xfmxg         1/1       Running            0          23h       10.87.2.12   10.87.2.12
kube-keepalived-vip-zjts7         1/1       Running            3          23h       10.87.2.4    10.87.2.4
```
可以隨機挑一個pod,去看裏面的配置

```sh
 $ kubectl exec kube-keepalived-vip-c4sxw cat /etc/keepalived/keepalived.conf
 
 
global_defs {
  vrrp_version 3
  vrrp_iptables KUBE-KEEPALIVED-VIP
}
 
vrrp_instance vips {
  state BACKUP
  interface eno1
  virtual_router_id 50
  priority 103
  nopreempt
  advert_int 1
 
  track_interface {
    eno1
  }
 
 
 
  virtual_ipaddress { 
    10.87.2.50
  }
}
 
 
# Service: default/nginx
virtual_server 10.87.2.50 80 { //此為service開的口
  delay_loop 5
  lvs_sched wlc
  lvs_method NAT
  persistence_timeout 1800
  protocol TCP
 
 
  real_server 10.2.49.30 8080 { //這裏說明 pod的真實狀況
    weight 1
    TCP_CHECK {
      connect_port 80
      connect_timeout 3
    }
  }
 
}
 
```

最後我們去測試這功能

```sh
$ curl  10.87.2.50
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

10.87.2.50:80(我們假設的VIP,實際上其實沒有node是用這IP)即可幫我們導向這個service


以上的程式代碼都在[github](https://github.com/kubernetes/contrib/tree/master/keepalived-vip)上可以找到。

## 參考文檔

- [kweisamx/kubernetes-keepalived-vip](https://github.com/kweisamx/kubernetes-keepalived-vip)
- [kubernetes/keepalived-vip](https://github.com/kubernetes/contrib/tree/master/keepalived-vip)
