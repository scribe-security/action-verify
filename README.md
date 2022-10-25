---
title: Bom
---
# Scribe GitHub actions - `gensbom bom`
Scribe offers GitHub actions for embedding evidence collecting and integrity verification to your workflows. \
Action 

## Other actions
* [bom - action](https://github.com/scribe-security/action-bom/README.md)
* [verify - action](https://github.com/scribe-security/action-verify/README.md)
* [installer - action](https://github.com/scribe-security/action-installer/README.md)

## Bom action
The action invokes a containerized `gensbom` sub-command `bom`. 
The command allows users to generate and manage SBOMs.
- GitHub-specific context attached to all SBOMs (GIT_URL, JOB_ID, JOB_NAME .. etc)
- Signing SBOMs, supporting Sigstore keyless flow while using GitHub's workload auth ODIC identity.
- Generates detailed SBOMs for images, directories, files and git repositories. 
- Store and manage SBOMs on Scribe service.
- Attach SBOM in your CI or releases.
- Generate SBOM directly from your private OCI registry support.
- Customizable SBOM with environments, labels, sections, etc.
- Attach external reports to your SBOM.
- Generate In-Toto attestation, statement or predicate for your SBOMs.

### Input arguments
```yaml
  type:
    description: 'Target source type options=[docker,docker-archive, oci-archive, dir, registry]'
    default: registry
  target:
    description: 'Target object name format=[<image:tag>, <dir_path>]'
    required: true
  verbose:
    description: 'Increase verbosity (-v = info, -vv = debug)'
    default: 1
  config:
    description: 'Application config file'
  format:
    description: 'Sbom formatter, options=[cyclonedx-json cyclonedx-xml attest-cyclonedx-json statement-cyclonedx-json predicate-cyclonedx-json attest-slsa statement-slsa predicate-slsa]'
    default: cyclonedxjson
  output-directory:
    description: 'Report output directory'
    default: ./scribe/gensbom
  output-file:
    description: 'Output result to file'
  name:
    description: 'Custom/project name'
  label:
    description: 'Custom label'
  env:
    description: 'Custom env'
  filter-regex:
    description: 'Filter out files by regex'
    default: .*\.pyc,\.git/.*
  collect-regex:
    description: 'Collect files content by regex'
  force:
    description: 'Force overwrite cache'
    default: false
  attest-config:
    description: 'Attestation config map'
  attest-name:
    description: 'Attestation config name (default "gensbom")'
  attest-default:
    description: 'Attestation default config, options=[sigstore sigstore-github x509]'
    default: sigstore-github
  scribe-enable:
    description: 'Enable scribe client'
    default: false
  scribe-client-id:
    description: 'Scribe client id' 
  scribe-client-secret:
    description: 'Scribe access token' 
  scribe-url:
    description: 'Scribe url' 
  context-dir:
    description: 'Context dir' 
```

### Output arguments
```yaml
  output-file:
    description: 'Bom output file path'
```

### Usage
```
- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    target: 'busybox:latest'
    verbose: 2
```

## Configuration
Use default configuration path `.gensbom.yaml`, or
provide a custom path using `config` input argument.
See detailed [documentation -](docs/configuration.md) config](docs/configuration.md)

You may add a `.cocosign.yaml` file to your repository or pass it with `--`config` \
for more [Cocosign configuration](https://github.com/scribe-security/cocosign)


## Attestations 
Attestations SBOMs allows you to sign and verify your SBOM targets. \
Attestations allow you to connect PKI-based identities to your evidence and policy management. 
Supported outputs:
- In-toto statements - cyclonedx BOM, SLSA Provenance
- In-toto predicate - cyclonedx, BOM, SLSA Provenance
- In-toto attestations -cyclonedx, BOM, SLSA Provenance


Use default configuration path `.cocosign.yaml`, or
provide custom path using `attest-config` input argument.

See details [documentation - attestation](docs/attestations.md) \
Source see [cocosign](https://github.com/scribe-security/cocosign), attestation manager



## .gitignore
Recommended to add output directory value to your .gitignore file.
By default add `**/scribe` to your `.gitignore`.

# Integrations
## Scribe service integration
Scribe provides a set of services to store, verify and manage the supply chain integrity. \
Following are some integration examples.

Scribe integrity flow - upload evidence using `gensbom` and download the integrity report using `valint`. \
You may collect evidence anywhere in your workflows.

<details>
  <summary>  Scribe integrity report - full workflow </summary>

Full workflow example of a workflow, upload evidence using gensbom and download report using Valint.

```YAML
name: example workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-report-test:
    runs-on: ubuntu-latest
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - uses: actions/checkout@v3
        with:
          repository: mongo-express/mongo-express
          ref: refs/tags/v1.0.0-alpha.4
          path: mongo-express-scm

      - name: gensbom Scm generate bom, upload to scribe
        id: gensbom_bom_scm
        uses: scribe-security/action-bom@master
        with:
           type: dir
           target: 'mongo-express-scm'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Build and push remote
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: mongo-express:1.0.0-alpha.4

      - name: gensbom Image generate bom, upload to scribe
        id: gensbom_bom_image
        uses: scribe-security/action-bom@master
        with:
           target: 'mongo-express:1.0.0-alpha.4'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Valint - download report
        id: valint_report
        uses: scribe-security/actions/valint/report@master
        with:
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - uses: actions/upload-artifact@v2
        with:
          name: scribe-reports
          path: |
            ${{ steps.gensbom_bom_scm.outputs.OUTPUT_PATH }}
            ${{ steps.gensbom_bom_image.outputs.OUTPUT_PATH }}
            ${{ steps.valint_report.outputs.OUTPUT_PATH }}
```
</details>


<details>
  <summary>  Scribe integrity report - Multi workflow </summary>

Full workflow example of a workflow, upload evidence using gensbom and download report using valint

```YAML
name: example workflow

on: 
  push:
    tags:
      - "*"

jobs:
  scribe-report-test:
    runs-on: ubuntu-latest
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - uses: actions/checkout@v3
        with:
          repository: mongo-express/mongo-express
          ref: refs/tags/v1.0.0-alpha.4
          path: mongo-express-scm

      - name: Build and push remote
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: mongo-express:1.0.0-alpha.4

      - name: gensbom Image generate bom, upload to scribe
        id: gensbom_bom_image
        uses: scribe-security/action-bom@master
        with:
           target: 'mongo-express:1.0.0-alpha.4'
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - name: Valint - download report
        id: valint_report
        uses: scribe-security/actions/valint/report@master
        with:
           verbose: 2
           scribe-enable: true
           product-key:  ${{ secrets.product-key }}
           scribe-client-id: ${{ secrets.client-id }}
           scribe-client-secret: ${{ secrets.client-secret }}

      - uses: actions/upload-artifact@v2
        with:
          name: scribe-reports
          path: |
            ${{ steps.gensbom_bom_scm.outputs.OUTPUT_PATH }}
            ${{ steps.gensbom_bom_image.outputs.OUTPUT_PATH }}
            ${{ steps.valint_report.outputs.OUTPUT_PATH }}
```
</details>

## Integrity report examples
<details>
  <summary>  Scribe integrity report </summary>

Valint downloading integrity report from scribe service

```YAML
  - name: Valint - download report
    id: valint_report
    uses: scribe-security/actions/valint/report@master
    with:
        verbose: 2
        scribe-enable: true
        product-key:  ${{ secrets.product-key }}
        scribe-client-id: ${{ secrets.client-id }}
        scribe-client-secret: ${{ secrets.client-secret }}
```
</details>

<details>
  <summary>  Scribe integrity report, select section </summary>

Valint downloading integrity report from scribe service

```YAML
  - name: Valint - download report
    id: valint_report
    uses: scribe-security/actions/valint/report@master
    with:
        verbose: 2
        scribe-enable: true
        product-key:  ${{ secrets.product-key }}
        scribe-client-id: ${{ secrets.client-id }}
        scribe-client-secret: ${{ secrets.client-secret }}
        section: packages
```
</details>

## Generating SBOMs examples
<details>
  <summary>  Public registry image </summary>

Create SBOM from remote `busybox:latest` image, skip if found by the cache.

```YAML
- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    target: 'busybox:latest'
    format: json
``` 
</details>

<details>
  <summary>  Docker built image </summary>

Create SBOM for image built by local docker `image_name:latest` image, overwrite cache.

```YAML
- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    type: docker
    target: 'image_name:latest'
    format: json
    force: true
``` 
</details>

<details>
  <summary>  Private registry image </summary>

Custom private registry, skip cache (using `Force`), output verbose (debug level) log output.
```YAML
- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    target: 'scribesecuriy.jfrog.io/scribe-docker-local/stub_remote:latest'
    verbose: 2
    force: true
```
</details>

<details>
  <summary>  Custom SBOM metadata </summary>

Custom metadata added to SBOM
Data will be included in the signed payload when the output is an attestation.
```YAML
- name: Generate cyclonedx json SBOM - add metadata - labels, envs, name
  id: gensbom_labels
  uses: scribe-security/action-bom@master
  with:
      target: 'busybox:latest'
      verbose: 2
      format: json
      force: true
      name: name_value
      env: test_env
      label: test_label
  env:
    test_env: test_env_value
```
</details>


<details>
  <summary> Save SBOM as artifact </summary>

Using action `output_path` you can access the generated SBOM and store it as an artifact.
```YAML
- name: Generate cyclonedx json SBOM
  id: gensbom_json
  uses: scribe-security/action-bom@master
  with:
    target: 'busybox:latest'
    format: json

- uses: actions/upload-artifact@v2
  with:
    name: gensbom-busybox-output-test
    path: ${{ steps.gensbom_json.outputs.OUTPUT_PATH }}
``` 
</details>

<details>
  <summary> Save SLSA provenance statement as artifact </summary>
Using action `output_path` you can access the generated SBOM and store it as an artifact.

```YAML
- name: Generate SLSA provenance statement
  id: gensbom_slsa_statement
  uses: scribe-security/action-bom@master
  with:
    target: 'busybox:latest'
    format: statement-slsa

- uses: actions/upload-artifact@v2
  with:
    name: scribe-evidence
    path: ${{ steps.gensbom_slsa_statement.outputs.OUTPUT_PATH }}
``` 
</details>

<details>
  <summary> Docker archive image </summary>

Create SBOM from local `docker save ...` output.
```YAML
- name: Build and save local docker archive
  uses: docker/build-push-action@v2
  with:
    context: .
    file: .GitHub/workflows/fixtures/Dockerfile_stub
    tags: scribesecuriy.jfrog.io/scribe-docker-public-local/stub_local:latest
    outputs: type=docker,dest=stub_local.tar

- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    type: docker-archive
    target: '/GitHub/workspace/stub_local.tar'
``` 
</details>

<details>
  <summary> OCI archive image </summary>

Create SBOM from the local oci archive.

```YAML
- name: Build and save local oci archive
  uses: docker/build-push-action@v2
  with:
    context: .
    file: .GitHub/workflows/fixtures/Dockerfile_stub
    tags: scribesecuriy.jfrog.io/scribe-docker-public-local/stub_local:latest
    outputs: type=docker,dest=stub_oci_local.tar

- name: Generate cyclonedx json SBOM
  uses: scribe-security/action-bom@master
  with:
    type: oci-archive
    target: '/GitHub/workspace/stub_oci_local.tar'
``` 
</details>

<details>
  <summary> Directory target </summary>

Create SBOM from a local directory. \
Note directory must be mapped to working dir for actions to access (containerized action).

```YAML
- name: Create dir
  run: |
    mkdir testdir
    echo "test" > testdir/test.txt

- name: gensbom attest dir
  id: gensbom_attest_dir
  uses: scribe-security/action-bom@master
  with:
    type: dir
    target: '/GitHub/workspace/testdir'
``` 
</details>

<details>
  <summary> Attest target (BOM) </summary>

Create and sign SBOM targets, skip if found signed SBOM by the cache. \
Targets: `registry`, `docker-archive`, `oci-archive`, `dir`.
Note: Default attestation config **Required** `id-token` permission access. \
Default attestation config: `sigstore-config` - GitHub workload identity and Sigstore (Fulcio, Rekor).


```YAML
job_example:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
  steps:
    - name: gensbom attest
    uses: scribe-security/action-bom@master
    with:
        target: 'busybox:latest'
        format: attest
``` 
</details>

<details>
  <summary> Attest target (SLSA) </summary>

Create and sign SBOM targets, skip if found signed SBOM by the cache. \
Targets: `registry`, `docker-archive`, `oci-archive`, `dir`.
Note: Default attestation config **Required** `id-token` permission access. \
Default attestation config: `sigstore-config` - GitHub workload identity and Sigstore (Fulcio, Rekor).

```YAML
job_example:
  runs-on: ubuntu-latest
  permissions:
    id-token: write
  steps:
    - name: gensbom attest
    uses: scribe-security/action-bom@master
    with:
        target: 'busybox:latest'
        format: attest-slsa
``` 
</details>

<details>
  <summary> Verify target (BOM) </summary>

Verify targets against a signed attestation. \
Note: `docker` in target `type` field (is not accessible because it requires docker daemon (containerized actions) \
Default attestation config: `sigstore-config` - sigstore (Fulcio, Rekor).
gensbom will look for both a bom or slsa attestation to verify against

```YAML
- name: gensbom verify
  uses: scribe-security/actions/gensbom/verify@master
  with:
    target: 'busybox:latest'
``` 

</details>

<details>
  <summary> Verify target (SLSA) </summary>

Verify targets against a signed attestation. \
Note: `docker` in target `type` field (is not accessible because it requires docker daemon (containerized actions) \
Default attestation config: `sigstore-config` - sigstore (Fulcio, Rekor).
gensbom will look for both a bom or slsa attestation to verify against

```YAML
- name: gensbom verify
  uses: scribe-security/actions/gensbom/verify@master
  with:
    target: 'busybox:latest'
    input-format: attest-slsa
``` 

</details>

<details>
  <summary> Attest and verify image (BOM) </summary>

Full job example of a image signing and verifying flow.

```YAML
 gensbom-busybox-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: gensbom attest
        id: gensbom_attest
        uses: scribe-security/action-bom@master
        with:
           target: 'busybox:latest'
           verbose: 2
           format: attest
           force: true

      - name: gensbom verify
        id: gensbom_verify
        uses: scribe-security/actions/gensbom/verify@master
        with:
           target: 'busybox:latest'
           verbose: 2

      - uses: actions/upload-artifact@v2
        with:
          name: gensbom-busybox-test
          path: gensbom_reports
``` 

</details>

<details>
  <summary> Attest and verify image (SLSA) </summary>

Full job example of a image signing and verifying flow.

```YAML
 gensbom-busybox-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: gensbom attest slsa
        id: gensbom_attest
        uses: scribe-security/action-bom@master
        with:
           target: 'busybox:latest'
           verbose: 2
           format: attest-slsa
           force: true

      - name: gensbom verify attest slsa
        id: gensbom_verify
        uses: scribe-security/actions/gensbom/verify@master
        with:
           target: 'busybox:latest'
           input-format: attest-slsa
           verbose: 2

      - uses: actions/upload-artifact@v2
        with:
          name: gensbom-busybox-test
          path: gensbom_reports
``` 

</details>

<details>
  <summary> Attest and verify directory </summary>

Full job example of a directory signing and verifying flow.

```YAML
  gensbom-dir-test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    steps:

      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: gensbom attest workdir
        id: gensbom_attest_dir
        uses: scribe-security/action-bom@master
        with:
           type: dir
           target: '/GitHub/workspace/'
           verbose: 2
           format: attest
           force: true

      - name: gensbom verify workdir
        id: gensbom_verify_dir
        uses: scribe-security/actions/gensbom/verify@master
        with:
           type: dir
           target: '/GitHub/workspace/'
           verbose: 2
      
      - uses: actions/upload-artifact@v2
        with:
          name: gensbom-workdir-reports
          path: |
            gensbom_reports      
``` 

</details>

<details>
  <summary> Install gensbom (tool) </summary>

Install gensbom as a tool
```YAML
- name: install gensbom
  uses: scribe-security/actions/gensbom/installer@master

- name: gensbom run
  run: |
    gensbom --version
    gensbom bom busybox:latest -vv
``` 
</details>

<details>
  <summary> Install Valint (tool) </summary>

Install Valint as a tool
```YAML
- name: install gensbom
  uses: scribe-security/actions/gensbom/installer@master
  with:
    tool: valint

- name: valint run
  run: |
    valint --version
    valint report --scribe.client-id $SCRIBE_CLIENT_ID $SCRIBE_CLIENT_SECRET
``` 
</details>
