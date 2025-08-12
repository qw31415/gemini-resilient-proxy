# Gemini Resilient Proxy

A resilient Cloudflare Worker proxy for Google Gemini, featuring multi-group routing and robust stream interruption handling.

This proxy is designed to sit between your application and your Gemini API upstream service. It intelligently handles common stream interruptions (e.g., `SAFETY`, network drops) by automatically retrying with the accumulated context, ensuring your users receive a complete and uninterrupted response.

## ✨ Features

- **Resilient Streaming**: Automatically retries on stream interruptions, providing a seamless experience.
- **Multi-Group Routing**: Dynamically routes requests to different upstream paths based on request attributes.
- **Flexible Group Selection**: Specify a routing group via API key prefix, custom header, URL parameter, or subdomain.
- **Configurable**: Easily configure upstream URLs, default groups, and other settings via Cloudflare environment variables.
- **Serverless**: Deploys in seconds to Cloudflare's global network for ultra-low latency.
- **Debug Mode**: Built-in debug logging for easy troubleshooting.

## 🚀 Deployment

1.  **Create a Cloudflare Worker**:
    - Log in to your Cloudflare dashboard.
    - Navigate to `Workers & Pages` > `Create application` > `Create Worker`.
    - Give your worker a name (e.g., `gemini-proxy`).

2.  **Copy the Code**:
    - Copy the entire content of `index.js` from this repository.
    - In the Cloudflare Worker editor, paste the code, replacing any boilerplate.

3.  **Configure Environment Variables**:
    - In your Worker's settings, go to `Settings` > `Variables`.
    - Add the following environment variables (all are optional):
      - `UPSTREAM_URL_BASE`: The base URL of your upstream service (e.g., `https://your-gemini-provider.com`). Defaults to the placeholder in the code.
      - `DEFAULT_GROUP`: The default routing group to use if none is specified (e.g., `default`).
      - `MAX_CONSECUTIVE_FAILURES`: The number of times to retry before giving up (e.g., `5`).
      - `DEBUG_MODE`: Set to `true` to enable console logging for debugging.

4.  **Save and Deploy**:
    - Click `Save and Deploy`. Your proxy is now live!

## ⚙️ Configuration

### Group Mapping

All routing logic is controlled by the `GROUP_MAPPING` object in `index.js`. To add, remove, or change groups, simply edit this object:

```javascript
const GROUP_MAPPING = {
  'default': '/proxy/default-group',
  'group-a': '/proxy/group-a',
  'group-b': '/proxy/group-b',
  // Add your new group here
  'my-new-group': '/proxy/my-new-group'
};
```

## 🔌 Usage

You can specify which upstream group to use in your API requests in four ways, listed by priority:

1.  **API Key Prefix (Highest Priority)**:
    - Prepend the group name to your API key, separated by a colon.
    - **Example**: `group-a:sk-xxxxxxxxxx`

2.  **Custom Header**:
    - Add an `X-Proxy-Group` header to your request.
    - **Example**: `X-Proxy-Group: group-b`

3.  **URL Parameter**:
    - Add a `group` query parameter to your request URL.
    - **Example**: `https://your-worker.workers.dev/v1beta/models/gemini-pro:streamGenerateContent?group=group-c`

4.  **Subdomain (Lowest Priority)**:
    - If you have custom domains configured for your worker, the first part of the hostname can be used as the group name.
    - **Example**: `https://group-a.your-domain.com`

If no group is specified, the `DEFAULT_GROUP` will be used.

### Example `curl` Request

```bash
# Using API Key Prefix to select 'group-a'
curl -X POST "https://your-worker.workers.dev/v1beta/models/gemini-pro:streamGenerateContent?alt=sse" \
-H "Content-Type: application/json" \
-d '{
  "contents": [{
    "parts": [{"text": "Why is the sky blue?"}]
  }],
  "generationConfig": {
    "temperature": 0.9
  },
  "apiKey": "group-a:YOUR_API_KEY"
}'
```

---

*This proxy is designed to enhance the reliability of Gemini API streams. Please use it responsibly.*
