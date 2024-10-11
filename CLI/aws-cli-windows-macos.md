# AWS CLI Installation & Configuration Guide

This guide covers the installation and configuration of the **AWS CLI** (Command Line Interface) on **Windows** and **macOS**, including information on AWS credentials, how to use multiple profiles, and output formatting options like JSON, YAML, and table.

## Prerequisites

- A valid **AWS account**. If you don’t have one, you can sign up at [aws.amazon.com](https://aws.amazon.com/).
- Administrator access to your computer for installation.
  
---

## Table of Contents
- [Installing AWS CLI on Windows](#installing-aws-cli-on-windows)
- [Installing AWS CLI on macOS](#installing-aws-cli-on-macos)
- [Verify the Installation](#verify-the-installation)
- [Configure AWS CLI](#configure-aws-cli)
- [AWS STS get-caller-identity Command](#aws-sts-get-caller-identity-command)
- [Managing AWS Credentials](#managing-aws-credentials)
  - [Default Location of AWS Credentials Files](#default-location-of-aws-credentials-files)
  - [Adding Multiple Profiles](#adding-multiple-profiles)
  - [Using Profiles with the AWS CLI](#using-profiles-with-the-aws-cli)
- [Formatting AWS CLI Output](#formatting-aws-cli-output)
  - [Output Formats: JSON, YAML, and Table](#output-formats-json-yaml-and-table)
- [Uninstalling AWS CLI](#uninstalling-aws-cli)

---

## Installing AWS CLI on Windows

To install the AWS CLI on Windows:

### Step 1: Download the AWS CLI Installer
- Visit [AWS CLI for Windows](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
- Download the **AWS CLI MSI installer**.

### Step 2: Run the Installer
- Open the downloaded `.msi` file.
- Follow the setup instructions:
  - Accept the license agreement.
  - Choose the installation folder.
  - Click **Install** and wait for the process to complete.

---

## Installing AWS CLI on macOS

To install the AWS CLI on macOS, follow these steps:

### Option 1: Install via Homebrew

1. Open **Terminal**.
2. Run the following command:

   ```bash
   brew install awscli
   ```

### Option 2: Install using AWS Installer

1. Go to the [AWS CLI download page for macOS](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
2. Download the **AWS CLI ZIP installer**.
3. Extract the file:

   ```bash
   unzip awscli-exe-macos.zip
   ```

4. Run the installer:

   ```bash
   sudo ./aws/install
   ```

---

## Verify the Installation

After installing the AWS CLI, confirm that it’s installed by running the following command in **Command Prompt** (Windows) or **Terminal** (macOS):

```bash
aws --version
```

The output should be similar to:

```
aws-cli/2.x.x Python/3.x.x <platform> botocore/2.x.x
```

---

## Configure AWS CLI

To use the AWS CLI, you need to configure it with your **AWS Access Key ID**, **Secret Access Key**, **default region**, and **output format**.

### Step 1: Run the AWS Configure Command

```bash
aws configure
```

You will be prompted to enter the following:

```
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: YOUR_REGION (e.g., us-east-1)
Default output format [None]: json
```

---

## AWS STS get-caller-identity Command

The `aws sts get-caller-identity` command retrieves details about the AWS account and identity being used to make the CLI call. It’s useful for verifying the IAM user or role you're operating with.

### Example Command

```bash
aws sts get-caller-identity
```

### Example Output

```json
{
  "UserId": "AIDACKCEVSQ6C2EXAMPLE",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/YourUserName"
}
```

This output displays:
- **UserId**: The IAM user's or role's ID.
- **Account**: Your AWS account ID.
- **Arn**: The Amazon Resource Name (ARN) of the IAM entity.

---

## Managing AWS Credentials

The AWS CLI stores credentials in a file so that they can be used automatically without requiring re-entry every time.

### Default Location of AWS Credentials Files

On both Windows and macOS, AWS credentials are stored in the following files:

- **AWS Credentials File**: This file contains access keys.
- **AWS Config File**: This file contains region and output format settings.

#### File Locations:
- **Windows**: `C:\Users\<Your-Username>\.aws\credentials` and `C:\Users\<Your-Username>\.aws\config`
- **macOS/Linux**: `~/.aws/credentials` and `~/.aws/config`

---

### Adding Multiple Profiles

You can create multiple profiles for different AWS accounts or roles in the AWS credentials file.

1. Open the `credentials` file:

   **Windows**: `C:\Users\<Your-Username>\.aws\credentials`  
   **macOS/Linux**: `~/.aws/credentials`

2. Add another profile by creating a new section. For example:

   ```ini
   [default]
   aws_access_key_id = YOUR_DEFAULT_ACCESS_KEY
   aws_secret_access_key = YOUR_DEFAULT_SECRET_KEY

   [dev]
   aws_access_key_id = YOUR_DEV_ACCESS_KEY
   aws_secret_access_key = YOUR_DEV_SECRET_KEY

   [prod]
   aws_access_key_id = YOUR_PROD_ACCESS_KEY
   aws_secret_access_key = YOUR_PROD_SECRET_KEY
   ```

---

### Using Profiles with the AWS CLI

You can specify which profile to use by using the `--profile` flag with any AWS CLI command.

#### Example using a specific profile:

```bash
aws s3 ls --profile dev
```

If no `--profile` flag is provided, the **default** profile will be used.

---

## Formatting AWS CLI Output

You can control how the AWS CLI formats its output using the `--output` flag. Available formats are:

- **json**: The default output format.
- **yaml**: Useful for more readable output.
- **table**: A columnar, human-readable format.

### Example Commands

#### JSON Output

```bash
aws s3 ls --output json
```

#### YAML Output

```bash
aws s3 ls --output yaml
```

#### Table Output

```bash
aws s3 ls --output table
```

The `--output` flag can be used with any AWS CLI command to control the output format.

---

## Uninstalling AWS CLI

### On Windows

1. Open **Control Panel**.
2. Go to **Programs > Uninstall a program**.
3. Select **AWS Command Line Interface** and click **Uninstall**.

### On macOS

To uninstall the AWS CLI, run the following command in Terminal:

```bash
sudo rm -rf /usr/local/aws-cli
sudo rm /usr/local/bin/aws
```

---

## Useful Resources

- [AWS CLI Official Documentation](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)

---

This documentation now covers all aspects of installing, configuring, using, and managing the AWS CLI on both Windows and macOS. It also explains how to work with AWS credentials, profiles, output formats, and the `aws sts get-caller-identity` command.
