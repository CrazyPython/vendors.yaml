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

### Specification (real)

Each vendor has a `provides` key, which is an array of regular expressions. The file is executed from top to bottom. The first vendor that matches is used. To use a vendor, send its `config` key to the model API.

#### Standardized key names

* `api_key` - `apiKey` will not work

#### Standard vendor names

* openai
* ai21
* forefront
* anthropic
* textsynth
* goose

### Implementations

#### Toy Python implementation

```
def create_continuation(model, vendor=None, vendor_config=None, **kwargs):
    if vendor is None:
        vendor = pick_vendor(model, vendor_config)
    if vendor_config is not None and vendor in vendor_config:
        kwargs = {**vendor_config[vendor]['config'], **kwargs}
    return openai.Completion.create(model=model, **kwargs)

def pick_vendor(model, custom_config=None):
    if custom_config is not None:
        for vendor_name, vendor in custom_config.items():
            if vendor['provides'] is not None:
                for pattern in vendor['provides']:
                    if re.match(pattern, model):
                        return vendor_name
    return "openai"
```
