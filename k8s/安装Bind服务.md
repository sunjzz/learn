K8S环境部署

五台虚拟机

10.4.7.11	hostname:hdss7-11.host.com

10.4.7.12	hostname:hdss7-12.host.com

10.4.7.21	hostname:hdss7-21.host.com

10.4.7.22	hostname:hdss7-22.host.com

10.4.7.200	hostname:hdss7-200.host.com

### 安装Bind服务

10.4.7.11 yum install -y bind9

bind服务配置：

vi /etc/named.conf，以下是修改配置文件内容修改：

```
        listen-on port 53 { 10.4.7.11; };
        //listen-on-v6 port 53 { ::1; };
        //allow-query     { localhost; };
        allow-query     { any; };
        forwarders      { 10.4.7.1; };
        
        recursion yes;
        //dnssec-enable yes;
        //dnssec-validation yes;
        dnssec-enable no;
        dnssec-validation no;
```

vi /etc/named.rfc1912.zones

```
zone "host.com" IN {
        type master;
        file "host.com.zone";
        allow-update { 10.4.7.11; };
};

zone "od.com" IN {
        type master;
        file "od.com.zone";
        allow-update { 10.4.7.11; };
};
```

vi /var/named/host.com.zone

```
$ORIGIN host.com.
$TTL 600        ; 10 minutes
@       IN SOA  dns.host.com. dnsadmin.host.com. (
                20210225        ; serial
                10800           ; refresh (3 hours)
                900             ; refresh (15 minutes)
                604800          ; expire (1 week)
                86400           ; minimum (1 day)
                )
           NS   dns.host.com.
$TTL 60 ; 1 minute
dns             A       10.4.7.11
HDSS7-11        A       10.4.7.11
HDSS7-12        A       10.4.7.12
HDSS7-21        A       10.4.7.21
HDSS7-22        A       10.4.7.22
HDSS7-200       A       10.4.7.200
```

vi /var/named/od.com.zone

```
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@       IN SOA  dns.od.com. dnsadmin.od.com. (
                20210225        ; serial
                10800           ; refresh (3 hours)
                900             ; refresh (15 minutes)
                604800          ; expire (1 week)
                86400           ; minimum (1 day)
                )
           NS   dns.od.com.
$TTL 60 ; 1 minute
dns             A       10.4.7.11
```



systemctl start named



修改五台主机的网卡配置文件，DNS1=10.4.7.11

vi /etc/resolve.conf，添加

search host.com	#表示短域名

### 证书签发

 访问https://pkg.cfssl.org/

```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/bin/cfssl-certinfo_linux-amd64
chmod +x /usr/bin/cfssl*
```

根证书签发：

```
vim /opt/certs/ca-csr.json
{
    "CN": "OldboyEdu",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ],
    "ca": {
        "expiry": "175200h"
    }
}
```

CN: Common Name ,浏览器使用该字段验证网站是否合法, 一般写的是域名。非常重要。浏览器使

用该字段验证网站是否合法

C: Country,国家

ST:State,州，省

L: Locality ,地区,城市

O: Organization Name ,组织名称,公司名称

OU: Organization Unit Name ,组织单位名称,公司部门

"expiry": "175200h"  # 过期时间（20年）



`cfssl gencert -initca ca-csr.json | cfssl-json -bare ca`



