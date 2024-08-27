# Student_Assignment_Hub_SystemDesign

# Student Assignment Hub
1. Features Expectation

A. Function requirement
1. Student can submit their submission for assignmetn
2. Faculties can post assignment
3. Students will get Email for their succesful submissions
B. Feature wil not be covered
1.real time collabration feature
R2. ealt time chat feature
C. who will use :
Faculties , students, IT team
D. what is scale of the system ?
10,000 stuednets, 200 Faculties
E. Usage patteern :
Hight activity near assignemmnt deadline
*
Estimation :
A. Read/ Write ratio : read 70% and write 30%
B. Throughtput Write QPS :
7000 submision * 2 attempss = 14,0000 submissions
2 write QPS (Uniform request) === 100 QPS (Burst request)
Read QPS
2(submission attempt) * 70/30 == 5 Read QPS (uniform reqeust) === 300 QPS (Burst
request)
C. Storage:
10000 (students)* 40(Submissions per semeneter) * 2 (submit attempts) * 5MB
(submission size) == 2,000,000 MB === 2 TB
Metadata + logs ==100 GB === 2 TB
D. Latency
Read latency == 200 ms
Write latency === 500 ms
Design Goal
A. Latency and Throughput
low latency (elasticcache and CDN) and high throughput
B. Consistency vs Availablity
For the Student Assignment Hub, strong consistency is more important than
availability because the accuracy and integrity of assignment submissions and
grading data are paramount. While high availability is still crucial, the potential
issues arising from inconsistent data (such as submission or grading disputes)
justify prioritizing consistency to ensure the system operates reliably and accurately
in all circumstances
High Level Design
A. API Design
GET : get all assignment
v1/assigment :
POST: submit a assignment
v1/assignment :
PUT : to update the assignment
v1/assignment/{assignment_id}:
DELETE: delete a specific assignment
v1/assignment/(assignment_id):
GET : get a specific assignemnt
v1/assignment/{assignment_id}:
Submission: submit submission for specific assignment
v1/assignment/{assignment_id}/submission
Databases Schema
A. Realtional Database
User, Assignment, Grades
B. Storage service : submission zip
C. Non- realtional databse (key-value) gather log on Submission
Deep Dive into System Architecture
---### **VPC Configuration**
- **VPC:** All services are present within the default VPC. The VPC contains multiple
subnets, with each subnet located in a specific Availability Zone (AZ).
- **Domain Name Setup:**
- **DNS:** A domain name was purchased from Namecheap, and the nameservers
from Route 53 were used to configure it.
- **Route 53:** A public hosted zone in Route 53 contains all DNS records (A, AAAA,
CNAME, ALIAS). The ALIAS record is used to map the domain name to the API Gateway,
directing the request appropriately.
### **API Gateway Configuration**
- **API Gateway:** Serves as the front-facing interface for all user requests. The API
Gateway can be placed either inside or outside the VPC, depending on the
architecture.
- **WAF Integration:**
AWS Web Application Firewall (WAF) can be integrated with the API Gateway to protect
the API from common web threats.
- **Public API:** The main API Gateway is placed outside the VPC to allow students,
teachers, and administrators to access the system over the internet. This API Gateway
handles operations such as: authentication, authorization, ratelimiting etc
### **Load Balancer and Traffic Management**
- **Load Balancer:** A Layer 7 load balancer is used to manage traffic.
why Application Load Balancer ?
- **Capabilities:** WebSockets, sticky sessions, and detailed traffic routing based on
application-layer data (such as URL paths, HTTP headers, and cookies) are supported,
offering more control compared to Layer 4 load balancers.
- **Server-Triggered Notifications:**
- When a participant posts a new message or updates a document, the server
processes the action. With WebSocket connections established, the server can
instantly push notifications to all connected clients without waiting for them to
request or poll for updates.
- **Example:** The server broadcasts a message like "New message from John Doe:
'Can we extend the deadline?'" to all participants connected via WebSockets.
- **SSL/TLS Termination:** Layer 7 load balancers can handle SSL/TLS termination,
meaning they decrypt incoming HTTPS traffic before passing it to the backend servers.
This offloads SSL processing work from the application servers, improving
performance.
- **ACM:** AWS Certificate Manager (ACM) is used to provision, manage, and deploy
SSL/TLS certificates for securing communications.
### **API Gateway to ALB Flow**
- **API Gateway as Passthrough:**
- The API Gateway does not decrypt the request. Instead, it forwards the encrypted
HTTPS request to the ALB, effectively acting as a proxy.
- **ALB Decrypts the Request:**
- The ALB, configured with an ACM certificate, receives the HTTPS request and
decrypts it. After decryption, the ALB can route the traffic to the appropriate backend
services (e.g., EC2 instances, containers) over plain HTTP or maintain HTTPS to the
backend if additional layers of security are required.
### **EC2 and RDS Configuration**
- **EC2:** EC2 instances are connected to the RDS using security groups. The EC2
instances are located in a public subnet.
- **RDS:** RDS is configured with a Multi-AZ setup to ensure strong consistency and
availability. The RDS instances are located in a private subnet.
- **KMS:** AWS Key Management Service (KMS) is used to encrypt data at rest in the
RDS, securing sensitive information.
### **DynamoDB Usage**
- **DynamoDB:** DynamoDB is used for auditing purposes to check if an assignment
is successfully saved in the S3 storage bucket. The status is saved as "success" for the
specific student ID and assignment ID, or "failure" with the reason provided.
### **SNS and Lambda Integration**
- **SNS:**
- An SNS topic is integrated into the post-submission endpoint of the web application
hosted on EC2.
- The request body of the submission endpoint is published to the SNS topic using
the AWS SDK.
- **Publisher:** The assignment submission triggers the SNS topic.
- **Subscriber:** A Lambda function subscribed to the SNS topic processes the
submission.
- **Lambda Function:**
- The Lambda function sends the assignment to the S3 bucket, and S3 stores it.
- DynamoDB is used to audit whether the submission was successful.
- Once successfully saved in S3 and DynamoDB, the Lambda function sends an email
to the user using the Simple Email Service (SES).
### **CloudWatch Monitoring**
- **CloudWatch:** Amazon CloudWatch is used for monitoring and managing AWS
resources, applications, and services.
- **Metrics Collection:** CloudWatch collects metrics for CPU and memory
utilization. When certain thresholds are breached, CloudWatch triggers alerts, and the
Auto Scaling Group (ASG) automatically scales the EC2 instances accordingly.
### **S3 Storage**
- **S3:** Amazon S3 is globally accessible and highly available across multiple
regions. It is located outside the VPC and used for storing assignment submissions.
### **Infrastructure as Code (IaC)**
- **Terraform:** Terraform is used for Infrastructure as Code (IaC), automating the
deployment of resources.
- **Packer:** Packer is used for automating the creation of machine images.
- **CI/CD Pipeline:** The continuous integration and continuous deployment (CI/CD)
pipeline ensures smooth and consistent development and deployment processes.
Justify
This architecture is designed to meet the needs of the Student Assignment Hub,
offering scalability, strong security, high availability, and an enhanced user
experience. Every decision, from the choice of AWS services to the implementation of
security protocols, has been made to ensure the system is robust, reliable, and
capable of handling the demands of a modern educational platform
