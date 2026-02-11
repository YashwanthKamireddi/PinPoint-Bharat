# CloudCost Copilot - Requirements Document

## Project Overview

**Project Name:** CloudCost Copilot  
**Track:** Student Track - AI for Learning & Developer Productivity  
**Tagline:** AI-powered AWS cost guardian that prevents bill shock and teaches cloud economics

## Problem Statement

### The Challenge
Students and developers learning AWS face a critical barrier: **fear of unexpected costs**. This fear prevents experimentation and learning, which are essential for cloud skill development.

**Real Statistics (Verified):**
- AWS free tier includes 750 hours/month EC2 (t2.micro/t3.micro only)
- Students commonly forget to stop instances after tutorials
- A single m5.large instance costs $69.12/month (exhausts $100 credits in 45 days)
- AWS Budgets alerts arrive 4-8 hours after threshold breach (too late)

### Current Pain Points
1. **Delayed Alerts:** AWS Budgets uses Cost Explorer data (4-hour delay)
2. **Complex Free Tier:** 60+ services with different rules (EC2: hours, S3: GB, Lambda: requests)
3. **No Context:** Alerts say "you spent $50" but not WHY or HOW TO FIX
4. **Fear-Based Learning:** Students avoid experimenting with new services
5. **Manual Monitoring:** Checking console every hour is impractical

### What Exists Today (Competitive Landscape)
- **AWS Budgets:** Free, but 4-hour delay, no recommendations
- **AWS Cost Anomaly Detection:** Detects unusual spend, but no prevention
- **CloudHealth/Cloudability:** Enterprise-focused ($500+/month), complex setup
- **AWS Trusted Advisor:** Basic checks, but requires Business/Enterprise support ($100+/month)

## Solution Overview

CloudCost Copilot is an **AI-powered AWS cost optimization platform** that prevents bill shock through intelligent monitoring, ML-based predictions, and agentic AI recommendations.

### What It Does
1. **Monitors AWS costs hourly** (limited by Cost Explorer 4-hour delay)
2. **Predicts future spend** using Amazon Forecast (7-day ML forecasts)
3. **Generates AI recommendations** via Amazon Bedrock Agents (autonomous cost optimization)
4. **Integrates with developer workflows** (Slack, GitHub Actions, CI/CD)
5. **Teaches cloud economics** through contextual explanations and gamification
6. **Executes optimizations safely** (user approval required, backups created)

### Core Value Propositions

**For Students:**  
*"Learn AWS fearlessly - your AI copilot prevents bill shock and teaches cloud economics"*

**For Developers:**  
*"Ship fast without surprise bills - automated cost reviews in your workflow"*

**For Startups:**  
*"Optimize AWS spend without hiring a FinOps engineer - save 30% on autopilot"*

### Key Differentiators
- ✅ **Agentic AI:** Uses Amazon Bedrock Agents (not just text generation)
- ✅ **Developer-first:** Slack bots, GitHub Actions, API-first architecture
- ✅ **Educational:** Gamification, learning paths, achievement system
- ✅ **Safe:** User approval required, backups created, audit logs
- ✅ **Latest AWS features:** Database Savings Plans, Compute Optimizer, Cost Anomaly Detection

### Honest Limitations
- **Not real-time:** Cost data has 4-hour delay (AWS Cost Explorer limitation)
- **MVP scope:** Supports EC2, S3, Lambda, RDS, DynamoDB (covers 85% of costs)
- **Requires IAM permissions:** Users must create read-only IAM role (5-min setup)
- **Predictions need history:** Requires 7+ days of usage data for accurate forecasts
- **Single AWS account in free tier:** Multi-account support in Pro tier ($10/month)

## Target Users

### Primary Users (Hackathon Focus)
- **Computer Science Students** (18-24 years)
  - Learning cloud computing for coursework
  - Building personal projects with AWS Educate credits
  - Preparing for AWS certifications

### Secondary Users (Broader Market)
- **Professional Developers**
  - Managing personal AWS accounts for side projects
  - Learning new AWS services without cost anxiety
  - Experimenting with serverless architectures
  
- **Indie Hackers & Solo Founders**
  - Building MVPs on tight budgets ($100-500/month)
  - Need cost visibility without hiring FinOps engineer
  - Shipping fast without surprise bills

- **Startup Engineering Teams (2-10 people)**
  - Pre-Series A startups optimizing burn rate
  - Need cost accountability per developer
  - Want automated cost reviews in CI/CD

- **Bootcamp Students & Career Switchers**
  - Learning cloud development
  - Building portfolio projects
  - Limited AWS credits ($100-200)

### User Personas

**Persona 1: "Anxious Student Alex"**
- Age: 21, CS major
- AWS credits: $100 (AWS Educate)
- Pain: Scared to try new services, checks console 10x/day
- Goal: Learn AWS without exhausting credits

**Persona 2: "Indie Hacker Priya"**
- Age: 28, solo founder
- AWS spend: $150/month
- Pain: No time to optimize costs, bills creeping up
- Goal: Automate cost monitoring, focus on product

**Persona 3: "Startup Dev Raj"**
- Age: 26, backend engineer at 5-person startup
- AWS spend: $2K/month (team account)
- Pain: No visibility into who's spending what
- Goal: Cost accountability, prevent waste before deployment

## Functional Requirements

### FR1: Hourly Cost Monitoring
**Priority:** Critical  
**Description:** Monitor AWS account costs with best-effort frequency

**Acceptance Criteria:**
- System polls AWS Cost Explorer API every 1 hour (respects 4-hour data delay)
- Tracks costs for MVP services: EC2, S3, Lambda, RDS (expandable later)
- Displays current spend vs. budget (updated hourly, labeled "Last updated: X hours ago")
- Shows cost breakdown by service
- Implements exponential backoff for API rate limits (4 requests/second max)
- Caches Cost Explorer responses for 1 hour to reduce API costs

**Technical Constraints:**
- Cost Explorer data is delayed 4-8 hours (AWS limitation)
- API cost: $0.01 per request (must minimize calls)
- Rate limit: 4 requests/second

### FR2: Cost Forecasting
**Priority:** Critical  
**Description:** Predict future costs using Amazon Forecast service

**Acceptance Criteria:**
- Uses Amazon Forecast (time-series ML) for 7-day and 30-day predictions
- Requires minimum 7 days of historical data (shows "Insufficient data" otherwise)
- Displays prediction with confidence interval (e.g., "$45-55 with 80% confidence")
- Alerts trigger when forecasted spend exceeds thresholds:
  - Warning: 70% of budget
  - Critical: 90% of budget
- Notifications via email and in-app (SMS optional, costs extra)
- Prediction accuracy target: ±20% (realistic for student usage patterns)

**Technical Implementation:**
- Amazon Forecast dataset: daily cost data
- Predictor: DeepAR+ algorithm (handles seasonality)
- Forecast horizon: 7 days (updated daily)
- Fallback: Linear regression if Forecast API fails

### FR3: Waste Detection (CloudWatch-Based)
**Priority:** Critical  
**Description:** Identify idle or underutilized resources using CloudWatch metrics

**Acceptance Criteria:**
- **EC2 Idle Detection:**
  - CPU < 5% for 4+ hours (excludes Spot instances)
  - Network I/O < 1 MB/hour
  - Excludes instances with tags: `Environment=production` or `AutoStop=false`
- **Unattached EBS Volumes:**
  - State = "available" (not attached to any instance)
  - Age > 7 days (grace period for new volumes)
- **S3 Infrequent Access:**
  - Buckets with < 100 GET requests/month (CloudWatch S3 metrics)
  - Recommend Glacier/Intelligent-Tiering
- **RDS Low Usage:**
  - DatabaseConnections < 5 for 24+ hours
  - CPU < 10% for 24+ hours
- **Free Tier Tracking (MVP: EC2, S3, Lambda only):**
  - EC2: 750 hours/month (t2.micro/t3.micro only)
  - S3: 5 GB storage, 20K GET, 2K PUT
  - Lambda: 1M requests, 400K GB-seconds
- Provides cost impact: "Stopping this instance saves $0.0116/hour = $8.35/month"

**Safety Rules:**
- Never flag resources with `DoNotDelete` or `Production` tags
- Exclude Reserved Instances and Savings Plans (stopping doesn't save money)
- Exclude instances in Auto Scaling Groups

### FR4: Agentic AI Recommendations (Bedrock Agents)
**Priority:** Critical  
**Description:** Autonomous AI agents that analyze costs, generate recommendations, and execute optimizations

**Architecture:**
- Uses **Amazon Bedrock Agents** with multi-agent collaboration
- Three specialized agents working together:
  1. **Analyzer Agent:** Detects waste, analyzes patterns, identifies anomalies
  2. **Optimizer Agent:** Generates recommendations using AWS Compute Optimizer + Database Savings Plans
  3. **Executor Agent:** Executes approved optimizations via function calling (tool use)

**Acceptance Criteria:**

**Agent 1: Analyzer Agent**
- **Purpose:** Continuous cost monitoring and anomaly detection
- **Model:** Amazon Nova Lite
- **Data Sources:**
  - Cost Explorer API (hourly polling)
  - CloudWatch metrics (EC2 CPU, RDS connections, S3 requests)
  - AWS Cost Anomaly Detection API
- **Capabilities:**
  - Detects idle resources (EC2 CPU < 5% for 4+ hours)
  - Identifies cost spikes (>20% increase week-over-week)
  - Analyzes spending patterns (weekday vs weekend, time of day)
  - Triggers Optimizer Agent when waste detected
- **Output:** Structured waste report with resource IDs and cost impact

**Agent 2: Optimizer Agent**
- **Purpose:** Generate actionable cost optimization recommendations
- **Model:** Amazon Nova Lite (primary), Claude 3.5 Haiku (fallback)
- **Integrations:**
  - **AWS Compute Optimizer API:** Official EC2/Lambda rightsizing recommendations
  - **Database Savings Plans API:** Analyze RDS/DynamoDB for commitment opportunities
  - **AWS Trusted Advisor API:** Cross-reference with TA recommendations (if available)
- **Recommendation Types:**
  1. **Immediate:** Stop idle EC2, delete unattached EBS (saves money today)
  2. **Short-term:** Rightsize instances, switch storage classes (1-week implementation)
  3. **Long-term:** Reserved Instances, Savings Plans (requires commitment)
- **Each recommendation includes:**
  - Estimated savings ($/month) with detailed calculation
  - Implementation difficulty (Easy/Medium/Hard)
  - Step-by-step instructions (AWS Console + CLI + Terraform)
  - Trade-offs and risks (e.g., "Stopping instance = downtime")
  - Rollback instructions
  - Confidence score (0-100%)
  - Data sources (CloudWatch, Compute Optimizer, Cost Explorer)

**Agent 3: Executor Agent**
- **Purpose:** Safely execute approved optimizations via AWS API calls
- **Model:** Amazon Nova Lite
- **Tool Use (Function Calling):**
  - `stop_ec2_instance(instance_id)` → Calls `ec2:StopInstances`
  - `create_ami_backup(instance_id, name)` → Calls `ec2:CreateImage`
  - `delete_ebs_volume(volume_id)` → Calls `ec2:DeleteVolume`
  - `create_ebs_snapshot(volume_id)` → Calls `ec2:CreateSnapshot`
  - `modify_s3_lifecycle(bucket, policy)` → Calls `s3:PutLifecycleConfiguration`
  - `purchase_savings_plan(commitment)` → Calls `savingsplans:CreateSavingsPlan`
- **Safety Mechanisms:**
  - Requires explicit user approval (no auto-execution)
  - Creates backups before destructive actions (AMI, snapshots)
  - Validates IAM permissions before execution
  - Logs all actions in DynamoDB audit trail
  - Sends email confirmation after each action
  - Rate limit: Max 5 actions per user per day

**Multi-Agent Collaboration Example:**
```
1. Analyzer Agent detects: "EC2 i-abc123 idle for 6 hours"
2. Analyzer triggers Optimizer Agent with waste data
3. Optimizer Agent:
   - Queries AWS Compute Optimizer: "Recommends stopping"
   - Checks CloudWatch: CPU 2% avg, Network 0.1 MB/hr
   - Generates recommendation: "Stop instance, save $8.35/month"
4. User approves recommendation in Slack
5. Executor Agent:
   - Creates AMI backup (takes 5 minutes)
   - Stops EC2 instance
   - Sends confirmation: "Instance stopped, backup: ami-xyz789"
```

**Integration with Latest AWS Features:**
- **AWS Compute Optimizer:** Get official rightsizing recommendations (not just our rules)
- **Database Savings Plans (Dec 2025):** Recommend RDS/DynamoDB commitments (up to 35% savings)
- **AWS Cost Anomaly Detection:** Leverage AWS's ML for anomaly detection
- **Model Context Protocol (MCP):** Future-proof agent-tool communication

**Cost Control:**
- Limit Bedrock Agent invocations to 1,000/day across all users
- Use prompt caching for repeated queries (90% cost reduction)
- Estimated cost: $5-10/month for 1,000 users

**Example Recommendation (Enhanced):**
```json
{
  "id": "rec_abc123",
  "type": "idle_ec2",
  "title": "Stop idle EC2 instance i-1234567890abcdef0",
  "description": "This t3.micro instance has been idle (CPU < 5%) for 6 hours. AWS Compute Optimizer also recommends stopping it.",
  "savings": {
    "monthly": 8.35,
    "annual": 100.20,
    "calculation": "t3.micro: $0.0116/hour × 720 hours/month"
  },
  "difficulty": "easy",
  "implementation": {
    "console": ["Go to EC2 Console", "Select instance i-1234567890abcdef0", "Actions → Stop"],
    "cli": "aws ec2 stop-instances --instance-ids i-1234567890abcdef0",
    "terraform": "# Set desired_capacity = 0 in your ASG config",
    "one_click": "Approve in Slack to execute via Bedrock Executor Agent"
  },
  "tradeoffs": "Instance will be unavailable. EBS data persists. You can restart anytime.",
  "rollback": "aws ec2 start-instances --instance-ids i-1234567890abcdef0",
  "confidence": 95,
  "sources": ["CloudWatch metrics (6h avg CPU: 2%)", "AWS Compute Optimizer", "Cost Explorer"],
  "tags": ["quick_win", "no_risk", "reversible"]
}
```

### FR5: Assisted Optimization (NOT Fully Automated)
**Priority:** High  
**Description:** Help users execute optimizations safely

**Acceptance Criteria:**
- **User must explicitly approve each action** (no auto-execution)
- **Confirmation flow:**
  1. User clicks "Approve" on recommendation
  2. System shows detailed confirmation modal:
     - Resource details (ID, type, current cost)
     - Exact action to be taken
     - Estimated savings
     - Risks and rollback steps
  3. User must type resource ID to confirm (prevents accidental clicks)
  4. System creates snapshot/backup (if applicable)
  5. System executes action via AWS SDK
  6. System sends confirmation email with rollback instructions

- **Supported Actions (MVP):**
  - Stop EC2 instances (creates AMI backup first)
  - Delete unattached EBS volumes (creates snapshot first)
  - Change S3 storage class to Intelligent-Tiering (reversible)
  - ~~Modify RDS instance types~~ (NOT in MVP - too risky, causes downtime)

- **Safety Mechanisms:**
  - Dry-run mode: Preview changes without executing
  - Rollback instructions provided for every action
  - Audit log stored in DynamoDB (immutable)
  - Email notification after every action
  - Rate limit: Max 5 actions per user per day (prevent bulk mistakes)

- **IAM Permissions Required (User's AWS Account):**
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:StopInstances",
          "ec2:CreateImage",
          "ec2:DeleteVolume",
          "ec2:CreateSnapshot",
          "s3:PutLifecycleConfiguration"
        ],
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "aws:RequestedRegion": ["us-east-1", "us-west-2"]
          }
        }
      }
    ]
  }
  ```
  - User must explicitly grant these (separate from read-only role)
  - Permissions are optional (app works without them, just no auto-execution)

### FR6: Educational Insights
**Priority:** High  
**Description:** Teach cloud economics through contextual explanations

**Acceptance Criteria:**

**Contextual Learning:**
- **Real-time Explanations:** AI explains WHY resources cost money when user views them
  - Example: "EC2 charges per hour ($0.0116/hr for t3.micro), even when idle. Stopping = no compute charges, but EBS storage still costs $0.10/GB/month"
- **Cost Comparisons:** Shows alternatives with calculations
  - Example: "Lambda is 80% cheaper for your use case (runs 10 min/day): Lambda $0.50/month vs EC2 $8.35/month"
- **AWS Documentation Links:** Direct links to relevant pricing pages
- **Best Practices:** Suggests AWS Well-Architected cost optimization patterns

**Achievement System (Simple):**
- **Milestone Badges (6 total):**
  - 🏆 **First Save:** Implemented first recommendation
  - 💰 **Cost Saver:** Saved $50 total
  - 🔥 **Week Streak:** Stayed under budget for 7 days
  - 📚 **AWS Learner:** Read 10 cost explanations
  - 🎯 **Budget Master:** Stayed under budget for 30 days
  - 🌟 **Power User:** Connected Slack + GitHub

**Progress Tracking:**
- Dashboard shows: "You've saved $X this month"
- Savings history chart (last 30 days)
- Recommendations implemented count
- Free tier usage percentage

**No Leaderboards, No Points, No Challenges** - Keep it simple and privacy-focused

### FR7: Budget Management
**Priority:** High  
**Description:** Set and track spending budgets

**Acceptance Criteria:**
- Users can set daily/weekly/monthly budgets
- Visual progress bars show spend vs. budget
- Automatic pause of resources when budget exceeded (optional)
- Budget recommendations based on usage patterns

### FR8: Developer Workflow Integration
**Priority:** Critical (for broader market)  
**Description:** Integrate with tools developers actually use

**Acceptance Criteria:**

**Slack Integration:**
- Slack bot: `/cloudcost status` shows current spend
- Real-time alerts in #aws-costs channel
- Interactive buttons: "Approve" recommendation directly in Slack
- Daily digest: "Yesterday's spend: $12.50 (+5% vs. avg)"
- Team notifications: "@channel Budget at 90%!"

**GitHub Actions Integration:**
- GitHub Action runs on every PR
- Comments on PR: "This deployment will cost ~$X/month"
- Blocks merge if cost > threshold (configurable)
- Example comment:
  ```
  💰 CloudCost Copilot Analysis
  
  Estimated monthly cost: $45.00
  Changes:
  - New Lambda function: +$5.00/month
  - New DynamoDB table: +$2.50/month
  
  ✅ Within budget ($100/month)
  ```

**VS Code Extension:**
- Shows cost estimates inline in Terraform/CDK files
- Hover over resource: "This EC2 instance costs $0.096/hour = $69.12/month"
- Suggests cheaper alternatives: "💡 Switch to t3.medium to save $20/month"
- Real-time validation: Red underline if resource exceeds budget
- Commands:
  - `CloudCost: Estimate Current File` - Calculate total cost
  - `CloudCost: Check Budget` - Show current spend
  - `CloudCost: Optimize` - Get recommendations
- Supports: Terraform (.tf), AWS CDK (TypeScript/Python), CloudFormation (.yaml)

**CI/CD Integration:**
- Pre-deployment cost estimation
- Terraform/CDK cost analysis
- Alert if infrastructure change increases costs by >20%

**API-First Architecture:**
- RESTful API for all features
- Webhooks for custom integrations
- SDKs: Python, JavaScript, Go
- Example:
  ```python
  from cloudcost import CloudCostClient
  
  client = CloudCostClient(api_key="...")
  spend = client.get_current_spend()
  if spend > 80:
      client.send_alert("Budget warning!")
  ```

**CLI Tool:**
- `cloudcost status` - Show current spend
- `cloudcost recommendations` - List optimizations
- `cloudcost approve <rec_id>` - Execute recommendation
- `cloudcost budget set 100` - Set monthly budget
- `cloudcost terraform estimate` - Estimate Terraform costs

### FR9: Multi-Account & Team Features
**Priority:** High (for startups)  
**Description:** Support teams and multiple AWS accounts

**Acceptance Criteria:**

**Multi-Account Management:**
- Connect up to 10 AWS accounts (Pro tier)
- Consolidated cost view across accounts
- Per-account breakdown
- Account tagging: "Production", "Staging", "Dev"

**Team Collaboration:**
- Invite team members (email-based)
- Role-based access control:
  - **Owner:** Full access, billing
  - **Admin:** Approve optimizations, view all data
  - **Member:** View-only, can't approve actions
  - **Viewer:** Dashboard access only
- Activity feed: "Raj stopped EC2 instance i-abc123"
- Comments on recommendations: "Should we approve this?"

**Cost Allocation:**
- Tag-based cost allocation
- Per-developer cost tracking (via tags)
- Department/project cost breakdown
- Chargeback reports: "Frontend team: $120, Backend team: $180"

**Shared Budgets:**
- Team budget: $500/month
- Per-developer budgets: $50/month each
- Alerts when individual exceeds personal budget

### FR9: Cost Comparison & Benchmarking
**Priority:** Medium  
**Description:** Compare costs against similar projects/users

**Acceptance Criteria:**
- Anonymous benchmarking: "Your costs are 40% higher than similar projects"
- Best practices from efficient users
- Industry-standard cost baselines

### FR10: Integration with Learning Platforms
**Priority:** Low  
**Description:** Integrate with educational platforms (Coursera, Udemy, AWS Educate)

**Acceptance Criteria:**
- Detect course-related resources (via tags)
- Provide course-specific cost guidance
- Share cost reports with instructors

## Non-Functional Requirements

### NFR1: Performance
- Dashboard loads in < 3 seconds (realistic with API calls)
- Cost predictions generated in < 10 seconds (Amazon Forecast latency)
- Alerts delivered within 1 hour of threshold breach (limited by Cost Explorer delay)

### NFR2: Scalability
- Support 1,000 concurrent users (MVP target)
- Handle 10 AWS accounts per user
- Process 100K+ cost data points per day

### NFR3: Security
- AWS credentials stored encrypted (AWS Secrets Manager)
- **Read-only IAM permissions by default** (minimal scope):
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ce:GetCostAndUsage",
          "ce:GetCostForecast",
          "cloudwatch:GetMetricStatistics",
          "cloudwatch:ListMetrics",
          "ec2:DescribeInstances",
          "ec2:DescribeVolumes",
          "ec2:DescribeInstanceTypes",
          "rds:DescribeDBInstances",
          "s3:ListAllMyBuckets",
          "s3:GetBucketLocation",
          "s3:GetMetricsConfiguration"
        ],
        "Resource": "*"
      }
    ]
  }
  ```
- Write permissions require explicit user approval (separate role)
- Data encrypted at rest (DynamoDB KMS, S3 SSE)
- Data encrypted in transit (TLS 1.3)

### NFR4: Reliability
- 99% uptime SLA (realistic for student project)
- Automated error recovery (retry with exponential backoff)
- Data backup every 24 hours (DynamoDB point-in-time recovery)

### NFR5: Usability
- Onboarding completed in < 10 minutes (includes IAM role setup)
- Mobile-responsive design (works on phones)
- Accessible (WCAG 2.1 AA compliance target)

### NFR6: Cost Efficiency
- Platform operational cost < $0.05 per user/month (target)
- Serverless architecture to minimize fixed costs

## User Stories

### US1: Prevent Bill Shock
**As a** student learning AWS  
**I want** real-time alerts when I'm about to overspend  
**So that** I don't exhaust my credits unexpectedly

### US2: Learn Cloud Economics
**As a** beginner developer  
**I want** explanations of why resources cost money  
**So that** I can make informed architectural decisions

### US3: Optimize Automatically
**As a** busy student  
**I want** one-click cost optimizations  
**So that** I don't waste time manually managing resources

### US4: Track Free Tier Usage
**As a** AWS free tier user  
**I want** visibility into free tier limits  
**So that** I can maximize free resources before paying

### US5: Compare Alternatives
**As a** developer choosing services  
**I want** cost comparisons between options (EC2 vs Lambda vs Fargate)  
**So that** I can pick the most cost-effective solution

## Technical Constraints

### TC1: AWS Service Limitations
- **Cost Explorer API:** 4-8 hour data delay (cannot be avoided)
- **Cost Explorer API:** $0.01 per request (must minimize calls)
- **Cost Explorer API:** 4 requests/second rate limit
- **CloudWatch metrics:** 5-minute granularity (best available)
- **Free tier limits:** Vary by service and region

### TC2: AI Model Constraints
- **Amazon Bedrock:** Rate limits vary by model (Nova Lite: 100 req/min)
- **Amazon Forecast:** Requires 7+ days of data for accurate predictions
- **Prediction accuracy:** ±20% realistic for variable student workloads

### TC3: User Permissions
- Users must grant IAM read permissions (setup friction)
- Write permissions optional (for auto-remediation)
- AWS Organizations accounts may have restricted billing access

### TC4: Regional Availability
- Cost Explorer available in us-east-1 only (global data, but API endpoint is regional)
- Amazon Forecast available in limited regions (us-east-1, us-west-2, eu-west-1)
- Solution works for accounts in any region, but APIs called from us-east-1

## Success Metrics

### Primary Metrics (Hackathon Evaluation)
1. **Cost Savings:** Average 30% reduction in AWS spend per user
2. **Adoption:** 500 active users within 3 months (students + developers)
3. **Engagement:** 50% weekly retention rate

### Secondary Metrics
4. **Learning Impact:** 70% of users report better understanding of AWS pricing
5. **Bill Shock Prevention:** 80% of users avoid unexpected charges
6. **Recommendation Acceptance:** 40% of recommendations implemented
7. **Developer Integration:** 30% of users connect Slack/GitHub within first week

### Business Metrics (Post-Hackathon)
8. **Revenue:** $5K MRR within 6 months (500 users × $10/month)
9. **Viral Coefficient:** 1.2 (each user refers 1.2 new users)
10. **Churn Rate:** <10% monthly
11. **NPS Score:** >50 (promoters - detractors)

### Technical Metrics
12. **API Uptime:** 99% (realistic for MVP)
13. **Alert Latency:** <2 hours (limited by Cost Explorer delay)
14. **Prediction Accuracy:** ±20% for 70% of forecasts
15. **Bedrock Agent Success Rate:** >90% of recommendations are actionable

## Out of Scope (Future Enhancements)

### Phase 2 (Post-Hackathon)
- Multi-cloud support (Azure, GCP)
- Advanced FinOps features (chargeback, showback, cost allocation tags)
- Custom pricing models for enterprises
- Mobile native apps (iOS/Android)
- Terraform/CDK cost estimation (pre-deployment)
- AWS Marketplace listing

### Phase 3 (Scale)
- White-label solution for universities
- Enterprise SSO (SAML, Okta)
- Advanced analytics (BigQuery, Snowflake integration)
- Custom ML models (per-user prediction tuning)
- Multi-region deployment (global availability)

### Not Planned
- Support for non-AWS clouds in MVP
- Real-time monitoring (limited by AWS Cost Explorer 4-hour delay)
- Automatic optimization without user approval (safety-first approach)
- Support for all 200+ AWS services (focus on top 10 that cover 85% of costs)

## Assumptions

1. Users have basic AWS knowledge (can create IAM users)
2. Users have access to AWS Cost Explorer (enabled by default)
3. Users are willing to grant read-only IAM permissions
4. Internet connectivity available for real-time monitoring

## Dependencies

### External Dependencies
- AWS Cost Explorer API (cost data)
- AWS CloudWatch API (resource metrics)
- Amazon Bedrock API (AI recommendations)
- AWS SDK for resource management

### Internal Dependencies
- User authentication system
- Database for historical cost data
- Background job scheduler for monitoring

## Risks & Mitigation

### Risk 1: AWS API Rate Limits
**Impact:** High  
**Probability:** Medium  
**Mitigation:** 
- Implement exponential backoff (2^n seconds delay)
- Cache Cost Explorer responses for 1 hour
- Batch requests where possible
- Monitor rate limit errors in CloudWatch

### Risk 2: Inaccurate Cost Predictions
**Impact:** High  
**Probability:** High (student usage is unpredictable)  
**Mitigation:** 
- Display confidence intervals (e.g., "$45-55 with 70% confidence")
- Require 7+ days of data before showing predictions
- Use conservative estimates (round up)
- Clearly label as "estimates, not guarantees"

### Risk 3: User Distrust of Optimization Actions
**Impact:** Medium  
**Probability:** High  
**Mitigation:** 
- Require explicit approval for every action
- Show detailed confirmation with risks
- Create backups before destructive actions
- Provide rollback instructions
- Build trust through transparency

### Risk 4: AWS Pricing Changes
**Impact:** Medium  
**Probability:** Low  
**Mitigation:** 
- Use AWS Price List API for up-to-date pricing
- Update pricing data weekly
- Notify users of significant pricing changes

### Risk 5: Cost Explorer API Costs Exceed Revenue
**Impact:** Critical  
**Probability:** Medium  
**Mitigation:**
- Poll hourly (not every 5 minutes)
- Aggressive caching (1-hour TTL)
- Limit free tier to 10 API calls/day per user
- Charge premium users for more frequent updates

### Risk 6: AWS Organizations Billing Access
**Impact:** Medium  
**Probability:** Medium (many students use university accounts)  
**Mitigation:**
- Document Organizations limitations in onboarding
- Provide workaround guide (request management account access)
- Scope MVP to "individual AWS accounts only"

## Compliance & Legal

- **Data Privacy:** GDPR, CCPA compliant (no PII stored)
- **AWS Terms of Service:** Comply with AWS Acceptable Use Policy
- **Open Source Licenses:** MIT license for codebase

## Glossary

- **Free Tier:** AWS's free usage limits for new accounts (12 months)
- **Idle Resource:** AWS resource consuming costs with minimal utilization
- **Cost Explorer:** AWS service for viewing and analyzing costs
- **IAM:** Identity and Access Management (AWS permissions system)
- **Bedrock:** AWS's managed generative AI service

---

**Document Version:** 1.0  
**Last Updated:** February 11, 2026  
**Author:** AI for Bharat Hackathon Team
