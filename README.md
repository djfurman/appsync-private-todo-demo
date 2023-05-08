# Todo Demo

This demo explores the structure of internal-use GraphQL APIs using the new Private APIs on AWS AppSync.

Note; this repo and discussion are for demonstration purposes only. A production setup will require appropriate lockdown and should be integrated with your existing deployments.

## Architecture

This new private API capability on AppSync allows us to create a private visibility endpoint. In this repo, we'll explore the following flow.

```mermaid
flowchart LR
    client[GraphQL Client]-->lb[Load Balancer]
    lb-->vpce[PrivateLink Vpce]
    vpce-->appsync[AWS AppSync]
    appsync-->resolver[Data Resolver]
    resolver-->producer[Producer]
```

### Diving Deeper

For our use case we will have two VPCs setup and for convenience will use the same region. Multi-regional availability is beyond this scope.

```mermaid
sequenceDiagram
    box Purple Client VPC
    actor client as Client
    end
    box Orange GraphQL Service VPC
    participant vpce as VPC Endpoint
    participant appsync as Private GraphQL API
    participant resolver as Resolver
    participant producer as Data Store
    end
    client->>vpce: GraphQL Request
    vpce->>appsync: VPC only Route to AppSync
    appsync-->resolver: Invoke correct resolver(s)
    resolver-->producer: Process data
    producer-->resolver: Data response
    resolver-->appsync: Resolver response
    appsync->>vpce: Private API response
    vpce->>client: GraphQL response
```

This allows us to simulate an internal-use AppSync API to support our todos GraphQL API.

## Tasks

- [ ] Setup Client VPC via CloudFormation
- [ ] Setup GraphQL VPC via CloudFormation
  - [ ] Setup VPCe for GraphQL VPC
  - [ ] Set Policies for Least-privilege Access
- [ ] [Service Control Policy (SCP)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) to block any public API creation
  - [x] Build [SCP Enforcement Policy](./iam/private-appsync-only-scp.json)
- [ ] Setup Todo Private API on AppSync
  - [ ] Setup Schema
  - [ ] Setup Private Visibility AppSync API via CloudFormation
  - [ ] Setup DynamoDB as data store
  - [ ] Setup IAM Execution Role for AppSync
    - [ ] Allow DynamoDB access directly from AppSync
  - [ ] Configure AppSync Schema
  - [ ] Configure AppSync resolver for DynamoDB
- [ ] Setup Client Calling Application

## References

1. [Introducing Private APIs on AWS AppSync](https://aws.amazon.com/blogs/mobile/introducing-private-apis-on-aws-appsync/)
1. [Using IAM policies to limit public API creation](https://docs.aws.amazon.com/appsync/latest/devguide/using-private-apis.html#blocking-public-apis)
