# Gamdom Expense Approval Automation - n8n Workflow

Automated expense approval system built with n8n, featuring multi-level validation, email-based approval workflow, external API integration, and comprehensive logging.

**Built for:** Gamdom Automation Engineer Assessment  
**Implementation Time:** ~8 hours  
**Total Workflows:** 3 workflows, ~25 nodes

---

## Table of Contents

- [Overview](#overview)
- [Workflow Architecture](#workflow-architecture)
- [Setup Instructions](#setup-instructions)
- [Workflow Logic](#workflow-logic)
- [Design Decisions](#design-decisions)
- [Assumptions](#assumptions)
- [Testing Guide](#testing-guide)

---

## Overview

This project implements an automated expense approval system that:

- Validates expense data from Google Sheets
- Sends email-based approval requests to managers
- Integrates with external API for expense processing
- Logs all actions to Supabase database
- Sends real-time Slack notifications

**Technology Stack:** n8n, Google Sheets, Gmail, Slack, Supabase

---

## Workflow Architecture

### System Flow
```
Google Sheets (Input)
    ↓
Workflow 1: Validation & Approval Request
    ↓ (Webhook)
Workflow 2: Approval Handler
    ↓ (HTTP Request)
Workflow 3: Mock External API
```

**Workflow 1:** Validates data and sends approval email  
**Workflow 2:** Processes approval decision and calls API  
**Workflow 3:** Simulates external expense system

![Workflow 1](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/expense-validation-approval%20.png)
![Workflow 2](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/expense-approval-%20handler.png)
![Workflow 3](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/mock-external-api.png)

---

## Setup Instructions

### Prerequisites

- n8n instance (v1.111.0+)
- Google account (Sheets + Gmail)
- Slack workspace
- Supabase account

### 1. Database Setup

Create Supabase table:
```sql
CREATE TABLE expense_logs (
  id BIGSERIAL PRIMARY KEY,
  transaction_id TEXT NOT NULL,
  name TEXT,
  email TEXT,
  amount DECIMAL(10,2),
  department TEXT,
  expense_type TEXT,
  status TEXT NOT NULL,
  action_type TEXT NOT NULL,
  message TEXT,
  error_details TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

Get credentials: Project URL + API Key

### 2. Google Sheets Setup

Create sheet with columns:

| Column | Name | Required |
|--------|------|----------|
| A | Timestamp | No |
| B | Name | Yes |
| C | Email | Yes |
| D | Department | No |
| E | Expense_Type | No |
| F | Amount | Yes |
| G | Transaction_ID | Yes |
| H | Description | No |

### 3. Slack Setup

1. Create app → Enable Incoming Webhooks
2. Add webhook to `#expense-automation` channel
3. Copy webhook URL

### 4. Gmail Setup

1. Enable 2-Step Verification
2. Generate App Password
3. Use in n8n Gmail credential

### 5. Import Workflows

Import in order:
1. `workflows/mock-external-api.json`
2. `workflows/expense-approval-handler.json`
3. `workflows/expense-validation-approval.json`

### 6. Configure & Activate

1. Add credentials (Google Sheets, Gmail, Slack, Supabase)
2. Update webhook URLs between workflows
3. Set manager email in Workflow 1
4. Activate all workflows

---

## Workflow Logic

### Workflow 1: Validation & Approval Request

1. Google Sheets trigger
2. Validate required fields
3. Validate email format
4. Validate amount (positive)
5. Send approval email
6. Log to Supabase

![Validation Error-Slack notification](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/slack-validation-error.png)
![Approval Email](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/gmail-approval-request.png)

### Workflow 2: Approval Handler

**If Approved:**
- Log "approved"
- Call Mock API
- Log "api_success" or "api_failed"
- Send Slack notification

**If Rejected:**
- Log "rejected"
- Email submitter

![Rejection E-mail](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/gmail-rejection-notification.png)
![Slack Success](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/slack-success.png)
![Slack Error](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/slack-api-failed.png)

### Workflow 3: Mock API

Returns success response:
```json
{
  "status": "success",
  "message": "Expense processed",
  "transaction_id": "TXN-XXX"
}
```

---

## Design Decisions

### 1. Supabase for Logging

**Why not Google Sheets?**
- Professional SQL database
- Better querying
- Scalable

### 2. Three Separate Workflows

**Why?**
- Decoupled architecture
- No timeout issues
- Independent testing

### 3. Email-Based Approval

**Why?**
- Accessible from anywhere
- One-click UX
- Audit trail

### 4. Logging Structure
```
action_type: approval_request | api_call
status: approval_pending | approved | rejected | api_success | api_failed
```

---

## Assumptions

- Google Sheets columns match exactly
- Transaction ID is unique
- All amounts in USD
- Single manager approval
- No approval timeout
- No retry mechanism for API failures

---

## Testing Guide

### Test Scenarios

#### ✅ Test 1: Success Flow
**Data:** John Smith, TXN-2025-001  
**Result:** Validation → Approval → API Success → Slack ✅

#### ❌ Test 2: Missing Name
**Data:** Empty name  
**Result:** Slack error "Missing fields: Name" ✅

#### ❌ Test 3: Invalid Email
**Data:** david@company  
**Result:** Slack error "Invalid email format" ✅

#### ❌ Test 4: Negative Amount
**Data:** -500.00  
**Result:** Slack error "Amount must be positive" ✅

#### ⛔ Test 5: Rejection
**Data:** James Taylor, TXN-2025-006  
**Result:** Email to submitter + Slack rejection ✅

#### ⚠️ Test 6: API Failure
**Setup:** Workflow 3 deactivated  
**Result:** Slack "Approved but API failed" ✅


![Test Results](https://github.com/berkaycan90/gamdom-expense-approval/blob/main/setup/test-data.csv%20.csv)

![Log Results]Log Result:(https://github.com/berkaycan90/gamdom-expense-approval/blob/main/assets/supabase-logs-overview.png)

**Coverage:** 6/6 scenarios passed (100%)

---

## Repository Structure
```
gamdom-expense-approval/
├── README.md
├── workflows/
│   ├── expense-validation-approval.json
│   ├── expense-approval-handler.json
│   └── mock-external-api.json
├── assets/
│   ├── expense-validation-approval.png
│   ├── expense-approval-handler.png
│   ├── mock-external-api.png
│   ├── gmail-approval-request.png
│   ├── gmail-rejection-notification.png
│   ├── slack-success.png
│   ├── slack-validation-error.png
│   ├── slack-api-failed.png
│   └── supabase-logs-overview.png
└── setup/
    ├── supabase-schema-logs.csv
    └── test-data.csv
```

---

## Implementation Summary

### ✅ Requirements Met

- Logical workflow design
- Proper approval and error handling
- Efficient n8n node usage
- Clear documentation

### 📊 Metrics

- **Workflows:** 3
- **Nodes:** ~25
- **Integrations:** 5
- **Test Coverage:** 100%
- **Implementation Time:** ~8 hours

---

**n8n Version:** 1.111.0 (Cloud)  
**Author:** Berkay Can  
**Date:** November 2025

Submitted for Gamdom Automation Engineer assessment.