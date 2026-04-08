# Automation Context: Manage test automation contexts via MarioBot

Interact with MarioBot to manage test automation context data (test accounts, beneficiaries, etc.) used by E2E automation tests.

## Channel & Bot Info

- **Channel**: `#zeno-testing` (C094E6V7ELS)
- **Bot mention**: `<@U08JT4ML6HL>` (MarioBot)

## Valid Environments (`--run_on`)

| Category | Environments |
|----------|-------------|
| **TymeBusiness** | `tb-tst`, `tb-stg` |
| **South Africa** | `sa-tst`, `sa-stg` |
| **Philippines** | `ph-dev`, `ph-tst`, `ph-stg` |
| **TymeConnect** | `tc-dev`, `tc-tst`, `tc-stg` |
| **SA (MX)** | `sa-mx-tst`, `sa-mx-stg` |
| **PH (MX)** | `ph-mx-dev`, `ph-mx-tst`, `ph-mx-stg` |
| **MCA SEA** | `mx-mca-sea-dev`, `mx-mca-sea-tst`, `mx-mca-sea-stg`, `mca-sea-dev`, `mca-sea-tst`, `mca-sea-stg` |
| **MCA TC** | `mx-mca-tc-dev`, `mx-mca-tc-tst`, `mx-mca-tc-stg` |
| **MCA PH** | `mx-mca-ph-dev`, `mx-mca-ph-tst`, `mx-mca-ph-stg` |

**Important**: Raw JSON in `--json_data` is only supported for **MX** environments (those containing `mx` in the name, e.g., `ph-mx-stg`, `sa-mx-stg`). For non-MX environments, JSON must be passed as a single-line compact JSON string.

## Actions

### Add Context (`/context/add`)

Add new test context data. The JSON data must be an **array** of context objects.

**Required fields** (varies by bank):
- `context` (string): The context identifier (e.g., `beneficiary_transfer`, `intra_transfer_happy_case`)
- `isInUse` (string): `"true"` or `"false"`
- `consumer_data` (object): Contains `passCode`, `cellPhone`, and optionally `accountNumber`
- `userName` (string): **Required for MX environments**

**SA-specific fields**:
- `saId` (string): South Africa ID number
- `payload` (object): Usually `{}`
- `note` (string): Usually `""`

**PH-specific fields**:
- `userName` (string): Username for the account
- `bank_name` (string): e.g., `"GoTyme Bank"`
- `Account`, `Mobile`, `Email` (objects): Transfer recipient info with `receive_info` and `account_name`

### Get Context (`/context/get`)

Retrieve existing context data.

### Delete Context (`/context/delete`)

Remove context data.

## Command Format

All commands are sent as Slack messages in the `#zeno-testing` channel:

```
<@U08JT4ML6HL> --run_on=<environment> --action=<action> --json_data=<json>
```

For actions without JSON data:
```
<@U08JT4ML6HL> --run_on=<environment> --action=<action> --<param>=<value>
```

## How to Send Commands

1. **Format the JSON** as a compact single-line string (no newlines, no extra spaces).
   - **CRITICAL**: JSON string values must NOT contain spaces. MarioBot parses `--json_data` by splitting on spaces. Replace spaces with underscores (e.g., `"For Automation"` → `"For_Automation"`).
2. **Send via Slack** using `mcp__slack__slack_post_message`:
   - `channel_id`: `C094E6V7ELS`
   - `text`: the full command string

## Examples

### Add SA beneficiary transfer context (MX):
```
<@U08JT4ML6HL> --run_on=sa-mx-stg --action=/context/add --json_data=[{"payload":{},"saId":"9812033179085","consumer_data":{"passCode":"1357","cellPhone":"+27833095235"},"note":"","isInUse":"true","context":"beneficiary_transfer"}]
```

### Add PH intra-transfer context (MX):
```
<@U08JT4ML6HL> --run_on=ph-mx-stg --action=/context/add --json_data=[{"Account":{"receive_info":"0164 0866 4847","account_name":"LE P."},"isInUse":"false","Mobile":{"account_name":"Naruto U.","receive_info":"119 039 7464"},"context":"intra_transfer_happy_case","userName":"intratransfer","bank_name":"GoTyme Bank","Email":{"receive_info":"smmdemotestuat70@tyme.com","account_name":"Madara U."}}]
```

### Get linked device:
```
<@U08JT4ML6HL> --run_on=sa-mx-stg --action=/device/get/linked/device --username=7011250013083
```

## Input Validation

Before processing any request, validate these **required** inputs. If any are missing, **stop and ask the user** before proceeding:

1. **Country / Environment (`--run_on`)** — REQUIRED. The user must explicitly specify which environment to target. Do NOT guess or default. Ask:
   > "Which environment should I use? (e.g., `sa-mx-stg`, `ph-mx-stg`, `sa-stg`)"
2. **Action** — REQUIRED. Determine from the user's intent (add/get/delete). If ambiguous, ask.
3. **Context name** — REQUIRED for add/get/delete. The user must explicitly provide the context identifier. Do NOT guess or default. Ask:
   > "What is the context name? (e.g., `beneficiary_transfer`, `intra_transfer_happy_case`, `transfer_multiple_eda`)"
4. **Data fields** — REQUIRED for add. The account/consumer data. If incomplete, list what's missing and ask.

### Country-to-Environment Mapping

Use this to help users pick the right `--run_on` value:

| Country | Default MX STG | Default MX TST | Non-MX STG | Non-MX TST |
|---------|---------------|---------------|------------|------------|
| **South Africa (SA)** | `sa-mx-stg` | `sa-mx-tst` | `sa-stg` | `sa-tst` |
| **Philippines (PH)** | `ph-mx-stg` | `ph-mx-tst` | `ph-stg` | `ph-tst` |
| **TymeBusiness (TB)** | — | — | `tb-stg` | `tb-tst` |
| **TymeConnect (TC)** | — | — | `tc-stg` | `tc-tst` |

If the user says "SA" or "South Africa", ask whether they mean MX or non-MX, and STG or TST.

## Workflow

When the user invokes `/automation-context`:

1. **Validate required inputs** (see Input Validation above). If the environment or any required field is missing, **ask the user** — do not proceed without them.

2. **Validate the environment** against the valid environments table above. Reject invalid values and suggest the closest match.

3. **Format the command**:
   - JSON data must be a **single-line compact string** (use `JSON.stringify` with no spaces).
   - Wrap the full command as: `<@U08JT4ML6HL> --run_on=<env> --action=<action> --json_data=<json>`

4. **Send the message** to channel `C094E6V7ELS` using `mcp__slack__slack_post_message`.

5. **Wait briefly**, then read the thread reply using `mcp__slack__slack_get_thread_replies` to check MarioBot's response.

6. **Report the result** to the user.

## Safety

- **NEVER** modify or fabricate test account credentials — only use data provided by the user.
- Always validate the environment before sending.
- If the user provides ambiguous input, ask for clarification before sending.
- Show the user the exact command that will be sent before posting to Slack.
