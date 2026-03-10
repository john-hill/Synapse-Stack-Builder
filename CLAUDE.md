# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Synapse Stack Builder is a Java 11 application that generates and deploys AWS CloudFormation stacks for the Synapse platform. It uses Apache Velocity templates to produce CloudFormation JSON and manages infrastructure including VPCs, Elastic Beanstalk environments, S3 buckets, CDN, NLB, data warehouses, and more.

## Build & Test Commands

```bash
# Build (creates fat JAR via maven-assembly-plugin)
mvn clean install

# Build without tests
mvn clean install -DskipTests

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=SubnetBuilderTest

# Run a single test method
mvn test -Dtest=SubnetBuilderTest#testBuild
```

CI runs: `mvn --batch-mode clean test`

## Architecture

### Dependency Injection
Google Guice wires everything together. `TemplateGuiceModule` is the central module that binds all builders, configuration, and AWS clients. Each infrastructure domain has a Builder interface and Impl class.

### Entry Points
Each infrastructure component has its own `*Main.java` class (e.g., `RepositoryBuilderMain`, `VpcBuilderMain`, `S3BuilderMain`). These bootstrap Guice and invoke the corresponding builder.

### Template Generation
Velocity templates in `src/main/resources/templates/` generate CloudFormation JSON. Builders populate Velocity contexts and merge them with `.vtp` template files. `CloudFormationClient` wraps the AWS SDK to create/update stacks.

### Key Package Layout (`org.sagebionetworks.template`)
- `repo/` — Repository stack (Elastic Beanstalk, queues, Kinesis, AppConfig, Athena, CloudWatch, alarms)
- `vpc/` — VPC and subnet configuration
- `s3/` — S3 bucket definitions
- `global/` — Global resources (Bedrock, knowledge bases)
- `cdn/` — CloudFront CDN and Web ACL
- `nlb/` — Network Load Balancer
- `datawarehouse/` — Data warehouse and backfill
- `ip/address/` — IP address pool management
- `config/` — Configuration loading from S3 and properties
- `jobs/` — Async admin job execution via Synapse client

### Configuration
- JSON config files in `src/main/resources/templates/` define infrastructure components (SNS/SQS topics, S3 buckets, Kinesis streams, etc.)
- `Constants.java` centralizes config file paths, CloudFormation parameter names, and stack naming conventions
- Runtime configuration is loaded from an S3 configuration bucket per environment

### AWS SDK Usage
The project uses both AWS SDK v1 (legacy S3, CloudFormation) and v2 (most services). AWS clients are provided via Guice `@Provides` methods in `TemplateGuiceModule`.

## Testing
- JUnit 5 (Jupiter) with Mockito
- Tests mirror the main source structure under `src/test/java/`
- Test resources (JSON fixtures, SQL files) in `src/test/resources/`

## Pre-commit Hooks
- `git-secrets` scans for hardcoded secrets
- `yamllint` validates YAML files
