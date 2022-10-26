---
title: Verify
---
# Scribe action for `gensbom bom`
Scribe offers GitHub actions for embedding evidence collecting and validated integrity of your supply chain. \

`gensbom verify` is used to verify evidence (attestations) and policies. \

Further documentation [Github integration](https://scribe-security.netlify.app/docs/ci-integrations/github/)


## Other actions
* [bom - action](https://github.com/scribe-security/action-bom/README.md)
* [verify - action](https://github.com/scribe-security/action-verify/README.md)
* [integrity report - action](https://github.com/scribe-security/action-report/README.md)
* [installer - action](https://github.com/scribe-security/action-installer/README.md)

## Verify action
Action for `gensbom verify`.
The command allows users to verify an image via a signed attestation (In-toto).
- Verify targets using signed sbom, SLSA provanence attestations.
- Signed SBOM supports, the action will verify Sigstore keyless flow (Fulcio CA + Rekor log) while using GitHub (See example below).
- Verify signer identity, for example, GitHub workflow ids.
- Download attestations (signed SBOMs) from Scribe service.
- Verify attestations via OPA/CUE policies (see cocosign documentation).
- Verify the trust of target images, directories, files or git repositories.

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
  input-format:
    description: 'Sbom input formatter, options=[attest-cyclonedx-json attest-slsa] (default "attest-cyclonedx-json")'
    default: attest-cyclonedx-json
  output-directory:
    description: 'report output directory'
    default: ./scribe/gensbom
  output-file:
    description: 'Output result to file'
  filter-regex:
    description: 'Filter out files by regex'
    default: .*\.pyc,\.git/.*
  attest-config:
    description: 'Attestation config map'
  attest-name:
    description: 'Attestation config name (default "gensbom")'
  attest-default:
    description: 'Attestation default config, options=[sigstore sigstore-github x509]'
    default: sigstore-github
```

### Usage
```
- name: Gensbom verify
  id: gensbom_verify
  uses: scribe-security/action-installer@master
  with:
      target: 'busybox:latest'
      verbose: 2
```

## Configuration
Use default configuration path `.gensbom.yaml`, or
provide a custom path using `config` input argument.

You may add a `.cocosign.yaml` file to your repository or pass it with `--`config` \
<!-- for more [Cocosign configuration](https://github.com/scribe-security/cocosign) -->


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
<!-- Source see [cocosign](https://github.com/scribe-security/cocosign), attestation manager -->

## .gitignore
Recommended to add output directory value to your .gitignore file.
By default add `**/scribe` to your `.gitignore`.

## Verify SBOMs examples

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
  uses: scribe-security/action-verify@master
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
  uses: scribe-security/action-verify@master
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
        uses: scribe-security/action-verify@master
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
        uses: scribe-security/action-verify@master
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
        uses: scribe-security/action-verify@master
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
