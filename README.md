# Groniz Authentication: CLI, MCP, and REST API Compared

Groniz authentication depends on the interface you are using. MCP accepts a Bearer authorization header, while the public REST API expects the API key directly in the `Authorization` header. The CLI resolves its own credential sources, with stored credentials taking precedence over environment configuration.

When one interface works and another fails, test the failing route in its actual runtime. Reusing a header from a working example can introduce the error you are trying to fix.

The [AI agent setup guide](https://groniz.com/blog/ai-agent-social-media-publishing-setup-a-client-by-client-guide) covers client selection. Use the checks below to identify which credentials the failing interface uses and record a diagnosis without exposing them.

## Match credentials to the interface

Use this matrix when configuring a client or reviewing an integration. Supply a real key in place of `YOUR_API_KEY` through your approved credential mechanism. Keep it out of the repository.

| Interface | Credential route | Important distinction | Useful first verification |
| --- | --- | --- | --- |
| Groniz CLI | Stored credentials, then environment configuration, then defaults | Stored credentials can override `GRONIZ_API_KEY` | `groniz whoami` in the actual runtime |
| Groniz MCP | Bearer header, API key in URL, or OAuth | Header form is `Authorization: Bearer YOUR_API_KEY` | Read-only account discovery through that client |
| Groniz public REST API | API key directly in `Authorization` | Header form is `Authorization: YOUR_API_KEY` | A documented read-only request on that API route |

The MCP server endpoint is `https://mcp.groniz.com/mcp`. Use the bare API key for REST requests and the Bearer form for MCP headers. Use the public API documentation for the exact endpoint you intend to call rather than deriving a route from an MCP tool name.

Groniz also handles OAuth connections to social platforms. Keep that destination connection distinct from authenticating your AI client or script to Groniz. A client may reach Groniz successfully and still need the intended social account connected.

## Verify CLI identity where execution happens

For an interactive CLI setup, the documented login and identity commands are:

```bash
groniz auth:login
groniz whoami
```

A setup that supplies credentials through the environment can use `GRONIZ_API_KEY`. However, if that runtime already has stored credentials, they take precedence. After setting a different environment key, run `whoami` to check which account the CLI actually uses.

Check identity before account discovery:

```bash
groniz whoami
groniz integrations:list
```

Compare the returned account and integration IDs with the task's intended destination. If they differ, inspect the credential source in that runtime. Use the CLI's supported credential management for any change; avoid improvising configuration deletion during a publishing task.

Run these checks where the publishing command will execute. A host terminal, container, automation process, and editor agent may have different credential sources. Record which one produced the result.

## Match MCP syntax to the client

For an MCP connection that uses a header, its resolved value must contain `Bearer` followed by the Groniz API key. How a client inserts an environment variable into that header depends on its configuration format.

For example, the [Hermes MCP guide](https://groniz.com/blog/how-to-connect-hermes-agent-to-social-media-with-mcp) shows the syntax for Hermes. Copying variable interpolation syntax between clients without checking their configuration can leave the client sending an unresolved placeholder or an empty value.

Verify the client's credential source and then request read-only integration discovery. Check this MCP connection even if CLI authentication already works.

If you use Groniz's supported API-key-in-URL route, treat the entire configured URL as sensitive. Keep it out of shared logs and screenshots. Remove the embedded key from any URL recorded for diagnosis, including keys in the URL path or query. OAuth is another supported MCP route; check the state of the actual client connection when using it.

## Keep REST authentication explicit

The public REST header is:

```text
Authorization: YOUR_API_KEY
```

The MCP Bearer header is:

```text
Authorization: Bearer YOUR_API_KEY
```

These examples show the header sent with each request. Store and supply the secret using your deployment's approved mechanism. Avoid embedding it in a command that will be copied into an issue, shell transcript, or shared runbook.

If a REST request fails, inspect that request's documented endpoint and authentication format. Do not infer a token scope, expiry behavior, or rotation endpoint from an error message alone. Those details need their own documentation.

## Leave a diagnostic note

Record the result without credentials, for example:

```text
Runtime: publishing worker container
Interface: Groniz CLI
Credential source category: stored credentials detected
Check: groniz whoami
Result: authenticated; returned account differs from intended account
Destination check: not attempted
Next action: resolve credential selection in this runtime
Secrets included: none
```

Include the interface, runtime, credential source category, observed result, and account comparison. Omit keys, authorization headers, and URLs that contain credentials.

After authentication works, retrieve the selected integration's current settings before preparing a post. Review the content, media, destination, and schedule before authorizing a delivery. The [MCP, CLI, skill, and REST comparison](https://groniz.com/blog/mcp-vs-cli-vs-skill-vs-rest-api-for-social-publishing) can help you choose the interface for that workflow.

To finish the destination setup, [connect the intended social account in Groniz](https://groniz.com/console/connectors), then confirm it appears through the same route your executor will use.
