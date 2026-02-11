# CloudCost Copilot

> AI-powered AWS cost monitoring platform for students - Learn cloud computing without fear of bill shock

**Hackathon:** AI for Bharat 2026 (Student Track - AI for Learning & Developer Productivity)

---

## 🎯 Problem

Students learning AWS face a critical barrier: **fear of unexpected costs**. 

- AWS free tier includes 750 hours/month EC2, but only for t2.micro/t3.micro
- A single forgotten m5.large instance costs $69.12/month (exhausts $100 credits in 45 days)
- AWS Budgets alerts arrive 4-8 hours after threshold breach (too late)
- Students avoid experimenting with new services due to cost anxiety

**Existing solutions:**
- AWS Budgets: Free, but 4-hour delay, no recommendations
- CloudHealth/Cloudability: Enterprise-focused ($500+/month)
- AWS Trusted Advisor: Requires Business Support ($100+/month)

---

## 💡 Solution

CloudCost Copilot provides:

1. **Hourly Cost Monitoring** - Track AWS spend with 1-hour updates (limited by Cost Explorer 4-hour delay)
2. **ML-Based Predictions** - Amazon Forecast predicts 7-day costs with confidence intervals
3. **AI Recommendations** - Amazon Bedrock (Nova Lite) generates optimization suggestions
4. **Educational Insights** - Learn WHY resources cost money, not just HOW MUCH
5. **Safe Optimizations** - User approval required for all actions (no accidental deletions)

---

## 🏗️ Architecture

### AWS Services Used

| Service | Purpose | Why This Service |
|---------|---------|------------------|
| **Amazon Bedrock (Nova Lite)** | AI recommendations | Cost-effective ($0.06/1M tokens vs. $3 for Claude) |
| **Amazon Forecast** | Cost predictions | Purpose-built for time-series ML (not generic LLM) |
| **AWS Lambda** | Serverless compute | Pay-per-use, auto-scales, no servers to manage |
| **Amazon DynamoDB** | NoSQL database | Serverless, fast, pay-per-request |
| **AWS Cost Explorer API** | Cost data source | Only API for AWS billing data |
| **Amazon CloudWatch** | Resource metrics | Detect idle EC2, underutilized RDS |
| **Amazon API Gateway** | REST API | Managed API with Cognito integration |
| **Amazon Cognito** | Authentication | Secure user management |
| **AWS Amplify** | Frontend hosting | CI/CD + custom domain + SSL |
| **Amazon SNS** | Notifications | Email/SMS alerts |
| **Amazon EventBridge** | Scheduled tasks | Cron-like scheduling for Lambda |

### High-Level Flow

```
User → Amplify (React App) → API Gateway → Lambda Functions
                                              ↓
                                    ┌─────────┴─────────┐
                                    ↓                   ↓
                            Cost Explorer API    CloudWatch Metrics
                                    ↓                   ↓
                            Amazon Bedrock      Amazon Forecast
                                    ↓                   ↓
                                DynamoDB (Storage)
                                    ↓
                            SNS (Alerts) → User Email/SMS
```

---

## 🎨 Key Features

### 1. Cost Monitoring
- Hourly polling of AWS Cost Explorer API
- Service-wise cost breakdown (EC2, S3, Lambda, RDS)
- Free tier usage tracking (EC2: 750h, S3: 5GB, Lambda: 1M requests)
- Budget vs. actual spend visualization

### 2. Predictive Alerts
- Amazon Forecast generates 7-day predictions
- Alerts at 70% (warning) and 90% (critical) of budget
- Confidence intervals displayed (e.g., "$45-55 with 80% confidence")
- Requires 7+ days of historical data

### 3. Waste Detection
- **Idle EC2:** CPU < 5% for 4+ hours (excludes Spot, ASG, Production tags)
- **Unattached EBS:** Volumes not attached for 7+ days
- **Underutilized RDS:** < 5 connections for 24h, CPU < 10%
- **Infrequent S3:** < 100 GET requests/month

### 4. AI Recommendations
- Amazon Bedrock (Nova Lite) generates 3-5 suggestions per day
- Each recommendation includes:
  - Estimated savings ($/month)
  - Step-by-step instructions
  - Trade-offs and risks
  - Rollback instructions
- Cached for 24 hours (reduces API costs)

### 5. Safe Optimizations
- User must approve each action
- Confirmation requires typing resource ID
- Creates backups before destructive actions (AMI for EC2, snapshot for EBS)
- Audit log of all changes
- Rate limit: Max 5 actions/day per user

---

## 📊 Cost Analysis

### Operational Costs (1,000 users)

| Service | Monthly Cost |
|---------|--------------|
| Lambda | $19.07 |
| DynamoDB | $4.38 |
| API Gateway | $7.00 |
| Bedrock (Nova Lite) | $0.27 |
| Forecast | $6.60 |
| Cost Explorer API | $24.00 |
| CloudWatch | $11.00 |
| Other (SNS, S3, Amplify) | $37.55 |
| **TOTAL** | **$109.87/month** |

**Per-User Cost:** $0.11/month  
**Revenue Target:** $10/user/month (freemium)  
**Gross Margin:** 99%

### Cost Optimizations Applied
1. Use Nova Lite (60x cheaper than Claude Sonnet)
2. Poll hourly (not every 5 minutes) - saves $216/month
3. Cache aggressively (24h for recommendations, 1h for costs)
4. Use DynamoDB on-demand (no provisioned capacity)

---

## 🚧 Known Limitations

### 1. Cost Explorer Data Delay (4-8 hours)
- **Impact:** Cannot provide true "real-time" alerts
- **Mitigation:** Honest labeling ("Last updated: 4 hours ago")
- **Why:** AWS limitation, no workaround

### 2. MVP Scope (EC2, S3, Lambda, RDS Only)
- **Impact:** Doesn't cover all 200+ AWS services
- **Mitigation:** Covers 80% of student costs
- **Future:** Add more services based on feedback

### 3. Prediction Accuracy (±20%)
- **Impact:** Forecasts may be off by 20%
- **Mitigation:** Display confidence intervals
- **Why:** Student usage is unpredictable

### 4. IAM Setup Required
- **Impact:** 5-10 minute setup (create IAM role)
- **Mitigation:** Step-by-step guide with screenshots
- **Why:** Required for security

### 5. AWS Organizations Limitations
- **Impact:** University accounts may not have billing access
- **Mitigation:** Document workaround
- **Why:** AWS Organizations restriction

---

## 🎯 Success Metrics

### Primary
- **Cost Savings:** 30% reduction in AWS spend per user
- **Adoption:** 500 active users in 3 months
- **Engagement:** 50% weekly retention

### Secondary
- **Learning:** 70% report better understanding of AWS pricing
- **Bill Shock Prevention:** 80% avoid unexpected charges
- **Recommendation Acceptance:** 40% implement suggestions

---

## 🔒 Security

### IAM Permissions (Read-Only by Default)
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
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "rds:DescribeDBInstances",
        "s3:ListAllMyBuckets"
      ],
      "Resource": "*"
    }
  ]
}
```

### Write Permissions (Optional, User-Approved)
- `ec2:StopInstances`, `ec2:CreateImage`
- `ec2:DeleteVolume`, `ec2:CreateSnapshot`
- `s3:PutLifecycleConfiguration`

### Data Protection
- Credentials encrypted (AWS Secrets Manager)
- Data encrypted at rest (DynamoDB KMS, S3 SSE)
- Data encrypted in transit (TLS 1.3)
- No PII stored (only AWS account IDs)

---

## 🚀 Deployment

### Prerequisites
- AWS Account with admin access
- Node.js 18+
- AWS CDK installed
- GitHub account (for CI/CD)

### Infrastructure as Code (AWS CDK)
```bash
npm install
npx cdk bootstrap
npx cdk deploy --all
```

### CI/CD (GitHub Actions)
- Automatic deployment on push to `main`
- Preview deployments for pull requests
- Automated testing before deployment

---

## 📚 Documentation

- **[requirements.md](./requirements.md)** - Detailed functional requirements
- **[design.md](./design.md)** - System architecture and AWS services

---

## 🏆 Competitive Differentiation

| Feature | AWS Budgets | CloudHealth | CloudCost Copilot |
|---------|-------------|-------------|-------------------|
| **Cost** | Free | $500+/month | $10/month |
| **Data Delay** | 4-8 hours | 4-8 hours | 4-8 hours (honest) |
| **Recommendations** | ❌ None | ✅ Rule-based | ✅ AI-powered |
| **Predictions** | ❌ None | ✅ Basic | ✅ ML (Forecast) |
| **Educational** | ❌ No | ❌ No | ✅ Yes |
| **Student Focus** | ❌ No | ❌ No | ✅ Yes |
| **Setup Time** | 10 min | 2+ hours | 10 min |

---

## 🎓 Educational Value

CloudCost Copilot teaches students:
1. **AWS Pricing Models** - On-demand, Reserved, Spot, Savings Plans
2. **Cost Optimization** - Right-sizing, idle detection, storage tiers
3. **Cloud Architecture** - Serverless, event-driven, microservices
4. **FinOps Principles** - Budget management, cost allocation, forecasting
5. **AWS Services** - Hands-on experience with 10+ AWS services

---

## 📝 License

MIT License - Open source for educational use

---

## 👥 Team

**AI for Bharat Hackathon 2026**  
Student Track - AI for Learning & Developer Productivity

---

## 🙏 Acknowledgments

- AWS for providing Bedrock, Forecast, and other services
- AI for Bharat hackathon organizers
- Open source community

---

**Built with ❤️ for students learning AWS**
