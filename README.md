# schemas

JSON Schemas for SGNL action metadata files.

## Schemas

| Schema | Description |
|--------|-------------|
| [`metadata.schema.json`](metadata.schema.json) | Validates `metadata.yaml` files in SGNL action repositories |

## Usage

### Editor Validation (VS Code, IntelliJ)

Add this comment to the top of your `metadata.yaml`:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/sgnl-actions/schemas/main/metadata.schema.json
name: my-action
description: My action description
# ...
```

This gives you real-time validation and autocomplete in any editor that supports the [YAML Language Server](https://github.com/redhat-developer/yaml-language-server).

**VS Code setup**: Install the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) by Red Hat. The `yaml-language-server` comment is detected automatically.

### Programmatic Validation (CI)

Use [ajv](https://ajv.js.org/) to validate in Node.js:

```javascript
import Ajv from 'ajv';
import fs from 'fs';
import yaml from 'js-yaml';

const schema = JSON.parse(fs.readFileSync('metadata.schema.json', 'utf8'));
const metadata = yaml.load(fs.readFileSync('metadata.yaml', 'utf8'));

const ajv = new Ajv();
const validate = ajv.compile(schema);

if (!validate(metadata)) {
  console.error(validate.errors);
  process.exit(1);
}
```

## Metadata Structure

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Action identifier (kebab-case) |
| `description` | string | Human-readable description |
| `inputs` | object | Input parameters |
| `outputs` | object | Output fields |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `secrets` | array or object | Required credentials |
| `environment` | object | Environment variables |
| `requires_one_of` | array | At least one complete group must be satisfied |
| `optional_one_of` | array | If any key present, one complete group must be satisfied |
| `runtime` | object | Runtime configuration (type, timeout, memory) |

### Input Types

`text`, `number`, `boolean`, `array`, `email`

### Output Types

`text`, `number`, `boolean`, `datetime`, `object`, `array`

### Validation Rules

| Rule | Applies To | Description |
|------|-----------|-------------|
| `min` | text, array | Minimum length or size |
| `max` | text, array | Maximum length or size |
| `regex` | text | Regular expression pattern |
| `enum` | text | Restricted set of allowed values |

## Validation Groups

Actions can declare validation requirements using `requires_one_of` and `optional_one_of`. These fields enable flexible validation at configuration time.

### `requires_one_of`

At least one complete group must be satisfied (mandatory OR validation).

**Example** - Action supports multiple auth types:

```yaml
requires_one_of:
  - ["secrets.BEARER_AUTH_TOKEN"]
  - ["secrets.OAUTH2_CLIENT_CREDENTIALS_CLIENT_ID", "secrets.OAUTH2_CLIENT_CREDENTIALS_CLIENT_SECRET", "environment.OAUTH2_CLIENT_CREDENTIALS_TOKEN_URL"]
  - ["secrets.BASIC_USERNAME", "secrets.BASIC_PASSWORD"]
```

This action accepts Bearer token **OR** OAuth2 Client Credentials **OR** Basic auth.

### `optional_one_of`

If any key from any group is provided, at least one complete group must be satisfied (conditional OR validation).

**Example** - Action with optional auth:

```yaml
optional_one_of:
  - ["secrets.BEARER_AUTH_TOKEN"]
  - ["secrets.BASIC_USERNAME", "secrets.BASIC_PASSWORD"]
```

Auth is optional, but if provided, it must be either complete Bearer **OR** complete Basic auth (not partial).

### Key Prefixes

- `inputs.` - References an input parameter
- `secrets.` - References a secret credential
- `environment.` - References an environment variable

## License

MIT License - Copyright (c) 2025 SGNL.ai, Inc.
