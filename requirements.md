# Requirements Document - RentGuard AI

## 1. Executive Summary

RentGuard AI is a dual-sided platform designed to bridge the trust gap between landlords and tenants in India's rental market through AI-powered compliance checking, privacy-first verification, and automated lifecycle management.

---

## 2. Problem Statement

### 2.1 Current Market Challenges

**For Tenants:**
- Lack of legal awareness regarding Model Tenancy Act 2021 and state-specific regulations
- Exposure to unfair and illegal lease clauses
- Privacy invasion during document verification process
- No mechanism to build credit score through rent payments
- Paper-heavy, time-consuming agreement processes

**For Landlords:**
- Difficulty in reliable tenant screening without invading privacy
- Challenges in rent collection and payment tracking
- Manual legal notice generation for defaulters
- High broker fees (typically 1 month's rent)
- Lack of digital tools for lease lifecycle management

**For Both Parties:**
- Absence of standardized, legally compliant agreement templates
- No centralized digital locker for rental documents
- Limited awareness of legal rights and obligations
- Expensive legal consultation for dispute resolution

### 2.2 Market Opportunity

- **Target Market Size**: 50+ million rental households in India
- **Primary Users**: Urban millennials/Gen Z in metros (Bangalore, Mumbai, Gurgaon, Pune)
- **Secondary Users**: Independent landlords (1-5 properties)
- **Tertiary Users**: Small real estate brokers and property managers

---

## 3. User Stories & Acceptance Criteria

### 3.1 Tenant User Stories

#### US-T1: Lease Analysis (Lease Guard AI)
**As a** tenant  
**I want to** upload my draft lease agreement and get AI-powered analysis  
**So that** I can identify illegal clauses and understand my legal risks

**Acceptance Criteria:**
- AC-T1.1: System accepts PDF and image formats (max 10MB)
- AC-T1.2: OCR extracts text with >95% accuracy
- AC-T1.3: AI generates risk score (0-100) within 12 seconds
- AC-T1.4: Red flags highlight clauses violating Model Tenancy Act 2021
- AC-T1.5: Yellow flags identify ambiguous or concerning clauses
- AC-T1.6: System provides specific section references (e.g., "violates Section 21")
- AC-T1.7: Downloadable PDF report generated
- AC-T1.8: AI generates negotiation email template based on findings

#### US-T2: Privacy Control (Privacy Shield)
**As a** tenant  
**I want to** control which documents landlords can access and for how long  
**So that** my privacy is protected during verification

**Acceptance Criteria:**
- AC-T2.1: Receive real-time notification when landlord requests document access
- AC-T2.2: View detailed request showing landlord name, property, and requested documents
- AC-T2.3: Approve or deny request with single click
- AC-T2.4: If approved, landlord gets view-only access for exactly 5 minutes
- AC-T2.5: No download or screenshot capability in secure viewer
- AC-T2.6: Receive notification when documents are viewed with timestamp
- AC-T2.7: Access automatically expires after 5 minutes
- AC-T2.8: Complete audit trail maintained in CloudWatch

#### US-T3: Rent Payment (Rent-to-Credit)
**As a** tenant  
**I want to** pay rent digitally and receive HRA-compliant receipts  
**So that** I can claim tax benefits and build credit score

**Acceptance Criteria:**
- AC-T3.1: Support for Credit Card, UPI, and Net Banking
- AC-T3.2: Payment processed through secure gateway (Razorpay/Stripe)
- AC-T3.3: HRA-compliant receipt generated automatically
- AC-T3.4: Revenue stamp (₹1) added for payments > ₹5,000
- AC-T3.5: Receipt includes landlord PAN number
- AC-T3.6: Receipt saved to digital locker (S3)
- AC-T3.7: Email and WhatsApp notification sent with PDF attachment
- AC-T3.8: Future: Payment reported to credit bureau for score building

#### US-T4: Digital Signature
**As a** tenant  
**I want to** sign agreements digitally using Aadhaar eSign  
**So that** I can complete the process without physical presence

**Acceptance Criteria:**
- AC-T4.1: Integration with DigiLocker API (mocked for prototype)
- AC-T4.2: Aadhaar-based authentication
- AC-T4.3: Legally valid digital signature on PDF
- AC-T4.4: Signed document stored in digital locker
- AC-T4.5: Both parties notified upon signature completion

---

### 3.2 Landlord User Stories

#### US-L1: Agreement Creation (Instant Drafter)
**As a** landlord  
**I want to** generate legally compliant lease agreements using AI  
**So that** I can save time and ensure regulatory compliance

**Acceptance Criteria:**
- AC-L1.1: Support for Residential and Commercial agreement types
- AC-L1.2: Input fields: rent amount, deposit, duration, escalation %
- AC-L1.3: Commercial-specific fields: GSTIN, lock-in period, force majeure
- AC-L1.4: AI generates draft compliant with Model Tenancy Act 2021
- AC-L1.5: State-specific clauses added based on property location
- AC-L1.6: Preview, edit, and regenerate options available
- AC-L1.7: Stamp duty calculated based on state rates
- AC-L1.8: Mock payment gateway for stamp duty payment
- AC-L1.9: Digital e-stamp with QR code overlaid on PDF
- AC-L1.10: Agreement saved to digital locker with status "Awaiting Tenant Signature"

#### US-L2: Tenant Verification (The Handshake)
**As a** landlord  
**I want to** request and view tenant documents securely  
**So that** I can verify tenant credibility without violating privacy

**Acceptance Criteria:**
- AC-L2.1: Input tenant email/phone to send request
- AC-L2.2: Select documents: Aadhaar, payslips, bank statements
- AC-L2.3: Request stored in DynamoDB with status "PENDING"
- AC-L2.4: Tenant receives email and push notification
- AC-L2.5: Dashboard shows "Pending Approval" status
- AC-L2.6: Upon approval, receive notification with "View Documents" button
- AC-L2.7: Secure viewer with 5-minute countdown timer
- AC-L2.8: View AI-generated "Risk Score" based on tenant financials
- AC-L2.9: No download or screenshot capability
- AC-L2.10: Access auto-expires after 5 minutes
- AC-L2.11: Status updated to "VIEWED" in database

#### US-L3: Rent Collection (Smart Collections)
**As a** landlord  
**I want to** track rent payments and send automated reminders  
**So that** I can reduce payment delays and manual follow-ups

**Acceptance Criteria:**
- AC-L3.1: Visual ledger showing monthly payments (Green: Paid, Red: Overdue)
- AC-L3.2: Automated WhatsApp reminders 3 days before due date
- AC-L3.3: Email reminders on due date
- AC-L3.4: For overdue > 15 days, "Send Legal Notice" button enabled
- AC-L3.5: AI generates Section 106 legal notice using Bedrock
- AC-L3.6: Notice includes tenant details, overdue amount, days, legal language
- AC-L3.7: Multi-channel delivery: Registered email + WhatsApp + SMS
- AC-L3.8: Email tracking: opened/unopened status
- AC-L3.9: WhatsApp tracking: delivered/read status
- AC-L3.10: Notice timestamp logged in payment ledger
- AC-L3.11: PDF saved to digital locker for audit trail

#### US-L4: E-Stamp Integration
**As a** landlord  
**I want to** pay stamp duty digitally and get e-stamped agreements  
**So that** my agreements are legally valid without visiting stamp offices

**Acceptance Criteria:**
- AC-L4.1: Stamp duty calculated based on state rates and rent amount
- AC-L4.2: Mock payment gateway integration (Signzy/Legistify API)
- AC-L4.3: Digital stamp overlay on PDF with QR code
- AC-L4.4: QR code links to verification page
- AC-L4.5: Timestamp and transaction ID embedded
- AC-L4.6: E-stamped document saved to digital locker

---

## 4. Functional Requirements

### 4.1 Authentication & Authorization

**FR-1.1**: User registration via Google OAuth or Auth0  
**FR-1.2**: Role-based access control (Tenant vs Landlord)  
**FR-1.3**: JWT token-based session management  
**FR-1.4**: Multi-factor authentication for sensitive operations  
**FR-1.5**: Password reset via email OTP

### 4.2 Document Management

**FR-2.1**: Upload documents to AWS S3 private buckets  
**FR-2.2**: Organize documents by user and lease ID  
**FR-2.3**: Generate presigned URLs for time-limited access  
**FR-2.4**: Implement digital locker for all rental documents  
**FR-2.5**: Version control for agreement revisions

### 4.3 AI/ML Processing

**FR-3.1**: OCR extraction using AWS Textract  
**FR-3.2**: RAG retrieval from Pinecone vector database  
**FR-3.3**: AI inference using AWS Bedrock (Claude 3 Haiku)  
**FR-3.4**: Risk score calculation (0-100 scale)  
**FR-3.5**: Clause classification (red/yellow/green flags)  
**FR-3.6**: Legal notice generation using AI templates  
**FR-3.7**: Negotiation email generation

### 4.4 Payment Processing

**FR-4.1**: Integration with Razorpay/Stripe payment gateway  
**FR-4.2**: Support for Credit Card, UPI, Net Banking  
**FR-4.3**: Webhook handling for payment confirmation  
**FR-4.4**: Automatic receipt generation using pdf-lib  
**FR-4.5**: Revenue stamp logic for payments > ₹5,000  
**FR-4.6**: GST calculation for commercial leases  
**FR-4.7**: Payment ledger updates in DynamoDB

### 4.5 Notification System

**FR-5.1**: Email notifications via AWS SES  
**FR-5.2**: WhatsApp notifications via Twilio/Meta API  
**FR-5.3**: SMS notifications for critical alerts  
**FR-5.4**: Push notifications via Firebase Cloud Messaging  
**FR-5.5**: Email tracking (opened/unopened)  
**FR-5.6**: WhatsApp delivery status tracking

### 4.6 Compliance & Legal

**FR-6.1**: Model Tenancy Act 2021 compliance checking  
**FR-6.2**: State-specific rent control act integration  
**FR-6.3**: Section 106 legal notice templates  
**FR-6.4**: HRA-compliant receipt format  
**FR-6.5**: Digital signature integration (Aadhaar eSign)  
**FR-6.6**: E-stamping API integration

---

## 5. Non-Functional Requirements

### 5.1 Performance

**NFR-1.1**: Lease analysis completion within 12 seconds  
**NFR-1.2**: API response time < 500ms for 95% of requests  
**NFR-1.3**: Support 1000 concurrent users  
**NFR-1.4**: Document upload speed: 1MB/second minimum  
**NFR-1.5**: Payment processing within 3 seconds

### 5.2 Security

**NFR-2.1**: All data encrypted at rest (AES-256)  
**NFR-2.2**: All data encrypted in transit (TLS 1.3)  
**NFR-2.3**: Presigned URLs expire after 5 minutes  
**NFR-2.4**: No download capability in secure viewer  
**NFR-2.5**: Audit trail for all document access  
**NFR-2.6**: GDPR-compliant data handling  
**NFR-2.7**: PCI-DSS compliance for payment processing  
**NFR-2.8**: Regular security audits and penetration testing

### 5.3 Scalability

**NFR-3.1**: Serverless architecture (AWS Lambda)  
**NFR-3.2**: Auto-scaling based on load  
**NFR-3.3**: Support 10,000 users in first 6 months  
**NFR-3.4**: Database sharding for horizontal scaling  
**NFR-3.5**: CDN for static asset delivery

### 5.4 Availability

**NFR-4.1**: 99.9% uptime SLA  
**NFR-4.2**: Automated failover mechanisms  
**NFR-4.3**: Regular database backups (daily)  
**NFR-4.4**: Disaster recovery plan with RTO < 4 hours  
**NFR-4.5**: Health monitoring via CloudWatch

### 5.5 Usability

**NFR-5.1**: Mobile-responsive design  
**NFR-5.2**: Support for Hindi and English languages  
**NFR-5.3**: Accessibility compliance (WCAG 2.1 Level AA)  
**NFR-5.4**: Maximum 3 clicks to complete any task  
**NFR-5.5**: Intuitive UI with minimal learning curve

### 5.6 Maintainability

**NFR-6.1**: Modular microservices architecture  
**NFR-6.2**: Comprehensive API documentation  
**NFR-6.3**: Automated testing (unit, integration, E2E)  
**NFR-6.4**: CI/CD pipeline for deployments  
**NFR-6.5**: Centralized logging via CloudWatch

---

## 6. Data Requirements

### 6.1 Data Sources

**DR-1.1**: Model Tenancy Act, 2021 (PDF text)  
**DR-1.2**: State Rent Control Acts (Karnataka, Maharashtra, etc.)  
**DR-1.3**: Legal clause database (illegal patterns)  
**DR-1.4**: Court precedents and case law  
**DR-1.5**: Section 106 notice templates  
**DR-1.6**: User-uploaded lease documents

### 6.2 Data Storage

**DR-2.1**: DynamoDB tables:
- Users (PK: userId)
- Leases (PK: leaseId, SK: version)
- LeaseAnalysis (PK: userId, SK: analysisId)
- AccessRequests (PK: requestId)
- PaymentLedger (PK: leaseId, SK: month)
- AuditLogs (PK: userId, SK: timestamp)

**DR-2.2**: S3 buckets:
- rentguard-temp-uploads (private)
- tenant-documents (private)
- rent-receipts (private)
- legal-notices (private)
- lease-templates (public-read)

**DR-2.3**: Pinecone vector database:
- Legal text embeddings (1536 dimensions)
- Indexed by section and act name

### 6.3 Data Retention

**DR-3.1**: Lease documents: 7 years (legal requirement)  
**DR-3.2**: Payment receipts: 7 years (tax compliance)  
**DR-3.3**: Audit logs: 3 years  
**DR-3.4**: Temporary uploads: 24 hours (auto-delete)  
**DR-3.5**: User data: Until account deletion + 30 days

---

## 7. Integration Requirements

### 7.1 Third-Party APIs

**IR-1.1**: AWS Bedrock (Claude 3 Haiku) - AI inference  
**IR-1.2**: AWS Textract - OCR processing  
**IR-1.3**: Pinecone - Vector database  
**IR-1.4**: Razorpay/Stripe - Payment gateway  
**IR-1.5**: Twilio/Meta - WhatsApp API  
**IR-1.6**: AWS SES - Email delivery  
**IR-1.7**: DigiLocker - Aadhaar verification (mocked)  
**IR-1.8**: Signzy/Legistify - E-stamping (mocked)

### 7.2 Future Integrations

**IR-2.1**: Credit bureau APIs (CIBIL, Experian)  
**IR-2.2**: Insurance providers (renter's insurance)  
**IR-2.3**: Legal consultation platforms  
**IR-2.4**: Property management systems  
**IR-2.5**: Co-living company APIs (Zolo, Stanza)

---

## 8. Compliance Requirements

### 8.1 Legal Compliance

**CR-1.1**: Model Tenancy Act, 2021 adherence  
**CR-1.2**: State-specific rent control acts  
**CR-1.3**: Transfer of Property Act, 1882 (Section 106)  
**CR-1.4**: Income Tax Act (HRA provisions)  
**CR-1.5**: Indian Stamp Act, 1899

### 8.2 Data Protection

**CR-2.1**: GDPR compliance for data handling  
**CR-2.2**: Right to data portability  
**CR-2.3**: Right to be forgotten  
**CR-2.4**: Consent management for document access  
**CR-2.5**: Data breach notification within 72 hours

### 8.3 Financial Compliance

**CR-3.1**: PCI-DSS Level 1 compliance  
**CR-3.2**: RBI guidelines for payment aggregators  
**CR-3.3**: GST compliance for commercial leases  
**CR-3.4**: TDS provisions for rent > ₹50,000/month

---

## 9. Success Metrics & KPIs

### 9.1 User Metrics

**KPI-1.1**: User acquisition: 10,000 users in 6 months  
**KPI-1.2**: Monthly active users (MAU): 60% of total users  
**KPI-1.3**: User retention rate: >70% after 3 months  
**KPI-1.4**: Net Promoter Score (NPS): >50

### 9.2 Product Metrics

**KPI-2.1**: Lease analyses completed: 5,000 in Year 1  
**KPI-2.2**: Agreements generated: 3,000 in Year 1  
**KPI-2.3**: Payment GMV: ₹1 Crore in Year 1  
**KPI-2.4**: Average risk score: <40 (indicating safer leases)  
**KPI-2.5**: Legal notice effectiveness: 80% payment within 7 days

### 9.3 Technical Metrics

**KPI-3.1**: API uptime: 99.9%  
**KPI-3.2**: Average response time: <500ms  
**KPI-3.3**: Error rate: <0.1%  
**KPI-3.4**: AI accuracy: >90% for clause identification  
**KPI-3.5**: OCR accuracy: >95%

### 9.4 Business Metrics

**KPI-4.1**: Revenue: ₹10 lakhs in Year 1  
**KPI-4.2**: Customer acquisition cost (CAC): <₹500  
**KPI-4.3**: Lifetime value (LTV): >₹2,000  
**KPI-4.4**: LTV:CAC ratio: >4:1  
**KPI-4.5**: Conversion rate (free to paid): >15%

---

## 10. Constraints & Assumptions

### 10.1 Constraints

**C-1.1**: Prototype budget: <₹1,500  
**C-1.2**: Development timeline: 4 weeks for MVP  
**C-1.3**: Team size: 2-4 developers  
**C-1.4**: AWS Free Tier limitations  
**C-1.5**: DigiLocker and e-stamping APIs mocked for prototype

### 10.2 Assumptions

**A-1.1**: Users have smartphone with internet access  
**A-1.2**: Users comfortable with digital payments  
**A-1.3**: Landlords willing to adopt digital tools  
**A-1.4**: Tenants concerned about privacy  
**A-1.5**: Model Tenancy Act adoption increases  
**A-1.6**: Credit bureaus will integrate rent payment data

---

## 11. Out of Scope (Phase 1)

**OS-1.1**: Blockchain-based immutable records  
**OS-1.2**: IoT smart lock integration  
**OS-1.3**: AI-powered dispute resolution  
**OS-1.4**: Property maintenance tracking  
**OS-1.5**: Multi-language support beyond Hindi/English  
**OS-1.6**: Mobile native apps (iOS/Android)  
**OS-1.7**: Real credit bureau integration  
**OS-1.8**: Legal marketplace with lawyer consultation

---

## 12. Glossary

**Model Tenancy Act 2021**: Central legislation standardizing rental agreements in India  
**Section 106**: Legal provision for termination notice under Transfer of Property Act  
**RAG (Retrieval Augmented Generation)**: AI technique combining retrieval and generation  
**HRA (House Rent Allowance)**: Tax-exempt component of salary for rent payments  
**Presigned URL**: Time-limited URL for secure S3 object access  
**E-Stamp**: Digital stamp duty payment and certificate  
**Risk Score**: 0-100 metric indicating lease compliance level  
**The Handshake**: Privacy-first document verification feature  
**Digital Locker**: Secure cloud storage for rental documents  
**GMV (Gross Merchandise Value)**: Total transaction value processed

---

**Document Version**: 1.0  
**Last Updated**: February 2026  
**Status**: Draft for AI Bharat Hackathon Submission
