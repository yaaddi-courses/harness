# Secrets and secret managers

**Covers:** secrets, HashiCorp Vault, HashiCorp

## The most expensive line in a pipeline {#overview}
A pipeline needs to log in to many things: a code host, an image store, a cloud account, a database. Each login needs a password or a token. If those values sit in plain text in a pipeline file, a script or a chat message, anyone who can read the file can use them. And files get copied, shared and kept forever in version control. Handling credentials correctly is one of the most important parts of a delivery platform, so let's see how Harness does it.

## secrets {#secrets}
Secrets are pieces of sensitive data, such as passwords, API tokens, private keys and certificates. Harness stores them encrypted, and pipelines refer to them by name, so the real value never appears in the pipeline definition. A step uses a reference such as `<+secrets.getValue("registry_token")>`, and the value is filled in only when the step runs, on the Delegate.

Four properties matter:
- **Hidden in logs:** Harness masks secret values in the logs. If a script prints one, the log shows asterisks.
- **Access control:** who may use a secret, who may see it and who may edit it are separate permissions.
- **Scope:** you can define a secret for one project, one organization or the whole account.
- **Rotation:** to change a credential, you update the one secret and not every pipeline.

## HashiCorp {#hashicorp}
HashiCorp is the company behind several widely used infrastructure tools, among them Terraform, for describing infrastructure as code, and Vault, for secrets. Organizations that automate infrastructure often use their tools.

## HashiCorp Vault {#hashicorp-vault}
HashiCorp Vault is a popular third-party tool for storing and managing secrets. Harness can plug into it instead of, or alongside, its own built-in storage. Large companies often already have a Vault as the one place where all secrets live, with its own audit trail, policies and automatic expiry.

You connect it through a secret manager connector. The pipeline still refers to the secret by name, but Harness reads the value from Vault when it is needed, through the Delegate, so Harness does not keep the value at all. Vault can also create short-lived credentials on demand, which expire after minutes, so a stolen one is soon useless.

## A worked example {#example}
A pipeline pushes an image to an image store, which needs a token.

First, the wrong way, with the token in the definition:

```yaml
- step:
    type: Run
    name: Push image
    spec:
      command: docker login -u ci -p EXAMPLE_TOKEN_DO_NOT_USE registry.example.com
```

Anyone who can read this file now has the token. That includes everyone with access to the repository and its full history.

Now the right way, with a secret:

```yaml
- step:
    type: Run
    name: Push image
    spec:
      envVariables:
        REGISTRY_TOKEN: <+secrets.getValue("registry_token")>
      command: echo "$REGISTRY_TOKEN" | docker login -u ci --password-stdin registry.example.com
```

Let's compare the two:

| | Token in the file | Secret reference |
|---|---|---|
| Visible in the repository | Yes, forever | No, only the name |
| Visible in logs | Yes | Masked |
| Rotating the token | Edit every file and the history cannot be cleaned | Change one secret |
| Who can use it | Anyone who can read the file | Only those with permission to the secret |

Now suppose the company keeps its secrets in Vault. The secret `registry_token` is stored there, and Harness reads it through the Vault connector at run time. When the security team rotates the token in Vault, the next pipeline run picks it up automatically, with no change in Harness.

Count the exposure. With 30 pipelines using the same token, plain text means 30 places to find and fix after a leak. With a secret, the answer is 1.

## A common mistake {#mistake}
A common slip is to print a secret in a script to "debug" it, or to encode it with base64 and think that hides it. The log masking works on the exact value, so an altered form passes through unmasked. Another slip is committing a real token, even once. Assume it has leaked and rotate it at once, because the history is kept.

## Key points {#key-points}
- Secrets are stored encrypted and referred to by name, so the value never appears in pipeline files.
- Harness masks secrets in logs. Do not print them or transform them.
- HashiCorp Vault is an external secret store that Harness can read from through a connector.
- Update a secret in one place to rotate it everywhere. Rotate immediately after any leak.
