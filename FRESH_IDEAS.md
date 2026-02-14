# 🎓 FRESH HACKATHON IDEAS (Student Track)
## Real Student Problems, Zero Generic AI Chatbots

**Date:** Feb 14, 2026  
**Context:** NO business ideas, NO generic AI assistants, ONLY unsolved student problems

---

## 🔥 IDEA #1: MessMate (Hostel Food Safety Tracker)

### **The REAL Problem:**

From research: *"Students constantly fall sick because of mess food. No number of strikes or protests changes this pathetic condition."*

**What actually happens:**
- 50 students get food poisoning from hostel mess
- Everyone blames the mess, but no proof
- Mess contractor denies everything
- Students have no evidence
- Problem repeats next month

**Current "solutions":**
- ❌ Complain to warden (nothing happens)
- ❌ WhatsApp group complaints (no data)
- ❌ Stop eating mess food (expensive, unsustainable)

**The gap:** No systematic way to track food quality and hold mess accountable.

---

### **The Solution: MessMate**

**Tagline:** *"Crowdsourced food safety tracker that holds your hostel mess accountable."*

**How it works:**
1. **Students rate every meal** (1-5 stars + photo)
2. **Report issues** (food poisoning, hair in food, undercooked)
3. **AI analyzes patterns** (Amazon Bedrock):
   - "30 students reported stomach issues after Thursday dinner"
   - "Chicken biryani has 2.1 avg rating (avoid)"
   - "Mess hygiene dropped 40% this week"
4. **Auto-generates report** for warden with evidence
5. **Tracks mess contractor performance** over time

**Example:**
```
Alert: Food Safety Issue Detected

Date: Feb 14, 2026
Meal: Dinner (Chicken Biryani)

Reports:
- 23 students reported stomach pain (within 4 hours)
- 15 students uploaded photos (undercooked chicken)
- Average rating: 1.2/5 (lowest this month)

AI Analysis (Bedrock):
"High probability of food poisoning outbreak. 
Chicken appears undercooked in 12/15 photos.
Recommend immediate inspection and contractor warning."

Evidence Package:
- 23 student testimonials
- 15 photos with timestamps
- Historical data (3 similar incidents in past 2 months)

Action: Auto-sent to Warden + Chief Warden + Dean
```

---

### **Why This Wins:**

✅ **REAL pain** (every hostel student faces this)  
✅ **No solution exists** (just WhatsApp complaints)  
✅ **Measurable impact** (reduce food poisoning by 60%)  
✅ **Uses AWS smartly:**
- Amazon Bedrock (image analysis + pattern detection)
- Amazon Rekognition (detect food quality from photos)
- DynamoDB (store ratings)
- SNS (alert warden)
- S3 (store evidence photos)

✅ **Clear demo:**
- Show 20 students rating bad meal
- Show AI detecting pattern
- Show auto-generated report
- Show warden taking action

✅ **India-specific** (hostel mess culture is unique to India)

---

### **Technical Architecture:**

```
┌─────────────────────────────────────────────────────┐
│              Students (Mobile App)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │ Rate meal│  │ Upload   │  │ Report   │          │
│  │ 1-5 stars│  │ photo    │  │ issue    │          │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘          │
└───────┼─────────────┼─────────────┼─────────────────┘
        │             │             │
        └─────────────┴─────────────┘
                      │
         ┌────────────▼────────────┐
         │   API Gateway           │
         │   + Lambda              │
         └────────────┬────────────┘
                      │
         ┌────────────▼────────────┐
         │   DynamoDB              │
         │   - Ratings             │
         │   - Reports             │
         │   - Photos (S3 URLs)    │
         └────────────┬────────────┘
                      │
         ┌────────────▼────────────┐
         │   EventBridge           │
         │   (Every 1 hour)        │
         └────────────┬────────────┘
                      │
         ┌────────────▼────────────┐
         │   Lambda (Analyzer)     │
         │   - Fetch last hour     │
         │   - Detect patterns     │
         └────────────┬────────────┘
                      │
         ┌────────────▼────────────┐
         │   Amazon Bedrock        │
         │   + Rekognition         │
         │                         │
         │   Analyze:              │
         │   - Rating patterns     │
         │   - Photo quality       │
         │   - Text sentiment      │
         └────────────┬────────────┘
                      │
         ┌────────────▼────────────┐
         │   If issue detected:    │
         │   - Generate report     │
         │   - Send to SNS         │
         │   - Email warden        │
         └─────────────────────────┘
```

---

### **MVP (24-Hour Scope):**

**What to build:**
1. Mobile web app (React) - rate meals, upload photos
2. Lambda + DynamoDB - store ratings
3. Bedrock - analyze patterns
4. Simple dashboard for warden

**Demo:**
1. Show 15 students rating "bad" meal
2. Upload photos of undercooked food
3. Show AI detecting issue
4. Show auto-generated report

**Tech stack:**
- React (mobile-first web app)
- Lambda + API Gateway
- DynamoDB
- Bedrock (Nova Lite)
- Rekognition (food quality)
- SNS (alerts)

**Time: 18 hours**

---

## 🔥 IDEA #2: RoomieMatch (AI Roommate Compatibility)

### **The REAL Problem:**

From research: *"Sometimes you might want to change the arrangement of the room but you can't because you need everyone's permission. Friends of your roommates keep barging in and out."*

**What actually happens:**
- College assigns random roommates
- You get paired with someone incompatible:
  - You sleep at 11 PM, they party till 3 AM
  - You're vegetarian, they cook non-veg in room
  - You study with silence, they blast music
- Entire year is miserable
- Can't change rooms (college policy)

**Current "solutions":**
- ❌ Random assignment (50% compatibility)
- ❌ "Adjust kar lo" (suffer in silence)
- ❌ Request room change (rarely approved)

**The gap:** No data-driven roommate matching.

---

### **The Solution: RoomieMatch**

**Tagline:** *"Tinder for roommates, but actually scientific."*

**How it works:**
1. **Students fill questionnaire:**
   - Sleep schedule (early bird / night owl)
   - Cleanliness (messy / organized)
   - Social (introvert / extrovert)
   - Study habits (silent / music)
   - Food preferences (veg / non-veg)
   - Guests (never / sometimes / often)
2. **AI matches compatible roommates** (Bedrock)
3. **Shows compatibility score** (85% match)
4. **Explains WHY** ("Both night owls, both organized")
5. **Students can request specific roommate**

**Example:**
```
Your Profile:
- Sleep: 11 PM - 7 AM (early bird)
- Cleanliness: 8/10 (organized)
- Social: Introvert
- Study: Needs silence
- Food: Vegetarian
- Guests: Rarely

Top Matches:

1. Rahul Kumar (92% compatible)
   - Sleep: 10:30 PM - 6:30 AM ✅
   - Cleanliness: 9/10 ✅
   - Social: Introvert ✅
   - Study: Needs silence ✅
   - Food: Vegetarian ✅
   - Guests: Never ✅
   
   Why compatible:
   "Both early birds, both value cleanliness,
   both introverts who need quiet study time.
   Zero conflicts predicted."

2. Amit Sharma (78% compatible)
   - Sleep: 12 AM - 8 AM ⚠️ (1 hour difference)
   - Cleanliness: 7/10 ✅
   - Social: Ambivert ⚠️
   - Study: Needs silence ✅
   - Food: Vegetarian ✅
   - Guests: Sometimes ⚠️
   
   Potential conflicts:
   "Amit sleeps 1 hour later (manageable).
   Occasionally has guests (minor issue)."
```

---

### **Why This Wins:**

✅ **REAL pain** (roommate conflicts ruin college life)  
✅ **No solution exists** (colleges do random assignment)  
✅ **Measurable impact** (reduce roommate conflicts by 70%)  
✅ **Uses AWS:**
- Amazon Bedrock (compatibility analysis)
- DynamoDB (student profiles)
- Amplify (web app)

✅ **Clear demo:**
- Show 2 incompatible roommates (conflicts)
- Show RoomieMatch suggesting better match
- Show compatibility score + explanation

✅ **Scalable** (every college in India needs this)

---

## 🔥 IDEA #3: LostAndFound.ai (Campus Item Recovery)

### **The REAL Problem:**

From research: *"Scores of your commodities like Cycle, Bottle, headphone, pendrive, laptop seems to be the property of the hostel mates."*

**What actually happens:**
- You lose your laptop charger in library
- Someone finds it, keeps it
- You post in WhatsApp group (500 messages/day, gets buried)
- Never find it
- Buy new one (₹2000 wasted)

**Current "solutions":**
- ❌ WhatsApp group (messages get buried)
- ❌ Lost & Found office (no one checks)
- ❌ Ask friends (limited reach)

**The gap:** No centralized, searchable system.

---

### **The Solution: LostAndFound.ai**

**Tagline:** *"AI-powered campus lost & found that actually works."*

**How it works:**
1. **Lost item:** Take photo, describe, location
2. **AI analyzes** (Rekognition + Bedrock):
   - "Black laptop charger, HP brand, found in Library"
3. **Matches with found items** automatically
4. **Notifies both parties** via SMS
5. **Arranges meetup** (safe location on campus)

**Example:**
```
You lost: Laptop charger
Description: "Black HP charger, 65W"
Location: Central Library, 2nd floor
Time: Feb 14, 2 PM

AI searches found items...

Match found! (95% confidence)

Found by: Priya Sharma
Description: "Black laptop charger, HP logo"
Location: Central Library, 2nd floor
Time: Feb 14, 2:30 PM
Photo: [shows your charger]

Action:
✅ SMS sent to both
✅ Meetup arranged: Library entrance, 5 PM today
✅ Verification code: 4729 (show to confirm)
```

---

### **Why This Wins:**

✅ **REAL pain** (students lose ₹5000+ worth of items/year)  
✅ **No solution exists** (WhatsApp groups don't work)  
✅ **Measurable impact** (recover 60% of lost items)  
✅ **Uses AWS:**
- Amazon Rekognition (image matching)
- Amazon Bedrock (description matching)
- SNS (SMS notifications)
- DynamoDB (lost/found database)

✅ **Clear demo:**
- Show student losing item
- Show another student finding it
- Show AI matching them
- Show successful recovery

---

## 🔥 IDEA #4: ClassSwap (Peer-to-Peer Class Exchange)

### **The REAL Problem:**

**What actually happens:**
- You have class at 8 AM Monday (you hate mornings)
- Your friend has same class at 2 PM Tuesday (they hate afternoons)
- You want to swap, but:
  - College doesn't allow section changes
  - Attendance is biometric (can't proxy)
  - No official way to swap

**Current "solutions":**
- ❌ Bunk class (attendance suffers)
- ❌ Suffer through 8 AM (miserable)
- ❌ Unofficial swaps (risky, no tracking)

**The gap:** No official peer-to-peer class exchange system.

---

### **The Solution: ClassSwap**

**Tagline:** *"Swap class timings with peers, officially."*

**How it works:**
1. **Post your class:** "Want to swap 8 AM Monday → 2 PM Tuesday"
2. **AI finds matches:** Students who want opposite swap
3. **Both approve swap**
4. **System notifies professor** (optional approval)
5. **Attendance updated** automatically

**Why professors allow this:**
- Same number of students in each section
- Both students committed to attending
- Reduces bunking
- Improves student satisfaction

---

## 🎯 WHICH IDEA TO CHOOSE?

### **Scoring Matrix:**

| Idea | Problem Clarity | Student Impact | 24h Feasibility | Judge Appeal | India-Specific | TOTAL |
|------|----------------|----------------|-----------------|--------------|----------------|-------|
| **MessMate** | 10/10 | 10/10 | 8/10 | 9/10 | 10/10 | **47/50** ⭐⭐⭐ |
| **RoomieMatch** | 9/10 | 9/10 | 9/10 | 8/10 | 8/10 | **43/50** ⭐⭐ |
| **LostAndFound.ai** | 8/10 | 7/10 | 9/10 | 7/10 | 6/10 | **37/50** ⭐ |
| **ClassSwap** | 7/10 | 6/10 | 7/10 | 6/10 | 5/10 | **31/50** |

---

## 🏆 RECOMMENDATION: **MessMate**

### **Why:**

1. **Every hostel student faces this** (universal pain)
2. **High impact** (food poisoning is serious)
3. **India-specific** (hostel mess culture unique to India)
4. **Uses AWS impressively:**
   - Bedrock (pattern detection)
   - Rekognition (food quality from photos)
   - Real-time alerts (SNS)
5. **Clear demo** (easy to show value)
6. **Measurable outcome** (reduce food poisoning incidents)
7. **Scalable** (every college hostel in India)

---

## 📋 24-HOUR PLAN FOR MESSMATE

### **Hour 0-3: Mobile Web App (React)**
```jsx
// MealRating.jsx
function MealRating() {
  const [rating, setRating] = useState(0);
  const [photo, setPhoto] = useState(null);
  const [issue, setIssue] = useState('');
  
  const submitRating = async () => {
    const formData = new FormData();
    formData.append('rating', rating);
    formData.append('photo', photo);
    formData.append('issue', issue);
    formData.append('meal', 'dinner');
    formData.append('timestamp', Date.now());
    
    await fetch('/api/ratings', {
      method: 'POST',
      body: formData
    });
    
    alert('Rating submitted! Thank you.');
  };
  
  return (
    <div className="meal-rating">
      <h2>Rate Today's Dinner</h2>
      
      <div className="stars">
        {[1,2,3,4,5].map(star => (
          <span 
            key={star}
            onClick={() => setRating(star)}
            className={star <= rating ? 'filled' : ''}
          >
            ⭐
          </span>
        ))}
      </div>
      
      <input 
        type="file" 
        accept="image/*"
        capture="environment"
        onChange={e => setPhoto(e.target.files[0])}
      />
      
      <textarea 
        placeholder="Any issues? (optional)"
        value={issue}
        onChange={e => setIssue(e.target.value)}
      />
      
      <button onClick={submitRating}>Submit</button>
    </div>
  );
}
```

### **Hour 3-6: Backend (Lambda + DynamoDB)**
```python
# lambda/submit_rating.py
import boto3
import json
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
s3 = boto3.client('s3')
table = dynamodb.Table('mess-ratings')

def lambda_handler(event, context):
    # Parse form data
    rating = int(event['rating'])
    photo = event['photo']  # base64
    issue = event.get('issue', '')
    meal = event['meal']
    timestamp = event['timestamp']
    
    # Upload photo to S3
    photo_key = f"photos/{timestamp}.jpg"
    s3.put_object(
        Bucket='messmate-photos',
        Key=photo_key,
        Body=photo
    )
    
    # Store rating in DynamoDB
    table.put_item(Item={
        'id': str(timestamp),
        'rating': rating,
        'photo_url': f"s3://messmate-photos/{photo_key}",
        'issue': issue,
        'meal': meal,
        'timestamp': timestamp
    })
    
    return {'statusCode': 200, 'body': 'Rating submitted'}
```

### **Hour 6-10: AI Analysis (Bedrock + Rekognition)**
```python
# lambda/analyze_patterns.py
import boto3
from datetime import datetime, timedelta

bedrock = boto3.client('bedrock-runtime')
rekognition = boto3.client('rekognition')
dynamodb = boto3.resource('dynamodb')

def analyze_meal():
    """Run every hour to detect issues"""
    
    # Get last hour's ratings
    table = dynamodb.Table('mess-ratings')
    one_hour_ago = datetime.now() - timedelta(hours=1)
    
    ratings = table.scan(
        FilterExpression='timestamp > :time',
        ExpressionAttributeValues={':time': one_hour_ago.timestamp()}
    )['Items']
    
    # Check for patterns
    low_ratings = [r for r in ratings if r['rating'] <= 2]
    
    if len(low_ratings) >= 10:  # 10+ bad ratings
        # Analyze photos with Rekognition
        food_quality_issues = []
        for rating in low_ratings:
            if rating.get('photo_url'):
                result = rekognition.detect_labels(
                    Image={'S3Object': {
                        'Bucket': 'messmate-photos',
                        'Name': rating['photo_url'].split('/')[-1]
                    }}
                )
                # Check for "undercooked", "burnt", "hair", etc.
                labels = [l['Name'] for l in result['Labels']]
                if any(bad in labels for bad in ['Burnt', 'Raw', 'Dirty']):
                    food_quality_issues.append(rating)
        
        # Use Bedrock to generate report
        prompt = f"""
        Analyze this hostel mess food safety issue:
        
        - {len(low_ratings)} students gave low ratings (1-2 stars)
        - {len(food_quality_issues)} photos show quality issues
        - Common complaints: {', '.join([r['issue'] for r in low_ratings if r['issue']])}
        
        Generate a formal report for the hostel warden including:
        1. Severity assessment
        2. Likely cause
        3. Recommended action
        """
        
        response = bedrock.invoke_model(
            modelId='amazon.nova-lite-v1:0',
            body=json.dumps({'prompt': prompt, 'max_tokens': 500})
        )
        
        report = json.loads(response['body'])['completion']
        
        # Send alert to warden
        sns = boto3.client('sns')
        sns.publish(
            TopicArn='arn:aws:sns:us-east-1:123456789:warden-alerts',
            Subject='URGENT: Mess Food Safety Issue Detected',
            Message=f"""
            {len(low_ratings)} students reported issues with today's dinner.
            
            AI Analysis:
            {report}
            
            Evidence:
            - {len(low_ratings)} low ratings
            - {len(food_quality_issues)} photos with quality issues
            
            View full report: https://messmate.app/reports/{datetime.now().date()}
            """
        )
        
        return {'alert_sent': True, 'report': report}
    
    return {'alert_sent': False}
```

### **Hour 10-14: Warden Dashboard (React)**
```jsx
// WardenDashboard.jsx
function WardenDashboard() {
  const [alerts, setAlerts] = useState([]);
  const [stats, setStats] = useState({});
  
  useEffect(() => {
    fetch('/api/warden/alerts').then(r => r.json()).then(setAlerts);
    fetch('/api/warden/stats').then(r => r.json()).then(setStats);
  }, []);
  
  return (
    <div className="dashboard">
      <h1>Mess Quality Dashboard</h1>
      
      <div className="stats">
        <div className="stat">
          <h3>Today's Average Rating</h3>
          <div className="value">{stats.avg_rating}/5</div>
        </div>
        <div className="stat">
          <h3>Total Ratings</h3>
          <div className="value">{stats.total_ratings}</div>
        </div>
        <div className="stat">
          <h3>Issues Reported</h3>
          <div className="value">{stats.issues}</div>
        </div>
      </div>
      
      <div className="alerts">
        <h2>Recent Alerts</h2>
        {alerts.map(alert => (
          <div key={alert.id} className="alert">
            <h3>{alert.meal} - {alert.date}</h3>
            <p>{alert.summary}</p>
            <button>View Full Report</button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### **Hour 14-16: Demo Video**
1. Show students rating bad meal (multiple phones)
2. Show photos of undercooked food
3. Show AI detecting pattern
4. Show warden receiving alert
5. Show warden dashboard with evidence

### **Hour 16-18: Deploy + Polish**
- Deploy to AWS Amplify
- Test end-to-end
- Create architecture diagram

### **Hour 18-20: Submit**
- Push to GitHub
- Write README
- Submit

---

## ✅ WHY THESE IDEAS ARE BETTER

| Old Ideas | New Ideas |
|-----------|-----------|
| ❌ Generic AI chatbot | ✅ Specific student problems |
| ❌ Business/commerce (overdone) | ✅ Campus life (unique) |
| ❌ Competes with existing tools | ✅ No solution exists |
| ❌ Vague value prop | ✅ Clear, measurable impact |
| ❌ Works anywhere | ✅ India-specific (hostel culture) |

---

## 🚀 PICK ONE AND BUILD

**My recommendation: MessMate**

**Why:**
- Biggest impact (health & safety)
- Most India-specific
- Best demo potential
- Uses AWS impressively

**You have 24 hours. Start now.**
