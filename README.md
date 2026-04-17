# mnemospark-website

Static site for mnemospark.

## Environments

- **Staging/Test**: GitHub Pages (existing test pages)
- **Production**: AWS CloudFront + private S3 origin

## Production architecture (v1)

- S3 bucket (private, versioned, encrypted)
- CloudFront distribution with:
  - custom domains: `mnemospark.ai`, `www.mnemospark.ai`
  - TLS cert from ACM (`us-east-1`)
  - OAC (Origin Access Control)
  - CloudFront Function redirecting `www` -> apex (`mnemospark.ai`)
- GitHub Actions deploy on `main` (OIDC, no static AWS keys)
- No WAF for v1

## App architecture (v1) — `app.mnemospark.ai`

The app shell is a **separate** static site (dedicated S3 bucket + CloudFront distribution) and does **not** share the marketing bucket or distribution.

- Content lives under `app/`
- Infra stack: `infra/cloudformation/website-app.yaml`
- Deploy workflow: `.github/workflows/deploy-app.yml` (OIDC, no static AWS keys)

## Repo layout

- `prod/` → production site content (deployed to S3 root)
- `app/` → app shell content for `app.mnemospark.ai` (deployed to a separate S3 bucket)
- `infra/cloudformation/website-prod.yaml` → infrastructure stack
- `infra/cloudformation/website-app.yaml` → app infrastructure stack
- `.github/workflows/deploy-prod.yml` → CI deploy pipeline
- `.github/workflows/deploy-app.yml` → CI deploy pipeline (app)

## Current production content source

Production HTML and static assets live under **`prod/`** (for example `prod/index.html`, `prod/favicon.svg`, `prod/og-image.png`). Edit those files directly; merging to `main` deploys them to S3.

## One-time setup

### 1) Request ACM certificate (us-east-1)

Request a public cert in **us-east-1** for:
- `mnemospark.ai`
- `www.mnemospark.ai`

Validate via DNS in Porkbun (ACM will provide CNAME validation records).

### 2) Ensure GitHub OIDC provider exists in IAM

If you already use GitHub OIDC in this account, reuse it.
Pass your existing GitHub OIDC provider ARN from **IAM → Identity providers**.

If missing, create once (outside this stack).

### 3) Deploy CloudFormation stack

```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name mnemospark-website-prod \
  --template-file infra/cloudformation/website-prod.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    ProjectName=mnemospark-website \
    DomainName=mnemospark.ai \
    WwwDomainName=www.mnemospark.ai \
    AcmCertificateArn=<YOUR_ACM_CERT_ARN> \
    GitHubOrg=pawlsclick \
    GitHubRepo=mnemospark-website \
    GitHubBranch=main \
    GitHubOidcProviderArn=<YOUR_GITHUB_OIDC_PROVIDER_ARN>
```

Get outputs:

```bash
aws cloudformation describe-stacks \
  --region us-east-1 \
  --stack-name mnemospark-website-prod \
  --query "Stacks[0].Outputs"
```

Record these outputs:
- `ApexDnsTarget` (CloudFront domain)
- `WwwDnsTarget` (same CloudFront domain)
- `GitHubDeployRoleArn`

### 4) Configure DNS in Porkbun

Porkbun supports apex ALIAS/CNAME flattening (their KB recommends ALIAS for root + CNAME for `www`).

Create records:
- `@` → **ALIAS** to `ApexDnsTarget`
- `www` → **CNAME** to `WwwDnsTarget`
- Add ACM validation CNAME records exactly as provided by ACM

### 5) GitHub Actions deploy

Workflow: `.github/workflows/deploy-prod.yml`

It assumes the deploy role via the `AWS_ROLE_ARN_PROD` repository (or `prod` environment) secret. After stack deploy, use output `GitHubDeployRoleArn`, or this pattern:

`<YOUR_GITHUB_DEPLOY_ROLE_ARN>`

(`mnemospark-website-github-deploy-main` is the default role name in the CloudFormation template unless you override it.)

On push to `main` affecting `prod/**` (or manual dispatch), it:
1. Reads stack outputs
2. Syncs `prod/` to S3
3. Invalidates CloudFront (`/*`)

## App (`app.mnemospark.ai`) one-time setup

### 1) Deploy CloudFormation stack

```bash
aws cloudformation deploy \
  --region us-east-1 \
  --stack-name mnemospark-website-app \
  --template-file infra/cloudformation/website-app.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    ProjectName=mnemospark-website \
    AppDomainName=app.mnemospark.ai \
    AcmCertificateArn=<YOUR_ACM_CERT_ARN> \
    GitHubOrg=pawlsclick \
    GitHubRepo=mnemospark-website \
    GitHubBranch=main \
    GitHubOidcProviderArn=<YOUR_GITHUB_OIDC_PROVIDER_ARN>
```

Record these outputs:
- `AppDnsTarget` (CloudFront domain)
- `GitHubDeployRoleArn`

### 2) Configure DNS in Porkbun

Create records:
- `app` → **CNAME** to `AppDnsTarget`

### 3) GitHub Actions deploy

Workflow: `.github/workflows/deploy-app.yml`

It assumes the deploy role via the `AWS_ROLE_ARN_APP` repository (or `app` environment) secret. Use output `GitHubDeployRoleArn`, or this pattern:

`<YOUR_GITHUB_DEPLOY_ROLE_ARN>`

## Deploy flow (day-to-day)

1. Edit files in `prod/`
2. Merge to `main`
3. GitHub Action deploys automatically

## Validation checklist

- `https://mnemospark.ai` loads production page
- `https://www.mnemospark.ai` 301 redirects to apex
- favicon + OG image load from production domain
- CloudFront invalidation succeeds in workflow logs
