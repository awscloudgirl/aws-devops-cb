# AWS Load Balancers

AWS offers several types of load balancers to distribute incoming application or network traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses. Each type of load balancer operates at different layers of the OSI model and is optimized for different use cases.

## Application Load Balancer (ALB)

- **Documentation**: [AWS ALB Introduction](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- **Layer**: 7 (Application Layer)
- **Features**:
  - **Port Based Routing**: ALB can route traffic to different targets based on the port number, allowing for more granular control over traffic distribution.
  - **Path Based Routing**: ALB can route requests to different target groups based on the URL path of the request. This is useful for directing traffic to different microservices within an application.
  - **Platform Specific Redirection**:
    - **Mobile**: ALB can redirect traffic from mobile devices to specific target groups optimized for mobile users.
    - **Desktop**: Similarly, traffic from desktop users can be redirected to target groups optimized for desktop experiences.
  - **Advanced Request Routing**: ALB supports host-based routing, query string parameter-based routing, and HTTP header-based routing.
  - **WebSocket Support**: ALB supports WebSocket and HTTP/2 protocols, which are essential for real-time applications.
- **Targets**:
  - **EC2 Instances**: Virtual servers in the cloud.
  - **Containers**: Such as those managed by Amazon ECS or Kubernetes.
  - **Lambdas**: Serverless functions that can be invoked in response to HTTP requests.
  - **Hybrid Infrastructure**: On-premises servers that are part of a hybrid cloud architecture.

## Network Load Balancer (NLB)

- **Documentation**: [AWS NLB Introduction](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)
- **Layer**: 4 (Transport Layer)
- **Features**:
  - **TCP/UDP Traffic Forwarding**: NLB is designed to handle TCP and UDP traffic, making it suitable for applications that require high throughput and low latency.
  - **High Performance**: Capable of handling millions of requests per second while maintaining ultra-low latency (~100 ms).
  - **Static IP Addresses**: NLB provides a static IP per Availability Zone, which can be useful for applications that require fixed IP addresses. It also supports Elastic IP assignment.
  - **Extreme Performance**: Ideal for applications that require extreme performance, such as gaming, IoT, and real-time data streaming.
- **Target Groups**:
  1. **Instances**: EC2 instances that receive the traffic.
  2. **Private IP Addresses**: Direct traffic to specific IP addresses within your VPC.
  3. **Application Load Balancer**: NLB can route traffic to an ALB, allowing for complex routing scenarios.
- **Health Check Protocols**: Supports TCP, HTTP, and HTTPS health checks to ensure that only healthy targets receive traffic.
- **Targets**:
  - **EC2 Instances**: For high-performance applications.
  - **Containers**: For containerized applications.
  - **Lambdas**: For serverless applications.
  - **Hybrid Infrastructure**: For integrating on-premises resources.

## Gateway Load Balancer (GLB)

- **Documentation**: [AWS GLB Introduction](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html)
- **Layer**: 3 (Network Layer)
- **Features**:
  - **Network Virtual Appliances**: GLB allows you to deploy, scale, and manage a fleet of third-party network virtual appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems.
  - **Transparent Network Gateway**: Acts as a single entry and exit point for all traffic, simplifying the network architecture.
  - **Load Balancer**: Distributes traffic to your virtual appliances, ensuring even load distribution and high availability.
  - **GENEVE Protocol**: Uses the GENEVE protocol on port 6081, which is designed for network virtualization and supports encapsulation of network packets.
- **Use Cases**:
  - **Security**: Deploy firewalls and intrusion detection systems to protect your network.
  - **Traffic Inspection**: Use deep packet inspection systems to analyze and manipulate network traffic.
  - **Network Management**: Simplify the management of complex network architectures by centralizing traffic through a single gateway.

Each type of load balancer is designed to meet specific needs and use cases, allowing you to choose the best option for your application's requirements.
