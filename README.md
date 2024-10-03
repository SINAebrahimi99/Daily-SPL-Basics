# Daily-SPL
Some Splunk queries that might be useful for SOC daily routine (SOC L1 Diaries!)

All of these queries are just usecases that ive wrote to give you ideas and key concepts that could help you in your SOC environment

Each query can be better and more advanced , it only depends on you and your skills (+ your Company and your Log plicies).

### Incoming Traffic from Internet
#### Incoming allowed src_ip
```
index="your index name" eventtype=traffic action!=blocked 
NOT src_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
| timechart count by src_ip useother=false
```
#### Incoming allowed src_port
```
index="your index name" eventtype=traffic action!=blocked 
NOT src_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
| timechart count by src_port useother=false
```
### Outgoing Traffic from Inside
#### Outgoing allowed src_ip
```
index="your index name" eventtype=traffic action!=blocked 
src_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
NOT dest_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
| timechart count by src_ip useother=false
```
#### Outgoing allowed src_port
```
index="your index name" eventtype=traffic action!=blocked 
src_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
NOT dest_ip IN ("192.168.0.0/16", "172.16.0.0/12", "10.0.0.0/8")
| timechart count by src_port useother=false
```

for all of these we also must monitor the dest_ip and dest_port , I just wont rewrite them again 
 and all these queries for `blocked traffic ` 
 
the `blocked traffic` is very important to monitor 

The main goal of these charts is to monitor the baseline traffic of your company and detect any abnormal traffic activity.

### Important Services like DNS or HTTP

Its critical to monitor the traffic of important services to avoid any Denial of Service or data exfiltration through a specific app.

```
index="your index name" sourcetype=traffic
service IN (HTTP, HTTPS) 
| timechart count by src_ip useother=false
```
```
index="your index name" sourcetype=traffic service IN (HTTP, HTTPS) 
| stats sum(bytes_in) as rcv, sum(bytes_out) as sent by src_ip
```


### Scanning
#### Port Scan
```
index=* (tag=network AND tag=communicate) earliest=-1h
| stats dc(dest_port) as num_dest_port, dc(dest_ip) as num_dest_ip by src_ip
| where num_dest_port > 50 OR num_dest_ip > 50
```
#### Ping Scan
```
index="your_index" sourcetype="your_sourcetype" icmp_type=8 
| timechart span=1m count by src_ip 
| where count > 10
```
we also have other scans like SYN, ARP, SV and ... scans.

any Scanning acivity must be monitored.

