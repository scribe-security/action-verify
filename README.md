---
sidebar_label: "Verify"
title: Scribe GitHub Action for `valint verify`
sidebar_position: 3
toc_min_heading_level: 2
toc_max_heading_level: 5
---

Scribe offers GitHub Actions for embedding evidence collecting and validated integrity of your supply chain.

Use `valint verify` to verify evidence (attestations) and policies.

Further documentation **[GitHub integration](../../../integrating-scribe/ci-integrations/github)**.

### Verify Action
The command allows users to verify any target against its evidence.
- Verify image, directory, file or git targets.
- Verify evidence policy compliance across the supply chain.
- Pull evidence from scribe service.
- Download and search evidence in all enabled stores.
- Support Sigstore keyless verifying as well as **[GitHub workload identity](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)**.

### Input arguments
```yaml
  target:
    description: Target object name format=[<image:tag>, <dir path>, <git url>] (Optional)
    required: true
  all-evidence:
    description: Run all evidence verification
  attest-config:
    description: Attestation config path
  attest-default:
    description: Attestation default config, options=[sigstore sigstore-github x509 x509-env kms pubkey]
  attestation:
    description: Attestation for target
  base-image:
    description: Base image for the target
  beautify:
    description: Enhance the output using ANSI and Unicode characters
  bom:
    description: Create target SBOM evidence
  bundle:
    description: Policy bundle uri/path (early-availability)
  bundle-auth:
    description: 'Bundle repository authentication info, [format: ''username:password'']'
  bundle-branch:
    description: Bundle branch in the repository
  bundle-commit:
    description: Bundle commit hash in the repository
  bundle-depth:
    description: Bundle clone depth
  bundle-tag:
    description: Bundle tag in the repository
  ca:
    description: x509 CA Chain path
  cert:
    description: x509 Cert path
  common-name:
    description: Default policy allowed common names
  crl:
    description: x509 CRL path
  crl-full-chain:
    description: Enable Full chain CRL verfication
  depth:
    description: Git clone depth
  disable-crl:
    description: Disable certificate revocation verificatoin
  email:
    description: Default policy allowed emails
  force:
    description: Force skip cache
  format:
    description: Policy Result Evidence format, options=[statement-sarif attest-sarif sarif ]
  git-auth:
    description: 'Git repository authentication info, [format: ''username:password'']'
  git-branch:
    description: Git branch in the repository
  git-commit:
    description: Git commit hash in the repository
  git-tag:
    description: Git tag in the repository
  initiative:
    description: Initiative configuration file path (early-availability)
  initiative-id:
    description: Initiative id
  initiative-name:
    description: Initiative name
  input-format:
    description: Input Evidence format, options=[attest-cyclonedx-json attest-slsa statement-slsa statement-cyclonedx-json statement-generic attest-generic ]
  key:
    description: x509 Private key path
  kms:
    description: Provide KMS key reference
  md:
    description: Output Initiative result markdown report file
  oci:
    description: Enable OCI store
  oci-repo:
    description: Select OCI custom attestation repo
  pass:
    description: Private key password
  payload:
    description: path of the decoded payload
  platform:
    description: Select target platform, examples=windows/armv6, arm64 ..)
  provenance:
    description: Create target SLSA Provenance evidence
  pubkey:
    description: Public key path
  public-key:
    description: Public key path
  rule:
    description: Rule configuration file path (early-availability)
  rule-args:
    description: Policy arguments
  rule-label:
    description: Run only rules with specified label
  skip-bundle:
    description: Skip bundle download
  skip-confirmation:
    description: Skip Sigstore Confirmation
  skip-report:
    description: Skip Policy report stage
  uri:
    description: Default policy allowed uris
  cache-enable:
    description: Enable local cache
  config:
    description: Configuration file path
  deliverable:
    description: Mark as deliverable, options=[true, false]
  env:
    description: Environment keys to include in evidence
  gate-name:
    description: Policy Gate name
  gate-type:
    description: Policy Gate type
  input:
    description: Input Evidence target, format (\<parser>:\<file> or \<scheme>:\<name>:\<tag>)
  label:
    description: Add Custom labels
  level:
    description: Log depth level, options=[panic fatal error warning info debug trace]
  log-context:
    description: Attach context to all logs
  log-file:
    description: Output log to file
  output-directory:
    description: Output directory path
    default: ./scribe/valint
  output-file:
    description: Output file name
  pipeline-name:
    description: Pipeline name
  predicate-type:
    description: Custom Predicate type (generic evidence format)
  product-key:
    description: Product Key
  product-version:
    description: Product Version
  scribe-client-id:
    description: Scribe Client ID (deprecated)
  scribe-client-secret:
    description: Scribe Client Token
  scribe-disable:
    description: Disable scribe client
  scribe-enable:
    description: Enable scribe client (deprecated)
  scribe-url:
    description: Scribe API Url
  structured:
    description: Enable structured logger
  timeout:
    description: Timeout duration
  verbose:
    description: Log verbosity level [-v,--verbose=1] = info, [-vv,--verbose=2] = debug

```

### Usage
Containerized action can be used on Linux runners as following
```yaml
- name: valint verify
  id: valint_verify
  uses: scribe-security/action-verify@v2.0.4
  with:
      target: 'busybox:latest'
```

Composite Action can be used on Linux or Windows runners as following
```yaml
- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-verify-cli@v2.0.4
  with:
    target: 'hello-world:latest'
```

> Use `master` instead of tag to automatically pull latest version.


### 1. Obtain a Scribe Hub API Token
1. Sign in to [Scribe Hub](https://app.scribesecurity.com). If you don't have an account you can sign up for free [here](https://scribesecurity.com/scribe-platform-lp/ "Start Using Scribe For Free").

2. Create a API token in [Scribe Hub > Settings > Tokens](https://app.scribesecurity.com/settings/tokens). Copy it to a safe temporary notepad until you complete the integration.

:::note Important
The token is a secret and will not be accessible from the UI after you finalize the token generation. 
:::

### 2. Add the API token to GitLab secrets

Set your Scribe Hub API token in Github with a key named SCRIBE_TOKEN as instructed in *GitHub instructions](https://docs.github.com/en/actions/security-guides/encrypted-secrets/ "GitHub Instructions")

### 3. Instrument your build scripts

#### Usage

```yaml
name:  scribe_github_workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-sign-verify
    runs-on: ubuntu-latest
    steps:
      
      # - uses: scribe-security/action-evidence@master
      # - uses: scribe-security/action-slsa@master
      - uses: scribe-security/action-bom@master
        with:
          target: [target]
          format: [attest, statement]
          scribe-client-secret: ${{ secrets.SCRIBE_TOKEN }}

      - uses: scribe-security/action-verify@master
        with:
          target: [target]
          input-format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          scribe-client-secret: ${{ secrets.SCRIBE_TOKEN }}
```

#### Example: Enforcing SP 800-190 Controls with Valint Initiatives

This workflow demonstrates how to leverage Valint’s initiative engine to automatically generate evidence (SBOM and vulnerability reports), submit it to Valint, and enforce SP 800-190 controls on your container image.

```yaml
name: sp-800-190-policy-check

on:
  pull_request:

jobs:
  image-policy-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          file: Dockerfile
          push: false
          tags: |
            ${{ github.sha }}

      - name: Scan image with Trivy
        uses: aquasecurity/trivy-action@0.28.0
        continue-on-error: true
        with:
          image-ref: ${{ github.sha }}
          format: sarif
          output: trivy-report.sarif
          vuln-type: os,library
          severity: CRITICAL,HIGH
          ignore-unfixed: true
          exit-code: 0

      - name: Collect Evidence & Evaluate SP 800-190 Initiative
        uses: scribe-security/action-verify@main
        with:
          initiative: sp-800-190@v2
          target: ${{ github.sha }}
          bom: true                   # Generate CycloneDX SBOM
          base-image: Dockerfile      # Include base image in SBOM
          input: sarif:trivy-report.sarif
          input-format: attest 
          beautify: true
```

> **Note:** Enabling `bom`, `provenance`, or `input` flags ensures Valint generates and ingests the necessary evidence before policy evaluation.


### Configuration
If you prefer using a custom configuration file instead of specifying arguments directly, you have two choices. You can either place the configuration file in the default path, which is `.valint.yaml`, or you can specify a custom path using the `config` argument.

For a comprehensive overview of the configuration file's structure and available options, please refer to the CLI configuration documentation.


### Attestations 
Attestations allow you to sign and verify your targets. <br />
Attestations allow you to connect PKI-based identities to your evidence and policy management.  <br />

Supported outputs:
- In-toto predicate - CycloneDX SBOM, SLSA Provenance (unsigned evidence)
- In-toto statements - CycloneDX SBOM, SLSA Provenance (unsigned evidence)
- In-toto attestations -CycloneDX SBOM, SLSA Provenance (signed evidence)

Select default configuration using `--attest.default` flag. <br />
Select a custom configuration by providing `cocosign` field in the **[configuration](../configuration)** or custom path using `--attest.config`.
Scribe uses the **cocosign** library we developed to deal with digital signatures for signing and verification.

* See details of in-toto spec **[here](https://github.com/in-toto/attestation)**.
* See details of what attestations are and how to use them **[here](../attestations)**.

### Storing Keys in Secret Vault

GitHub exposes secrets from its vault using environment variables, you may provide these environments as secret to Valint.

> Paths names prefixed with `env://[NAME]` are read from the environment matching the name.

<details>
  <summary> GitHub Secret Vault </summary>

X509 Signer enables the utilization of environments for supplying key, certificate, and CA files in order to sign and verify attestations. It is commonly employed in conjunction with Secret Vaults, where secrets are exposed through environments.

>  path names prefixed with `env://[NAME]` are extracted from the environment corresponding to the specified name.


For example the following configuration and Job.

configuration File, `.valint.yaml`
```yaml
attest:
  default: "" # Set custom configuration
  cocosign:
    signer:
        x509:
            enable: true
            private: env://SIGNER_KEY
            cert: env://SIGNER_CERT
            ca: env://COMPANY_CA
    verifier:
        x509:
            enable: true
            cert: env://SIGNER_CERT
            ca: env://COMPANY_CA
```
Job example
```yaml
name:  github_vault_workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-sign-verify
    runs-on: ubuntu-latest
    steps:
        uses: scribe-security/action-bom@master
        with:
          target: busybox:latest
          format: attest
        env:
          SIGNER_KEY: ${{ secrets.SIGNER_KEY }}
          SIGNER_CERT: ${{ secrets.SIGNER_CERT }}
          COMPANY_CA:  ${{ secrets.COMPANY_CA }}

        uses: scribe-security/action-verify@master
        with:
          target: busybox:latest
          input-format: attest
        env:
          SIGNER_CERT: ${{ secrets.SIGNER_CERT }}
          COMPANY_CA:  ${{ secrets.COMPANY_CA }}
```
</details>

### Running action as non root user
By default, the action runs in its own pid namespace as the root user. You can change the user by setting specific `USERID` and `USERNAME` environment variables.

```YAML
- name: Verify image
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
    format: json
  env:
    USERID: 1001
    USERNAME: runner
``` 

<details>
  <summary> Non root user with HIGH UID/GID </summary>
By default, the action runs in its own pid namespace as the root user. If the user uses a high UID or GID, you must specify all the following environment variables. You can change the user by setting specific `USERID` and `USERNAME` variables. Additionally, you may group the process by setting specific `GROUPID` and `GROUP` variables.

```YAML
- name: Verify image
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
    format: json
  env:
    USERID: 888000888
    USERNAME: my_user
    GROUPID: 777000777
    GROUP: my_group
``` 

</details>
``` 

### Platform-Specific Image Handling
The Valint tool is compatible with both Linux and Windows images. Set the desired platform using the 'platform' field in your configuration:

```yaml
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: hello-world:latest
    platform: linux/amd64
```

Docker is configured by default to pull images matching the runner's platform. For analyzing images across different platforms, you need to pull the image from the registry and specify the platform.

```yaml
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: registry:hello-world:latest
    platform: windows/amd64
```

### Windows Runner Compatibility
> On Windows Github runners, containerized actions are currently not supported. It's recommended to use CLI actions in such cases.

```yaml
- name: valint verify
  uses: scribe-security/action-verify-cli@master
  with:
    target: hello-world:latest
```

### Verify SBOMs examples
<details>
  <summary> Verify target (SBOM) </summary>

Verify targets against a signed attestation.  
Default attestation config: `sigstore-github` - Sigstore (Fulcio, Rekor).  
Valint will look for either an SBOM or SLSA attestation to verify against.  

```yaml
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
``` 

</details>

<details>
  <summary> Verify target (SLSA) </summary>

Verify targets against a signed attestation. <br />
Default attestation config: `sigstore-github` - Sigstore (Fulcio, Rekor). <br />
Valint will look for either an SBOM or SLSA attestation to verify against. <br />


```yaml
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
    input-format: attest-slsa
``` 

</details>

<details>
  <summary> Verify target (Generic) </summary>

Verify targets against a signed attestation. <br />
Default attestation config: `sigstore-github` - Sigstore (Fulcio, Rekor). <br />
Valint will look for either an SBOM or SLSA attestation to verify against. <br />


```yaml
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
    input-format: attest-generic
``` 

</details>

<details>
  <summary> Verify Policy flow - image target (Signed SBOM) </summary>

Full job example of a image signing and verifying flow.

```YAML
 valint-busybox-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: valint attest
        id: valint_attest
        uses: scribe-security/action-bom@master
        with:
           target: 'busybox:latest'
           format: attest
           force: true

      - name: valint verify
        id: valint_verify
        uses: scribe-security/action-verify@master
        with:
           target: 'busybox:latest'

      - uses: actions/upload-artifact@v4
        with:
          name: valint-busybox-test
          path: scribe/valint
``` 

</details>

<details>
  <summary> Verify Policy flow - image target (Signed SLSA) </summary>

Full job example of a image signing and verifying flow.

```YAML
 valint-busybox-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: valint attest slsa
        id: valint_attest
        uses: scribe-security/action-bom@master
        with:
           target: 'busybox:latest'
           format: attest-slsa
           force: true

      - name: valint verify attest slsa
        id: valint_verify
        uses: scribe-security/action-verify@master
        with:
           target: 'busybox:latest'
           input-format: attest-slsa

      - uses: actions/upload-artifact@v4
        with:
          name: valint-busybox-test
          path: scribe/valint
``` 

</details>

<details>
  <summary> Verify Policy flow - Directory target (Signed SBOM) </summary>

Full job example of a directory signing and verifying flow.

```YAML
  valint-dir-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: valint attest workdir
        id: valint_attest_dir
        uses: scribe-security/action-bom@master
        with:
           type: dir
           target: '/GitHub/workspace/'
           format: attest
           force: true

      - name: valint verify workdir
        id: valint_verify_dir
        uses: scribe-security/action-verify@master
        with:
           type: dir
           target: '/GitHub/workspace/'
      
      - uses: actions/upload-artifact@v4
        with:
          name: valint-workdir-evidence
          path: |
            scribe/valint      
``` 

</details>


<details>
  <summary> Verify Policy flow - Git repository target (Signed SBOM) </summary>

Full job example of a git repository signing and verifying flow.
> Support for both local (path) and remote git (URL) repositories.

```YAML
  valint-dir-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: valint attest local repo
        id: valint_attest_dir
        uses: scribe-security/action-bom@master
        with:
           type: git
           target: '/GitHub/workspace/my_repo'
           format: attest
           force: true

      - name: valint verify local repo
        id: valint_verify_dir
        uses: scribe-security/action-verify@master
        with:
           type: git
           target: '/GitHub/workspace/my_repo'
      
      - uses: actions/upload-artifact@v4
        with:
          name: valint-git-evidence
          path: |
            scribe/valint      
``` 
</details>

<details>
  <summary> Attest and verify evidence on OCI (SBOM, SLSA) </summary>

Store any evidence on any OCI registry. <br />
Support storage for all targets and both SBOM and SLSA evidence formats.

> Use input variable `format` to select between supported formats. <br />
> Write permission to `oci-repo` is required. 

```YAML
valint-dir-test:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
  env:
    DOCKER_CONFIG: $HOME/.docker
  steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY_URL }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_TOKEN }}

      - uses: scribe-security/action-bom@master
        id: valint_attest
        with:
          target: busybox:latest
          force: true
          format: attest
          oci: true
          oci-repo: ${{ env.REGISTRY_URL }}/attestations    
``` 

Following actions can be used to verify a target over the OCI store.
```yaml
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY_URL }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_TOKEN }}

      - uses: scribe-security/action-verify@master
        id: valint_attest
        with:
          target: busybox:latest
          input-format: attest
          oci: true
          oci-repo: ${{ env.REGISTRY_URL }}/attestations   
```
> Read permission to `oci-repo` is required. 

</details>

<details>
  <summary> Install Valint (tool) </summary>

Install Valint as a tool:
```yaml
- name: install valint
  uses: scribe-security/action-installer@master

- name: valint run
  run: |
    valint --version
    valint bom busybox:latest
``` 
</details>


### Alternative evidence stores

> You can learn more about alternative stores **[here](https://scribe-security.netlify.app/docs/integrating-scribe/other-evidence-stores)**.

<details>
  <summary> <b> OCI Evidence store </b></summary>
Valint supports both storage and verification flows for `attestations` and `statement` objects utilizing OCI registry as an evidence store.

Using OCI registry as an evidence store allows you to upload, download and verify evidence across your supply chain in a seamless manner.

Related flags:
* `oci` Enable OCI store.
* `oci-repo` - Evidence store location.

### Before you begin
Evidence can be stored in any accusable registry.
* Write access is required for upload (generate).
* Read access is required for download (verify).

You must first login with the required access privileges to your registry before calling Valint.
For example, using `docker login` command or `docker/login-action` action.

### Usage
```yaml
name:  scribe_github_workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-sign-verify
    runs-on: ubuntu-latest
    steps:

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.my_registry }}
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name:  Generate evidence step
      
      # uses: scribe-security/action-evidence@master
        # uses: scribe-security/action-slsa@master
      - uses: scribe-security/action-bom@master
        with:
          target: [target]
          format: [attest, statement]
          oci: true
          oci-repo: [oci_repo]

      - name:  Verify policy step
        uses: scribe-security/action-verify@master
        with:
          target: [target]
          input-format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          oci: true
          oci-repo: [oci_repo]
```
</details>

### .gitignore
It's recommended to add an output directory value to your .gitignore file.
By default add `**/scribe` to your `.gitignore`.

## Other Actions
* [bom](action-bom.md), [source](https://github.com/scribe-security/action-bom)
* [slsa](action-slsa.md), [source](https://github.com/scribe-security/action-slsa)
* [evidence](action-evidence.md), [source](https://github.com/scribe-security/action-evidence)
* [verify](action-verify.md), [source](https://github.com/scribe-security/action-verify)
* [installer](action-installer.md), [source](https://github.com/scribe-security/action-installer)
