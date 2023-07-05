---
title: Verify
sidebar_position: 2
---
# Scribe Github Action for `valint bom`
Scribe offers GitHub Actions for embedding evidence collecting and validated integrity of your supply chain.

Use `valint verify` to verify evidence (attestations) and policies.

Further documentation [Github integration](https://scribe-security.netlify.app/docs/ci-integrations/github/)


## Other Actions
* [bom](action-bom.md), [source](https://github.com/scribe-security/action-bom)
* [verify](action-verify.md), [source](https://github.com/scribe-security/action-verify)
* [installer](action-installer.md), [source](https://github.com/scribe-security/action-installer)


## Verify Action
Action for `valint verify`.
The command allows users to verify any target against its evidence.
- Verify image, directory, file or git targets.
- Verify evidence policy compliance across the supply chain.
- Pull evidence from scribe service.
- Download and search evidence in all enabled stores.
- Support Sigstore keyless verifying as well as [Github workload identity](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect).

### Input arguments
```yaml
  type:
    description: 'Target source type options=[docker,docker-archive, oci-archive, dir, registry, git]'
  target:
    description:
    required: true
  attestation:
    description: Attestation for target
  common-name:
    description: Default policy allowed common names
  email:
    description: Default policy allowed emails
  force:
    description: Force skip cache
  input-format:
    description: Evidence format, options=[attest-cyclonedx-json attest-slsa statement-slsa statement-cyclonedx-json statement-generic]
    default: attest-cyclonedx-json
  uri:
    description: Default policy allowed uris
  attest-config:
    description: Attestation config path
  attest-default:
    description: Attestation default config, options=[sigstore sigstore-github x509]
    default: sigstore
  backoff:
    description: Backoff duration
    default: 15s
  cache-enable:
    description: Enable local cache
    default: true
  config:
    description: Configuration file path
  context-dir:
    description: Context dir
  filter-regex:
    description: Filter out files by regex
    default: '**/*.pyc,**/.git/**'
  filter-scope:
    description: Filter packages by scope
  git-branch:
    description: Git branch in the repository
  git-commit:
    description: Git commit hash in the repository
  git-tag:
    description: Git tag in the repository
  label:
    description: Add Custom labels
  level:
    description: Log depth level, options=[panic fatal error warning info debug trace]
  oci:
    description: Enable OCI store
  oci-repo:
    description: Select OCI custom attestation repo
  output-directory:
    description: Output directory path
    default: ./scribe/valint
  output-file:
    description: Output file name
  product-key:
    description: Scribe Project Key
  scribe-audience:
    description: Scribe auth audience
    default: api.production.scribesecurity.com
  scribe-client-id:
    description: Scribe Client ID
  scribe-client-secret:
    description: Scribe Client Secret
  scribe-enable:
    description: Enable scribe client
  scribe-login-url:
    description: Scribe login url
    default: https://scribesecurity-production.us.auth0.com
  scribe-url:
    description: Scribe API Url
    default: https://api.production.scribesecurity.com
  timeout:
    description: Timeout duration
    default: 120s
  verbose:
    description: Log verbosity level [-v,--verbose=1] = info, [-vv,--verbose=2] = debug
    default: 1

```

### Usage
```
- name: valint verify
  id: valint_verify
  uses: scribe-security/action-installer@master
  with:
      target: 'busybox:latest'
```

## Configuration
Use default configuration path `.valint.yaml`, or provide a custom path using `--config` flag.

See detailed [configuration](docs/configuration.md)


## Attestations 
Attestations allow you to sign and verify your targets. <br />
Attestations allow you to connect PKI-based identities to your evidence and policy management.  <br />

Supported outputs:
- In-toto predicate - Cyclonedx SBOM, SLSA Provenance (unsigned evidence)
- In-toto statements - Cyclonedx SBOM, SLSA Provenance (unsigned evidence)
- In-toto attestations -Cyclonedx SBOM, SLSA Provenance (signed evidence)

Select default configuration using `--attest.default` flag. <br />
Select a custom configuration by providing `cocosign` field in the [configuration](docs/configuration.md) or custom path using `--attest.config`.
Scribe uses the **cocosign** library we developed to deal with digital signatures for signing and verification.

See details [In-toto spec](https://github.com/in-toto/attestation)
See details [attestations](docs/attestations.md)

### Storing Keys in Secret Vault

Github exposes secrets from its vault using envrionment varuables, you may provide these envrionments as secret to valint.

> Paths names prefixed with `env://[NAME]` are read from the envrionment matching the name.

<details>
  <summary> Github Secret Vault </summary>

X509 Signer enables the utilization of environments for supplying key, certificate, and CA files in order to sign and verify attestations. It is commonly employed in conjunction with Secret Vaults, where secrets are exposed through environments.

>  path names prefixed with `env://[NAME]` are extracted from the environment corresponding to the specified name.


For example the following configuration and Job.

Configuraiton File, `.valint.yaml`
```yaml
attest:
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
          SIGNER_CERT: ${{ secrets.SIGNER_KEY }}
          COMPANY_CA:  ${{ secrets.COMPANY_CA }}

        uses: scribe-security/action-verify@master
        with:
          target: busybox:latest
          input-format: attest
        env:
          SIGNER_CERT: ${{ secrets.SIGNER_KEY }}
          COMPANY_CA:  ${{ secrets.COMPANY_CA }}
```
</details>


## Target types - `[target]`
---
Target types are types of artifacts produced and consumed by your supply chain.
Using supported targets, you can collect evidence and verify compliance on a range of artifacts.

> Fields specified as [target] support the following format.

### Format

`[scheme]:[name]:[tag]` 

> Backwards compatibility: It is still possible to use the `type: [scheme]`, `target: [name]:[tag]` format.

| Sources | target-type | scheme | Description | example
| --- | --- | --- | --- | --- |
| Docker Daemon | image | docker | use the Docker daemon | docker:busybox:latest |
| OCI registry | image | registry | use the docker registry directly | registry:busybox:latest |
| Docker archive | image | docker-archive | use a tarball from disk for archives created from "docker save" | image | docker-archive:path/to/yourimage.tar |
| OCI archive | image | oci-archive | tarball from disk for OCI archives | oci-archive:path/to/yourimage.tar |
| Remote git | git| git | remote repository git | git:https://github.com/yourrepository.git |
| Local git | git | git | local repository git | git:path/to/yourrepository | 
| Directory | dir | dir | directory path on disk | dir:path/to/yourproject | 
| File | file | file | file path on disk | file:path/to/yourproject/file | 

### Evidence Stores
Each storer can be used to store, find and download evidence, unifying all the supply chain evidence into a system is an important part to be able to query any subset for policy validation.

| Type  | Description | requirement |
| --- | --- | --- |
| scribe | Evidence is stored on scribe service | scribe credentials |
| OCI | Evidence is stored on a remote OCI registry | access to a OCI registry |

## Scribe Evidence store
Scribe evidence store allows you store evidence using scribe Service.

Related Flags:
> Note the flag set:
>* `scribe-client-id`
>* `scribe-client-secret`
>* `scribe-enable`

### Before you begin
Integrating Scribe Hub with your environment requires the following credentials that are found in the **Integrations** page. (In your **[Scribe Hub](https://prod.hub.scribesecurity.com/ "Scribe Hub Link")** go to **integrations**)

* **Client ID**
* **Client Secret**

<img src='../../../img/ci/integrations-secrets.jpg' alt='Scribe Integration Secrets' width='70%' min-width='400px'/>

* Add the credentials according to the [GitHub instructions](https://docs.github.com/en/actions/security-guides/encrypted-secrets/ "GitHub Instructions"). Based on the code example below, be sure to call the secrets **clientid** for the **client_id**, and **clientsecret** for the **client_secret**.

* Use the Scribe custom pipe as shown in the example bellow

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

        uses: scribe-security/action-bom@master
        with:
          target: [target]
          format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          scribe-enable: true
          scribe-client-id: ${{ secrets.clientid }}
          scribe-client-secret: ${{ secrets.clientsecret }}

        uses: scribe-security/action-verify@master
        with:
          target: [target]
          format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          scribe-enable: true
          scribe-client-id: ${{ secrets.clientid }}
          scribe-client-secret: ${{ secrets.clientsecret }}
```

## OCI Evidence store
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
        uses: scribe-security/action-bom@master
        with:
          target: [target]
          format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          oci: true
          oci-repo: [oci_repo]

      - name:  Verify policy step
        uses: scribe-security/action-verify@master
        with:
          target: [target]
          format: [attest, statement, attest-slsa, statement-slsa, attest-generic, statement-generic]
          oci: true
          oci-repo: [oci_repo]
```


## Verify SBOMs examples
<details>
  <summary> Verify target (SBOM) </summary>

Verify targets against a signed attestation.
Default attestation config: `sigstore-github` - sigstore (Fulcio, Rekor). <br />
valint will look for both a bom or slsa attestation to verify against. <br />

```YAML
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
``` 

</details>

<details>
  <summary> Verify target (SLSA) </summary>

Verify targets against a signed attestation. <br />
Default attestation config: `sigstore-github` - sigstore (Fulcio, Rekor). <br />
valint will look for both a bom or slsa attestation to verify against. <br />


```YAML
- name: valint verify
  uses: scribe-security/action-verify@master
  with:
    target: 'busybox:latest'
    input-format: attest-slsa
``` 

</details>

<details>
  <summary> Verify target (Gernic) </summary>

Verify targets against a signed attestation. <br />
Default attestation config: `sigstore-github` - sigstore (Fulcio, Rekor). <br />
valint will look for both a bom or slsa attestation to verify against. <br />


```YAML
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

      - uses: actions/upload-artifact@v2
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

      - uses: actions/upload-artifact@v2
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
      
      - uses: actions/upload-artifact@v2
        with:
          name: valint-workdir-evidence
          path: |
            scribe/valint      
``` 

</details>


<details>
  <summary> Verify Policy flow - Git repository target (Signed SBOM) </summary>

Full job example of a git repository signing and verifying flow.
> Support for both local (path) and remote git (url) repositories.

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
      
      - uses: actions/upload-artifact@v3
        with:
          name: valint-git-evidence
          path: |
            scribe/valint      
``` 
</details>

<details>
  <summary> Attest and verify evidence on OCI (SBOM,SLSA) </summary>

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

      - uses: scribe-security/action-bom@dev
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

      - uses: scribe-security/action-verify@dev
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
  <summary> Install valint (tool) </summary>

Install valint as a tool
```YAML
- name: install valint
  uses: scribe-security/action-installer@master

- name: valint run
  run: |
    valint --version
    valint bom busybox:latest
``` 
</details>

## .gitignore
Recommended to add output directory value to your .gitignore file.
By default add `**/scribe` to your `.gitignore`.
