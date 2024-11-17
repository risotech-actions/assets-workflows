# Assets Factory Workflows Test Cases

This document details all test cases covered by the Assets Factory Workflows.

## Overview

The workflows handle three main scenarios:

1. Creating new NPM packages from level 1 branches
2. Managing beta packages during feature development
3. Automating version updates when features are merged

## Test Cases by Workflow

### 1. Create Product Workflow

Handles the creation of new packages from level 1 branches.

#### Branch Validation Tests

| Test Case | Input | Expected Result |
|-----------|-------|-----------------|
| Level 1 branch | `product-name` | ✅ Proceed with package creation |
| Sub-branch | `product/feature` | ⏭️ Skip with sub-branch message |
| Feature branch | `product-features/name` | ⏭️ Skip with feature branch message |
| Main/Master branch | `main`, `master` | ⏭️ Skip with protected branch message |

#### Package Name Formatting Tests

| Input | Expected Package Name |
|-------|---------------------|
| `Product-Name` | `product-name` |
| `product.name` | `product-name` |
| `product--name` | `product-name` |
| `-product-name-` | `product-name` |
| `Product_Name!` | `product-name` |

#### Version Generation Tests

| Scenario | Input Date | Expected Version |
|----------|------------|------------------|
| New package | 2024-03-20 | `2024.0320.0` |
| Same day, different package | 2024-03-20 | `2024.0320.1` |
| Next day package | 2024-03-21 | `2024.0321.0` |

### 2. Create Beta Workflow

Manages beta package creation for feature branches.

#### Branch Pattern Tests

| Test Case | Branch Name | Target | Expected Result |
|-----------|-------------|--------|-----------------|
| Valid feature branch | `product-features/login` | `product` | ✅ Create beta |
| Wrong target | `product-features/login` | `main` | ⏭️ Skip with wrong target message |
| Invalid pattern | `feature/login` | `product` | ⏭️ Skip with invalid pattern message |
| Protected target | `main-features/test` | `main` | ⏭️ Skip with protected branch message |

#### Beta Version Sequence Tests

| Scenario | Current Version | PR # | Push # | Expected Beta Version |
|----------|----------------|------|--------|---------------------|
| First push | `2024.0320.0` | 42 | 1 | `2024.0320.0-beta.42.1` |
| Second push | `2024.0320.0` | 42 | 2 | `2024.0320.0-beta.42.2` |
| New day, first push | `2024.0321.0` | 42 | 1 | `2024.0321.0-beta.42.1` |

#### PR Communication Tests

| Event | Expected Action |
|-------|----------------|
| PR opened | Create comment with installation instructions |
| New beta published | Update comment with new version |
| PR closed | Remove comment |

### 3. Merge Handler Workflow

Manages version updates and cleanup after PR merges.

#### Merge Validation Tests

| Test Case | Source Branch | Target Branch | Expected Result |
|-----------|--------------|---------------|-----------------|
| Valid merge | `product-features/login` | `product` | ✅ Process merge |
| Wrong target | `product-features/login` | `main` | ⏭️ Skip with wrong target message |
| Invalid branch | `feature/login` | `product` | ⏭️ Skip with invalid branch message |
| Non-feature branch | `hotfix/bug` | `product` | ⏭️ Skip with non-feature message |

#### Version Update Tests

| Current Version | Merge Date | Last Update | Expected Version |
|----------------|------------|-------------|------------------|
| `2024.0320.0` | 2024-03-20 | 2024-03-20 | `2024.0320.1` |
| `2024.0320.1` | 2024-03-20 | 2024-03-20 | `2024.0320.2` |
| `2024.0320.2` | 2024-03-21 | 2024-03-20 | `2024.0321.0` |

#### Beta Cleanup Tests

| Scenario | Expected Cleanup |
|----------|-----------------|
| PR with multiple betas | Remove all `*-beta.PR_NUMBER.*` versions |
| No beta versions | Skip cleanup |
| Some versions deleted | Continue with remaining |

## Error Handling

### Common Error Scenarios

1. **Package Registry Access**
   - Missing credentials
   - Invalid token scope
   - Network issues
   - Rate limiting

2. **Git Operations**
   - Concurrent updates
   - Push permission issues
   - Tag conflicts
   - Branch protection violations

3. **Package Operations**
   - Build failures
   - Version conflicts
   - Dependency issues
   - Registry timeouts

### Expected Error Responses

| Error Type | Expected Behavior |
|------------|------------------|
| Permission denied | Clear error message, link to permission docs |
| Network failure | Retry mechanism, timeout after 3 attempts |
| Build failure | Preserve logs, show debug information |
| Version conflict | Show conflicting versions, suggest resolution |

# Output Format Tests

### Summary Output Tests

Each workflow should provide clear summaries for:

1. **Validation Results**

```markdown
## ✅ Validation Passed
- Branch: `feature-branch`
- Target: `parent-branch`
- Type: `feature`
```

or

```markdown
## ⏭️ Process Skipped
- Reason: `Invalid branch pattern`
- Details: `Expected pattern...`
```

2. **Process Results**

```markdown
## 📦 Package Published
- Name: `@org/package`
- Version: `2024.0320.0`
- Type: `beta`
```

or

```markdown
## ❌ Process Failed
- Step: `publish`
- Error: `Registry unreachable`
```

3. **Installation Instructions**

```markdown
## Installation
$ npm install @org/package@version
```

# Integration Tests

### Cross-workflow Scenarios

1. **Full Feature Lifecycle**

```
Create Product → Create Betas → Merge → Cleanup
```

2. **Multiple Features**

```
Feature 1 (Beta) ----→ Merge
                   ↗
Feature 2 (Beta) --→ Merge
```

3. **Concurrent Updates**

```
Feature 1 Push →→ Version Update
Feature 2 Push →→ Wait → Update
```

### Version Management Integration

| Action | Previous State | New State |
|--------|---------------|-----------|
| Create Product | Empty | `1.0.0` |
| First Beta | `1.0.0` | `1.0.0-beta.1.1` |
| Merge Feature | `1.0.0` | `1.0.1` |
| Same-day Merge | `1.0.1` | `1.0.2` |

# Performance Tests

### Expected Timing Benchmarks

| Operation | Expected Time |
|-----------|--------------|
| Branch validation | < 5s |
| Package creation | < 30s |
| Beta publish | < 45s |
| Version update | < 20s |
| Full PR cycle | < 2m |

### Resource Usage Limits

| Resource | Maximum Usage |
|----------|--------------|
| CPU | 2 cores |
| Memory | 4GB |
| Disk | 1GB |
| Network | 100Mbps |

# Security Tests

### Permission Validation

| Action | Required Permissions |
|--------|---------------------|
| Create package | `packages:write` |
| Update version | `contents:write` |
| Manage PR | `pull-requests:write` |
| Delete package | `packages:delete` |

### Secret Management

| Secret | Usage |
|--------|-------|
| `GITHUB_TOKEN` | Auth for all operations |
| `NPM_TOKEN` | Package registry access |
| `DEPLOY_KEY` | Repository access |

# Recovery Tests

### Failure Recovery Scenarios

1. **Interrupted Publish**
   - Detect incomplete publish
   - Clean up partial artifacts
   - Retry from last successful step

2. **Conflicting Updates**
   - Detect version conflict
   - Auto-increment version
   - Update references

3. **Cleanup Failures**
   - List failed deletions
   - Retry individual items
   - Report unrecoverable items

### State Recovery Tests

| Failure Point | Recovery Action |
|---------------|-----------------|
| Pre-publish | Reset working directory |
| Mid-publish | Remove partial package |
| Post-publish | Verify package integrity |
| During cleanup | Continue with remaining |

# Documentation Tests

### Required Documentation Elements

1. **Installation Instructions**
   - Package name
   - Version
   - Registry configuration
   - Dependencies

2. **Error Messages**
   - Clear description
   - Cause
   - Resolution steps
   - Support links

3. **Process Summaries**
   - Steps completed
   - Resources created
   - Next actions
   - Reference links
