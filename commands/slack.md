# Slack Messaging Workflow

## Channel Name to ID Mapping

Use this mapping to resolve channel names to IDs **without** calling `slack_list_channels`.

| Channel Name | Channel ID |
|---|---|
| #zeno-testing | C073TT0GB6U |
| #mobile-transfer-squad | C07CEJ4FXPS |
| #project-susanoo | C0A8YPX211N |

## Send Message Flow

When the user asks to **send a Slack message**, proceed immediately without asking for confirmation:

1. **Resolve the channel ID** from the mapping table above:
   - Match the channel name the user mentioned (with or without `#` prefix) against the **Channel Name** column.
   - If found, use the corresponding **Channel ID** directly. **Do NOT call `slack_list_channels`.**
   - If **not found** in the mapping, call `mcp__slack__slack_list_channels` to look it up. Then tell the user:
     > "Channel `#{name}` is not in the skill mapping. Add it to `slack.md` to skip the lookup next time. The channel ID is `{id}`."

2. **Send the message** using `mcp__slack__slack_post_message`:
   - `channel_id`: the resolved channel ID
   - `text`: the message content

   If the user did not provide explicit message content, generate an appropriate message from the conversation context.

## Reply to Thread Flow

When the user asks to **reply to a Slack thread**:

1. **Resolve the channel ID** using the same mapping logic from step 1 above.

2. **Reply to the thread** using `mcp__slack__slack_reply_to_thread`:
   - `channel_id`: the resolved channel ID
   - `thread_ts`: the thread timestamp (user must provide this, or extract from a Slack message URL)
   - `text`: the reply content

## Slack Message URL Parsing

If the user provides a Slack message URL, extract the channel ID and thread timestamp from it:
- URL format: `https://{workspace}.slack.com/archives/{channel_id}/p{timestamp}`
- The `p{timestamp}` part needs conversion: insert a `.` so that 6 digits come after it.
  - Example: `p1234567890123456` -> thread_ts = `1234567890.123456`

## Examples

Send a message:
```
/slack send #zeno-testing "Build passed for PNL-12345"
```

Reply to a thread:
```
/slack reply #zeno-testing 1234567890.123456 "Fix has been merged"
```

Send from context (no explicit message):
```
/slack send #zeno-testing
```
