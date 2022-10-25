## Attestations
gensbom utilized our `cocosign` framework to allow users to sign sbombs in to a In-toto attestations.
Framework allows users to select between signing and verifying schemes. 
gensbom creates a cyclonedx json attestation according to the In-toto spec.
Please see full documentation of `cocosign` 

Source see [cocosign](https://github.com/scribe-security/cocosign), attestation manager
Source see [sigstore](https://github.com/sigstore)

## Signer types
Signers/Verifiers are selected via configuration (supported) or provided by application (custom), objects allow application to chose between signed and verifying flows .

**Supported signers/verifiers**

- KMS - Cosign based KMS signer allows users to sign via kms.
[doc](https://github.com/sigstore/cosign/blob/main/KMS.md) for Ref details.
    - Support `KMSREF` environment variable (when configuration field is empty).
    - Support static ref set by configuration or env.
    - Support in-band ref verification flow by using the `REF` signature option.
- Fulcio - Cosign based fulcio signer allows users to sign attestation via fulcio (sigstore) project.
Simply put you can utilize a OIDC connection to gain a short living certificate signed to your identity.
[keyless](https://github.com/sigstore/cosign/blob/main/KEYLESS.md), [fulcio_doc](https://github.com/sigstore/fulcio) for more details.
    - Interactive - Ask user to authorize the signature via browser or security code url.
    - Token - Static OIDC identity token provided (extracted via external tool)
    - Providers
        - google-workload-identity
        - google-impersonate
        - github-workload-identity
        - spiffe
        - google-impersonate-cred (`GOOGLE_APPLICATION_CREDENTIALS`)
- x509/KML - Key management library, go library abstracting the key type from applications. (Supports TPM). [doc](https://github.com/scribe-security/KML) for details.
Note: KML has other values such as managing a basic x509 CA.
**KML support table:**
    
    ```jsx
    - Key formats - PEM formatted keys.
    - File CA API - rsa-2048,4096 (pss), ecdsa (p256).
    - TPM CA API - rsa-2048 (pss).
    - File installer API - rsa-2048,4096 (pss), ecdsa (p256), ed25519.
    - TPM installer API - rsa-2048 (pss).
    - File key API - rsa-2048,4096 (pss), ecdsa (p256), ed25519.
    - TPM key API rsa-2048,(pss).
    - File TLS API - rsa-2048,4096,(pss), , ecdsa (p256).
    - TPM TLS API - rsa-2048,(pss).
    ```

## Custom cocosign configuration
```jsx
signer:
	x509:
	    enable: <true|false>
	    private: <key_path>
	    cert: <cert_path>
	    ca: <ca_path>
	fulcio:
	    enable: <true|false>
	    url: <sigstore_url>
	    oidc:
	        issuer: <sigstore_issuer_url>
	        client-id: <sigstore_client_id>
	        client-secret: <sigstore_client_secret>
	        token:<external_token> - for auth=token, enter the OIDC identity token
	kms:
	    enable: <true|false>
	    ref: <kms_ref> # Or KMSREF envrionment
verifier:
	x509:
	    enable: <true|false>
	    cert: <cert_path>
	    ca: <ca_path>
	fulcio:
	    enable: <true|false>
	kms:
	    enable: <true|false>
	    ref: <kms_ref> # Or KMSREF envrionment
	policies:
		<list of rego/cue policies>
	certemail: 
		<email to verify certificate>
	certuris: 
		<uris to verify certificate>
  untrustedpublic: <true|false> // Allow verifiers with only publics
storer:
	rekor:
	    enable: <true|false>
	    url: <rekor_url>
	    disablebundle: <true|false>
```

## Usage
<details>
  <summary> Sigstore public instance </summary>

### Sigstore
Sigstore signer and verifier allow you to use ephemeral short living keys based on OIDC identity (google, microsoft, github).
You may can use the default Sigstore `cocosign` configuration flag.
Sigstore will also provide a transperancy log for any one to verify your signatures against (`rekor`)

```bash
gensbom bom busybox:latest -o attest --attest.default sigstore -v
gensbom verify busybox:latest --attest.default sigstore -v
``` 
Default config
```
signer:
    fulcio:
        enable: true
        url: https://fulcio.sigstore.dev
        oidc:
            auth: interactive
            issuer: https://oauth2.sigstore.dev/auth
            client-id: sigstore
verifier:
    fulcio:
        enable: true
storer:
    rekor:
        enable: true
        url: https://rekor.sigstore.dev
        disablebundle: false
```

</details>

<details>
  <summary> X509 local keys </summary>

### X509 local keys
X509 flows allow you to use local keys, cert and CA file to sign and verify you sboms.
You may can use the default x509 `cocosign` configuration flag.

```bash
gensbom bom busybox:latest -o attest --attest.default x509 -v
gensbom verify busybox:latest --attest.default fulcio -v
```
Default config
```
signer:
    x509:
        enable: true
        private: /etc/cocosign/keys/private/default.pem
        cert: /etc/cocosign/keys/public/cert.pem
        ca: /etc/cocosign/keys/public/ca.pem
verifier:
    x509:
        enable: true
        cert: /etc/cocosign/keys/public/cert.pem
        ca: /etc/cocosign/keys/public/ca.pem
```
</details>

<details>
  <summary> Scribe service - TBD </summary>
</details>

<details>
  <summary> Custom configuration </summary>

### Custom
You may use any configuration supported by `cocosign` as well.
Create a configuration file (default .cocosign)

```bash
gensbom bom busybox:latest -o attest --attest.config <config_path> -v
gensbom verify busybox:latest --attest.config <config_path> -v
``` 
</details>

<details>
  <summary> Cosign integration </summary>

## Cosign integration
Gensbom allows you to use cosign cli tool `attest`,`verify-attestation` subcommands.
You may use cosign to connect the attestation to your OCI registry (sign and verify).
Example uses keyless (sigstore) but you may use any cosign signer/verifer supported.
```
gensbom bom <image> -vv -o predicate -f --output-file gensbom_predicate.json
COSIGN_EXPERIMENTAL=1 cosign attest --predicate gensbom_predicate.json <image> --type https://scribesecurity.com/predicate/cyclondex
COSIGN_EXPERIMENTAL=1 cosign verify-attestation <image>
```
</details>
