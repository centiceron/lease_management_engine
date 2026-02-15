# Design Document - RentGuard AI

## 1. System Overview

RentGuard AI is a serverless, cloud-native platform built on AWS infrastructure that provides AI-powered rental agreement management. The system follows a microservices architecture with event-driven workflows.

### 1.1 Design Principles

- **Privacy-First**: Tenant data protected with time-limited access controls
- **AI-Powered**: Leverage LLMs for compliance checking and document generation
- **Serverless**: Cost-effective, auto-scaling Lambda functions
- **Event-Driven**: Asynchronous processing for better performance
- **Mobile-First**: Responsive design for smartphone users

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Web App    │  │  Mobile Web  │  │   Admin      │         │
│  │  (React.js)  │  │  (Responsive)│  │  Dashboard   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTPS/WSS
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              API Gateway (REST + GraphQL)                │  │
│  │  - JWT Authentication  - Rate Limiting  - CORS          │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    BUSINESS LOGIC LAYER                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│  │ Lambda 1 │ │ Lambda 2 │ │ Lambda 3 │ │ Lambda 4 │         │
│  │ Analyze  │ │  Access  │ │  Notice  │ │ Payment  │         │
│  │  Lease   │ │  Grant   │ │   Gen    │ │ Process  │         │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DATA & AI LAYER                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│  │DynamoDB  │ │    S3    │ │ Pinecone │ │ Bedrock  │         │
│  │  NoSQL   │ │  Storage │ │  Vector  │ │   AI     │         │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

**Frontend:**
- Framework: React.js 18+ with Next.js 14
- State Management: Redux Toolkit / Zustand
- UI Library: Tailwind CSS + shadcn/ui
- Forms: React Hook Form + Zod validation
- File Upload: react-dropzone
- PDF Viewer: react-pdf

**Backend:**
- Runtime: Node.js 20.x on AWS Lambda
- API: AWS API Gateway (REST + WebSocket)
- Authentication: Auth0 / AWS Cognito
- Serverless Framework: AWS SAM / Serverless Framework

**Database:**
- Primary: DynamoDB (NoSQL)
- Vector: Pinecone / MongoDB Atlas Vector Search
- Cache: Redis (ElastiCache)

**AI/ML:**
- LLM: AWS Bedrock (Claude 3 Haiku)
- OCR: AWS Textract
- Embeddings: text-embedding-ada-002

**Storage:**
- Object Storage: AWS S3
- CDN: CloudFront

**External Services:**
- Payments: Razorpay / Stripe
- Email: AWS SES
- WhatsApp: Twilio / Meta Business API
- SMS: AWS SNS

---

## 3. Data Model Design

### 3.1 DynamoDB Tables

#### Table 1: Users
```
PK: userId (String)
Attributes:
- email (String)
- name (String)
- phone (String)
- role (String) // "tenant" | "landlord"
- createdAt (Number)
- lastLogin (Number)
- profileComplete (Boolean)
- kycStatus (String) // "pending" | "verified"
```

#### Table 2: Leases
```
PK: leaseId (String)
SK: version (Number)
Attributes:
- landlordId (String)
- tenantId (String)
- propertyAddress (Object)
- rentAmount (Number)
- depositAmount (Number)
- startDate (String)
- endDate (String)
- leaseType (String) // "residential" | "commercial"
- status (String) // "draft" | "active" | "expired"
- documentUrl (String) // S3 path
- stampDutyPaid (Boolean)
- signatures (Object)
  - landlord: { signed: Boolean, timestamp: Number }
  - tenant: { signed: Boolean, timestamp: Number }
```

#### Table 3: LeaseAnalysis
```
PK: userId (String)
SK: analysisId (String)
Attributes:
- fileId (String)
- fileName (String)
- riskScore (Number) // 0-100
- redFlags (List<Object>)
  - clause (String)
  - text (String)
  - violation (String)
  - severity (String)
- yellowFlags (List<Object>)
- suggestions (List<String>)
- status (String) // "processing" | "completed" | "failed"
- createdAt (Number)
- completedAt (Number)
```

#### Table 4: AccessRequests
```
PK: requestId (String)
Attributes:
- landlordId (String)
- tenantId (String)
- propertyId (String)
- documents (List<String>) // ["aadhaar", "payslips"]
- status (String) // "pending" | "approved" | "denied" | "viewed"
- createdAt (Number)
- expiresAt (Number)
- approvedAt (Number)
- viewedAt (Number)
- viewDuration (Number) // seconds
- presignedUrls (List<String>)
```

#### Table 5: PaymentLedger
```
PK: leaseId (String)
SK: month (String) // "2024-03"
Attributes:
- amount (Number)
- status (String) // "pending" | "paid" | "overdue"
- dueDate (String)
- paidAt (Number)
- paymentId (String)
- paymentMethod (String)
- receiptUrl (String) // S3 path
- noticesSent (List<Object>)
  - type (String)
  - sentAt (Number)
  - channels (List<String>)
```

### 3.2 S3 Bucket Structure

```
rentguard-temp-uploads/
  └── temp/
      └── {userId}/
          └── {fileId}.pdf

tenant-documents/
  └── {userId}/
      ├── aadhaar.pdf
      ├── payslips/
      └── bank_statements/

lease-documents/
  └── {leaseId}/
      ├── draft_v1.pdf
      ├── signed_v1.pdf
      └── stamped_v1.pdf

rent-receipts/
  └── {userId}/
      └── {year}/
          └── {month}.pdf

legal-notices/
  └── {leaseId}/
      └── {noticeId}.pdf
```

---

## 4. API Design

### 4.1 Authentication APIs

**POST /auth/register**
```json
Request:
{
  "email": "user@example.com",
  "name": "John Doe",
  "phone": "+919876543210",
  "role": "tenant"
}

Response:
{
  "userId": "usr_123",
  "token": "jwt_token",
  "expiresIn": 3600
}
```

**POST /auth/login**
```json
Request:
{
  "email": "user@example.com",
  "password": "hashed_password"
}

Response:
{
  "userId": "usr_123",
  "token": "jwt_token",
  "role": "tenant"
}
```

### 4.2 Lease Analysis APIs

**POST /lease/analyze**
```json
Request:
{
  "fileId": "file_123",
  "fileName": "lease_draft.pdf"
}

Response:
{
  "analysisId": "ana_456",
  "status": "processing",
  "estimatedTime": 10
}
```

**GET /lease/analysis/{analysisId}**
```json
Response:
{
  "analysisId": "ana_456",
  "riskScore": 72,
  "status": "completed",
  "redFlags": [
    {
      "clause": "5.2",
      "text": "Landlord may evict without notice",
      "violation": "Section 21 of Model Tenancy Act",
      "severity": "HIGH"
    }
  ],
  "yellowFlags": [...],
  "suggestions": [...]
}
```

### 4.3 Access Request APIs

**POST /access/request**
```json
Request:
{
  "tenantId": "usr_789",
  "propertyId": "prop_456",
  "documents": ["aadhaar", "payslips", "bank_statement"]
}

Response:
{
  "requestId": "req_123",
  "status": "pending",
  "expiresAt": 1709251200
}
```

**POST /access/approve**
```json
Request:
{
  "requestId": "req_123",
  "action": "approve"
}

Response:
{
  "requestId": "req_123",
  "status": "approved",
  "viewUrl": "/secure-viewer/req_123",
  "expiresIn": 300
}
```

### 4.4 Payment APIs

**POST /payment/create**
```json
Request:
{
  "leaseId": "lease_123",
  "month": "2024-03",
  "amount": 15000,
  "method": "credit_card"
}

Response:
{
  "orderId": "order_xyz",
  "amount": 15000,
  "currency": "INR",
  "razorpayKey": "rzp_test_key"
}
```

**POST /webhooks/payment-success**
```json
Request:
{
  "event": "payment.captured",
  "payload": {
    "orderId": "order_xyz",
    "paymentId": "pay_abc",
    "amount": 15000,
    "method": "card"
  }
}

Response:
{
  "status": "processed",
  "receiptUrl": "s3://receipts/usr_123/2024/march.pdf"
}
```

---

## 5. Component Design

### 5.1 Frontend Components

**Tenant Dashboard**
```
TenantDashboard/
├── LeaseGuardUploader
│   ├── FileDropzone
│   ├── UploadProgress
│   └── AnalysisResults
├── PrivacyCenter
│   ├── AccessRequestList
│   ├── RequestDetails
│   └── ApprovalModal
├── PaymentSection
│   ├── RentCalendar
│   ├── PaymentForm
│   └── ReceiptViewer
└── DigitalLocker
    ├── DocumentGrid
    └── DocumentViewer
```

**Landlord Dashboard**
```
LandlordDashboard/
├── LeaseCreator
│   ├── AgreementTypeSelector
│   ├── DetailsForm
│   ├── AIPreview
│   └── StampDutyPayment
├── TenantScreener
│   ├── RequestForm
│   ├── PendingRequests
│   └── SecureViewer
├── PaymentLedger
│   ├── MonthlyGrid
│   ├── OverdueAlerts
│   └── NoticeGenerator
└── Analytics
    ├── RevenueChart
    └── TenantRiskScores
```

### 5.2 Backend Lambda Functions

**fn-analyze-lease**
```javascript
exports.handler = async (event) => {
  // 1. Get file from S3
  // 2. Extract text via Textract
  // 3. Query Pinecone for relevant laws
  // 4. Send to Bedrock for analysis
  // 5. Store results in DynamoDB
  // 6. Send SNS notification
};
```

**fn-grant-access**
```javascript
exports.handler = async (event) => {
  // 1. Validate tenant approval
  // 2. Generate presigned URLs (300s expiry)
  // 3. Update DynamoDB status
  // 4. Send WebSocket notification to landlord
  // 5. Schedule auto-expire Lambda
};
```

**fn-legal-notice**
```javascript
exports.handler = async (event) => {
  // 1. Fetch lease metadata
  // 2. Load Section 106 template
  // 3. Fill template via Bedrock
  // 4. Generate PDF
  // 5. Send via SES + Twilio
  // 6. Update ledger
};
```

**fn-process-rent**
```javascript
exports.handler = async (event) => {
  // 1. Verify webhook signature
  // 2. Update payment ledger
  // 3. Check lease type (residential/commercial)
  // 4. Generate receipt PDF
  // 5. Save to S3
  // 6. Send notifications
};
```

---

## 6. AI/ML Design

### 6.1 RAG Pipeline

**Step 1: Document Ingestion**
```python
# Preprocess legal documents
def ingest_legal_docs():
    docs = load_pdfs(["Model_Tenancy_Act_2021.pdf", ...])
    chunks = split_into_chunks(docs, chunk_size=512)
    embeddings = generate_embeddings(chunks)
    pinecone.upsert(embeddings)
```

**Step 2: Retrieval**
```python
def retrieve_context(query):
    query_embedding = generate_embedding(query)
    results = pinecone.query(query_embedding, top_k=10)
    return [r.metadata['text'] for r in results]
```

**Step 3: Generation**
```python
def analyze_lease(lease_text):
    context = retrieve_context(lease_text)
    prompt = f"""
    You are a legal expert in Indian rental law.
    
    LEASE TEXT:
    {lease_text}
    
    LEGAL CONTEXT:
    {context}
    
    Analyze and return JSON:
    {{
      "riskScore": 0-100,
      "redFlags": [...],
      "yellowFlags": [...],
      "suggestions": [...]
    }}
    """
    response = bedrock.invoke(prompt)
    return parse_json(response)
```

### 6.2 Prompt Templates

**Lease Analysis Prompt**
```
System: You are an expert in Indian rental law with deep knowledge of the Model Tenancy Act 2021.

User: Analyze this lease agreement and identify violations:

LEASE TEXT:
{extracted_text}

RELEVANT LAWS:
{rag_context}

Provide:
1. Risk Score (0-100): Higher = more risky
2. Red Flags: Critical violations with section references
3. Yellow Flags: Ambiguous or concerning clauses
4. Recommendations: How to fix issues

Format as JSON.
```

**Legal Notice Prompt**
```
System: You are a legal notice drafter specializing in Section 106 notices.

User: Fill this template with the provided details:

TEMPLATE:
{section_106_template}

DATA:
- Tenant: {tenant_name}
- Address: {tenant_address}
- Rent: ₹{rent_amount}
- Overdue: {overdue_days} days
- Amount Due: ₹{overdue_amount}

Ensure professional legal language and accurate citations.
```

---

## 7. Security Design

### 7.1 Authentication Flow

```
User → Frontend → API Gateway → Lambda Authorizer
                                    ↓
                              Verify JWT Token
                                    ↓
                              Check User Role
                                    ↓
                              Allow/Deny Request
```

### 7.2 Document Access Control

**Presigned URL Generation**
```javascript
const params = {
  Bucket: 'tenant-documents',
  Key: `${userId}/aadhaar.pdf`,
  Expires: 300, // 5 minutes
  ResponseContentDisposition: 'inline', // No download
};
const url = s3.getSignedUrl('getObject', params);
```

**Secure Viewer Implementation**
```javascript
// Disable right-click
document.addEventListener('contextmenu', (e) => e.preventDefault());

// Disable keyboard shortcuts
document.addEventListener('keydown', (e) => {
  if (e.ctrlKey && (e.key === 's' || e.key === 'p')) {
    e.preventDefault();
  }
});

// Watermark overlay
<div className="watermark">
  Viewed by {landlordName} at {timestamp}
</div>
```

### 7.3 Data Encryption

**At Rest:**
- S3: AES-256 server-side encryption
- DynamoDB: AWS KMS encryption
- Secrets: AWS Secrets Manager

**In Transit:**
- TLS 1.3 for all API calls
- Certificate pinning for mobile apps

---

## 8. Performance Optimization

### 8.1 Caching Strategy

**Redis Cache:**
```javascript
// Cache user profile
await redis.set(`user:${userId}`, JSON.stringify(user), 'EX', 3600);

// Cache lease analysis
await redis.set(`analysis:${analysisId}`, result, 'EX', 86400);

// Cache legal templates
await redis.set('template:section106', template, 'EX', 604800);
```

**CloudFront CDN:**
- Static assets (JS, CSS, images)
- Public documents (templates, guides)
- API responses with Cache-Control headers

### 8.2 Database Optimization

**DynamoDB Indexes:**
```
GSI1: landlordId-createdAt-index
GSI2: tenantId-status-index
GSI3: leaseId-month-index
```

**Query Patterns:**
```javascript
// Get all leases for a landlord
const params = {
  TableName: 'Leases',
  IndexName: 'landlordId-createdAt-index',
  KeyConditionExpression: 'landlordId = :lid',
  ExpressionAttributeValues: { ':lid': landlordId }
};
```

### 8.3 Lambda Optimization

**Cold Start Reduction:**
- Provisioned concurrency for critical functions
- Minimize dependencies (use Lambda layers)
- Keep functions under 50MB

**Memory Allocation:**
- fn-analyze-lease: 2048 MB (AI processing)
- fn-grant-access: 512 MB (simple logic)
- fn-process-rent: 1024 MB (PDF generation)

---

## 9. Monitoring & Observability

### 9.1 CloudWatch Metrics

**Custom Metrics:**
- LeaseAnalysisTime (ms)
- RiskScoreDistribution
- PaymentSuccessRate
- AccessRequestApprovalRate
- APILatency (p50, p95, p99)

**Alarms:**
- Error rate > 1%
- API latency > 1000ms
- Lambda throttling
- DynamoDB capacity exceeded

### 9.2 Logging Strategy

**Structured Logging:**
```javascript
logger.info('Lease analysis started', {
  userId,
  analysisId,
  fileSize,
  timestamp: Date.now()
});

logger.error('Payment processing failed', {
  orderId,
  error: err.message,
  stack: err.stack
});
```

**Log Aggregation:**
- CloudWatch Logs Insights for querying
- Export to S3 for long-term storage
- Integration with third-party tools (Datadog, New Relic)

---

## 10. Deployment Architecture

### 10.1 CI/CD Pipeline

```
GitHub → GitHub Actions → Build & Test → Deploy to Dev
                              ↓
                         Integration Tests
                              ↓
                         Deploy to Staging
                              ↓
                         Manual Approval
                              ↓
                         Deploy to Production
```

### 10.2 Infrastructure as Code

**AWS SAM Template:**
```yaml
Resources:
  AnalyzeLeaseFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs20.x
      MemorySize: 2048
      Timeout: 30
      Environment:
        Variables:
          BEDROCK_MODEL: anthropic.claude-3-haiku
          PINECONE_INDEX: legal-docs
      Events:
        S3Upload:
          Type: S3
          Properties:
            Bucket: !Ref TempUploadsBucket
            Events: s3:ObjectCreated:*
```

---

## 11. Error Handling

### 11.1 Error Codes

```
4000: Invalid request format
4001: Missing required fields
4010: Unauthorized access
4011: Token expired
4030: Resource not found
4031: Lease not found
4040: Payment failed
5000: Internal server error
5001: AI service unavailable
5002: Database error
```

### 11.2 Retry Logic

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (err) {
      if (i === maxRetries - 1) throw err;
      await sleep(Math.pow(2, i) * 1000);
    }
  }
}
```

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Status**: Draft for AI Bharat Hackathon Submission
