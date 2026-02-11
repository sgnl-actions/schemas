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

## License

MIT License - Copyright (c) 2025 SGNL.ai, Inc.
