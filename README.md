# **System Architecture and Workflow Documentation**  

## **1. Overview**  

Our **system architecture** is designed to integrate multiple applications seamlessly, allowing for efficient business operations, user authentication, subscription management, and customer support. The infrastructure is built with **modular applications**, synchronized via **Temporal workflows** and protected by **APISIX API Gateway**.  

```mermaid
flowchart TD
    %% Define Colors
    classDef startStyle fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff;
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Frontend Deployment
    start((System Start)) --> frontendDeployment["Frontend Deployment via Vercel"]
    class start startStyle
    class frontendDeployment processStyle

    frontendDeployment --> proxyRouting["Proxy Routing via Vercel JSON Configuration"]
    proxyRouting --> multipleApps["Multiple Applications Functioning as One"]
    class proxyRouting processStyle
    class multipleApps processStyle

```
---

## **2. Frontend Deployment and Application Routing**  

Our frontend applications are deployed via **Vercel**, enabling multiple websites and services to function under a unified domain.  

- **Vercel JSON configuration** ensures that all applications are proxied under a single domain.  
- This configuration simplifies **routing, authentication, and API access** while ensuring a seamless user experience.  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Applications in the System
    applications{"Applications in Use"}
    class applications processStyle

    applications --> erpnext["ERPNext - Business Management & Seller Commissions"]
    applications --> saleor["Saleor - Product Purchases & Checkout"]
    applications --> vendure["Vendure - Subscription & Multi-User Plans"]
    applications --> zammad["Zammad - Customer Support & Ticketing"]
    applications --> qrCodeApp["QR Code App - Subscription Plan Registration"]
    applications --> multiUserApp["Multi-User App - Extends Vendure"]
    applications --> blogsApp["Blogs App - Post Management"]

```
---

## **3. Applications and Their Roles**  

Our ecosystem consists of interconnected applications that facilitate business operations, sales tracking, and user management.  

### **3.1 Business & Sales Applications**  
- **ERPNext** → Core business management system handling purchase history, order processing, and seller/affiliate commissions.  
- **Saleor** → E-commerce platform managing product sales and checkout flow.  
- **Vendure** → Manages **subscription-based services** and **multi-user plans**.  

### **3.2 Customer Support & User Management**  
- **Zammad** → Ticketing and live chat system for customer support.  
- **QR Code App** → Assists with user onboarding by linking their **Saleor purchase** to a **Vendure subscription plan**.  
- **Multi-User App** → Extends **Vendure** to support **multi-user subscription plans**.  

### **3.3 Blog Management**  
The **Blogs App** consists of two **Next.js applications** for content creation and distribution:  
- **Live Server** → Allows users to create and manage posts based on their assigned permissions.  
- **Static Server** → Publishes posts to **S3**, making them publicly accessible.  
- **Temporal** ensures real-time synchronization between the **Live and Static Servers**.  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;
    classDef storageStyle fill:#9C27B0,stroke:#333,stroke-width:2px,color:#fff;

    %% Blog Applications
    blogsApp["Blogs App - Post Management"]
    class blogsApp processStyle

    blogsApp --> liveServer["Live Server - Users Manage Posts"]
    blogsApp --> staticServer["Static Server - Publishes Posts to S3"]
    staticServer --> temporalSync["Temporal - Syncs Live & Static Servers"]
    class liveServer processStyle
    class staticServer storageStyle
    class temporalSync processStyle

```
---

## **4. Authentication, Security, and Synchronization**  

### **4.1 Centralized Authentication via Keycloak**  
- **Keycloak** is the **identity and access management system**, ensuring secure authentication across all applications.  
- **OpenID Connect (OIDC)** enforces strict authentication policies.  
- **Direct login APIs** for individual applications are disabled—users must authenticate through **Keycloak**.  

### **4.2 API Security with APISIX Gateway**  
- **APISIX acts as the API Gateway**, filtering traffic and **blocking unauthorized requests**.  
- **Only requests from Vercel** with valid OIDC tokens can access the API.  

### **4.3 Synchronization with Temporal**  
**Temporal handles real-time workflows** across applications, ensuring:  
- **Consistent user role management** across all services.  
- **Automated webhook processing** for event-driven workflows.  
- **Synchronization of blog posts** between the Live Server and Static Server.  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Authentication & Security
    authentication{"Authentication & Security"}
    class authentication processStyle

    authentication --> keycloak["Keycloak - Centralized Authentication"]
    authentication --> apisix["APISIX - API Gateway (Restricts Unauthorized Access)"]
    authentication --> temporalSecurity["Temporal - Synchronization & Role Management"]

    keycloak --> authFlow["Users Must Authenticate via Keycloak"]
    temporalSecurity --> webhookProcessing["Handles Webhooks & Application Sync"]
    temporalSecurity --> blogSync["Syncs Blog Posts"]

```
---

## **5. User Onboarding and Purchase Flow**  

### **5.1 Account Creation & Purchases**  
- Users create an account by **purchasing a product via Saleor**.  
- **Stripe** processes all payments, as **Saleor does not handle transactions internally**.  
- Purchase records are **synchronized with ERPNext**, allowing users to **track their order history**.  

### **5.2 User Synchronization Across Applications**  
- **Users registered in Saleor** are automatically created in **ERPNext** and **Zammad**.  
- **Vendure accounts require a separate onboarding process** via **QR Code App**.  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% User Onboarding
    onboarding["User Onboarding & Purchases"]
    class onboarding processStyle

    onboarding --> createAccount["User Registers & Purchases via Saleor"]
    createAccount --> stripePayments["Stripe - Payment Processing"]
    createAccount --> syncERPNext["Sync Purchase Records with ERPNext"]

```
---

## **6. Vendure Account Creation (QR Code-Based Flow)**  

To access Vendure, users must complete the **QR Code onboarding process**:  

1. **Scan QR Code**  
   - Users scan a **QR code** provided with their Saleor purchase.  
   - They are prompted to **log in or register an account**.  

2. **Accept Service Agreement**  
   - Users must **agree to the terms of service** before proceeding.  
   - A **temporary role** is assigned after agreement.  

3. **Zammad Ticket Workflow**  
   - A **support ticket is generated in Zammad** to track the onboarding process.  
   - Users **complete required steps**, such as filling out **registration forms**.  

4. **Completion & Role Assignment**  
   - Once all steps are completed, the **user is assigned their Vendure role**, allowing them to manage their subscription.  
```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% User Synchronization
    userSync{"User Synchronization Across Apps"}
    class userSync processStyle

    userSync --> syncERPNext["User Synced to ERPNext"]
    userSync --> syncZammad["User Auto-Registered in Zammad"]
    userSync --> vendureOnboarding["Vendure Onboarding via QR Code"]

```
---

## **7. Role-Based Access & Subscription Management**  

Users are **granted access** based on their role, which determines their privileges across applications.  

### **7.1 Role Progression**  
- **Temporary Role** → Assigned upon registration. Limited access.  
- **Basic Plan** → Default subscription access.  
- **Pro Plan** → Includes **AI-assisted support** in Zammad.  
- **Family Plan** → Enables **multi-user access**, allowing the primary user to **invite additional members**.  

### **7.2 Subscription Management & Upgrades**  
| **Plan** | **Features** |
|---------|-------------|
| **Basic** | Default plan with standard access to services. |
| **Pro** | Unlocks AI-assisted support in Zammad. |
| **Family** | Enables multi-user access and Pro-level benefits for all invited members. |

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;
    classDef decisionStyle fill:#FFC107,stroke:#333,stroke-width:2px,color:#000;
    classDef errorStyle fill:#F44336,stroke:#333,stroke-width:2px,color:#fff;

    %% Vendure Account Creation
    vendureOnboarding["Vendure Onboarding via QR Code"]
    class vendureOnboarding processStyle

    vendureOnboarding --> scanQRCode["Scan QR Code for Registration"]
    scanQRCode --> loginPrompt{"Prompt User to Log In or Sign Up?"}
    class loginPrompt decisionStyle

    loginPrompt -->|"No"| errorResponse["Error: User Must Log In"]
    loginPrompt -->|"Yes"| acceptService["User Accepts Service Agreement"]
    class errorResponse errorStyle
    class acceptService processStyle

    acceptService --> tempRole["Temporary Role Assigned"]
    tempRole --> zammadTicket["Zammad Ticket Created for Onboarding"]
    zammadTicket --> completeSteps["User Completes Registration Steps"]
    completeSteps --> assignVendureRole["User Assigned Final Vendure Role"]

```
---

## **8. Affiliate & Seller Commission Tracking**  

### **8.1 Seller & Influencer Tracking in ERPNext**  
- **ERPNext automatically tracks all sales** generated by sellers and influencers.  
- Sellers can access a **dedicated UI** to:  
  - View sales and commissions.  
  - Track **invoice payments** and **withdrawals**.  

### **8.2 Affiliate Marketing via Promo Codes**  
- **Affiliates receive unique promo codes** to distribute.  
- When a purchase is made using a promo code:  
  - **ERPNext records the transaction**.  
  - The sale is **attributed to the affiliate**.  

**Affiliates can access a commission dashboard**, displaying:  
- **Earnings summary**  
- **Sales reports**  
- **Payout history**  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;
    classDef decisionStyle fill:#FFC107,stroke:#333,stroke-width:2px,color:#000;

    %% Subscription & Role Management
    assignVendureRole --> roleBasedAccess{"Role-Based Access & Subscription Management"}
    class roleBasedAccess decisionStyle

    roleBasedAccess --> tempRole["Users Start with Temporary Role"]
    roleBasedAccess --> basicPlan["Basic Plan - Default Access to Vendure"]
    roleBasedAccess --> proPlan["Pro Plan - AI Support in Zammad"]
    roleBasedAccess --> familyPlan["Family Plan - Multi-User Access"]

    familyPlan --> inviteUsers["Primary User Invites Additional Members"]
    inviteUsers --> assignedProRole["Family Members Get Pro-Level Benefits"]

```
---

## **9. Customer Support & AI Assistance**  

- **Zammad handles customer support** via chat and ticketing.  
- **AI-assisted support** is available for **Pro Plan and Family Plan users only**.  
- **Users with AI-assisted support can generate interactive tickets**, allowing the AI to suggest solutions.  

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Commission Tracking
    commissionTracking{"Affiliate & Seller Commission Tracking"}
    class commissionTracking processStyle

    commissionTracking --> sellerTracking["Seller & Influencer Sales Tracking in ERPNext"]
    sellerTracking --> sellerDashboard["Seller Dashboard - View Sales & Commissions"]
    sellerTracking --> invoiceManagement["Monitor Invoice Payments & Withdrawals"]

    commissionTracking --> promoTracking["Affiliate Marketing with Promo Codes"]
    promoTracking --> affiliateDashboard["Affiliate Dashboard - Earnings, Sales, Payouts"]

```
---

## **10. Role & Permission Flow Management**  

### **10.1 Real-Time Role Synchronization with Temporal**  
- **Temporal ensures automatic role updates** across applications.  
- When a **user’s plan is upgraded** (e.g., from Basic to Pro):  
  - **All applications update access permissions in real-time**.  

### **10.2 API Gateway Enforcement**  
- **APISIX ensures only verified user requests** reach backend services.  
- Unauthorized role modifications are **automatically blocked**.  


```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Customer Support
    customerSupport{"Customer Support & AI Assistance"}
    class customerSupport processStyle

    customerSupport --> zammadChat["Zammad Chat - Customer Support"]
    customerSupport --> aiSupport["AI-Assisted Support for Pro & Family Plans"]
    aiSupport --> interactiveTickets["Interactive AI-Supported Ticketing"]

```

```mermaid
flowchart TD
    %% Define Colors
    classDef processStyle fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff;

    %% Role & Permission Flow
    roleSync["Real-Time Role Synchronization via Temporal"]
    class roleSync processStyle

    roleSync --> permissionUpdates["Permission Updates Across All Applications"]
    permissionUpdates --> planUpgrade["Upgrades: Temporary Role → Paid Plan Access"]
    planUpgrade --> accessSync["All Apps Update User Permissions"]

```
