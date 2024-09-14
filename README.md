**Install ETCD**

##
        mkdir /root/binaries
        cd /root/binaries
**download and copy binaries** 
##
        wget https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-amd64.tar.gz
        tar -xzvf etcd-v3.5.4-linux-amd64.tar.gz
        cd etcd-v3.5.4-linux-amd64
        cp etcd etcdctl /usr/local/bin/
**starting etcd from cli in the fore ground**
##
        etcd

**verification on the other ternimal we can add some data to etcd to check**
##
        etcdctl put db "uploading some text to etcd"
        etcdctl get db

This way of getting data without use of certificates is not sercure so we use certificates
Lets 1st generate certs and use them in etcd

**Steps Generate Client Certificate and Client Key**
##
    mkdir /root/certificates
    cd /root/certificates

##
        openssl genrsa -out ca.key 2048

        openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

        openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 1000

**step start etcd server**

##
        etcd --cert-file=/root/certificates/ca.crt --key-file=/root/certificates/ca.key --advertise-client-urls=https://127.0.0.1:2379 --listen-client-urls=https://127.0.0.1:2379

step verify
##
        etcdctl put course "cks"
**doesn't work**

step that works with https
##
        etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false put db "ck8s-security"
        etcdctl --endpoints=https://127.0.0.1:2379 --insecure-skip-tls-verify  --insecure-transport=false get db

so we use certificates as for security purpose

Running as daemon

create dir
##
                mkdir /var/lib/etcd
                chmod 700 /var/lib/etcd
**create systemd file**
##
                cat <<EOF | sudo tee /etc/systemd/system/etcd.service
                [Unit]
                Description=etcd
                Documentation=https://github.com/coreos

                [Service]
                ExecStart=/usr/local/bin/etcd \\
                  --cert-file=/root/certificates/etcd.crt \\
                  --key-file=/root/certificates/etcd.key \\
                  --trusted-ca-file=/root/certificates/ca.crt \\
                  --client-cert-auth \\
                  --listen-client-urls https://127.0.0.1:2379 \\
                  --advertise-client-urls https://127.0.0.1:2379 \\
                  --data-dir=/var/lib/etcd
                Restart=on-failure
                RestartSec=5

                [Install]
                WantedBy=multi-user.target
                EOF
**start etcd**
##
                systemctl daemon reload
                systemctl start etcd
                systemctl statud etcd

**check logs**
##
                journalctl -u etcd





