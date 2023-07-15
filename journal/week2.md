# Week 2 — Distributed Tracing

## Required Tasks

I did Instrument the backend application to use Open Telemetry (OTEL) with Honeycomb as the provider
I also Instrumented AWS X-Ray into the backend application
I configured and provisioned X-Ray daemon within docker-compose and send data back to X-Ray API
And observed X-Ray traces within the AWS Console
I integrated Rollbar for error logging, triggered an error and observed the error with Rollbar
I did Install watchtower and wrote a custom logger to send application log data to cloudwatch log group

### Proof of Cloudwatch logs Implementation
![cloudwatch log image](./assets/weektwo/cloudwatch.png)

### Proof of AWS X-RAY Implementation
![x-ray image](./assets/weektwo/xray.png)

### Proof of Honeycomb Instrumentation
![honeycomb image](./assets/weektwo/honeycomb.png)

### Proof of Rollbar Instrumentation
![x-ray image](./assets/weektwo/rollbar-proof.png)


