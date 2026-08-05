# fetch_blob

Download binary content from a WorkIQ path. The tool returns up to 4 MB of file bytes in an in-band JSON envelope with `statusCode`, `sizeBytes`, `base64Content`, `error`, and `requestId`. Use this for file content, email attachments, document downloads, profile photos, and other binary Microsoft 365 resources.

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | The relative WorkIQ path to the binary resource (e.g., `/me/drive/items/{id}/content`, `/me/messages/{id}/attachments/{attachmentId}/$value`). Do not include a base URL. |
| `format` | string | No | A `$format` conversion value such as `pdf`; honored only on compatible drive-content endpoints. |
| `agentId` | string | No | Target a specific M365 Copilot agent. |

## When to Use

- Downloading a file from OneDrive or SharePoint
- Retrieving an email attachment
- Downloading exported content

Distinguish from `fetch`: use `fetch_blob` when the path returns binary content (files, raw attachment bytes). Use `fetch` when the path returns JSON.

## Path Conventions

| Resource | Path pattern |
|----------|-------------|
| OneDrive file content | `/me/drive/items/{id}/content` |
| SharePoint file content | `/drives/{driveId}/items/{id}/content` |
| Email attachment (raw) | `/me/messages/{id}/attachments/{attachmentId}/$value` |
| Profile photo | `/me/photo/$value`, `/users/{id}/photo/$value` |

## Workflow

1. Use `fetch` to list items and retrieve their IDs (e.g., `/me/drive/root/children`)
2. Use `fetch_blob` with the content path to download the binary data.
3. Check `statusCode` before reading or decoding `base64Content`.
4. Decode `base64Content` only when the host needs to materialize the returned bytes locally.

On a non-200 response, do not retry path variants. For access denied, return the file's `webUrl` or the parent message's `webLink`; for profile photos, report the policy denial. For payloads over 4 MB, return the file's `webUrl`. For other errors, report `error` and `requestId`.

Never fabricate binary content or download URLs.

## Examples

### Download a file from OneDrive by item ID
```json
{ "path": "/me/drive/items/{id}/content" }
```

### Download an email attachment
```json
{ "path": "/me/messages/{messageId}/attachments/{attachmentId}/$value" }
```

### Download a file from a shared drive
```json
{ "path": "/drives/{driveId}/items/{itemId}/content" }
```

### Download a drive item converted to PDF
```json
{
  "path": "/me/drive/items/{id}/content",
  "format": "pdf"
}
```
