\# Web Application Security Lab



A controlled web application security laboratory built with \*\*GNS3, FortiGate, Alpine Linux, and Nginx\*\*. The lab is designed to demonstrate how a firewall can separate a client network from a web-server/DMZ network and provide a controlled environment for studying web traffic, firewall policies, logging, and basic security testing.



> \*\*Project type:\*\* Cybersecurity / Network Security Lab

> \*\*Environment:\*\* GNS3

> \*\*Firewall:\*\* FortiGate VM64-KVM

> \*\*Web Server:\*\* Alpine Linux + Nginx

> \*\*Protocol:\*\* HTTP

> \*\*Status:\*\* Functional lab topology with controlled security-testing workflow



\---



\## 1. Project Overview



Web applications are common targets for attacks because they are directly exposed to user traffic and frequently process untrusted input.



This project creates a small isolated environment where web traffic passes through a \*\*FortiGate firewall\*\* before reaching a web server located in a separate network.



The main objectives are to:



\* Build a segmented web application environment.

\* Place the web server in a DMZ/server network.

\* Control traffic between the client and web server using FortiGate firewall policies.

\* Configure an Alpine Linux server running Nginx.

\* Generate legitimate HTTP traffic.

\* Study how web requests appear in server logs.

\* Perform controlled and non-destructive security tests.

\* Provide a reusable environment for future web application security experiments.



\---



\## 2. Lab Architecture



The basic architecture consists of three logical zones:



```text

&#x20;                   Internet / NAT

&#x20;                        |

&#x20;                        |

&#x20;                   port3

&#x20;               +----------------+

&#x20;               |    FortiGate   |

&#x20;               |    Firewall    |

&#x20;               +----------------+

&#x20;                 |            |

&#x20;               port1        port2

&#x20;                 |            |

&#x20;         Client Network       DMZ

&#x20;         10.10.80.0/24    10.10.60.0/24

&#x20;                 |            |

&#x20;                 |            |

&#x20;         WEB-CLIENT       WEB SERVER

&#x20;         10.10.80.10      10.10.60.10

&#x20;                              |

&#x20;                            Nginx

&#x20;                            TCP/80

```



\### Network zones



| Zone                 | Network         | Purpose                                          |

| -------------------- | --------------- | ------------------------------------------------ |

| Client network       | `10.10.80.0/24` | Source of legitimate and controlled test traffic |

| DMZ / Server network | `10.10.60.0/24` | Hosts the web server                             |

| Internet/NAT side    | External/DHCP   | Provides controlled outbound connectivity        |



\---



\## 3. IP Addressing



\### FortiGate



| Interface | Address         | Role                      |

| --------- | --------------- | ------------------------- |

| `port1`   | `10.10.80.1/24` | Client-side gateway       |

| `port2`   | `10.10.60.1/24` | DMZ/server gateway        |

| `port3`   | External/DHCP   | Internet/NAT connectivity |



\### Client



| Device     | Address          | Gateway      |

| ---------- | ---------------- | ------------ |

| WEB-CLIENT | `10.10.80.10/24` | `10.10.80.1` |



\### Web server



| Device                  | Address          | Gateway      |

| ----------------------- | ---------------- | ------------ |

| Alpine/Nginx Web Server | `10.10.60.10/24` | `10.10.60.1` |



\---



\## 4. Technologies Used



\* \*\*GNS3\*\* — Network emulation and laboratory topology.

\* \*\*FortiGate VM64-KVM\*\* — Firewall, routing, NAT, and traffic control.

\* \*\*Alpine Linux\*\* — Lightweight Linux operating system for the web server.

\* \*\*Nginx\*\* — HTTP web server.

\* \*\*VPCS / Linux client\*\* — Used to generate client traffic.

\* \*\*Git/GitHub\*\* — Project version control and documentation.



\---



\## 5. FortiGate Configuration



The FortiGate firewall provides separation between the client network and the web-server network.



\### Policy 1 — WEB-CLIENT-to-DMZ



```text

Name:        WEB-CLIENT-to-DMZ

Incoming:    port1

Outgoing:    port2

Source:      10.10.80.0/24

Destination: 10.10.60.0/24

Service:     ALL

NAT:         Disabled

Logging:     All sessions

```



This policy permits traffic from the client network to the web-server/DMZ network.



The main purpose is to demonstrate that communication between two different security zones should pass through an explicitly defined firewall policy.



\### Policy 2 — WEB-SERVER-to-Internet



```text

Name:        WEB-SERVER-to-Internet

Incoming:    port2

Outgoing:    port3

Source:      10.10.60.0/24

Destination: Internet

Service:     ALL

NAT:         Enabled

Logging:     All sessions

```



This policy allows the DMZ server to make outbound connections through the FortiGate.



NAT is enabled because the private DMZ address is not directly routable on the external network.



\---



\## 6. Web Server



The web server runs \*\*Nginx on Alpine Linux\*\*.



The server is located in the DMZ network:



```text

IP address: 10.10.60.10

Subnet:     /24

Gateway:    10.10.60.1

Web port:   TCP/80

```



The server hosts a simple test web page representing a bank-style web application environment.



The purpose of the page is not to reproduce a real banking application, but to provide a controlled HTTP endpoint for security and network testing.



\---



\## 7. Connectivity Validation



Before security testing, basic network connectivity was established.



The following communication paths were successfully tested during the lab build:



```text

WEB-CLIENT

10.10.80.10

&#x20;    |

&#x20;    | ICMP

&#x20;    v

FortiGate port1

10.10.80.1

&#x20;    |

&#x20;    | Firewall policy

&#x20;    v

FortiGate port2

10.10.60.1

&#x20;    |

&#x20;    v

Web Server

10.10.60.10

```



HTTP connectivity to the Nginx server was also successfully established.



The server returned the expected web page over HTTP.



\---



\## 8. Web Traffic and Logging



Nginx records incoming HTTP requests in its access log.



The primary log used during the laboratory work is:



```text

/var/log/nginx/access.log

```



A typical legitimate request appears in the form:



```text

GET / HTTP/1.1

```



The access log can therefore be used to correlate:



1\. Client activity.

2\. HTTP requests.

3\. Server responses.

4\. Request timestamps.

5\. Requested URLs.

6\. Response status codes.



This provides a simple foundation for studying web-server monitoring.



\---



\## 9. Security Testing Methodology



The security-testing component of this project follows a controlled and non-destructive approach.



Testing is performed only against the isolated laboratory environment.



\### Test categories



\#### Test 1 — Normal HTTP Traffic



The first test establishes a baseline using an ordinary HTTP request.



Example:



```text

GET /

```



Expected behavior:



```text

Client → FortiGate → Web Server

&#x20;                   ↓

&#x20;                HTTP 200

```



The purpose is to establish what normal application traffic looks like before introducing unusual requests.



\#### Test 2 — Controlled Suspicious HTTP Input



The second test introduces a harmless suspicious-looking parameter into an HTTP request.



Example test pattern:



```text

GET /?id=' OR '1'='1

```



The objective is to observe how unusual input appears in the web-server logs and to provide a basis for discussing web-application security monitoring.



This test is intended for \*\*observation and detection\*\*, not exploitation.



\---



\## 10. Security Principles Demonstrated



This laboratory demonstrates several important cybersecurity concepts.



\### Network segmentation



The client and web server are placed in different IP networks.



```text

Client network: 10.10.80.0/24

DMZ:            10.10.60.0/24

```



This prevents the web server from simply being placed on the same network as the client.



\### Firewall enforcement



Traffic between the two networks is controlled by the FortiGate firewall rather than being allowed through an unrestricted Layer-2 connection.



\### DMZ concept



The web server is placed in a dedicated server/DMZ network.



This reflects the principle that externally accessed services should be isolated from more trusted internal networks.



\### Least privilege



The lab can be extended from the current broad testing policy to specific services such as:



```text

HTTP  → TCP/80

HTTPS → TCP/443

DNS   → UDP/TCP 53

```



A production design should avoid using `ALL` services where they are not required.



\### Logging and monitoring



Nginx access logs provide visibility into HTTP requests, while FortiGate logging provides visibility into network traffic passing through the firewall.



\---



\## 11. Repository Structure



The Git repository is organized to separate the topology, documentation, security testing, and supporting material.



```text

web-application-security-lab/

│

├── Web\_App\_Security\_Lab.gns3

├── .gitignore

│

├── documentation/

│   ├── lab-overview.md

│   ├── network-design.md

│   └── security-testing.md

│

├── fortigate/

│   ├── firewall-policies.md

│   └── configuration-notes.md

│

├── nginx/

│   ├── server-notes.md

│   └── log-analysis.md

│

├── tests/

│   ├── baseline-test.md

│   └── test-results.md

│

└── screenshots/

```



The current repository contains the GNS3 topology and Git configuration. Additional documentation and evidence can be added as the project develops.



\---



\## 12. Why GNS3?



GNS3 provides an isolated environment for experimenting with network-security configurations without affecting a production network.



It makes it possible to reproduce scenarios involving:



\* Firewalls

\* Routers

\* Linux servers

\* Network segmentation

\* NAT

\* Web servers

\* Traffic inspection

\* Security testing



This makes the environment suitable for cybersecurity training and internship projects.



\---



\## 13. Future Improvements



The current lab provides a foundation that can be expanded without changing its basic architecture.



Possible improvements include:



\### HTTPS



Configure Nginx with TLS and compare:



```text

HTTP  → TCP/80

HTTPS → TCP/443

```



\### More restrictive firewall policies



Replace the current broad testing rules with service-specific policies.



\### FortiGate inspection



Enable appropriate security profiles and examine how the firewall handles suspicious web traffic.



\### Web application logging



Expand the application to produce more realistic authentication and application events.



\### Security monitoring



Add centralized logging or a SIEM platform to correlate:



```text

FortiGate logs

&#x20;     +

Nginx access logs

&#x20;     +

Nginx error logs

&#x20;     +

Application events

```



\### Additional security tests



The isolated environment can later be used to study:



\* Input validation

\* Authentication security

\* HTTP header security

\* Access-control weaknesses

\* Suspicious request detection

\* Basic vulnerability scanning

\* Web-server hardening



All testing should remain restricted to the authorized laboratory environment.



\---



\## 14. Security and Ethical Use



This project is intended strictly for \*\*authorized laboratory and educational use\*\*.



The security tests are designed to run against systems controlled by the lab owner or with explicit permission.



The techniques demonstrated in this repository should not be used against systems, networks, or applications without authorization.



\---



\## 15. Current Project Status



\### Completed



\* \[x] GNS3 laboratory topology created

\* \[x] FortiGate firewall deployed

\* \[x] Client network configured

\* \[x] DMZ/server network configured

\* \[x] Alpine Linux web server deployed

\* \[x] Nginx installed and configured

\* \[x] HTTP connectivity established

\* \[x] FortiGate client-to-DMZ policy configured

\* \[x] FortiGate server-to-Internet policy configured

\* \[x] Basic web traffic successfully generated





\---



\## 16. Project Learning Outcomes



This laboratory provides practical experience with:



\* Network segmentation

\* Firewall policy design

\* FortiGate administration

\* DMZ architecture

\* Linux server administration

\* Nginx configuration

\* HTTP traffic

\* Network troubleshooting

\* Security logging

\* Controlled security testing



The project also demonstrates how network security and web application security can be combined in a single isolated laboratory environment.



\---



\## 17. Author



\*\*Meleng Collins\*\*



Cybersecurity / IT Internship Project



The project was developed as a practical cybersecurity laboratory for learning and demonstrating firewall configuration, network segmentation, web-server deployment, traffic monitoring, and controlled web application security testing.



