# LinkedIn Post Draft

🚀 **BGP Security Monitoring & Anomaly Detection Lab**

I recently completed a hands-on BGP security monitoring project built in EVE-NG using Cisco IOSv, Logstash, Elasticsearch and Kibana.

The lab focuses on two practical BGP security scenarios:

🔹 **Prefix Hijacking Detection** — identifying when a monitored prefix is originated by an unexpected AS.

🔹 **Route Leak Detection** — identifying an unexpected AS-path for a monitored prefix.

### Lab architecture
Cisco IOSv -> SSH/Syslog Collection -> Logstash -> Elasticsearch -> Kibana

The lab also uses Chrony/NTP for centralized time synchronization.

### Validation performed
✅ Injected a simulated prefix hijack from AS200
✅ Detected the anomaly in Elasticsearch/Logstash
✅ Visualized the active hijack in Kibana
✅ Withdrew the hijacked route
✅ Confirmed the active hijack metric returned to 0
✅ Simulated and monitored an AS-path route leak

The final dashboard provides active security indicators, BGP events, neighbor activity, notifications, route-leak trends and an AS topology view.

This project helped me strengthen my practical skills in **BGP, network security, Logstash, Elasticsearch, Kibana, Linux and security monitoring**.

📌 Full project report and lab documentation are available in my GitHub repository.

#BGP #NetworkSecurity #Networking #Cisco #EVE-NG #Elasticsearch #Logstash #Kibana #CyberSecurity #NetworkEngineering #Routing #NOC #SOC
