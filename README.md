# api-platform-090-custom-policies

API Platform 0.9.0 Gateway with Custom Policies

## Prerequisites

1. Download the Gateway ZIP from the [Gateway v0.9.0 release](https://github.com/wso2/api-platform/releases/tag/gateway/v0.9.0).
2. Download the ap CLI from the [ap CLI v0.6.0 release](https://github.com/wso2/api-platform/releases/tag/ap/v0.6.0).
3. Unzip the Gateway ZIP and CLI ZIP files.
4. Add the `ap` CLI to the `PATH` variable:
   ```bash
   export PATH=./ap-darwin-arm64-v0.6.0:$PATH
   ```

## Sample Policy: transform-payload-case

A policy is a Go module that implements the gateway policy interface. Refer to the Policy Hub policy implementations: [gateway-controllers/policies](https://github.com/wso2/gateway-controllers/tree/main/policies)

```go
// Policy is the base interface that all policies must implement
type Policy interface {

    // Mode returns the policy's processing mode for each phase
    // Used by the kernel to optimize execution (e.g., skip body buffering if not needed)
    Mode() ProcessingMode

    // OnRequest executes the policy during request phase
    // Called with request context including headers and body (if body mode is BUFFER)
    // Returns RequestAction with modifications or immediate response
    // Returns nil if policy has no action (pass-through)
    OnRequest(ctx *RequestContext, params map[string]interface{}) RequestAction

    // OnResponse executes the policy during response phase
    // Called with response context including headers and body (if body mode is BUFFER)
    // Returns ResponseAction with modifications
    // Returns nil if policy has no action (pass-through)
    OnResponse(ctx *ResponseContext, params map[string]interface{}) ResponseAction
}
```

This policy intercepts the incoming request body and transforms its text to uppercase or lowercase before forwarding it to the backend service. It accepts a `targetCase` parameter with two options:

- `uppercase` (default) — converts the entire request body to UPPERCASE
- `lowercase` — converts the entire request body to lowercase

## Build the Gateway

### 1. Update `build.yaml`

The `build.yaml` file defines the gateway version and the policies to include. Add your custom policy using `filePath` to reference the local policy module:

```yaml
  # Custom Policy: Transform Payload Case
  - name: transform-payload-case
    filePath: ../transform-payload-case
```

The file also contains the default policies from the Policy Hub. You can remove any default policies you don't need.

### 2. Build the Gateway Image

```bash
ap gateway image build
```

This will build the following Docker images:

- `ghcr.io/wso2/api-platform/hr-gateway-gateway-controller:0.9.0`
- `ghcr.io/wso2/api-platform/hr-gateway-gateway-runtime:0.9.0`

### 3. Update `docker-compose.yaml`

Update the images in `hr-gateway-v0.9.0/docker-compose.yaml` to use the built images:

```yaml
gateway-controller:
  image: ghcr.io/wso2/api-platform/hr-gateway-gateway-controller:0.9.0

gateway-runtime:
  image: ghcr.io/wso2/api-platform/hr-gateway-gateway-runtime:0.9.0
```

### 4. Start the Gateway

```bash
docker compose up -d
```

### 5. Deploy the API

```bash
ap api deploy --file users-api.yaml
```

