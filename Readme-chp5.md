Chapter 5
Integrating MCP (Model Context Protocol)
In earlier chapters, your agent learned to follow instructions, ground itself in your own data using File Search (RAG), and call custom tools.

In this final chapter, we'll connect your agent to a live MCP server - giving it access to external capabilities like live menus, toppings, and order management through a standard, secure protocol.

What Is MCP and Why Use It?
MCP (Model Context Protocol) is an open standard for connecting AI agents to external tools, data sources, and services through interoperable MCP servers.
Instead of integrating with individual APIs, you connect once to an MCP server and automatically gain access to all the tools that server exposes.

Benefits of MCP
🧩 Interoperability: a universal way to expose tools from any service to any MCP-aware agent.
🔐 Security & governance: centrally manage access and tool permissions.
⚙️ Scalability: add or update server tools without changing your agent code.
🧠 Simplicity: keep integrations and business logic in the server; keep your agent focused on reasoning.
Install the Azure AI Agents SDK (with MCP support)
First, make sure you have the latest SDK version that supports MCP integration.

bash
TypeCopy
pip install "azure-ai-agents>=1.2.0b5"
Then, update your imports in agent.py to include MCP-related classes and utilities:

python
TypeCopy
from azure.ai.agents.models import McpTool, ToolApproval, ThreadRun, RequiredMcpToolCall, RunHandler
import time
from typing import Any
Your full import section should now look like this:

python
TypeCopy
import os
import time
from typing import Any
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import (
    MessageRole, FilePurpose, FunctionTool, FileSearchTool, ToolSet,
    McpTool, ToolApproval, ThreadRun, RequiredMcpToolCall, RunHandler
)
from tools import calculate_pizza_for_people
from dotenv import load_dotenv
The Contoso Pizza MCP Server
For Contoso Pizza, the MCP server exposes APIs for:

🧀 Pizzas: available menu items and prices
🍅 Toppings: categories, availability, and details
📦 Orders: create, view, and cancel customer orders
You'll connect your agent to this server and grant it explicit permission to use a curated list of tools for these operations.

Create and Add the MCP Tool
You'll define the MCP Tool right before creating your ToolSet, alongside other tools (like File Search or the Pizza Calculator).

Add the MCP Tool
python
TypeCopy
# Add MCP tool so the agent can call Contoso Pizza microservices
mcp_tool = McpTool(
    server_label="contoso_pizza",
    server_url="https://ca-pizza-mcp-sc6u2typoxngc.graypond-9d6dd29c.eastus2.azurecontainerapps.io/sse",
    allowed_tools=[
        "get_pizzas",
        "get_pizza_by_id",
        "get_toppings",
        "get_topping_by_id",
        "get_topping_categories",
        "get_orders",
        "get_order_by_id",
        "place_order",
        "delete_order_by_id",
    ],
)
mcp_tool.set_approval_mode("never")
Then, add it to the toolset:

python
TypeCopy
toolset.add(mcp_tool)
Parameters Explained
Parameter	Description
server_label	A human-readable name for logs and debugging.
server_url	The MCP server endpoint.
allowed_tools	A whitelist of MCP tools your agent can call.
approval_mode	Defines whether calls require manual approval ("never" disables prompts).
In production, use more restrictive approval modes and scoped tool access.

Handling Tool Approvals
When the agent calls an MCP tool, you can intercept and approve these calls dynamically.
This gives you visibility and fine-grained control over what's executed.

For this we will create a custom run handler:

python
TypeCopy
# Custom RunHandler to approve MCP tool calls
class MyRunHandler(RunHandler):
    def submit_mcp_tool_approval(
        self, *, run: ThreadRun, tool_call: RequiredMcpToolCall, **kwargs: Any
    ) -> ToolApproval:
        print(f"[RunHandler] Approving MCP tool call: {tool_call.id} for tool: {tool_call.name}")
        return ToolApproval(
            tool_call_id=tool_call.id,
            approve=True,
            headers=mcp_tool.headers,
        )
This is added after defining and enabling the toolset, but before creating the agent.

Then, pass the handler when running the agent:

python
TypeCopy
run = project_client.agents.runs.create_and_process(
    thread_id=thread.id,
    agent_id=agent.id,
    run_handler=MyRunHandler()  # Enables controlled MCP approvals
)
🧠 Think of this as a middleware that intercepts all remote tool calls for logging, auditing, or dynamic security rules.

Add a User ID
To place orders, the agent must identify the customer.

Get your User ID
Open a new browser tab in Edge and navigate to: https://aka.ms/LAB571/registration.

Click on log in with Microsoft

You can login with:
Username:
User1-56904705@LODSPRODMSLEARNMCA.onmicrosoft.com
Access Token:
9ps7fxx^

You will get a permissions requested pop up. Click on Accept then on Grant Consent on the following page

Copy the resulting User ID before leaving the page!

Navigate back to VSCode to update your instructions.txt with your user details or pass the GUID in chat.

txt
TypeCopy
## User details:
Name: <YOUR NAME>
UserId: <YOUR USER GUID>
The name will be User1-56904705@LODSPRODMSLEARNMCA.onmicrosoft.com, while the UserId will be the ID you copied on step 4.

(Optional) View your order dashboard: Open a new browser tab in Edge and navigate to: https://aka.ms/LAB571/dashboard.

This dashboard will showcase a live-feed of orders as you run the agent. If you have time, you should come back to monitor the orders after you have interacted with your agent.

Trying It Out
Now it's time to test your connected agent!
Run the agent and try out these prompts:

TypeCopy
Show me the available pizzas.
TypeCopy
What is the price for a pizza hawai?
TypeCopy
Place an order for 2 large pepperoni pizzas.
The agent will automatically call the appropriate MCP tools, retrieve data from the live Contoso Pizza API, and respond conversationally - following your instructions.txt rules (e.g., tone, local currency, and time zone conversions).

Best Practices for MCP Integration
🔒 Principle of least privilege: only allow tools the agent truly needs.
📜 Observability: log all tool calls for traceability and debugging.
🔁 Resilience: handle connection errors gracefully and retry failed tool calls.
🧩 Versioning: pin MCP server versions to prevent breaking changes.
👩‍💼 Human-in-the-loop: use approval modes for sensitive actions (like order placement).
Recap
In this chapter, you:

Learned what MCP is and why it matters for scalable agent design.
Installed the updated Azure AI Agents SDK with MCP support.
Connected your agent to the Contoso Pizza MCP Server.
Implemented a custom run handler for tool approvals.
Tested real-time integration with menu, toppings, and order tools.
🎉 Congratulations - you've completed the workshop!
Your agent can now:
✅ Follow system instructions
✅ Access and reason over private data (RAG)
✅ Call custom tools
✅ Interact with live services via MCP

Your Contoso PizzaBot is now a fully operational, intelligent, and extensible AI assistant.
