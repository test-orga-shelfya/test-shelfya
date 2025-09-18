# Email Service

## Overview
The Email Service module provides core email sending capabilities for the system, specifically focused on sending verification emails to users during account registration or email change workflows. It abstracts over the email provider (via SMTP) and ensures that verification links are reliably delivered to users' inboxes.

## Key Features

- **Send Verification Emails**: Delivers individualized verification emails containing secure tokens to user email addresses, facilitating email account validation during onboarding or security workflows.
- **SMTP Provider Integration**: Connects to the configured external SMTP server, allowing system-wide email delivery using environment-based configuration.

## System Errors

- **SMTP Configuration Error**: Triggered when required SMTP environment variables (host or port) are missing or incorrectly configured.  
  **Resolution**: Ensure `SMTP_HOST` and `SMTP_PORT` are correctly set in the environment variables.
- **Email Delivery Failure**: Occurs when an email fails to send (due to networking issues, authentication problems, or provider rejections).  
  **Resolution**: Check the error message for details, confirm SMTP server availability, and verify credentials and network connectivity.

## Usage Examples

```typescript
import { EmailService } from "./services/email.service";

const emailService = new EmailService();

const email = "user@example.com";
const verificationToken = "secureRandomToken123";

emailService.sendVerificationEmail(email, verificationToken)
  .then(() => {
    console.log("Verification email sent.");
  })
  .catch((err) => {
    console.error("Error sending verification email:", err);
  });
```

## System Integration

```mermaid
flowchart LR
  dependencies["nodemailer", "SMTP server", "Environment Variables (SMTP_HOST, SMTP_PORT, CLIENT_URL)"] --> thisModule["Email Service"]
  thisModule --> process["Sends Verification Emails"]
  thisModule --> usedBy["User Registration Flow", "Email Update Flow"]
  usedBy --> consumers["Frontend (triggers API endpoint)", "Admin Dashboard"]
```
