# CI/CD Workshop: SAST Integration with Snyk

## Practical 4: Security Scanning in GitHub Actions

### Executive Summary

This project demonstrates the integration of Static Application Security Testing (SAST) using Snyk within a CI/CD pipeline powered by GitHub Actions. The implementation achieves automated vulnerability detection for Java Maven projects with non-blocking security scans and centralized reporting.

### Objectives Achieved

1. **Automated Security Scanning**: Integrated Snyk Open Source scanner into GitHub Actions workflow
2. **CI/CD Pipeline Integration**: Implemented security scanning as part of automated build process
3. **SARIF Reporting**: Configured Security Alert Interchange Format (SARIF) upload to GitHub Security tab
4. **Non-blocking Workflow**: Ensured security scans report vulnerabilities without failing builds

### Technical Implementation

#### 1. Workflow Configuration

- **File**: `.github/workflows/maven.yml`
- **Trigger**: Push and pull requests to `main` branch
- **Jobs**:
  - `build and test`: Maven compilation and unit tests
  - `security`: Snyk vulnerability scanning

#### 2. Security Scanning Process

```yaml
- Snyk CLI installation and authentication
- Maven dependency analysis
- SARIF report generation
- GitHub Security tab upload
```

#### 3. Permissions Configuration

- `contents: read` - Repository access
- `security-events: write` - SARIF upload capability
- `actions: read` - Workflow metadata access

### Results

#### Vulnerability Detection

- **Total Alerts**: 9 vulnerabilities identified
- **Primary Issues**:
  - Stack-based Buffer Overflow in `org.yaml:snakeyaml@1.23`
  - Introduced via `com.github.javafaker:javafaker@1.0.2` dependency
- **Severity Distribution**: Low severity issues in YAML parsing library

#### CI/CD Performance

- **Build Duration**: ~1 minute 13 seconds
- **Test Job**: 20 seconds
- **Security Scan**: 46 seconds
- **Status**: ✅ All workflows passing

### Key Features

1. **Continuous Monitoring**: Every code push triggers security analysis
2. **Centralized Dashboard**: All vulnerabilities viewable in GitHub Security tab
3. **Developer-Friendly**: Non-blocking scans allow development to continue
4. **Actionable Reports**: Detailed remediation guidance for each vulnerability

### Technologies Used

- **CI/CD Platform**: GitHub Actions
- **Security Tool**: Snyk Open Source (v1.1301.0)
- **Build System**: Apache Maven 3.x
- **Runtime**: Java 17 (Temurin distribution)
- **Framework**: Spring Boot 3.4.11

### Learning Outcomes

- ✅ Understanding of SAST integration in CI/CD pipelines
- ✅ Practical experience with Snyk security scanner
- ✅ Knowledge of SARIF format and GitHub Security features
- ✅ Best practices for non-blocking security workflows

### Repository

- **GitHub**: [Wangpos/cicd-demo-swe302-p4a](https://github.com/Wangpos/cicd-demo-swe302-p4a)
- **Branch**: `main`
- **Workflow Status**: Active and operational

### Conclusion

Successfully implemented automated security scanning within a CI/CD pipeline, demonstrating industry best practices for DevSecOps. The system provides continuous vulnerability monitoring while maintaining development velocity through non-blocking workflow design.
