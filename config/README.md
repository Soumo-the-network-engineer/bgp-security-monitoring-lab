# Configuration

Sanitized configuration reference for the BGP Security Monitoring Lab. Runtime credentials are redacted.

## 7.1 MAIN-INT-RTR Base Configuration

```cisco
hostname MAIN-INT-RTR
ip domain name bgp-lab.local
username bgpcollector privilege 15 secret <LAB_PASSWORD_REDACTED>
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
!
interface Loopback10
 ip address 100.100.100.1 255.255.255.0
!
interface GigabitEthernet0/0
 ip address 10.10.10.1 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 11.11.11.1 255.255.255.0
!
interface GigabitEthernet0/2.10
 encapsulation dot1Q 10
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
!
interface GigabitEthernet0/3
 ip address dhcp
 ip nat outside
 ip virtual-reassembly in
!
interface GigabitEthernet0/4
 ip address 12.12.12.1 255.255.255.0
!
router bgp 100
 ! Legitimate lab prefixes include:
 network 10.10.10.0 mask 255.255.255.0
 network 11.11.11.0 mask 255.255.255.0
 network 100.100.100.0 mask 255.255.255.0
!
logging source-interface GigabitEthernet0/2.10
logging host 192.168.20.10
ntp source GigabitEthernet0/2.10
ntp server 192.168.20.11 prefer

```

> The exact BGP neighbor statements were maintained in the live lab but were not fully preserved in the session excerpts used for this report. The report therefore does not invent missing interface IP values or neighbor addresses beyond the verified topology table.

## 7.2 AS200 Router-Reflector Design
AS200 uses vIOS4 and vIOS3 as RR nodes. vIOS7 and vIOS6 act as RR clients. The lab intentionally uses iBGP only inside AS200, with no dependency on a separate IGP for the demonstration. MAIN peers with vIOS7 over 10.10.10.0/24 and vIOS6 over 11.11.11.0/24.
AS200 logical role summary
-------------------------
vIOS4 = RR1
vIOS3 = RR2
vIOS7 = RR1 client + prefix-hijack injector
vIOS6 = RR2 client

Core route-reflector requirement:
- Client sessions are marked as route-reflector-client on the RR nodes.
- AS200 remains a single iBGP AS.


## 7.3 vIOS7 Baseline and Hijack Test Configuration
interface Loopback100
 ip address 70.70.70.7 255.255.255.255
!
router bgp 200
 network 70.70.70.7 mask 255.255.255.255

Controlled prefix-hijack injection
conf t
interface Loopback200
 ip address 100.100.100.254 255.255.255.0
 no shutdown
router bgp 200
 network 100.100.100.0 mask 255.255.255.0
end

Controlled prefix-hijack withdrawal
conf t
router bgp 200
 no network 100.100.100.0 mask 255.255.255.0
exit
interface Loopback200
 shutdown
end


## 7.4 vIOS8 Route-Leak Test
vIOS8 is AS300 and receives the legitimate route from AS100 as path 100. The test also exposes an observed path 200 100, which is treated by the parser as a possible route leak.
vIOS8 route-state parser output observed during validation:
{"router":"vIOS8","prefix":"100.100.100.0/24","as_path":"100","origin_as":"100","possible_route_leak":false}
{"router":"vIOS8","prefix":"100.100.100.0/24","as_path":"200 100","origin_as":"100","possible_route_leak":true}


# 8. Connectivity, NAT and Management Network

## 8.1 Management / Lab Route
After the environment was moved to the 10.158.47.0/24 management network, Windows required a persistent route for the lab service subnet. The route used MAIN at 10.158.47.88 as the gateway for 192.168.20.0/24.
Windows (Administrator CMD)
route -p add 192.168.20.0 mask 255.255.255.0 10.158.47.88


## 8.2 PAT Design
ip access-list extended BGPELK-NAT
 deny ip 192.168.20.0 0.0.0.255 10.158.47.0 0.0.0.255
 permit ip 192.168.20.0 0.0.0.255 any
!
route-map BGPELK-PAT permit 10
 match ip address BGPELK-NAT
!
ip nat inside source route-map BGPELK-PAT interface GigabitEthernet0/3 overload

The deny statement prevents translation between the lab service subnet and the management subnet, while the permit statement allows Internet-bound lab traffic to use PAT. Rebuilding this policy restored cross-network connectivity during troubleshooting.

## 8.3 DHCP Validation on MAIN Gi0/3
show dhcp lease

Temp IP addr: 10.158.47.88
Temp sub net mask: 255.255.255.0
DHCP Lease server: 10.158.47.93, state: 5 Bound
Lease: 3599 secs, Renewal: 1799 secs, Rebind: 3149 secs
Temp default-gateway addr: 10.158.47.93

The DHCP lease was verified as bound, with zero DHCP conflicts and successful gateway ping tests. Therefore the later INTVULN software issue was not attributed to DHCP.

## 8.4 Windows Firewall Resolution
A separate connectivity issue was traced to Windows firewall behavior. Traffic sourced from 192.168.20.1 could reach Windows only after a temporary inbound ICMP rule was added for the lab subnet. This confirmed that Cisco routing and DHCP were healthy and that the host firewall was the immediate blocker for that test.
netsh advfirewall firewall add rule name="Allow ICMP from BGP Lab" dir=in action=allow protocol=icmpv4:8,any remoteip=192.168.20.0/24


# 9. BGP State Collection

## 9.1 MAIN Collector
MAIN BGP state is collected over SSH from 192.168.20.1 using the bgpcollector local account. The collector invokes the IOS command show ip bgp and passes the result to a parser that emits one JSON object per route.
Observed MAIN parser output examples:
{"router":"MAIN-INT-RTR","prefix":"10.10.10.0/24","next_hop":"0.0.0.0","as_path":"","origin_as":"100","origin":"i","best":true}
{"router":"MAIN-INT-RTR","prefix":"70.70.70.7/32","next_hop":"10.10.10.2","as_path":"200","origin_as":"200","origin":"i","best":true}
{"router":"MAIN-INT-RTR","prefix":"100.100.100.0/24","next_hop":"0.0.0.0","as_path":"","origin_as":"100","origin":"i","best":true}


## 9.2 vIOS8 Collector
SSH compatibility options used by get_vios8_bgp.sh:
-o KexAlgorithms=+diffie-hellman-group14-sha1
-o HostKeyAlgorithms=+ssh-rsa
-o PubkeyAuthentication=no
-o PreferredAuthentications=password
-o StrictHostKeyChecking=no
-o UserKnownHostsFile=/dev/null

The vIOS8 collector is executed by Logstash and feeds a JSON-lines parser. File permissions were arranged so the logstash user could execute the collector.

# 10. Logstash Pipelines and Detection Logic

## 10.1 Pipeline Layout
Pipeline ID	Config path	Purpose
main	/etc/logstash/conf.d/*.conf	Cisco syslog on TCP/5514 -> bgp-%{+YYYY.MM.dd}
route_state	/etc/logstash/routes/*.conf	MAIN BGP route-state collection and origin validation
vios8_route	/etc/logstash/vios8/*.conf	vIOS8 route-state collection and AS-path validation


## 10.2 Syslog Pipeline
input {
  tcp {
    id => "bgp_syslog_tcp"
    port => 5514
    codec => line
  }
}

filter {
  grok {
    match => {
      "message" => "^<%{INT:[cisco][syslog_pri]}>%{SYSLOGTIMESTAMP:[cisco][syslog_timestamp]} %{IP:[cisco][router_ip]} %{GREEDYDATA:[cisco][syslog_body]}"
    }
    tag_on_failure => ["_cisco_parse_failure"]
  }
}

output {
  elasticsearch {
    hosts => ["https://127.0.0.1:9200"]
    user => "bgp_logstash"
    password => "<REDACTED>"
    ssl_enabled => true
    ssl_certificate_authorities => ["/etc/logstash/certs/http_ca.crt"]
    index => "bgp-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}


## 10.3 Route-State Security Rules
Expected origin rules:
10.10.10.0/24 -> 100
11.11.11.0/24 -> 100
100.100.100.0/24 -> 100
70.70.70.7/32 -> 200

For each route:
- matching origin -> security.event_type = normal_origin
- mismatch      -> security.event_type = possible_prefix_hijack
                  tag = _possible_prefix_hijack


## 10.4 vIOS8 AS-Path Detection Logic
For prefix 100.100.100.0/24:
expected AS path = 100

AS path 100       -> normal_route / as_path_match=true
AS path 200 100   -> possible_route_leak / as_path_match=false
other paths       -> unknown_as_path


## 10.5 Exact vIOS8 Pipeline Configuration
input {
  exec {
    id => "vios8_bgp_route_collector"
    command => "/usr/local/bin/parse_vios8_bgp.sh"
    interval => 60
    codec => json_lines
  }
}
filter {
  mutate {
    add_tag => ["bgp_route", "vios8_route"]
    add_field => {
      "[event][dataset]" => "bgp.routes"
      "[event][kind]" => "state"
    }
  }
  if [prefix] == "100.100.100.0/24" {
    mutate { add_field => { "[security][expected_as_path]" => "100" } }
    if [as_path] == "100" {
      mutate { add_field => {
        "[security][as_path_match]" => "true"
        "[security][event_type]" => "normal_route"
      }}
    } else if [as_path] == "200 100" {
      mutate { add_field => {
        "[security][as_path_match]" => "false"
        "[security][event_type]" => "possible_route_leak"
      }}
      add_tag => ["_possible_route_leak"]
    } else {
      mutate { add_field => {
        "[security][as_path_match]" => "unknown"
        "[security][event_type]" => "unknown_as_path"
      }}
    }
  } else {
    mutate { add_field => {
      "[security][as_path_match]" => "unknown"
      "[security][event_type]" => "unknown_route"
    }}
  }
}
output {
  elasticsearch {
    id => "vios8_routes_elasticsearch"
    hosts => ["https://127.0.0.1:9200"]
    user => "bgp_logstash"
    password => "<REDACTED>"
    ssl_enabled => true
    ssl_certificate_authorities => ["/etc/logstash/certs/http_ca.crt"]
    index => "bgp-routes-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}


## 10.6 Pipeline Registration
- pipeline.id: vios8_route
  path.config: "/etc/logstash/vios8/*.conf"

- pipeline.id: route_state
  path.config: "/etc/logstash/routes/*.conf"

- pipeline.id: main
  path.config: "/etc/logstash/conf.d/*.conf"


# 11. Elasticsearch and Security Indexing

## 11.1 Cluster Configuration
cluster.name: bgp-security-lab
node.name: BGP-ELK
discovery.type: single-node
http.host: 0.0.0.0

Elasticsearch 9.5.4 was used. TLS was enabled and the Logstash CA was copied to /etc/logstash/certs/http_ca.crt. A dedicated bgp_logstash writer account was created with index privileges on bgp-* and cluster privileges sufficient to write route and syslog data. Credentials are deliberately redacted in this report.

## 11.2 Index Families
Index pattern	Content	Example fields
bgp-YYYY.MM.dd	Cisco syslog	cisco.router_ip, cisco.message, cisco.syslog_timestamp, cisco.facility
bgp-routes-YYYY.MM.dd	BGP route-state snapshots	router, prefix, origin_as, as_path, next_hop, best, security.*


## 11.3 Example Detected Hijack Document
{
  "router": "MAIN-INT-RTR",
  "prefix": "100.100.100.0/24",
  "origin_as": "200",
  "as_path": "200",
  "best": false,
  "security": {
    "expected_origin_as": "100",
    "origin_match": false,
    "event_type": "possible_prefix_hijack"
  },
  "tags": ["bgp_route", "_possible_prefix_hijack"]
}


## 11.4 Elasticsearch Startup Incident
During the lab, Elasticsearch once failed to start because the machine-learning native code crashed. The logs explicitly indicated that the ML native component was the failing subsystem and identified xpack.ml.enabled: false as the bypass. Disk space was not the cause; the node had tens of gigabytes free. The incident was treated as an infrastructure troubleshooting event rather than a BGP detection failure.

