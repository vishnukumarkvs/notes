aws sdk
=======

  Identifying SDK Components
  Here is the status of the AWS integration in the  coroot-cluster-agent , details on the deprecated SDK, and what a migration to the AWS SDK for Go v2 would entail.
  ──────
  ### 1. AWS Code in Coroot Cluster Agent

  The agent gathers cloud infrastructure metrics for Amazon Web Services (AWS) managed databases. Specifically, the AWS discovery code resides in  and performs the following tasks:

  • Discovery: It lists database instances via RDS ( DescribeDBInstances ) and caches/clusters via ElastiCache ( DescribeCacheClusters ).
  • OS/System Metrics: It retrieves detailed operating system metrics (CPU, Memory, I/O, Network) by pulling events from the  RDSOSMetrics  Log Group via CloudWatch Logs ( GetLogEvents ).
  • Database Engine Logs: It downloads database engine logs (e.g. Postgres engine log files) via the RDS file API ( DownloadDBLogFilePortion ) and parses them for warning/error patterns.
  ──────
  ### 2. Are we using the deprecated 1.0 SDK?

  Yes. The agent currently relies on  github.com/aws/aws-sdk-go  (v1.55.7), which is the AWS SDK for Go v1.
  AWS officially transitioned the AWS SDK for Go v1 into Maintenance Mode in July 2025 and is planning its End-of-Support. AWS strongly recommends that all active Go projects migrate to the AWS SDK for Go v2 ( github.com/aws/aws-sdk-go-v2 ).
  ──────
  ### 3. Can we migrate to SDK v2? Are there any breaking changes?

  Yes, we can migrate. However, doing so requires rewriting the Go code that interacts with AWS services, as SDK v2 introduces several compile-time breaking API changes:

  #### Breaking Changes & Refactoring Required:

  1. Session & Configuration:
      • In v1: The codebase uses  session.NewSession  to configure the credentials and connection (see aws.go).
      • In v2:  session  has been removed. You load configuration via:
        cfg, err := config.LoadDefaultConfig(ctx,
            config.WithRegion(awsConfig.Region),
            config.WithCredentialsProvider(credentials.NewStaticCredentialsProvider(...)),
        )

  2. Explicit Context Support ( context.Context ):
      • In v1: Standard API calls (e.g.  svc.DescribeDBInstances(input) ) do not require a context parameter.
      • In v2: Every API call strictly requires a  context.Context  as its first argument:
        output, err := client.DescribeDBInstances(ctx, input)
      This means the  discoverRDS() ,  discoverEC() , and  refresh()  goroutines in the agent will need to pass down valid contexts.
  3. Service Client Initialization:
      • In v1: Clients are created using  rds.New(sess) .
      • In v2: Clients are created using  rds.NewFromConfig(cfg) .
  4. Helper Function Name Changes:
      • In v1: The codebase uses  aws.StringValue(ptr)  or  aws.Int64Value(ptr)  to dereference values.
      • In v2: These helper functions are renamed to  aws.ToString(ptr) ,  aws.ToInt64(ptr) , etc.
  5. Error Handling:
      • In v1: The codebase uses  awserr.Error  to assert and extract errors (see aws.go).
      • In v2: Generic AWS errors are handled via  smithy.APIError  (using  github.com/aws/smithy-go ), or specific SDK-typed errors using standard Go  errors.As .