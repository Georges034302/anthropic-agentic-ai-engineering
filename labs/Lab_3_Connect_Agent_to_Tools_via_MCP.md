
# Lab 3: Connect an Agent to Tools via MCP (Pro Version)

## Objective
Build or configure a Model Context Protocol (MCP) server and client. Expose a tool (e.g., calculator, file search) and have an agent use it through the protocol. This lab will guide you through:
- Setting up a minimal MCP server and client
- Exposing and discovering tools
- Logging and debugging the agent-tool interaction

---

## Step 1: Set Up MCP Server

We'll use a simple Python implementation with `json-rpc` for demonstration. Install dependencies:
```bash
pip install json-rpc
```


**mcp_server.py**

```python
# Import required libraries
from jsonrpc import JSONRPCResponseManager, dispatcher
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
```

```python
# Example tool: Calculator (exposed via MCP)
@dispatcher.add_method
def add(a, b):
	# Add two numbers and return the result
	return a + b
```

```python
# HTTP handler for MCP JSON-RPC requests
class MCPHandler(BaseHTTPRequestHandler):
	def do_POST(self):
		# Read and decode the incoming JSON-RPC request
		content_length = int(self.headers['Content-Length'])
		request_json = self.rfile.read(content_length).decode()
		# Handle the request using the dispatcher
		response = JSONRPCResponseManager.handle(request_json, dispatcher)
		# Send the JSON-RPC response
		self.send_response(200)
		self.send_header('Content-type', 'application/json')
		self.end_headers()
		self.wfile.write(json.dumps(response.data).encode())
```

```python
# Start the MCP server
def run(server_class=HTTPServer, handler_class=MCPHandler, port=4000):
	server_address = ('', port)
	httpd = server_class(server_address, handler_class)
	print(f'Starting MCP server on port {port}...')
	httpd.serve_forever()
```

```python
# Entry point for running the MCP server
if __name__ == '__main__':
	run()
```

---

## Step 2: Implement MCP Client (Agent)


**mcp_client.py**

```python
# Import required libraries
import requests
import json
```

```python
# Function to call a tool via MCP (JSON-RPC)
def call_tool(method, params):
	payload = {
		"jsonrpc": "2.0",
		"method": method,
		"params": params,
		"id": 1
	}
	# Send the JSON-RPC request to the MCP server
	response = requests.post("http://localhost:4000", json=payload)
	return response.json()
```

```python
# Entry point for running the MCP client (agent)
if __name__ == "__main__":
	# Discover tool (hardcoded for demo)
	print("Calling add tool via MCP...")
	result = call_tool("add", {"a": 2, "b": 3})
	print("Response:", result)
```

---

## Step 3: Run and Test

1. Start the MCP server:
   ```bash
   python mcp_server.py
   ```
2. In another terminal, run the client:
   ```bash
   python mcp_client.py
   ```

**Sample Output:**
```
Calling add tool via MCP...
Response: {'jsonrpc': '2.0', 'result': 5, 'id': 1}
```

---

## Step 4: Explanation
- **MCP Server:** Exposes a calculator tool via JSON-RPC.
- **MCP Client:** Discovers and invokes the tool, logging the request/response.
- **Extensible:** Add more tools by defining new methods on the server.

---

## Step 5: Deliverables
- MCP server and client code
- Tool definition (e.g., add)
- Sample agent run and output

---

## Pro Tips
- Use JSON-RPC for easy tool integration and debugging.
- Log all requests and responses for traceability.
- Extend the server with more tools (e.g., file search, weather lookup).

---

**Proceed to Lab 4 when you have a working MCP server, client, and sample run.**
