# learn-aws-iam-password-policy-config
how to aws config for compliance and set password policy and deny creation without mfa

To **configure IAM best practices using Terraform** and **enforce compliance with AWS Config**, follow these steps:  

---

# **1️⃣ Use Terraform to Configure AWS IAM Password Policy**
This Terraform code will enforce a **strong password policy** to meet security best practices.  

### **Terraform Code for IAM Password Policy**
```hcl
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 14
  require_uppercase_characters   = true
  require_lowercase_characters   = true
  require_numbers                = true
  require_symbols                = true
  allow_users_to_change_password = true
  max_password_age               = 90
  password_reuse_prevention      = 24
  hard_expiry                    = false
}
```

✅ **This ensures IAM users have strong passwords and meet compliance standards.**  

---

# **2️⃣ Use Terraform to Enforce MFA for IAM Users**
To **require MFA**, create a **Deny-All policy** that blocks actions unless the user is authenticated with MFA.  

### **Terraform Code for MFA Enforcement**
```hcl
resource "aws_iam_policy" "enforce_mfa" {
  name        = "EnforceMFA"
  description = "IAM policy to enforce MFA for all users"
  policy      = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect   = "Deny",
        Action   = "*",
        Resource = "*",
        Condition = {
          BoolIfExists = {
            "aws:MultiFactorAuthPresent": "false"
          }
        }
      }
    ]
  })
}

resource "aws_iam_group" "mfa_users" {
  name = "MFA-Users"
}

resource "aws_iam_group_policy_attachment" "attach_mfa_policy" {
  group      = aws_iam_group.mfa_users.name
  policy_arn = aws_iam_policy.enforce_mfa.arn
}
```
✅ **This denies all actions unless MFA is enabled for the user.**  

---

# **3️⃣ Use AWS Config to Monitor IAM Compliance**
AWS Config helps ensure that IAM security settings are **continuously monitored**.  

## **Terraform Code for AWS Config Rule: IAM Password Policy Compliance**
```hcl
resource "aws_config_config_rule" "iam_password_policy_compliance" {
  name        = "iam-password-policy"
  description = "Ensure IAM password policy meets security best practices"
  source {
    owner             = "AWS"
    source_identifier = "IAM_PASSWORD_POLICY"
  }
}
```

✅ **AWS Config will now monitor if the IAM password policy is compliant.**  

---

## **4️⃣ Use AWS Config to Monitor MFA Compliance**
To check if **IAM users have MFA enabled**, deploy the following **AWS Config rule**:  

### **Terraform Code for MFA Compliance**
```hcl
resource "aws_config_config_rule" "iam_mfa_enabled" {
  name        = "iam-mfa-enabled"
  description = "Ensure all IAM users have MFA enabled"
  source {
    owner             = "AWS"
    source_identifier = "IAM_USER_MFA_ENABLED"
  }
}
```
✅ **This will flag any IAM users who have not enabled MFA.**  

---

# **5️⃣ (Optional) Enable AWS Config to Store Compliance Data**
AWS Config needs a **S3 bucket and SNS topic** for logs and notifications.  

### **Terraform Code for AWS Config Setup**
```hcl
resource "aws_s3_bucket" "config_bucket" {
  bucket = "my-aws-config-bucket"
}

resource "aws_config_configuration_recorder" "config_recorder" {
  name     = "default"
  role_arn = aws_iam_role.config_role.arn
}

resource "aws_config_delivery_channel" "config_channel" {
  name           = "default"
  s3_bucket_name = aws_s3_bucket.config_bucket.bucket
}

resource "aws_config_configuration_recorder_status" "config_status" {
  name       = aws_config_configuration_recorder.config_recorder.name
  is_enabled = true
}
```
✅ **Now, AWS Config will continuously monitor IAM compliance and log changes.**  

---

# **🔹 Summary: What This Terraform Setup Does**
✅ **IAM Security Best Practices Implemented**:
- **Strong password policy** (length, complexity, expiration).  
- **MFA enforcement** (deny all actions if MFA is not enabled).  
- **AWS Config rules** to **monitor compliance** and detect violations.  
- **AWS Config logging** to store compliance data in S3.  

---

## **Would You Like Additional Features?**
- **Auto-remediation:** Automatically disable non-compliant IAM users.  
- **Notification alerts:** Send an SNS notification if AWS Config detects a violation.  

Let me know if you want these added! 🚀
