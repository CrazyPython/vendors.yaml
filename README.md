# vendors.yml
A file format for configuring access to language models

## Motivation

Some language model applications need to easily switch between different models. Different accounts have access to different models, and managing keys and organization IDs across vendors can be difficult. `vendors.yml` is a standardized way to tell language model apps how to access differnt models.

## Specification

### Example

Imagine you have access to the GPT-3.5 base through the [openai-cd2-proxy](https://github.com/cosmicoptima/openai-cd2-proxy), you have access to ChatGPT-4 through an organization `org-myOrgID`, and you have a free trial account for everything else. Your free trial account is called "foo foundation," and has several fine-tuned models on it. The following vendors.yml cause compliant language model libraries to correctly route requests:

```
openai-cd2-proxy:
    config:
        api_key: mySecretProxyKey
        api_base: https://my.proxy.url/v1
    provides: ["code-davinci-002"]
openai-gpt-4:
    config:
        api_key: mySecretKey
        organization: org-myOrgId
    provides: ["gpt-4.*"]
openai:
    config:
        api_key: mySecretKey
        organization: org-myOtherOrgId
    provides: [".*:ft-foo-foundation.*"]
```
