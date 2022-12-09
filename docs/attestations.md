---
title: Attestations
---

# Attestations
Attestations represents authenticated metadata about a set of software artifacts (evidence). <br /> 
scribe utilizes both attestations (signed) and statement (unsigned) to validate the integrity and policy compliance of your supply chain.

## Evidence 
`cocosign` supports both signed and unsigned evidence.
* InToto statement - unsigned evidence
* InToto attestation - signed evidence

See details [In-toto spec](https://github.com/in-toto/attestation)


## Default configuration
You can select from a set of prefilled default configuration.

> Use flag `--attest.default`, supported values are `sigstore,sigstore-github,x509`.

<details>
  <summary> Sigstore public instance </summary>

Sigstore signer and verifier allow you to use ephemeral short living keys based on OIDC identity (google, microsoft, github).
Sigstore will also provide a transperancy log for any one to verify your signatures against (`rekor`)

> Use flag `--attest.default=sigstore`.

Default config
```yaml
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
  <summary> Sigstore public instance - Github workfload identity </summary>

Sigstore signer and verifier allow you to use ephemeral short living keys based on OIDC identity (google, microsoft, github).
Sigstore will also provide a transperancy log for any one to verify your signatures against (`rekor`)

> Select by using `--attest.default=sigstore-github`

Default config

```yaml
signer:
  fulcio:
    enable: true
    url: https://fulcio.sigstore.dev
    oidc:
      auth: provider
      issuer: https://token.actions.githubusercontent.com
      clientid: sigstore
verifier:
  fulcio:
    enable: true
```
</details>


<details>
  <summary> X509 local keys </summary>

X509 flows allow you to use local keys, cert and CA file to sign and verify you sboms.
You may can use the default x509 `cocosign` configuration flag.

> Use flag `--attest.default=x509`.

```bash
gensbom busybox:latest -o attest --attest.default x509 -v
gensbom verify busybox:latest --attest.default fulcio -v
```

Default config
```yaml
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


## Custom configuration
Edit your main configuration, add the following subsection. \
For full configuration details see [configuration-format](#configuration-format).

Usage:
```yaml
attest:
    default: ""
    cocosign: #Custom cocosign configuration
        signer:
            x509:
                enable: true
                private: ./private/key.pem
        verifier:
            x509:
                enable: true
                cert: ./public/cert.pem
                ca: ./public/ca.pem
```
> Use flag `--attest.config` to provide a external cocosign config.


## Signers and verifiers support

### **KMS**
Sigstore based KMS signer allows users to sign via kms.
[doc](https://github.com/sigstore/cosign/blob/main/KMS.md) for Ref details.
    - Support `KMSREF` environment variable (when configuration field is empty).
    - Support static ref set by configuration or env.
    - Support in-band ref verification flow by using the `REF` signature option.

### **Fulcio**
Sigstore based fulcio signer allows users to sign InToto statement using fulcio (Sigstore) project.

Simply put you can utilize a OIDC connection to gain a short living certificate signed to your identity.

[keyless](https://github.com/sigstore/cosign/blob/main/KEYLESS.md)
[fulcio_doc](https://github.com/sigstore/fulcio)

#### Support
- Interactive - User must authorize the signature via browser, device or security code url.
- Token - Static OIDC identity token provided provided by an external tool.
- Built-in identity providers flows
    - google-workload-identity
    - google-impersonate
    - github-workload-identity
    - spiffe
    - google-impersonate-cred (`GOOGLE_APPLICATION_CREDENTIALS`)

### **x509** 
File based key management library, go library abstracting the key type from applications (Supports TPM).

> See [KML](https://github.com/scribe-security/KML) for details.

| Media type | TPM | RSA (pss)| ECDSA (p256) | ED25519 |
| --- | --- | --- | --- | --- |
| file | | 2048, 4096 | yes | yes |
| TPM | yes | 2048 | | |

 > PEM formatted files 

### OCI storer
Storer uploads evidence to your OCI registry.
Evidence can be attached to a specific image or uploaded to a general repo location.

OCI store capability allows your evidence collection to span across your supply chain.

> Use flag `--oci` and `--oci-repo` to enable.

Default config, 
``` 
storer:
    oci:
        enable: false
        ref: ""
```

> Supports cosign verification

## Configuration format
```yaml
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
	    ref: <kms_ref>
verifier:
	x509:
	    enable: <true|false>
	    cert: <cert_path>
	    ca: <ca_path>
	fulcio:
	    enable: <true|false>
	kms:
	    enable: <true|false>
	    ref: <kms_ref>
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
    oci:
        enable: <true|false>
        ref: <oci ref to upload evidence to>
```