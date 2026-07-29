# AWS Storybook Reading Order

> Read from business pressure to service choice, then from individual service contracts to complete application and architecture decisions.

Each article links directly to the next. Article length reflects the depth needed by the lesson; it is not a measure of importance.

## 01 — Infrastructure

Start here for the cloud, managed-service responsibility, events, and architectural thinking.

- [The Country of AWS](./01-infrastructure/01-aws-country.md)
- [AWS Services: Why Cook the Kitchen When You Just Want the Meal?](./01-infrastructure/02-service-meal.md)
- [Shared Responsibility: The Restaurant Agreement](./01-infrastructure/03-shared-responsibility.md)
- [Event-Driven Thinking: Stop Hiring People to Wait](./01-infrastructure/04-event-driven-thinking.md)
- [How to Think Like an Architect](./01-infrastructure/05-how-to-think-like-an-architect.md)

## 02.01 — Identity and Security

Who is making the request, what may they do, and which boundary protects it?

- [AWS Identity and Security: The Protected Restaurant](./02-services/01-identity-security/00-identity-security-the-protected-restaurant.md)
- [Amazon Cognito: The Tiny Company’s Guest List](./02-services/01-identity-security/01-cognito-guest-list.md)
- [AWS Identity and Access Management: Which Cape Are You Wearing?](./02-services/01-identity-security/02-iam-which-cape-are-you-wearing.md)
- [AWS Security Token Service: Polyjuice Potion](./02-services/01-identity-security/03-sts-polyjuice-potion.md)
- [AWS Key Management Service: The Elder Wand Answers to Its True Owner](./02-services/01-identity-security/04-kms-elder-wand.md)
- [AWS Systems Manager Parameter Store: The Filing Cabinet](./02-services/01-identity-security/05-parameterstore-filingcabinet.md)
- [AWS Secrets Manager: The Private Security Guard](./02-services/01-identity-security/06-secretsmanager-private-security-guard.md)
- [AWS WAF: The Front-Door Screen](./02-services/01-identity-security/07-waf-front-door-screen.md)
- [AWS Certificate Manager and Amazon EC2: The Protected Staff Entrance](./02-services/01-identity-security/08-aws-certificate-manager-and-amazon-ec2-the-protected-staff-entrance.md)
- [AWS Identity and Security: Many Boundaries, One Restaurant](./02-services/01-identity-security/09-identity-security-many-boundaries-one-restaurant.md)

## 02.02 — Compute

What shape of work needs to run?

- [AWS Compute: The Wand Chooses the Wizard](./02-services/02-compute/00-compute-wizards-wand.md)
- [Amazon EC2: Bits Boba That Stays Open](./02-services/02-compute/01-ec2-boba-shop.md)
- [Amazon Machine Image (AMI): Bottle the Shop Setup, Not the Business](./02-services/02-compute/02-ami-blueprint.md)
- [Amazon EC2 Auto Scaling: The Fleet Manager Who Opens Bits Boba Locations Before the Line Gets Too Long](./02-services/02-compute/03-auto-scaling-the-fleet-manager.md)
- [Elastic Load Balancing: The Host Who Seats Every Customer](./02-services/02-compute/04-elastic-load-balancing-the-host.md)
- [Containers: The Food Truck That Carries the Kitchen](./02-services/02-compute/05-containers-the-food-truck.md)
- [Amazon ECS: The Operations Manager Behind Your Food Truck Festival](./02-services/02-compute/06-ecs-operations-manager.md)
- [Amazon ECS Task Definitions: The Operating Manual Every Food Truck Follows](./02-services/02-compute/06.5-task-definitions-operating-manual.md)
- [AWS Fargate: The RV Resort for Your Food Trucks](./02-services/02-compute/07-fargate-rv-resort.md)
- [Amazon ECR: The Warehouse of Approved Food Truck Blueprints](./02-services/02-compute/08-ecr-warehouse.md)
- [Amazon EKS: The National Food Truck Association](./02-services/02-compute/09-eks-the-national-food-truck-association.md)
- [AWS Batch: The Catering Company](./02-services/02-compute/10-aws-batch-catering-company.md)
- [AWS Lambda: The Errand Runner](./02-services/02-compute/11-lambda-the-errand-runner.md)
- [AWS Lambda: The Dispatch Board](./02-services/02-compute/11.1-lambda-dispatch-board.md)
- [AWS Lambda: The Recovery Shelf](./02-services/02-compute/11.2-lambda-recovery-shelf.md)
- [AWS Lambda: The Runner's Kit and Clearance](./02-services/02-compute/11.3-lambda-runners-kit-and-clearance.md)
- [AWS Lambda: The Shift Change](./02-services/02-compute/11.4-lambda-shift-change.md)
- [AWS Compute: The Work Chooses the Compute](./02-services/02-compute/12-epilogue-the-work-chooses-the-compute.md)

## 02.03 — Storage

Where should files and durable objects live?

- [AWS Storage: The Memory Chooses Its Home](./02-services/03-storage/00-the-memory-chooses-its-home.md)
- [Instance Store and Amazon EBS: The Photographer’s Workbench](./02-services/03-storage/01-the-photographers-workbench.md)
- [Amazon EFS: The Shared Darkroom](./02-services/03-storage/02-the-shared-darkroom.md)
- [Amazon S3: The Gallery of Objects](./02-services/03-storage/03-the-gallery-of-objects.md)
- [Amazon S3: The Gallery Doors](./02-services/03-storage/04-the-gallery-doors.md)
- [Amazon S3: The Retouching Lab](./02-services/03-storage/05-the-retouching-lab.md)
- [Amazon S3: The Negative Vault](./02-services/03-storage/06-the-negative-vault.md)
- [Amazon S3: The Archive Elevator](./02-services/03-storage/07-the-archive-elevator.md)
- [Amazon EBS, Amazon EFS, and Amazon S3: Choose the Right Home](./02-services/03-storage/08-choose-the-right-home.md)
- [AWS Storage: What Remains](./02-services/03-storage/09-what-remains.md)
- [Amazon Athena: The Photo Archivist’s Reading Room](./02-services/03-storage/10-amazon-athena-the-photo-archivists-reading-room.md)

## 02.04 — Databases

Where should each kind of truth live?

- [AWS Databases: The Truth Chooses Its Home](./02-services/04-database/00-databases-where-should-the-truth-live.md)
- [Amazon DynamoDB: The Warehouse Built Around Retrieval Lanes](./02-services/04-database/01-dynamodb-warehouse-built-around-retrieval-lanes.md)
- [Amazon DynamoDB: Keys, Access Patterns, and the Retrieval Map](./02-services/04-database/02-dynamodb-keys-and-access-patterns.md)
- [Amazon DynamoDB: The Second Door Into the Warehouse](./02-services/04-database/03-dynamodb-indexes-second-door.md)
- [Amazon DynamoDB: The Warehouse Under Pressure](./02-services/04-database/04-dynamodb-under-pressure.md)
- [Amazon DynamoDB: The Developer’s Workbench](./02-services/04-database/05-dynamodb-developers-workbench.md)
- [Amazon DynamoDB: After Closing Time](./02-services/04-database/06-dynamodb-after-closing-time.md)
- [Amazon RDS and Amazon Aurora: The Accounting Office](./02-services/04-database/07-rds-aurora-accounting-office.md)
- [Amazon ElastiCache: The Memory Desk](./02-services/04-database/08-elasticache-memory-desk.md)
- [Amazon OpenSearch Service: The Search Catalog](./02-services/04-database/09-opensearch-search-catalog.md)
- [AWS Databases: The Right Room for the Question](./02-services/04-database/10-aws-databases-comparison.md)
- [AWS Databases: Choose the Home for the Truth](./02-services/04-database/11-databases-epilogue.md)

## 02.05 — APIs and SDKs

How does an application make and govern a request?

- [Amazon API Gateway and AWS SDK: Byte Burger's Three Zones](./02-services/05-api-sdk/00-api-gateway-and-aws-sdk-the-three-restaurant-zones.md)
- [Amazon API Gateway: The Customer Front](./02-services/05-api-sdk/01-api-gateway-the-customer-front.md)
- [Amazon API Gateway: The Order Contract](./02-services/05-api-sdk/02-api-gateway-the-order-contract.md)
- [Amazon API Gateway: The Membership Desk](./02-services/05-api-sdk/03-api-gateway-the-membership-desk.md)
- [Amazon API Gateway: The Crowd-Control Rope](./02-services/05-api-sdk/04-amazon-api-gateway-the-crowd-control-rope.md)
- [Amazon API Gateway: The Street Address](./02-services/05-api-sdk/05-amazon-api-gateway-the-street-address.md)
- [AWS SDK: The Operations Assistant](./02-services/05-api-sdk/06-aws-sdk-the-operations-assistant.md)
- [AWS SDK: The Busy Supplier Line](./02-services/05-api-sdk/07-aws-sdk-the-busy-supplier-line.md)
- [AWS SDK: The Long Inventory Receipt](./02-services/05-api-sdk/08-aws-sdk-the-long-inventory-receipt.md)
- [AWS SDK and Amazon S3: The Temporary Loading Dock Pass](./02-services/05-api-sdk/09-aws-sdk-and-amazon-s3-the-temporary-loading-dock-pass.md)
- [Amazon API Gateway and AWS SDK: The Byte Burger Contract](./02-services/05-api-sdk/10-amazon-api-gateway-and-aws-sdk-the-restaurant-contract.md)

## 02.06 — Application Integration

How should independent parts exchange work, facts, and state?

- [AWS Application Integration: The Lunch Rush](./02-services/06-application-integrations/00-the-lunch-rush.md)
- [Amazon SQS: The Kitchen Ticket Rail](./02-services/06-application-integrations/01-the-kitchen-ticket-rail.md)
- [Amazon SQS: The Ticket That Came Back](./02-services/06-application-integrations/02-the-ticket-that-came-back.md)
- [Amazon SNS: Order 42 Is Ready](./02-services/06-application-integrations/03-order-42-is-ready.md)
- [Amazon EventBridge: The Operations Switchboard](./02-services/06-application-integrations/04-the-operations-switchboard.md)
- [AWS Step Functions: The Fulfillment Runbook](./02-services/06-application-integrations/05-the-fulfillment-runbook.md)
- [AWS Step Functions: When the Fryer Fails](./02-services/06-application-integrations/06-when-the-fryer-fails.md)
- [Amazon Kinesis Data Streams: The River of Receipts](./02-services/06-application-integrations/07-the-river-of-receipts.md)
- [AWS AppSync: The Order in Your Pocket](./02-services/06-application-integrations/08-the-order-in-your-pocket.md)
- [AWS Service Integrations: The Retry Storm](./02-services/06-application-integrations/09-the-retry-storm.md)
- [AWS Application Integration: Who Gets the Message?](./02-services/06-application-integrations/10-who-gets-the-message.md)
- [AWS Application Integration: Byte Burger Keeps Moving](./02-services/06-application-integrations/11-the-restaurant-keeps-moving.md)

## 02.07 — Deployment

How does a change reach production safely?

- [AWS Deployment: The Friday Franchise Rollout](./02-services/07-deployment/00-the-friday-franchise-rollout.md)
- [AWS Deployment Packages: The Sealed Release Kit](./02-services/07-deployment/01-the-sealed-release-kit.md)
- [Amazon ECR and Amazon ECS: The Container Depot](./02-services/07-deployment/02-the-container-depot.md)
- [AWS CloudFormation: The Franchise Blueprint](./02-services/07-deployment/03-the-franchise-blueprint.md)
- [AWS CloudFormation: The Renovation Preview](./02-services/07-deployment/04-the-renovation-preview.md)
- [AWS SAM: The Serverless Store Plan](./02-services/07-deployment/05-the-serverless-store-plan.md)
- [AWS CDK: The Blueprint Factory](./02-services/07-deployment/06-the-blueprint-factory.md)
- [AWS Deployment Testing: The Test Kitchen](./02-services/07-deployment/07-the-test-kitchen.md)
- [AWS AppConfig: The Same Release, Different Store](./02-services/07-deployment/08-the-same-release-different-store.md)
- [AWS CodeBuild: The Central Build Kitchen](./02-services/07-deployment/09-the-central-build-kitchen.md)
- [AWS CodePipeline: The Release Conveyor](./02-services/07-deployment/10-the-release-conveyor.md)
- [AWS CodeDeploy: The Rollout Manager](./02-services/07-deployment/11-the-rollout-manager.md)
- [AWS Deployment Strategies: How Many Stores Change Tonight?](./02-services/07-deployment/12-how-many-stores-change-tonight.md)
- [AWS Elastic Beanstalk: The Turnkey Franchise](./02-services/07-deployment/13-the-turnkey-franchise.md)
- [AWS Deployment: The Morning After the Rollout](./02-services/07-deployment/14-the-morning-after-the-rollout.md)
- [AWS CloudShell, AWS CLI, and Amazon Q Developer: The Franchise Control Desk](./02-services/07-deployment/15-aws-cloudshell-cli-and-amazon-q-developer-the-franchise-control-desk.md)

## 02.08 — Observability and Optimization

How do we prove what happened and make the next shift better?

- [Amazon CloudWatch, AWS X-Ray, and AWS CloudTrail: The Operations Room](./02-services/08-observability/00-observability-operations-room.md)
- [Amazon CloudWatch: The Operations Manager](./02-services/08-observability/01-cloudwatch-operations-manager.md)
- [Amazon CloudWatch Logs: The Incident Notebook](./02-services/08-observability/02-cloudwatch-logs-incident-notebook.md)
- [Amazon CloudWatch Logs Insights: The Evidence Wall](./02-services/08-observability/03-cloudwatch-logs-insights-evidence-wall.md)
- [AWS X-Ray: The Store Manager](./02-services/08-observability/04-xray-store-manager.md)
- [AWS X-Ray: The Marked Receipt](./02-services/08-observability/05-xray-marked-receipt.md)
- [AWS CloudTrail: The Security Guard](./02-services/08-observability/06-cloudtrail-security-guard.md)
- [AWS Lambda: The Failing Station](./02-services/08-observability/07-lambda-failing-station.md)
- [Amazon API Gateway and Integrations: The Blocked Counter](./02-services/08-observability/08-api-gateway-integration-blocked-counter.md)
- [Amazon DynamoDB: The Hot Pantry](./02-services/08-observability/09-dynamodb-hot-pantry.md)
- [Troubleshooting and Optimization: The General Manager](./02-services/08-observability/10-troubleshooting-general-manager.md)
- [Troubleshooting and Optimization: The Better Shift](./02-services/08-observability/11-optimization-better-shift.md)

## 02.09 — Networking and Content Delivery

How do requests find the right entrance and intended path?

- [Amazon VPC, Amazon Route 53, and Amazon CloudFront: Byte Burger's Perimeter](./02-services/09-networking/00-the-restaurant-perimeter.md)
- [Amazon VPC: Byte Burger's Property Line](./02-services/09-networking/01-vpc-property-line.md)
- [Security Groups and Network ACLs: The Doors and the Fence](./02-services/09-networking/02-security-groups-and-nacls.md)
- [Amazon Route 53: The Address, Signpost, and Branch Director](./02-services/09-networking/03-route-53-address-and-director.md)
- [Amazon CloudFront: The Lunchtime Food Truck](./02-services/09-networking/04-cloudfront-lunchtime-food-truck.md)
- [VPC Endpoints and NAT Gateway: The Private Supplier Gate and Service Exit](./02-services/09-networking/05-vpc-endpoints-and-nat.md)
- [Networking: The Locked Kitchen](./02-services/09-networking/06-networking-locked-kitchen.md)

## 02.10 — Systems Under Pressure

What happens when traffic, failure, and change arrive together?

- [AWS Applications: The Saturday Rush](./02-services/10-systems-under-pressure/00-aws-applications-the-saturday-rush.md)
- [Amazon API Gateway, Amazon SQS, and AWS Lambda: Accepted Is Not Complete](./02-services/10-systems-under-pressure/01-api-gateway-sqs-lambda-accepted-is-not-complete.md)
- [AWS SDK and AWS Step Functions: The Payment Supplier Slows Down](./02-services/10-systems-under-pressure/02-sdk-step-functions-payment-supplier-slows-down.md)
- [Amazon DynamoDB, Amazon ElastiCache, and Amazon SQS: The Promotion Finds the Hot Shelf](./02-services/10-systems-under-pressure/03-dynamodb-elasticache-sqs-promotion-finds-hot-shelf.md)
- [Amazon CloudWatch, AWS X-Ray, and AWS CloudTrail: The Change Nobody Noticed](./02-services/10-systems-under-pressure/04-cloudwatch-xray-cloudtrail-change-nobody-noticed.md)
- [Amazon SQS, AWS Step Functions, and Amazon DynamoDB: The Recovery Shift](./02-services/10-systems-under-pressure/05-sqs-step-functions-dynamodb-recovery-shift.md)
- [AWS Applications: The Restaurant Under Pressure](./02-services/10-systems-under-pressure/06-aws-applications-restaurant-under-pressure.md)

## 03 — Architecture

When is an architectural decision right, when does it expire, and how can complexity be removed?

- [Software Architecture: Byte Burger Grows Up](./03-architecture/00-software-architecture-byte-burger-grows-up.md)
- [Software Architecture: One Restaurant, One Application](./03-architecture/01-software-architecture-one-restaurant-one-application.md)
- [Software Architecture: The Friday Night Rush](./03-architecture/02-software-architecture-the-friday-night-rush.md)
- [Software Architecture: The Kitchen Cannot Answer the Phone](./03-architecture/03-software-architecture-the-kitchen-cannot-answer-the-phone.md)
- [Software Architecture: Store Number Two](./03-architecture/04-software-architecture-store-number-two.md)
- [Software Architecture: Franchisees Want Their Own Rules](./03-architecture/05-software-architecture-franchisees-want-their-own-rules.md)
- [Software Architecture: Delivery Apps Arrive](./03-architecture/06-software-architecture-delivery-apps-arrive.md)
- [Software Architecture: The Menu Says Available, the Kitchen Says Sold Out](./03-architecture/07-software-architecture-menu-and-kitchen-disagree.md)
- [Software Architecture: Marketing Creates a Stampede](./03-architecture/08-software-architecture-marketing-creates-a-stampede.md)
- [Software Architecture: Twenty Engineers, One Deployment](./03-architecture/09-software-architecture-twenty-engineers-one-deployment.md)
- [Software Architecture: The First Service Extraction](./03-architecture/10-software-architecture-the-first-service-extraction.md)
- [Software Architecture: The Incident Nobody Could Diagnose](./03-architecture/11-software-architecture-the-incident-nobody-could-diagnose.md)
- [Software Architecture: Orders and Accounting Stop Agreeing](./03-architecture/12-software-architecture-orders-and-accounting-stop-agreeing.md)
- [Software Architecture: The Country-Wide Promotion](./03-architecture/13-software-architecture-country-wide-promotion.md)
- [Software Architecture: Going International](./03-architecture/14-software-architecture-going-international.md)
- [Software Architecture: We Built Too Much](./03-architecture/15-software-architecture-we-built-too-much.md)
- [Software Architecture: The Great Consolidation](./03-architecture/16-software-architecture-the-great-consolidation.md)
- [Software Architecture: What We Would Build Today](./03-architecture/17-software-architecture-what-we-would-build-today.md)
