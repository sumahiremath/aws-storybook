---
description: "A useful test does not merely repeat the happy path. It challenges the contracts most likely to fail after deployment."
tags:
  - "aws"
  - "deployment"
  - "devops"
  - "concept"
---

# AWS Deployment Testing: The Test Kitchen

> A useful test does not merely repeat the happy path. It challenges the contracts most likely to fail after deployment.

## The Business Goal

Every unit test passed.

In staging, the first mobile order failed.

The payment provider returned a success response after the client's timeout. The client retried. The order function created two orders because the test suite had mocked the provider as instant and perfect.

Then a queued event arrived twice. A test event used the wrong field shape. The staging API worked at its raw execute URL but failed behind the production-style custom domain.

The code had been tested.

The system's boundaries had not.

---

## The Story

Marco stocked the test kitchen with bad days.

One drawer held representative JSON events:

- A valid API Gateway request
- A request missing a required field
- An SQS batch containing one poison message
- A duplicate order event
- An expired token
- A payment timeout followed by success

Another station ran fast unit tests. A payment mock made failure cases deterministic. A sandbox integration test exercised the provider's real contract. A deployed test called the development API stage and inspected logs and downstream state.

Amazon Q Developer suggested additional test cases around null input and exception paths. Marco reviewed the suggestions, corrected assumptions about the event shape, and kept the valuable cases.

The tool expanded the test kitchen.

It did not become the chef.

---

## The Wrong Way

Mocks are useful when they isolate behavior and reproduce failure. They are dangerous when they replace every real integration and quietly encode an invented contract.

Testing only the Lambda handler can miss:

- IAM permissions
- Event-source mapping and batch behavior
- API Gateway transformations and authorizers
- Environment configuration
- Network access
- Service quotas and throttling
- Serialization differences
- Retry and duplicate-delivery behavior

Testing only in production discovers real integration behavior at the highest possible cost.

---

## Meet the AWS Tools

AWS services provide several useful testing surfaces:

> **Core idea:** Use fast local tests for code, realistic test events for contracts, deployed environments for managed integrations, and automated promotion gates for repeatability.

- **AWS SAM CLI** can invoke Lambda functions and API routes locally.
- **Lambda** and **API Gateway** accept representative test events or development requests.
- **API Gateway stages** expose isolated deployed API configurations.
- **CodeBuild** can run unit and integration test commands and publish reports.
- **CloudFormation**, **SAM**, and **CDK** create disposable or repeatable test stacks.
- **Amazon Q Developer** can assist in generating tests that developers validate.

---

## How It Works

### Unit Tests

Unit tests isolate small logic and should be fast enough to run on every change.

Inject clients or adapters around external services so application logic can simulate:

- Success
- Timeout
- Throttling
- Permission denial
- Malformed response
- Partial batch failure

Test business idempotency explicitly. A retry-safe API call is not the same thing as retry-safe business behavior.

### Integration Tests

Integration tests cross a real boundary: a database, queue, managed API, or third-party sandbox.

They are slower and require cleanup, but they reveal contract and permission failures that mocks cannot.

Use unique test identifiers. Make cleanup safe and scoped. Avoid tests that depend on execution order or shared mutable state.

### Mock APIs

A mock server can reproduce rare or expensive external behavior. Define the mock from the provider's documented or observed contract.

Keep at least a smaller set of real integration tests so the mock cannot drift unnoticed.

### Test Events

Store test events as reviewed files. Include the exact envelope the service delivers, not only the inner business object.

Event-driven tests should cover:

- Duplicate delivery
- Out-of-order events where possible
- Batches with partial failure
- Retry exhaustion
- Poison messages
- Missing optional fields
- Old and new schema versions

For Lambda batch sources, verify the intended failure-reporting behavior. Retrying an entire batch can repeat successful work unless the handler and event-source configuration support partial results correctly.

### Development Endpoints

Deploy an API to a development stage and call it through the same integration path used by clients. Test authentication, request validation, mapping, CORS, throttling, status codes, and downstream permissions.

A stage's endpoint and configuration may differ from production. The difference should be deliberate and represented in the environment plan.

### Generated Tests

Amazon Q Developer can propose tests from code and context. Generated tests can expose overlooked branches and accelerate boilerplate.

Review:

- Whether assertions prove behavior rather than implementation detail
- Whether mocks match real contracts
- Whether sensitive data enters prompts or generated fixtures
- Whether the test can fail for the right reason
- Whether important distributed failure modes are missing

Generation assists coverage; ownership remains with the developer.

---

## Architectural Mapping

```text
commit
  |
unit tests ---- fast logic feedback
  |
build artifact
  |
test stack -> API stage -> real AWS integrations
     |              \
test events       mock / sandbox dependency
     |
reports + cleanup
```

The test role should have permissions only for the isolated environment. Production credentials do not belong in a test runner.

---

## When to Use It

Use layered tests when:

- The application calls managed services or third parties
- Events can retry, duplicate, batch, or arrive with evolving schemas
- IAM and environment configuration affect behavior
- A deployment strategy depends on trustworthy health checks
- The team needs fast feedback and realistic evidence

## When Not to Use It

Do not make every unit test call a live service. Do not make every integration test a brittle end-to-end journey. Use the narrowest test that proves the intended contract.

Do not accept generated tests solely because they compile.

---

## Painkiller

> **Problem:** Unit tests model every dependency as instant and perfect.  
> **Pain:** Staging reveals duplicates, timeouts, IAM failures, event envelopes, and endpoint differences too late.  
> **AWS solution:** Combine unit tests, contract-aware mocks, versioned test events, real integration tests, deployed development endpoints, and reviewed generated tests.

---

## Knife Cut

> **A mock proves your code handles the behavior you modeled. An integration test checks whether you modeled reality.**

---

## The Masthead

### What Actually Just Happened

|In the story|In testing|What it actually means|
|---|---|---|
|Ingredient test|Unit test|Fast isolated logic verification|
|Pretend supplier|Mock API|Controlled dependency behavior|
|Supplier delivery|Integration test|Real boundary and contract|
|Order-event drawer|JSON test events|Replayable service envelopes and edge cases|
|Practice counter|API development stage|Deployed endpoint for environment testing|
|Suggested recipes|Amazon Q Developer tests|Generated starting points requiring review|

---

## A Note From the Author

No test environment perfectly reproduces production traffic, data, quotas, and failure timing. The goal is not theatrical realism; it is useful evidence at progressively more expensive boundaries.

Test data and logs deserve the same privacy discipline as production. Do not copy sensitive customer records into fixtures casually.

- [Testing with AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-start-api.html)
- [Test your serverless application with AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-test-and-debug.html)

---

## The Last Bite

Release 4.7.2 passed the test kitchen.

Then production booted with the staging payment endpoint because the endpoint had been compiled into the artifact.

> **Testing the right artifact is useless if production changes the artifact to configure it.**

---

**Next chapter:** *[AWS AppConfig: The Same Release, Different Store](08-the-same-release-different-store.md)*

Imani separates code from environment and gives every surface—API, Lambda, frontend, container, and feature flag—a deliberate production identity.
