# docker-lvs


Run from Kubernetes

```
Demo1:
==============================================================================
kind: Deployment
apiVersion: apps/v1
metadata:
  annotations:
    miaoyun.io/os: linux
  labels:
    app: "{{ .__APP__ }}-master"
    miaoyun.io/application.name: "{{ .__APP__ }}"
    miaoyun.io/application.workload: "{{ .__APP__ }}-master"
  name: "{{ .__APP__ }}-master"
spec:
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: "{{ .__APP__ }}-master"
        miaoyun.io/application.name: "{{ .__APP__ }}"
        miaoyun.io/application.workload: "{{ .__APP__ }}-master"
    spec:
      securityContext: {}
      terminationGracePeriodSeconds: 30
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - args:
            - /app/run.sh
          securityContext:
            allowPrivilegeEscalation: true
            readOnlyRootFilesystem: false
            runAsNonRoot: false
            privileged: true
          ports:
            - containerPort: 11234
              protocol: TCP
          env:
            - name: PATH
              value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
            - name: LVS_TYPE
              value: MASTER
            - name: LVS_PRI
              value: "100"
            - name: LVS_VIP
              value: {{ .LVS_VIP }}
            - name: LVS_NIC
              value: {{ .LVS_NIC }}
            - name: LVS_PORT
              value: "22345"
          volumeMounts:
            - name: volume-akgg5a
              mountPath: /lib/modules
            - mountPath: /etc/localtime
              name: chiwen-sync-timezone
              readOnly: true
            - name: volume-rffly5
              subPath: keepalived.tmpl
              mountPath: /app/keepalived.tmpl
            - name: volume-iv93j8
              subPath: run.sh
              mountPath: /app/run.sh
          name: container-288824
          image: 10.192.31.1:18443/default/miaoyun-lvs:v10
          pullingSpec: false
          tty: false
      hostNetwork: true
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                      - miaoyun-flannel-master
      volumes:
        - name: volume-akgg5a
          hostPath:
            path: /lib/modules
        - name: volume-rffly5
          configMap:
            name: "{{ .__APP__ }}-keepalived.conf"
            items:
              - key: keepalived.tmpl
                path: keepalived.tmpl
            defaultMode: null
        - name: volume-iv93j8
          configMap:
            name: "{{ .__APP__ }}-keepalived-startup-sh"
            items:
              - key: run.sh
                path: run.sh
            defaultMode: 493
        - hostPath:
            path: /etc/localtime
          name: chiwen-sync-timezone
  selector:
    matchLabels:
      app: "{{ .__APP__ }}-master"
      miaoyun.io/application.name: "{{ .__APP__ }}"
      miaoyun.io/application.workload: "{{ .__APP__ }}-master"
  replicas: 1

---
kind: Deployment
apiVersion: apps/v1
metadata:
  annotations:
    miaoyun.io/os: linux
  labels:
    app: "{{ .__APP__ }}-backup"
    miaoyun.io/application.name: "{{ .__APP__ }}"
    miaoyun.io/application.workload: "{{ .__APP__ }}-backup"
  name: "{{ .__APP__ }}-backup"
spec:
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: "{{ .__APP__ }}-backup"
        miaoyun.io/application.name: "{{ .__APP__ }}"
        miaoyun.io/application.workload: "{{ .__APP__ }}-backup"
    spec:
      securityContext: {}
      terminationGracePeriodSeconds: 30
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - args:
            - /app/run.sh
          securityContext:
            allowPrivilegeEscalation: true
            readOnlyRootFilesystem: false
            runAsNonRoot: false
            privileged: true
          ports:
            - containerPort: 11234
              protocol: TCP
          env:
            - name: PATH
              value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
            - name: LVS_TYPE
              value: BACKUP
            - name: LVS_PRI
              value: "90"
            - name: LVS_VIP
              value: {{ .LVS_VIP }}
            - name: LVS_NIC
              value: {{ .LVS_NIC }}
            - name: LVS_PORT
              value: "22345"
          volumeMounts:
            - name: volume-gxbqrq
              mountPath: /lib/modules
            - mountPath: /etc/localtime
              name: chiwen-sync-timezone
              readOnly: true
            - name: volume-qriszi
              subPath: keepalived.tmpl
              mountPath: /app/keepalived.tmpl
            - name: volume-cs3rvl
              subPath: run.sh
              mountPath: /app/run.sh
          name: container-047649
          image: 10.192.31.1:18443/default/miaoyun-lvs:v10
          pullingSpec: false
          tty: false
      hostNetwork: true
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                      - miaoyun-flannel-node1
      volumes:
        - name: volume-gxbqrq
          hostPath:
            path: /lib/modules
        - name: volume-qriszi
          configMap:
            name: "{{ .__APP__ }}-keepalived.conf"
            items:
              - key: keepalived.tmpl
                path: keepalived.tmpl
            defaultMode: null
        - name: volume-cs3rvl
          configMap:
            name: "{{ .__APP__ }}-keepalived-startup-sh"
            items:
              - key: run.sh
                path: run.sh
            defaultMode: 493
        - hostPath:
            path: /etc/localtime
          name: chiwen-sync-timezone
  selector:
    matchLabels:
      app: "{{ .__APP__ }}-backup"
      miaoyun.io/application.name: "{{ .__APP__ }}"
      miaoyun.io/application.workload: "{{ .__APP__ }}-backup"
  replicas: 1

---
kind: Deployment
apiVersion: apps/v1
metadata:
  annotations:
    miaoyun.io/os: linux
  labels:
    app: "{{ .__APP__ }}-realserver"
    miaoyun.io/application.name: "{{ .__APP__ }}"
    miaoyun.io/application.workload: "{{ .__APP__ }}-realserver"
  name: "{{ .__APP__ }}-realserver"
spec:
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: "{{ .__APP__ }}-realserver"
        miaoyun.io/application.name: "{{ .__APP__ }}"
        miaoyun.io/application.workload: "{{ .__APP__ }}-realserver"
    spec:
      securityContext: {}
      terminationGracePeriodSeconds: 30
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - args:
            - nginx
            - -g
            - daemon off;
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: false
            runAsNonRoot: false
            privileged: false
          ports:
            - containerPort: 80
              protocol: TCP
          env:
            - name: PATH
              value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
          volumeMounts:
            - name: volume-n3nz9a
              subPath: nginx.conf
              mountPath: /etc/nginx/nginx.conf
          name: container-392965
          image: 10.192.31.1:18443/default/docker-2048:latest
          pullingSpec: false
      hostNetwork: true
      initContainers:
        - args:
            - sh
            - /realserver.sh
            - start
          securityContext:
            allowPrivilegeEscalation: true
            readOnlyRootFilesystem: false
            runAsNonRoot: false
            privileged: true
          env:
            - name: LVS_VIP
              value: {{ .LVS_VIP }}
          volumeMounts:
            - name: volume-dtmtrq
              subPath: realserver.sh
              mountPath: /realserver.sh
          name: init-container-532817
          image: 10.192.31.1:18443/default/busybox:1.28
      volumes:
        - name: volume-n3nz9a
          configMap:
            name: "{{ .__APP__ }}-nginx-conf"
            items:
              - key: nginx.conf
                path: nginx.conf
            defaultMode: null
        - configMap:
            defaultMode: 493
            items:
              - key: realserver.sh
                path: realserver.sh
            name: "{{ .__APP__ }}-realserver-sh"
          name: volume-dtmtrq
  selector:
    matchLabels:
      app: "{{ .__APP__ }}-realserver"
      miaoyun.io/application.name: "{{ .__APP__ }}"
      miaoyun.io/application.workload: "{{ .__APP__ }}-realserver"
  replicas: 1

---
kind: Service
apiVersion: v1
metadata:
  annotations:
    miaoyun.io/service.url: /
  labels:
    miaoyun.io/application.name: "{{ .__APP__ }}"
    app: "{{ .__APP__ }}-realserver"
  name: "{{ .__APP__ }}-realserver-svc"
spec:
  type: NodePort
  selector:
    miaoyun.io/application.name: "{{ .__APP__ }}"
    app: "{{ .__APP__ }}-realserver"
  ports:
    - port: 22345
      targetPort: 22345
      protocol: TCP
      name: tcp-22345

---
apiVersion: v1
metadata:
  name: "{{ .__APP__ }}-keepalived.conf"
  labels:
    miaoyun.io/application.name: "{{ .__APP__ }}"
kind: ConfigMap
data:
  keepalived.tmpl: |
    ! Configuration File for keepalived
    global_defs {
       notification_email {
         acassen@miaoyun.io
         failover@miaoyun.io
         sysadmin@miaoyun.io
       }
       notification_email_from wenqiang.feng@miaoyun.io
       smtp_server 10.221.161.1
       smtp_connect_timeout 30
       router_id MiaoYun_LVS
       vrrp_mcast_group4 224.0.95.88
    }
              
    vrrp_instance VI_1 {
        state LVS_TYPE
        interface LVS_NIC 
        virtual_router_id 88
        priority LVS_PRI
        advert_int 1
        authentication {
            auth_type PASS  
            auth_pass miaoyun.io 
        }
        virtual_ipaddress {
    	LVS_VIP
        }
    }
              
    virtual_server LVS_VIP LVS_PORT { 
        delay_loop 6
        lb_algo rr          
        lb_kind DR         
        nat_mask 255.255.255.224
        persistence_timeout 50  
        protocol TCP       

        real_server 222.66.209.229 LVS_PORT {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }
        real_server 222.66.209.230 LVS_PORT {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }  
    }

---
apiVersion: v1
metadata:
  name: "{{ .__APP__ }}-keepalived-startup-sh"
  labels:
    miaoyun.io/application.name: "{{ .__APP__ }}"
kind: ConfigMap
data:
  run.sh: >
    #!/bin/bash


    cp /app/keepalived.tmpl /tmp/keepalived.tmpl


    sed -i "s/LVS_TYPE/${LVS_TYPE}/g" /tmp/keepalived.tmpl

    sed -i "s/LVS_PRI/${LVS_PRI}/g" /tmp/keepalived.tmpl

    sed -i "s/LVS_VIP/${LVS_VIP}/g" /tmp/keepalived.tmpl

    sed -i "s/LVS_NIC/${LVS_NIC}/g" /tmp/keepalived.tmpl

    sed -i "s/LVS_PORT/${LVS_PORT}/g" /tmp/keepalived.tmpl

    cp /tmp/keepalived.tmpl /etc/keepalived/keepalived.conf

    keepalived --dont-fork --dump-conf --log-console --log-detail -f /etc/keepalived/keepalived.conf

---
apiVersion: v1
metadata:
  name: "{{ .__APP__ }}-realserver-sh"
  labels:
    miaoyun.io/application.name: "{{ .__APP__ }}"
kind: ConfigMap
data:
  realserver.sh: |
    #!/bin/bash
    VIP=$LVS_VIP

    case "$1" in
    start)
           ifconfig lo:0 $VIP netmask 255.255.255.255 broadcast $VIP
           ip route add $VIP dev lo:0
           echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
           echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
           echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
           echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
           echo "RealServer Start OK"
           ;;
    stop)
           ifconfig lo:0 down
           ip route del $VIP dev lo:0
           echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
           echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
           echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
           echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
           echo "RealServer Stoped"
           ;;
           *)
           echo "Usage: $0 {start|stop}"
           exit 1
    esac
    exit 0

---
apiVersion: v1
metadata:
  name: "{{ .__APP__ }}-nginx-conf"
  labels:
    miaoyun.io/application.name: "{{ .__APP__ }}"
kind: ConfigMap
data:
  nginx.conf: >
    
    #user  nobody;

    worker_processes  1;


    #error_log  logs/error.log;

    #error_log  logs/error.log  notice;

    #error_log  logs/error.log  info;


    #pid        logs/nginx.pid;



    events {
        worker_connections  1024;
    }



    http {
        include       mime.types;
        default_type  application/octet-stream;

        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';

        #access_log  logs/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        #keepalive_timeout  0;
        keepalive_timeout  65;

        #gzip  on;

        server {
            listen       22345;
            server_name  localhost;

            #charset koi8-r;

            #access_log  logs/host.access.log  main;

            location / {
                root   html;
                index  index.html index.htm;
            }

            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
            #    root           html;
            #    fastcgi_pass   127.0.0.1:9000;
            #    fastcgi_index  index.php;
            #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            #    include        fastcgi_params;
            #}

            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }


        # another virtual host using mix of IP-, name-, and port-based configuration
        #
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;

        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}


        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;

        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;

        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;

        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;

        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}

    }
```

LVS Dockerfile
==============================================================================
```
FROM centos:6.6

MAINTAINER ErganziWudi

RUN yum install -y ipvsadm keepalived
RUN rm -rf /etc/keepalived/keepalived.conf

EXPOSE 11234

CMD ["/app/run.sh"]
```

