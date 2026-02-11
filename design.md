# CloudCost Copilot - Design Document

## Executive Summary

CloudCost Copilot is an **AI-powered AWS cost optimization platform** that prevents bill shock through intelligent monitoring, ML-based forecasting, and **autonomous agentic AI**. Built for students, developers, and startups, it provides actionable cost insights integrated directly into developer workflows (Slack, GitHub, CI/CD).

**Key Innovations:**
- **Agentic AI:** Uses Amazon Bedrock Agents with multi-agent collaboration (not just text generation)
- **Developer-first:** Slack bots, GitHub Actions, API-first architecture
- **Latest AWS features:** Database Savings Plans (Dec 2025), AWS Compute Optimizer integration
- **Educational:** Gamification system with badges, leaderboards, learning paths
- **Safe:** User approval required for all actions, backups created automatically

**Target Market:**
- **Primary:** CS students (AWS Educate, $100 credits)
- **Secondary:** Indie developers, solo founders, small startups (2-10 people)
- **Expansion:** Bootcamps, universities (white-label licensing)

**Competitive Advantage:**
- AWS Budgets: Free but no recommendations → We add AI recommendations
- CloudHealth: $500+/month, enterprise-focused → We're $10/month, developer-focused
- Trusted Advisor: Requires Business Support ($100+/month) → We're standalone

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          User Layer                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Web App  │  │  Slack   │  │ Discord  │  │  GitHub  │           │
│  │ (React)  │  │   Bot    │  │   Bot    │  │ Actions  │           │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    API Gateway + Auth                                │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Amazon API Gateway (REST + WebSocket)                       │   │
│  │  - Amazon Cognito (Authentication)                           │   │
│  │  - Rate Limiting (1000 req/min per user)                     │   │
│  │  - Request Validation                                        │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   Agentic AI Layer (NEW)                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Amazon Bedrock Agents (Multi-Agent System)                  │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │   │
│  │  │   Cost      │  │  Optimizer  │  │  Educator   │         │   │
│  │  │  Analyzer   │  │    Agent    │  │    Agent    │         │   │
│  │  │   Agent     │  │             │  │             │         │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘         │   │
│  │         │                 │                 │                │   │
│  │         └─────────────────┴─────────────────┘                │   │
│  │                           │                                   │   │
│  │                  ┌────────▼────────┐                         │   │
│  │                  │  Executor Agent │                         │   │
│  │                  │  (Tool Use)     │                         │   │
│  │                  └─────────────────┘                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   Application Layer                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Cost Monitor │  │ Prediction   │  │ Integration  │             │
│  │ (Lambda)     │  │ (Forecast)   │  │ Hub (Lambda) │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Data Layer                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │  DynamoDB    │  │  S3          │  │ ElastiCache  │             │
│  │  (Metadata)  │  │  (Archives)  │  │ (Cache)      │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   AWS Services Layer                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Cost Explorer│  │ CloudWatch   │  │ Compute      │             │
│  │ API          │  │ Metrics      │  │ Optimizer    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Cost Anomaly │  │ AWS SDK      │  │ Trusted      │             │
│  │ Detection    │  │ (Actions)    │  │ Advisor      │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└─────────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   Integration Layer (NEW)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Slack API    │  │ Discord API  │  │ GitHub API   │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└─────────────────────────────────────────────────────────────────────┘
```

## Component Design

### 1. Frontend Application

**Technology Stack:**
- React 18 with TypeScript
- Vite (build tool)
- TailwindCSS (styling)
- Recharts (data visualization)
- AWS Amplify (hosting + auth)

**Key Components:**

#### Dashboard Component
```typescript
interface DashboardProps {
  currentSpend: number;
  budget: number;
  predictions: CostPrediction[];
  recommendations: Recommendation[];
}
```

**Features:**
- Real-time cost meter (current spend vs budget)
- 7-day cost trend chart
- Top 5 cost-generating services
- Active alerts panel
- Quick action buttons

#### Recommendations Panel
- AI-generated optimization suggestions
- Estimated savings per recommendation
- One-click approval/rejection
- Implementation status tracking

#### Cost Analytics
- Service-wise cost breakdown (pie chart)
- Daily/weekly/monthly cost trends (line chart)
- Free tier usage tracking (progress bars)
- Cost comparison with previous periods

### 2. Backend Services

#### Agentic AI System (Amazon Bedrock Agents) - ENHANCED

**Architecture:** Multi-agent system with 3 specialized agents

**Agent 1: Analyzer Agent**
- **Purpose:** Continuous cost monitoring and anomaly detection
- **Model:** Amazon Nova Lite
- **Tools:**
  - `get_cost_data()` - Fetch from Cost Explorer API
  - `detect_anomalies()` - Call AWS Cost Anomaly Detection API
  - `analyze_cloudwatch_metrics()` - Get EC2 CPU, RDS connections, S3 requests
  - `identify_idle_resources()` - Detect waste patterns
- **Output:** Structured waste report → triggers Optimizer Agent

**Agent 2: Optimizer Agent**
- **Purpose:** Generate actionable cost optimization recommendations
- **Model:** Amazon Nova Lite (primary), Claude 3.5 Haiku (fallback)
- **Tools:**
  - `get_compute_optimizer_recommendations()` - AWS Compute Optimizer API
  - `analyze_database_savings_plans()` - RDS/DynamoDB commitment opportunities
  - `check_trusted_advisor()` - Cross-reference with TA (if available)
  - `calculate_savings()` - Estimate monthly/annual savings
  - `generate_implementation_steps()` - Console + CLI + Terraform instructions
- **Output:** 3-5 recommendations per day with detailed implementation guides

**Agent 3: Executor Agent**
- **Purpose:** Safely execute approved optimizations via AWS APIs
- **Model:** Amazon Nova Lite
- **Tools (Function Calling):**
  - `stop_ec2_instance(instance_id)` → `ec2:StopInstances`
  - `create_ami_backup(instance_id, name)` → `ec2:CreateImage`
  - `delete_ebs_volume(volume_id)` → `ec2:DeleteVolume`
  - `create_ebs_snapshot(volume_id)` → `ec2:CreateSnapshot`
  - `modify_s3_lifecycle(bucket, policy)` → `s3:PutLifecycleConfiguration`
  - `purchase_savings_plan(commitment)` → `savingsplans:CreateSavingsPlan`
- **Safety:** User approval required, backups created, audit logs

**Implementation (Bedrock Agents with Tool Use):**

```python
import boto3
import json

bedrock_agent = boto3.client('bedrock-agent-runtime', region_name='us-east-1')

def invoke_analyzer_agent(account_id):
    """Invoke Analyzer Agent to detect waste"""
    
    response = bedrock_agent.invoke_agent(
        agentId='AGENT_ID_ANALYZER',
        agentAliasId='AGENT_ALIAS_ID',
        sessionId=f"analyze_{account_id}",
        inputText=json.dumps({
            "action": "analyze_costs",
            "account_id": account_id,
            "time_range": "last_7_days"
        })
    )
    
    # Parse agent response
    waste_report = parse_agent_response(response)
    
    # If waste detected, trigger Optimizer Agent
    if waste_report['total_waste'] > 10:  # $10/month threshold
        return invoke_optimizer_agent(account_id, waste_report)
    
    return {"status": "no_waste_detected"}

def invoke_optimizer_agent(account_id, waste_data):
    """Invoke Optimizer Agent to generate recommendations"""
    
    # Agent uses multiple tools to generate recommendations
    response = bedrock_agent.invoke_agent(
        agentId='AGENT_ID_OPTIMIZER',
        agentAliasId='AGENT_ALIAS_ID',
        sessionId=f"optimize_{account_id}",
        inputText=json.dumps({
            "action": "generate_recommendations",
            "account_id": account_id,
            "waste_data": waste_data,
            "integrations": {
                "compute_optimizer": True,
                "database_savings_plans": True,
                "trusted_advisor": False  # Requires Business Support
            }
        })
    )
    
    # Agent calls tools:
    # 1. get_compute_optimizer_recommendations() - Gets AWS's official recommendations
    # 2. analyze_database_savings_plans() - Checks RDS/DynamoDB usage patterns
    # 3. calculate_savings() - Estimates monthly savings
    # 4. generate_implementation_steps() - Creates step-by-step guide
    
    recommendations = parse_agent_response(response)
    
    # Cache recommendations for 24 hours
    cache_recommendations(account_id, recommendations, ttl=86400)
    
    return recommendations

def execute_optimization_with_agent(recommendation_id, user_approval):
    """Use Executor Agent to perform optimization"""
    
    if not user_approval:
        return {"error": "User approval required"}
    
    rec = get_recommendation(recommendation_id)
    
    # Invoke Executor Agent with tool use
    response = bedrock_agent.invoke_agent(
        agentId='AGENT_ID_EXECUTOR',
        agentAliasId='AGENT_ALIAS_ID',
        sessionId=f"exec_{recommendation_id}",
        inputText=json.dumps({
            "action": rec['action'],
            "resource_id": rec['resource_id'],
            "create_backup": True,
            "send_confirmation": True
        })
    )
    
    # Agent workflow:
    # 1. Validates IAM permissions
    # 2. Creates backup (AMI or snapshot)
    # 3. Executes action (stop EC2, delete EBS, etc.)
    # 4. Logs action in DynamoDB audit trail
    # 5. Sends email confirmation
    
    result = parse_agent_response(response)
    
    # Store in audit log
    log_action(recommendation_id, result)
    
    return result

# Tool definitions for Executor Agent
EXECUTOR_TOOLS = [
    {
        "name": "stop_ec2_instance",
        "description": "Stop an EC2 instance to save costs. Creates AMI backup first.",
        "parameters": {
            "type": "object",
            "properties": {
                "instance_id": {
                    "type": "string",
                    "description": "EC2 instance ID (e.g., i-1234567890abcdef0)"
                },
                "create_backup": {
                    "type": "boolean",
                    "description": "Whether to create AMI backup before stopping",
                    "default": True
                }
            },
            "required": ["instance_id"]
        }
    },
    {
        "name": "delete_ebs_volume",
        "description": "Delete unattached EBS volume. Creates snapshot first.",
        "parameters": {
            "type": "object",
            "properties": {
                "volume_id": {
                    "type": "string",
                    "description": "EBS volume ID (e.g., vol-1234567890abcdef0)"
                },
                "create_snapshot": {
                    "type": "boolean",
                    "description": "Whether to create snapshot before deletion",
                    "default": True
                }
            },
            "required": ["volume_id"]
        }
    },
    {
        "name": "modify_s3_lifecycle",
        "description": "Change S3 storage class to save costs",
        "parameters": {
            "type": "object",
            "properties": {
                "bucket_name": {"type": "string"},
                "storage_class": {
                    "type": "string",
                    "enum": ["INTELLIGENT_TIERING", "GLACIER", "DEEP_ARCHIVE"]
                }
            },
            "required": ["bucket_name", "storage_class"]
        }
    }
]

# Multi-agent collaboration example
def analyze_and_optimize_workflow(account_id):
    """Complete workflow: Analyze → Optimize → Execute"""
    
    # Step 1: Analyzer Agent detects waste
    print(f"[Analyzer Agent] Analyzing costs for account {account_id}...")
    waste_report = invoke_analyzer_agent(account_id)
    
    if waste_report['status'] == 'no_waste_detected':
        return {"message": "No optimization opportunities found"}
    
    # Step 2: Optimizer Agent generates recommendations
    print(f"[Optimizer Agent] Generating recommendations...")
    recommendations = waste_report['recommendations']
    
    # Step 3: User reviews and approves
    print(f"[User] Reviewing {len(recommendations)} recommendations...")
    approved_recs = get_user_approvals(recommendations)
    
    # Step 4: Executor Agent executes approved actions
    results = []
    for rec_id in approved_recs:
        print(f"[Executor Agent] Executing recommendation {rec_id}...")
        result = execute_optimization_with_agent(rec_id, user_approval=True)
        results.append(result)
    
    return {
        "total_savings": sum(r['savings'] for r in results),
        "actions_executed": len(results),
        "details": results
    }
```

**Why 3 Agents (Not 4):**
- ✅ **Analyzer:** Specialized in data analysis and anomaly detection
- ✅ **Optimizer:** Specialized in recommendation generation (uses Compute Optimizer, Database Savings Plans)
- ✅ **Executor:** Specialized in safe AWS API execution
- ❌ **Educator:** Not needed as separate agent - just use contextual text explanations

**Integration with Latest AWS Features:**
- **AWS Compute Optimizer API:** Official rightsizing recommendations
- **Database Savings Plans (Dec 2025):** RDS/DynamoDB commitment analysis
- **AWS Cost Anomaly Detection:** ML-powered anomaly detection
- **Model Context Protocol (MCP):** Future-proof agent-tool communication

#### Cost Monitoring Service (AWS Lambda)

**Function:** `cost-monitor-handler`  
**Runtime:** Python 3.12  
**Trigger:** EventBridge (every 1 hour)  
**Memory:** 512 MB  
**Timeout:** 90 seconds

**Responsibilities:**
1. Fetch cost data from AWS Cost Explorer API
2. Calculate current spend and trends
3. Store data in DynamoDB
4. Trigger alerts if thresholds exceeded

**Implementation with Error Handling:**
```python
import boto3
import time
from botocore.exceptions import ClientError

ce_client = boto3.client('ce', region_name='us-east-1')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('cloudcost-costs')

def lambda_handler(event, context):
    accounts = get_monitored_accounts()
    
    for account in accounts:
        retry_count = 0
        max_retries = 3
        
        while retry_count < max_retries:
            try:
                # Fetch cost data with specific date range
                cost_data = ce_client.get_cost_and_usage(
                    TimePeriod={
                        'Start': get_date_7_days_ago(),
                        'End': get_today()
                    },
                    Granularity='DAILY',
                    Metrics=['UnblendedCost'],
                    GroupBy=[{'Type': 'SERVICE', 'Key': 'SERVICE'}],
                    Filter={
                        'Dimensions': {
                            'Key': 'LINKED_ACCOUNT',
                            'Values': [account['account_id']]
                        }
                    }
                )
                
                # Process and store
                current_spend = calculate_current_spend(cost_data)
                budget = account.get('budget', 100.0)
                
                # Check thresholds
                if current_spend > budget * 0.9:
                    send_alert(account, "critical", current_spend, budget)
                elif current_spend > budget * 0.7:
                    send_alert(account, "warning", current_spend, budget)
                
                # Store in DynamoDB
                store_cost_data(account['account_id'], cost_data)
                
                break  # Success, exit retry loop
                
            except ClientError as e:
                error_code = e.response['Error']['Code']
                
                if error_code == 'ThrottlingException':
                    # Exponential backoff
                    wait_time = 2 ** retry_count
                    print(f"Rate limited. Waiting {wait_time}s before retry {retry_count+1}/{max_retries}")
                    time.sleep(wait_time)
                    retry_count += 1
                    
                elif error_code == 'DataUnavailableException':
                    # Cost data not yet available (within 4-hour delay window)
                    print(f"Cost data not available for account {account['account_id']}")
                    break
                    
                else:
                    # Unexpected error
                    print(f"Error fetching costs: {e}")
                    send_alert_to_ops_team(account, e)
                    break
                    
            except Exception as e:
                print(f"Unexpected error: {e}")
                send_alert_to_ops_team(account, e)
                break
        
        if retry_count >= max_retries:
            print(f"Failed after {max_retries} retries for account {account['account_id']}")
            send_alert_to_ops_team(account, "Max retries exceeded")
    
    return {"statusCode": 200, "body": "Cost monitoring completed"}

def calculate_current_spend(cost_data):
    """Calculate total spend from Cost Explorer response"""
    total = 0.0
    for result in cost_data['ResultsByTime']:
        total += float(result['Total']['UnblendedCost']['Amount'])
    return round(total, 2)

def send_alert(account, severity, current_spend, budget):
    """Send SNS notification"""
    sns = boto3.client('sns')
    topic_arn = f"arn:aws:sns:us-east-1:123456789012:cloudcost-alerts-{severity}"
    
    message = f"""
    CloudCost Copilot Alert
    
    Account: {account['account_id']}
    Current Spend: ${current_spend:.2f}
    Budget: ${budget:.2f}
    Usage: {(current_spend/budget)*100:.1f}%
    
    Recommendation: Review your AWS Console for idle resources.
    """
    
    sns.publish(
        TopicArn=topic_arn,
        Subject=f"AWS Cost Alert: {severity.upper()}",
        Message=message
    )
```

#### AI Recommendation Engine (AWS Lambda)

**Function:** `ai-recommendation-handler`  
**Runtime:** Python 3.12  
**Trigger:** EventBridge (every 6 hours)  
**Memory:** 1024 MB  
**Timeout:** 180 seconds

**Responsibilities:**
1. Analyze cost patterns using Amazon Bedrock (Nova Lite)
2. Detect waste using CloudWatch metrics
3. Generate optimization recommendations
4. Calculate potential savings

**Amazon Bedrock Integration (Nova Lite):**
```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

def generate_recommendations(account_id, cost_data, resource_metrics):
    """Generate cost optimization recommendations using Amazon Nova Lite"""
    
    # Check cache first (24-hour TTL)
    cached = get_cached_recommendations(account_id)
    if cached:
        return cached
    
    # Prepare context for AI
    context = f"""
    AWS Account Cost Analysis:
    
    Current Month Spend: ${cost_data['total']:.2f}
    Budget: ${cost_data['budget']:.2f}
    
    Service Breakdown:
    - EC2: ${cost_data['ec2']:.2f} ({len(resource_metrics['idle_ec2'])} idle instances)
    - S3: ${cost_data['s3']:.2f}
    - Lambda: ${cost_data['lambda']:.2f}
    - RDS: ${cost_data['rds']:.2f}
    
    Idle Resources Detected:
    {format_idle_resources(resource_metrics)}
    """
    
    prompt = f"""You are an AWS cost optimization expert helping a student learn cloud computing.

{context}

Provide exactly 3 cost optimization recommendations in this JSON format:
[
  {{
    "title": "Brief action title",
    "description": "Explain what to do and why",
    "savings_monthly": 12.50,
    "difficulty": "easy|medium|hard",
    "steps": ["Step 1", "Step 2", "Step 3"],
    "tradeoffs": "What the student should know",
    "rollback": "How to undo this change"
  }}
]

Focus on:
1. Immediate savings (stop idle resources)
2. Educational value (teach AWS concepts)
3. Safety (no data loss)"""

    try:
        response = bedrock.invoke_model(
            modelId='amazon.nova-lite-v1:0',  # Cost-effective model
            contentType='application/json',
            accept='application/json',
            body=json.dumps({
                "messages": [{
                    "role": "user",
                    "content": prompt
                }],
                "max_tokens": 2000,
                "temperature": 0.3,  # Lower = more factual
                "top_p": 0.9
            })
        )
        
        response_body = json.loads(response['body'].read())
        recommendations = json.loads(response_body['content'][0]['text'])
        
        # Cache for 24 hours
        cache_recommendations(account_id, recommendations, ttl=86400)
        
        # Track Bedrock usage for cost monitoring
        log_bedrock_usage(account_id, response_body['usage'])
        
        return recommendations
        
    except ClientError as e:
        error_code = e.response['Error']['Code']
        
        if error_code == 'ThrottlingException':
            print("Bedrock rate limit hit. Using fallback recommendations.")
            return generate_fallback_recommendations(resource_metrics)
            
        elif error_code == 'ModelNotReadyException':
            print("Nova Lite not available. Falling back to Haiku.")
            return generate_recommendations_haiku(account_id, cost_data, resource_metrics)
            
        else:
            print(f"Bedrock error: {e}")
            return generate_fallback_recommendations(resource_metrics)

def generate_fallback_recommendations(resource_metrics):
    """Rule-based recommendations when Bedrock unavailable"""
    recommendations = []
    
    # Rule 1: Idle EC2 instances
    for instance in resource_metrics['idle_ec2']:
        recommendations.append({
            "title": f"Stop idle EC2 instance {instance['id']}",
            "description": f"This {instance['type']} instance has been idle (CPU < 5%) for {instance['idle_hours']} hours.",
            "savings_monthly": instance['cost_per_month'],
            "difficulty": "easy",
            "steps": [
                "Go to EC2 Console",
                f"Select instance {instance['id']}",
                "Actions → Instance State → Stop"
            ],
            "tradeoffs": "Instance will be unavailable. EBS data persists. You can restart anytime.",
            "rollback": "Start the instance from EC2 Console"
        })
    
    return recommendations[:3]  # Max 3 recommendations

def format_idle_resources(metrics):
    """Format resource metrics for AI context"""
    lines = []
    
    if metrics['idle_ec2']:
        lines.append(f"- {len(metrics['idle_ec2'])} idle EC2 instances")
    if metrics['unattached_ebs']:
        lines.append(f"- {len(metrics['unattached_ebs'])} unattached EBS volumes")
    if metrics['low_usage_rds']:
        lines.append(f"- {len(metrics['low_usage_rds'])} underutilized RDS instances")
    
    return "\n".join(lines) if lines else "- No idle resources detected"
```

#### Waste Detection Service (AWS Lambda)

**Function:** `waste-detector-handler`  
**Runtime:** Python 3.12  
**Trigger:** EventBridge (every 1 hour)  
**Memory:** 512 MB  
**Timeout:** 120 seconds

**Detection Rules with Safety Checks:**
1. **Idle EC2:** CPU < 5% for 4+ hours (excludes Spot, ASG, Production tags)
2. **Unattached EBS:** State="available", Age > 7 days
3. **Underutilized RDS:** Connections < 5 for 24h, CPU < 10%
4. **Infrequent S3:** < 100 GET requests/month

**Implementation:**
```python
import boto3
from datetime import datetime, timedelta

def detect_idle_ec2(account_id, region='us-east-1'):
    """Detect idle EC2 instances with safety checks"""
    cloudwatch = boto3.client('cloudwatch', region_name=region)
    ec2 = boto3.client('ec2', region_name=region)
    
    # Get all running instances
    response = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )
    
    idle_instances = []
    
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            instance_type = instance['InstanceType']
            
            # Safety check 1: Skip production instances
            tags = {tag['Key']: tag['Value'] for tag in instance.get('Tags', [])}
            if tags.get('Environment') == 'production' or tags.get('AutoStop') == 'false':
                continue
            
            # Safety check 2: Skip Spot instances
            if instance.get('InstanceLifecycle') == 'spot':
                continue
            
            # Safety check 3: Skip instances in Auto Scaling Groups
            if any(tag['Key'].startswith('aws:autoscaling:') for tag in instance.get('Tags', [])):
                continue
            
            # Get CPU metrics for last 4 hours
            end_time = datetime.utcnow()
            start_time = end_time - timedelta(hours=4)
            
            try:
                metrics = cloudwatch.get_metric_statistics(
                    Namespace='AWS/EC2',
                    MetricName='CPUUtilization',
                    Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
                    StartTime=start_time,
                    EndTime=end_time,
                    Period=300,  # 5-minute intervals
                    Statistics=['Average']
                )
                
                if not metrics['Datapoints']:
                    # No metrics = likely just started, skip
                    continue
                
                avg_cpu = sum(dp['Average'] for dp in metrics['Datapoints']) / len(metrics['Datapoints'])
                
                if avg_cpu < 5.0:
                    # Get pricing
                    cost_per_hour = get_ec2_pricing(instance_type, region)
                    
                    idle_instances.append({
                        'instance_id': instance_id,
                        'instance_type': instance_type,
                        'avg_cpu': round(avg_cpu, 2),
                        'idle_hours': 4,
                        'cost_per_hour': cost_per_hour,
                        'cost_per_month': round(cost_per_hour * 720, 2),  # 720 hours/month
                        'tags': tags
                    })
                    
            except ClientError as e:
                print(f"Error getting metrics for {instance_id}: {e}")
                continue
    
    return idle_instances

def detect_unattached_ebs(account_id, region='us-east-1'):
    """Detect unattached EBS volumes (with grace period)"""
    ec2 = boto3.client('ec2', region_name=region)
    
    response = ec2.describe_volumes(
        Filters=[{'Name': 'status', 'Values': ['available']}]
    )
    
    unattached_volumes = []
    grace_period_days = 7
    
    for volume in response['Volumes']:
        volume_id = volume['VolumeId']
        create_time = volume['CreateTime']
        age_days = (datetime.now(create_time.tzinfo) - create_time).days
        
        # Grace period: Skip volumes created < 7 days ago
        if age_days < grace_period_days:
            continue
        
        # Safety check: Skip volumes with DoNotDelete tag
        tags = {tag['Key']: tag['Value'] for tag in volume.get('Tags', [])}
        if tags.get('DoNotDelete') == 'true':
            continue
        
        # Calculate cost
        size_gb = volume['Size']
        volume_type = volume['VolumeType']
        cost_per_gb_month = get_ebs_pricing(volume_type, region)
        monthly_cost = size_gb * cost_per_gb_month
        
        unattached_volumes.append({
            'volume_id': volume_id,
            'size_gb': size_gb,
            'volume_type': volume_type,
            'age_days': age_days,
            'monthly_cost': round(monthly_cost, 2)
        })
    
    return unattached_volumes

def get_ec2_pricing(instance_type, region):
    """Get EC2 on-demand pricing (simplified)"""
    # In production, use AWS Price List API
    # For MVP, use hardcoded common instance prices (us-east-1)
    pricing = {
        't2.micro': 0.0116,
        't3.micro': 0.0104,
        't2.small': 0.023,
        't3.small': 0.0208,
        't2.medium': 0.0464,
        't3.medium': 0.0416,
        'm5.large': 0.096,
        'm5.xlarge': 0.192
    }
    return pricing.get(instance_type, 0.10)  # Default fallback

def get_ebs_pricing(volume_type, region):
    """Get EBS pricing per GB-month"""
    pricing = {
        'gp2': 0.10,
        'gp3': 0.08,
        'io1': 0.125,
        'io2': 0.125,
        'st1': 0.045,
        'sc1': 0.015
    }
    return pricing.get(volume_type, 0.10)
```

#### Integration Hub (Lambda Functions) - NEW

**Purpose:** Connect CloudCost Copilot with developer tools

**Slack Bot Integration:**

```python
import boto3
from slack_sdk import WebClient
from slack_sdk.signature import SignatureVerifier

slack_client = WebClient(token=os.environ['SLACK_BOT_TOKEN'])
verifier = SignatureVerifier(os.environ['SLACK_SIGNING_SECRET'])

def lambda_handler(event, context):
    """Handle Slack slash commands and interactive buttons"""
    
    # Verify Slack request
    if not verifier.is_valid_request(event['body'], event['headers']):
        return {"statusCode": 401}
    
    body = json.loads(event['body'])
    command = body.get('command')
    user_id = body['user_id']
    
    if command == '/cloudcost':
        action = body.get('text', 'status')
        
        if action == 'status':
            # Get current spend
            spend_data = get_user_spend(user_id)
            
            slack_client.chat_postMessage(
                channel=body['channel_id'],
                blocks=[
                    {
                        "type": "section",
                        "text": {
                            "type": "mrkdwn",
                            "text": f"*Current AWS Spend*\n${spend_data['current']:.2f} / ${spend_data['budget']:.2f}"
                        }
                    },
                    {
                        "type": "section",
                        "text": {
                            "type": "mrkdwn",
                            "text": f"*Top Services:*\n• EC2: ${spend_data['ec2']:.2f}\n• S3: ${spend_data['s3']:.2f}"
                        }
                    }
                ]
            )
        
        elif action == 'recommendations':
            # Get AI recommendations
            recs = get_recommendations(user_id)
            
            blocks = [{"type": "header", "text": {"type": "plain_text", "text": "💡 Cost Optimization Recommendations"}}]
            
            for rec in recs[:3]:
                blocks.append({
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*{rec['title']}*\nSaves: ${rec['savings']:.2f}/month"
                    },
                    "accessory": {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "Approve"},
                        "value": rec['id'],
                        "action_id": "approve_recommendation"
                    }
                })
            
            slack_client.chat_postMessage(channel=body['channel_id'], blocks=blocks)
    
    return {"statusCode": 200}

def handle_interactive_action(event):
    """Handle button clicks (approve recommendation)"""
    payload = json.loads(event['body'])['payload']
    action = payload['actions'][0]
    
    if action['action_id'] == 'approve_recommendation':
        rec_id = action['value']
        
        # Execute optimization via Bedrock Executor Agent
        result = execute_optimization_with_agent(rec_id, user_approval=True)
        
        # Send confirmation
        slack_client.chat_postMessage(
            channel=payload['channel']['id'],
            text=f"✅ Optimization approved! {result['message']}"
        )
```

**GitHub Actions Integration:**

```yaml
# .github/workflows/cloudcost-check.yml
name: CloudCost Check

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  cost-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Analyze Infrastructure Costs
        uses: cloudcost-copilot/github-action@v1
        with:
          api_key: ${{ secrets.CLOUDCOST_API_KEY }}
          terraform_dir: ./infrastructure
          budget_threshold: 100
      
      - name: Comment on PR
        uses: actions/github-script@v6
        with:
          script: |
            const cost = process.env.ESTIMATED_COST;
            const body = `## 💰 CloudCost Analysis
            
            **Estimated Monthly Cost:** $${cost}
            
            **Changes:**
            - New Lambda function: +$5.00/month
            - New DynamoDB table: +$2.50/month
            
            ${cost > 100 ? '⚠️ **Warning:** Exceeds budget!' : '✅ Within budget'}
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
```

**Discord Bot Integration:**

```python
import discord
from discord.ext import commands

bot = commands.Bot(command_prefix='!')

@bot.command(name='cloudcost')
async def cloudcost_status(ctx):
    """Show current AWS spend"""
    user_id = str(ctx.author.id)
    spend_data = get_user_spend(user_id)
    
    embed = discord.Embed(
        title="💰 Your AWS Spend",
        color=discord.Color.blue()
    )
    embed.add_field(name="Current", value=f"${spend_data['current']:.2f}", inline=True)
    embed.add_field(name="Budget", value=f"${spend_data['budget']:.2f}", inline=True)
    embed.add_field(name="Remaining", value=f"${spend_data['remaining']:.2f}", inline=True)
    
    await ctx.send(embed=embed)

@bot.command(name='recommendations')
async def show_recommendations(ctx):
    """Show cost optimization recommendations"""
    user_id = str(ctx.author.id)
    recs = get_recommendations(user_id)
    
    for rec in recs[:3]:
        embed = discord.Embed(
            title=rec['title'],
            description=rec['description'],
            color=discord.Color.green()
        )
        embed.add_field(name="Savings", value=f"${rec['savings']:.2f}/month")
        embed.add_field(name="Difficulty", value=rec['difficulty'])
        
        await ctx.send(embed=embed)
```

**VS Code Extension Integration:**

```typescript
// VS Code Extension: cloudcost-copilot
import * as vscode from 'vscode';
import { CloudCostClient } from './client';

export function activate(context: vscode.ExtensionContext) {
    const client = new CloudCostClient(getApiKey());
    
    // Hover provider for cost estimates
    const hoverProvider = vscode.languages.registerHoverProvider(
        ['terraform', 'typescript', 'python', 'yaml'],
        {
            provideHover(document, position) {
                const line = document.lineAt(position).text;
                
                // Detect AWS resources
                if (line.includes('aws_instance') || line.includes('EC2.Instance')) {
                    const instanceType = extractInstanceType(line);
                    const cost = calculateEC2Cost(instanceType);
                    
                    return new vscode.Hover([
                        `💰 **Cost Estimate**`,
                        `Instance: ${instanceType}`,
                        `Hourly: $${cost.hourly}`,
                        `Monthly: $${cost.monthly}`,
                        ``,
                        `💡 Cheaper alternative: ${cost.alternative}`
                    ]);
                }
                
                if (line.includes('aws_db_instance') || line.includes('RDS.DBInstance')) {
                    const dbType = extractDBType(line);
                    const cost = calculateRDSCost(dbType);
                    
                    return new vscode.Hover([
                        `💰 **RDS Cost Estimate**`,
                        `Instance: ${dbType}`,
                        `Monthly: $${cost.monthly}`,
                        ``,
                        `💡 Consider Database Savings Plan: Save 35%`
                    ]);
                }
            }
        }
    );
    
    // Diagnostic provider for budget warnings
    const diagnosticCollection = vscode.languages.createDiagnosticCollection('cloudcost');
    
    vscode.workspace.onDidSaveTextDocument(async (document) => {
        if (isTerraformOrCDK(document)) {
            const resources = parseResources(document);
            const totalCost = await client.estimateCost(resources);
            const budget = await client.getUserBudget();
            
            if (totalCost > budget) {
                const diagnostic = new vscode.Diagnostic(
                    new vscode.Range(0, 0, 0, 0),
                    `⚠️ Estimated cost ($${totalCost}/month) exceeds budget ($${budget}/month)`,
                    vscode.DiagnosticSeverity.Warning
                );
                diagnosticCollection.set(document.uri, [diagnostic]);
            }
        }
    });
    
    // Commands
    context.subscriptions.push(
        vscode.commands.registerCommand('cloudcost.estimateFile', async () => {
            const editor = vscode.window.activeTextEditor;
            if (!editor) return;
            
            const resources = parseResources(editor.document);
            const estimate = await client.estimateCost(resources);
            
            vscode.window.showInformationMessage(
                `💰 Estimated monthly cost: $${estimate.total}\n` +
                `EC2: $${estimate.ec2}, RDS: $${estimate.rds}, S3: $${estimate.s3}`
            );
        }),
        
        vscode.commands.registerCommand('cloudcost.checkBudget', async () => {
            const spend = await client.getCurrentSpend();
            const budget = await client.getUserBudget();
            const percentage = (spend / budget * 100).toFixed(1);
            
            vscode.window.showInformationMessage(
                `💰 Current spend: $${spend} / $${budget} (${percentage}%)`
            );
        }),
        
        vscode.commands.registerCommand('cloudcost.optimize', async () => {
            const recommendations = await client.getRecommendations();
            
            const panel = vscode.window.createWebviewPanel(
                'cloudcostRecommendations',
                'Cost Optimization Recommendations',
                vscode.ViewColumn.Two,
                {}
            );
            
            panel.webview.html = generateRecommendationsHTML(recommendations);
        })
    );
    
    context.subscriptions.push(hoverProvider, diagnosticCollection);
}

function extractInstanceType(line: string): string {
    // Terraform: instance_type = "t3.micro"
    const terraformMatch = line.match(/instance_type\s*=\s*"([^"]+)"/);
    if (terraformMatch) return terraformMatch[1];
    
    // CDK: instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO)
    const cdkMatch = line.match(/InstanceClass\.(\w+).*InstanceSize\.(\w+)/);
    if (cdkMatch) return `${cdkMatch[1].toLowerCase()}${cdkMatch[2].toLowerCase()}`;
    
    return 't3.micro';
}

function calculateEC2Cost(instanceType: string): {hourly: number, monthly: number, alternative: string} {
    const pricing = {
        't3.micro': {hourly: 0.0104, monthly: 7.49, alternative: 'Lambda (if workload is intermittent)'},
        't3.small': {hourly: 0.0208, monthly: 14.98, alternative: 't3.micro (if CPU < 20%)'},
        't3.medium': {hourly: 0.0416, monthly: 29.95, alternative: 't3.small (if CPU < 40%)'},
        'm5.large': {hourly: 0.096, monthly: 69.12, alternative: 't3.medium (if not compute-intensive)'}
    };
    
    return pricing[instanceType] || pricing['t3.micro'];
}
```

**Why VS Code Extension is Valuable:**
- ✅ **Shift-left cost optimization:** Catch expensive resources before deployment
- ✅ **Developer workflow:** Developers spend 6+ hours/day in VS Code
- ✅ **Real-time feedback:** Instant cost estimates while coding
- ✅ **Prevents mistakes:** Red underline for budget-exceeding resources
- ✅ **Educational:** Hover tooltips teach AWS pricing

**Supported File Types:**
- Terraform (.tf)
- AWS CDK (TypeScript, Python)
- CloudFormation (.yaml, .json)
- Pulumi (TypeScript, Python)

```python
# cloudcost_client.py
import requests

class CloudCostClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.cloudcost-copilot.com/v1"
    
    def get_current_spend(self):
        """Get current AWS spend"""
        response = requests.get(
            f"{self.base_url}/costs/current",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        return response.json()
    
    def get_recommendations(self):
        """Get cost optimization recommendations"""
        response = requests.get(
            f"{self.base_url}/recommendations",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        return response.json()
    
    def approve_recommendation(self, rec_id):
        """Execute a recommendation"""
        response = requests.post(
            f"{self.base_url}/recommendations/{rec_id}/approve",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        return response.json()

# Usage
client = CloudCostClient(api_key="your_api_key")
spend = client.get_current_spend()
print(f"Current spend: ${spend['total']}")

# In CI/CD pipeline
if spend['total'] > 80:
    raise Exception("AWS spend exceeds threshold!")
```

**Function:** `optimization-executor-handler`  
**Runtime:** Python 3.12  
**Trigger:** API Gateway (user-initiated)  
**Memory:** 256 MB  
**Timeout:** 60 seconds

**Supported Actions:**
1. Stop EC2 instances
2. Delete unattached EBS volumes
3. Change S3 storage class to Glacier
4. Modify RDS instance types
5. Delete old snapshots

**Safety Mechanisms:**
- Require explicit user approval
- Create backup/snapshot before destructive actions
- Audit log all changes
- Rollback capability

#### Prediction Service (AWS Lambda + Amazon Forecast)

**Function:** `cost-predictor-handler`  
**Runtime:** Python 3.12  
**Trigger:** EventBridge (daily at 2 AM UTC)  
**Memory:** 1024 MB  
**Timeout:** 300 seconds

**Algorithm:**
1. Fetch historical cost data (last 30 days minimum)
2. Create/update Amazon Forecast dataset
3. Train predictor (DeepAR+ algorithm)
4. Generate 7-day forecast
5. Store predictions in DynamoDB

**Implementation:**
```python
import boto3
import json
from datetime import datetime, timedelta

forecast = boto3.client('forecast', region_name='us-east-1')
dynamodb = boto3.resource('dynamodb')

def lambda_handler(event, context):
    """Generate cost predictions using Amazon Forecast"""
    
    accounts = get_accounts_needing_predictions()
    
    for account in accounts:
        try:
            # Check if we have enough historical data
            historical_data = get_historical_costs(account['account_id'], days=30)
            
            if len(historical_data) < 7:
                # Not enough data for prediction
                store_prediction_status(account['account_id'], {
                    'status': 'insufficient_data',
                    'message': 'Need at least 7 days of cost data',
                    'days_available': len(historical_data)
                })
                continue
            
            # Create or update Forecast dataset
            dataset_arn = create_or_update_dataset(account['account_id'], historical_data)
            
            # Train predictor (or use existing if recent)
            predictor_arn = get_or_create_predictor(account['account_id'], dataset_arn)
            
            # Generate forecast
            forecast_result = generate_forecast(predictor_arn, horizon=7)
            
            # Calculate confidence intervals
            predictions = parse_forecast_result(forecast_result)
            
            # Store in DynamoDB
            store_predictions(account['account_id'], predictions)
            
            # Check if predicted spend exceeds budget
            check_prediction_alerts(account, predictions)
            
        except Exception as e:
            print(f"Error predicting costs for {account['account_id']}: {e}")
            # Fallback to simple linear regression
            predictions = fallback_linear_prediction(historical_data)
            store_predictions(account['account_id'], predictions, method='linear_fallback')

def create_or_update_dataset(account_id, historical_data):
    """Create Amazon Forecast dataset"""
    dataset_name = f"cloudcost-{account_id}"
    
    # Check if dataset exists
    try:
        response = forecast.describe_dataset(DatasetArn=f"arn:aws:forecast:us-east-1:123456789012:dataset/{dataset_name}")
        dataset_arn = response['DatasetArn']
    except forecast.exceptions.ResourceNotFoundException:
        # Create new dataset
        response = forecast.create_dataset(
            DatasetName=dataset_name,
            Domain='CUSTOM',
            DatasetType='TARGET_TIME_SERIES',
            DataFrequency='D',  # Daily
            Schema={
                'Attributes': [
                    {'AttributeName': 'timestamp', 'AttributeType': 'timestamp'},
                    {'AttributeName': 'target_value', 'AttributeType': 'float'},
                    {'AttributeName': 'item_id', 'AttributeType': 'string'}
                ]
            }
        )
        dataset_arn = response['DatasetArn']
    
    # Import data
    import_data_to_forecast(dataset_arn, historical_data)
    
    return dataset_arn

def get_or_create_predictor(account_id, dataset_arn):
    """Get existing predictor or train new one"""
    predictor_name = f"cloudcost-predictor-{account_id}"
    
    try:
        response = forecast.describe_predictor(PredictorArn=f"arn:aws:forecast:us-east-1:123456789012:predictor/{predictor_name}")
        
        # Check if predictor is recent (< 7 days old)
        created_time = response['CreationTime']
        age_days = (datetime.now(created_time.tzinfo) - created_time).days
        
        if age_days < 7:
            return response['PredictorArn']
        else:
            # Retrain with new data
            return train_new_predictor(predictor_name, dataset_arn)
            
    except forecast.exceptions.ResourceNotFoundException:
        return train_new_predictor(predictor_name, dataset_arn)

def train_new_predictor(predictor_name, dataset_arn):
    """Train Amazon Forecast predictor"""
    response = forecast.create_predictor(
        PredictorName=predictor_name,
        AlgorithmArn='arn:aws:forecast:::algorithm/Deep_AR_Plus',  # DeepAR+ for time-series
        ForecastHorizon=7,  # 7-day forecast
        PerformAutoML=False,
        InputDataConfig={'DatasetGroupArn': dataset_arn},
        FeaturizationConfig={
            'ForecastFrequency': 'D'
        }
    )
    
    # Wait for training to complete (async)
    # In production, use Step Functions for orchestration
    return response['PredictorArn']

def fallback_linear_prediction(historical_data):
    """Simple linear regression fallback when Forecast unavailable"""
    import numpy as np
    
    # Extract daily costs
    days = list(range(len(historical_data)))
    costs = [d['cost'] for d in historical_data]
    
    # Fit linear model: y = mx + b
    m, b = np.polyfit(days, costs, 1)
    
    # Predict next 7 days
    predictions = []
    for i in range(1, 8):
        next_day = len(days) + i
        predicted_cost = m * next_day + b
        
        # Add ±20% confidence interval
        predictions.append({
            'date': (datetime.now() + timedelta(days=i)).strftime('%Y-%m-%d'),
            'predicted_cost': round(max(0, predicted_cost), 2),
            'confidence_low': round(max(0, predicted_cost * 0.8), 2),
            'confidence_high': round(predicted_cost * 1.2, 2),
            'confidence_level': 0.7,  # 70% confidence
            'method': 'linear_regression'
        })
    
    return predictions
```

### 3. Data Models

#### User Account (DynamoDB)
```json
{
  "PK": "USER#<user_id>",
  "SK": "PROFILE",
  "email": "student@example.com",
  "aws_accounts": [
    {
      "account_id": "123456789012",
      "role_arn": "arn:aws:iam::123456789012:role/CloudCostCopilot",
      "budget": 100.00,
      "currency": "USD"
    }
  ],
  "notification_preferences": {
    "email": true,
    "sms": false,
    "threshold_warning": 0.7,
    "threshold_critical": 0.9
  },
  "created_at": "2026-02-11T20:00:00Z"
}
```

#### Cost Data (DynamoDB)
```json
{
  "PK": "ACCOUNT#123456789012",
  "SK": "COST#2026-02-11T20:00:00Z",
  "total_cost": 45.67,
  "service_breakdown": {
    "EC2": 30.00,
    "S3": 5.00,
    "Lambda": 2.50,
    "RDS": 8.17
  },
  "free_tier_usage": {
    "EC2": {"used": 500, "limit": 750, "unit": "hours"},
    "S3": {"used": 3.2, "limit": 5, "unit": "GB"}
  },
  "timestamp": "2026-02-11T20:00:00Z"
}
```

#### Recommendation (DynamoDB)
```json
{
  "PK": "ACCOUNT#123456789012",
  "SK": "REC#<recommendation_id>",
  "type": "idle_ec2",
  "title": "Stop idle EC2 instance",
  "description": "Instance i-1234567890abcdef0 has been idle (CPU < 5%) for 4 hours",
  "estimated_savings": 36.00,
  "savings_period": "monthly",
  "priority": "high",
  "difficulty": "easy",
  "actions": [
    {
      "service": "ec2",
      "action": "stop_instances",
      "params": {"instance_ids": ["i-1234567890abcdef0"]}
    }
  ],
  "status": "pending",
  "created_at": "2026-02-11T20:00:00Z",
  "expires_at": "2026-02-18T20:00:00Z"
}
```

## AWS Services Architecture

### Core Services

#### 1. Amazon Bedrock
**Purpose:** AI-powered cost analysis and recommendations

**Models Used:**
- **Amazon Nova Lite (Primary):** Cost-effective model for recommendations
  - Input: $0.06 per 1M tokens
  - Output: $0.24 per 1M tokens
  - Use case: Generate optimization suggestions
- **Claude 3.5 Haiku (Fallback):** When Nova unavailable
  - Input: $0.80 per 1M tokens
  - Output: $4.00 per 1M tokens

**Use Cases:**
- Generate optimization recommendations
- Explain cost patterns in natural language
- Answer user questions about AWS pricing (educational mode)

**Cost Control:**
- Limit to 5 recommendations per account per day
- Cache recommendations for 24 hours
- Use prompt caching feature (reduces repeat costs by 90%)
- Estimated cost: $5-10/month for 1,000 users

**Configuration:**
```python
bedrock_config = {
    "model_id": "amazon.nova-lite-v1:0",
    "inference_config": {
        "max_tokens": 2000,
        "temperature": 0.3,  # Lower for factual responses
        "top_p": 0.9
    },
    "cache_ttl": 86400  # 24 hours
}
```

#### 2. AWS Lambda
**Purpose:** Serverless compute for all backend logic

**Functions:**
- `cost-monitor-handler` (5-min schedule)
- `ai-recommendation-handler` (30-min schedule)
- `waste-detector-handler` (15-min schedule)
- `cost-predictor-handler` (6-hour schedule)
- `optimization-executor-handler` (API-triggered)
- `alert-sender-handler` (event-triggered)

**Cost Optimization:**
- Use ARM64 (Graviton2) for 20% cost savings
- Right-size memory based on profiling
- Enable Lambda SnapStart for Java functions

#### 3. Amazon DynamoDB
**Purpose:** NoSQL database for user data, costs, recommendations

**Tables:**
- `cloudcost-users` (user profiles, AWS accounts)
- `cloudcost-costs` (historical cost data)
- `cloudcost-recommendations` (optimization suggestions)
- `cloudcost-audit-log` (action history)

**Access Patterns:**
1. Get user profile by user_id
2. Get cost data for account + date range
3. Get active recommendations for account
4. Query audit log by account + timestamp

**Indexes:**
- GSI1: `account_id` + `timestamp` (for time-series queries)
- GSI2: `status` + `created_at` (for pending recommendations)

#### 4. Amazon API Gateway
**Purpose:** REST API for frontend communication

**Endpoints:**
- `GET /api/costs` - Fetch cost data
- `GET /api/recommendations` - Get optimization suggestions
- `POST /api/recommendations/{id}/approve` - Execute optimization
- `GET /api/predictions` - Get cost forecasts
- `POST /api/accounts` - Add AWS account
- `GET /api/alerts` - Fetch alert history

**Security:**
- Amazon Cognito authorizer
- API key for rate limiting
- Request validation schemas
- CORS configuration

#### 5. Amazon Cognito
**Purpose:** User authentication and authorization

**User Pools:**
- Email/password authentication
- Social login (Google, GitHub)
- MFA optional
- Password policies (min 8 chars, complexity)

**Identity Pools:**
- Federated identities for AWS SDK access
- Temporary credentials for S3 uploads

#### 6. AWS Cost Explorer API
**Purpose:** Fetch actual AWS cost and usage data

**API Calls:**
- `GetCostAndUsage` - Daily cost breakdown by service
- `GetCostForecast` - AWS's native 7-day predictions (optional, for comparison)

**Rate Limits & Costs:**
- **Rate limit:** 4 requests per second
- **Cost:** $0.01 per request
- **Data delay:** 4-8 hours (AWS limitation)

**Cost Optimization:**
- Poll hourly (not every 5 minutes): 24 calls/day per user
- Cache responses for 1 hour (reduces duplicate calls)
- Batch multiple accounts in single call where possible
- **Monthly cost for 1,000 users:** $24 (24K requests × $0.01)

**Error Handling:**
- Implement exponential backoff for rate limits
- Handle `DataUnavailableException` (data not yet available)
- Fallback to cached data if API fails

#### 7. Amazon CloudWatch
**Purpose:** Resource metrics for waste detection

**Metrics Monitored:**
- EC2: CPUUtilization, NetworkIn/Out
- RDS: DatabaseConnections, CPUUtilization
- Lambda: Invocations, Duration, Errors
- S3: NumberOfObjects, BucketSizeBytes

#### 8. Amazon EventBridge
**Purpose:** Event-driven architecture for scheduled tasks

**Rules:**
- `cost-monitor-schedule` (rate: 1 hour) - Fetch cost data
- `ai-recommendation-schedule` (rate: 6 hours) - Generate recommendations
- `waste-detector-schedule` (rate: 1 hour) - Detect idle resources
- `cost-predictor-schedule` (cron: daily at 2 AM UTC) - Update forecasts

**Why Not More Frequent:**
- Cost Explorer has 4-hour delay (no benefit to polling faster)
- Reduces Lambda invocations and API costs
- Still provides timely alerts (within 1-2 hours of threshold breach)

#### 9. Amazon SNS
**Purpose:** Multi-channel notifications

**Topics:**
- `cloudcost-alerts-critical` (SMS + Email)
- `cloudcost-alerts-warning` (Email only)
- `cloudcost-daily-summary` (Email)

#### 10. AWS Amplify
**Purpose:** Frontend hosting and CI/CD

**Features:**
- Automatic deployments from Git
- Custom domain with SSL
- Environment variables management
- Preview deployments for PRs

## Security Architecture

### IAM Roles and Permissions

#### User's AWS Account Role
**Role Name:** `CloudCostCopilotReadOnly`

**Permissions (Read-Only):**
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
        "ec2:Describe*",
        "rds:Describe*",
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Lambda Execution Role
**Role Name:** `CloudCostCopilotLambdaRole`

**Permissions:**
- DynamoDB: Read/Write to application tables
- Bedrock: InvokeModel
- SNS: Publish to alert topics
- CloudWatch Logs: CreateLogGroup, PutLogEvents
- Secrets Manager: GetSecretValue (for API keys)

### Data Encryption

**At Rest:**
- DynamoDB: AWS-managed encryption (KMS)
- S3: SSE-S3 encryption
- Secrets Manager: KMS encryption

**In Transit:**
- HTTPS/TLS 1.2+ for all API calls
- VPC endpoints for AWS service communication

### Authentication Flow

```
1. User logs in → Cognito User Pool
2. Cognito returns JWT tokens (ID, Access, Refresh)
3. Frontend includes ID token in API requests
4. API Gateway validates token with Cognito
5. Lambda receives authenticated user context
```

## Deployment Architecture

### Infrastructure as Code (AWS CDK)

**Stack Structure:**
```
cloudcost-copilot/
├── lib/
│   ├── frontend-stack.ts      # Amplify hosting
│   ├── api-stack.ts            # API Gateway + Lambda
│   ├── data-stack.ts           # DynamoDB tables
│   ├── monitoring-stack.ts     # EventBridge + SNS
│   └── security-stack.ts       # Cognito + IAM
├── lambda/
│   ├── cost-monitor/
│   ├── ai-recommendation/
│   ├── waste-detector/
│   └── optimization-executor/
└── frontend/
    └── src/
```

### CI/CD Pipeline

**GitHub Actions Workflow:**
```yaml
name: Deploy CloudCost Copilot

on:
  push:
    branches: [main]

jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
      - name: Install dependencies
        run: npm install
      - name: CDK Deploy
        run: npx cdk deploy --all --require-approval never
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Amplify
        run: |
          cd frontend
          npm install
          npm run build
          aws amplify start-deployment
```

## Cost Estimation (REALISTIC)

### Monthly Operational Costs (for 1,000 users)

| Service | Usage | Unit Cost | Monthly Cost |
|---------|-------|-----------|--------------|
| **Lambda** | 2M invocations, 512MB avg, 30s avg duration | $0.20 per 1M requests + $0.0000166667 per GB-second | $2.40 + $16.67 = **$19.07** |
| **DynamoDB** | 5M reads (eventually consistent), 500K writes, 10GB storage | $0.25 per 1M reads, $1.25 per 1M writes, $0.25 per GB | $1.25 + $0.63 + $2.50 = **$4.38** |
| **API Gateway** | 2M requests | $3.50 per 1M requests | **$7.00** |
| **Amazon Bedrock (Nova Lite)** | 50K requests, 500K input tokens, 1M output tokens | $0.06 per 1M input, $0.24 per 1M output | $0.03 + $0.24 = **$0.27** |
| **Amazon Forecast** | 10 predictors, 1K forecasts | $0.60 per predictor-hour, $0.60 per 1K forecasts | $6.00 + $0.60 = **$6.60** |
| **Cost Explorer API** | 24K requests (1,000 users × 24 calls/day) | $0.01 per request | **$240.00** |
| **CloudWatch** | 20 custom metrics, 10GB logs | $0.30 per metric, $0.50 per GB | $6.00 + $5.00 = **$11.00** |
| **SNS** | 5K notifications | $0.50 per 1K | **$2.50** |
| **Amplify** | Hosting + 100GB bandwidth | $0.15 per GB | **$15.00** |
| **S3** | 50GB storage, 1M requests | $0.023 per GB, $0.0004 per 1K requests | $1.15 + $0.40 = **$1.55** |
| **EventBridge** | 10M events | $1.00 per 1M events | **$10.00** |
| **Secrets Manager** | 10 secrets | $0.40 per secret | **$4.00** |
| **Data Transfer** | 50GB out | $0.09 per GB | **$4.50** |
| **TOTAL** | | | **$325.87/month** |

**Per-User Cost:** $0.33/month  
**With 50% AWS Activate Credits:** $162.94/month

### Cost Optimization Strategies

1. **Reduce Cost Explorer API Calls:**
   - Poll hourly instead of every 5 minutes: **Saves $216/month**
   - Cache responses for 1 hour: **Saves additional $50/month**
   - **New Cost Explorer cost: $24/month** (1,000 users × 1 call/hour × 24 hours = 24K calls/month)

2. **Use Nova Lite Instead of Claude Sonnet:**
   - Claude Sonnet: $3 input + $15 output = $18 per 1M tokens
   - Nova Lite: $0.06 input + $0.24 output = $0.30 per 1M tokens
   - **Saves $17.70 per 1M tokens (60x cheaper)**

3. **Aggressive Caching:**
   - Cache Bedrock recommendations for 24 hours
   - Cache Cost Explorer responses for 1 hour
   - **Reduces API calls by 80%**

4. **Use DynamoDB On-Demand:**
   - No provisioned capacity costs
   - Pay only for actual reads/writes
   - **Saves $10-20/month vs. provisioned**

### Revised Monthly Costs (Optimized)

| Service | Optimized Cost |
|---------|----------------|
| Lambda | $19.07 |
| DynamoDB | $4.38 |
| API Gateway | $7.00 |
| Bedrock (Nova Lite) | $0.27 |
| Forecast | $6.60 |
| **Cost Explorer** | **$24.00** (hourly polling) |
| CloudWatch | $11.00 |
| SNS | $2.50 |
| Amplify | $15.00 |
| S3 | $1.55 |
| EventBridge | $10.00 |
| Secrets Manager | $4.00 |
| Data Transfer | $4.50 |
| **TOTAL** | **$109.87/month** |

**Per-User Cost:** $0.11/month  
**Revenue Target:** $10/user/month (freemium: free tier + paid features)  
**Gross Margin:** 99% (after reaching 1,000 users)

## Monitoring and Observability

### CloudWatch Dashboards

**Dashboard 1: System Health**
- Lambda error rates
- API Gateway latency (p50, p99)
- DynamoDB throttles
- Bedrock API errors

**Dashboard 2: Business Metrics**
- Active users (DAU, MAU)
- Cost savings delivered
- Recommendations accepted
- Alert response time

### Alarms

1. **Lambda Errors > 5%** → Page on-call engineer
2. **API Latency > 2s** → Investigate performance
3. **DynamoDB Throttles > 0** → Increase capacity
4. **Bedrock Rate Limit** → Implement backoff

## Scalability Considerations

### Horizontal Scaling
- Lambda: Auto-scales to 1,000 concurrent executions
- DynamoDB: On-demand pricing (auto-scales)
- API Gateway: Handles 10,000 requests/second

### Vertical Scaling
- Increase Lambda memory for faster execution
- Use DynamoDB provisioned capacity for predictable workloads

### Caching Strategy
- ElastiCache (Redis) for frequently accessed data
- CloudFront for static assets
- API Gateway caching for GET requests (TTL: 5 minutes)

## Disaster Recovery

### Backup Strategy
- DynamoDB: Point-in-time recovery enabled (35-day retention)
- S3: Versioning enabled for critical data
- Lambda: Code stored in Git + deployed via CI/CD
- Secrets: AWS Secrets Manager with automatic rotation

### Recovery Objectives
- **RTO (Recovery Time Objective):** 2 hours (realistic for student project)
- **RPO (Recovery Point Objective):** 1 hour (DynamoDB PITR granularity)

### Failover Plan
1. Detect outage via CloudWatch alarms
2. Restore DynamoDB tables from point-in-time backup
3. Redeploy Lambda functions from Git
4. Verify data integrity
5. Resume monitoring

**Note:** Multi-region deployment is out of scope for MVP due to cost and complexity.

---

## Known Limitations & Trade-offs

### 1. Cost Explorer Data Delay (4-8 hours)
**Impact:** Cannot provide true "real-time" alerts  
**Mitigation:** Set user expectations ("hourly monitoring with 4-hour data delay")  
**Why We Accept This:** AWS limitation, no workaround exists

### 2. MVP Scope (EC2, S3, Lambda, RDS Only)
**Impact:** Doesn't cover all 200+ AWS services  
**Mitigation:** Focus on services students use most (covers 80% of student costs)  
**Future:** Add more services based on user feedback

### 3. Prediction Accuracy (±20%)
**Impact:** Forecasts may be off by 20%  
**Mitigation:** Display confidence intervals, use conservative estimates  
**Why We Accept This:** Student usage is unpredictable (learning, experimenting)

### 4. Cost Explorer API Costs ($0.01/request)
**Impact:** Largest operational expense ($24/month for 1,000 users)  
**Mitigation:** Hourly polling + aggressive caching  
**Why We Accept This:** No alternative API for cost data

### 5. IAM Setup Friction
**Impact:** Students must create IAM roles (5-10 minute setup)  
**Mitigation:** Provide step-by-step guide with screenshots  
**Why We Accept This:** Required for security (no way to access AWS account without credentials)

### 6. AWS Organizations Limitations
**Impact:** Students using university AWS accounts may not have billing access  
**Mitigation:** Document limitation, provide workaround guide  
**Why We Accept This:** AWS Organizations restriction, cannot be bypassed

### 7. No Multi-Region Deployment
**Impact:** Single region outage = service unavailable  
**Mitigation:** Use us-east-1 (most reliable AWS region)  
**Why We Accept This:** Multi-region adds 3x cost and complexity for MVP

### 8. Free Tier Tracking Complexity
**Impact:** Only tracks EC2, S3, Lambda free tier (not all 60+ services)  
**Mitigation:** Focus on most commonly used services  
**Why We Accept This:** Each service has unique free tier rules, full coverage requires months of development

### 9. Bedrock Rate Limits
**Impact:** Max 1,000 recommendations/day across all users  
**Mitigation:** Cache recommendations for 24 hours, use fallback rules  
**Why We Accept This:** Bedrock Nova Lite has 100 req/min limit

### 10. No Offline Mode
**Impact:** Requires internet connection  
**Mitigation:** None (cloud service requires connectivity)  
**Why We Accept This:** Cost data is in AWS cloud, cannot be accessed offline

---

## Competitive Differentiation

### vs. AWS Budgets (Free)
- **AWS Budgets:** 4-hour delay, no recommendations, complex setup
- **CloudCost Copilot:** Same delay, but adds AI recommendations + educational insights

### vs. CloudHealth/Cloudability ($500+/month)
- **Enterprise tools:** Complex, expensive, overkill for students
- **CloudCost Copilot:** Student-focused, $10/month, simple onboarding

### vs. AWS Trusted Advisor (Requires Business Support $100+/month)
- **Trusted Advisor:** Basic checks, no cost predictions
- **CloudCost Copilot:** ML-based predictions + AI explanations

### Our Unique Value
1. **Educational focus:** Teaches cloud economics, not just alerts
2. **Student pricing:** $10/month vs. $100-500/month for alternatives
3. **AI-powered:** Uses Bedrock + Forecast (not just rule-based)
4. **Safety-first:** Requires approval for all actions (no accidental deletions)

---

**Document Version:** 1.0  
**Last Updated:** February 11, 2026  
**Architecture Review:** Pending
