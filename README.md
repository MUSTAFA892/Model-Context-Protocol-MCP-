# MCP Server Setup Guide

## Overview

The **Model Context Protocol (MCP)** enables applications to provide context for Large Language Models (LLMs) in a standardized and structured way. By using MCP, applications can expose a variety of resources, tools, and prompts that LLMs can consume in order to perform tasks efficiently. This repository demonstrates how to set up an MCP server, integrate it with services like Claude Desktop, and utilize the functionalities of MCP. 

## What is MCP?

MCP is a protocol designed to expose resources, tools, and prompts in a structured way, providing LLMs with the necessary context to execute tasks. The protocol operates similarly to a REST API, but is specifically tailored for LLMs and machine learning models. 

- **Resources**: Data that is exposed and accessed, like API endpoints.
- **Tools**: Functions that LLMs can execute, similar to POST requests.
- **Prompts**: Reusable templates to interact with LLMs.

MCP also supports advanced functionalities such as database integration, image processing, and lifecycle management. 

This guide walks you through the setup process for an MCP server, including environment configuration, server implementation, and integration with Claude Desktop.

---

### Example Image of MCP Server Configuration:

![MCP Server Configuration](https://github.com/alejandro-ao/mcp-server-example/blob/master/img/mcp-diagram-bg.png)


## Prerequisites

Before setting up the MCP server, make sure you have the following:

1. **Python 3.7+** (Recommended Python 3.8 or later)
2. **Claude Desktop** (Download from [Claude Desktop](https://claude.app/)) for interacting with MCP servers.
3. **PowerShell** (Windows) or **Terminal** (macOS/Linux) to run installation commands.
4. **uv** (Universal Virtual Environment Manager) tool for managing project environments.

---

## Step 1: Installing **uv** (Universal Virtual Environment)

**uv** is a tool used to initialize and manage Python environments. To install **uv**, open PowerShell as an Administrator and run the following command:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

---

## Step 2: Initializing a New **uv** Project

After installing **uv**, you can create a new project using the following command:

```powershell
uv init <project_name>
```

Replace `<project_name>` with the desired name for your project. This command will set up a new project directory and initialize the necessary files for your MCP server.

---

## Step 3: Creating and Activating the Virtual Environment

Once the project is initialized, create a virtual environment using **uv**:

```powershell
uv venv
uv activate
```

This ensures that your project dependencies will be isolated from the rest of your system.

---

## Step 4: Installing Required Packages

Now you need to install the necessary dependencies. Run the following command to install the MCP package and related dependencies:

```powershell
uv add "mcp[cli]" httpx
pip install mcp
```

These packages will allow you to set up the MCP server and make HTTP requests in your tools.

---

## Step 5: Writing Your MCP Server Code

Create a new Python file, such as `server.py`, and add the following code to initialize the MCP server:

```python
from mcp.server.fastmcp import FastMCP

# Create an MCP server instance
mcp = FastMCP("Demo Server")

# Add a tool for adding two numbers
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

# Add a dynamic greeting resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting"""
    return f"Hello, {name}!"
```

In this example, the server exposes a tool for adding two numbers and a resource to generate a personalized greeting.

---

## Step 6: Running Your MCP Server

To run your MCP server, execute the following command:

```powershell
python server.py
```

Alternatively, you can use **uv** to run the server:

```powershell
uv run server.py
```

Your MCP server is now running locally and ready to be integrated with LLM clients like Claude Desktop.

---

## Integrating with **Claude Desktop**

To interact with your MCP server via Claude Desktop, follow these steps:

### 1. **Install Claude Desktop**

Download and install the **Claude Desktop** application from [Claude Desktop](https://claude.app/).

### 2. **Configure MCP Server in Claude Desktop**

Once Claude Desktop is installed, open it and go to **Files > Settings > Developer**. In the **Claude Desktop Configuration** (`claude_desktop_configuration`), add the following configuration:

```json
{
  "mcpServers": {
    "mcp-server": {
      "command": "C:/Users/YourUser/.local/bin/uv",
      "args": [
        "--directory",
        "C:/Projects/MCP-Implementation",
        "run",
        "server.py"
      ]
    }
  }
}
```

Make sure to adjust the paths based on your system configuration. Restart Claude Desktop for the changes to take effect.

---

## Step 7: Using MCP Client in Claude Desktop

Once the server is running, you can interact with it via the **MCP Client** in Claude Desktop. You should see a notification indicating that the "MCP server is active". From here, you can query the exposed resources and tools.

---

## Step 8: Debugging with MCP Inspector

For testing and debugging your server, use the **MCP Inspector**. It provides a GUI to interact with your MCP server and allows you to inspect the functionality of your resources and tools.

To run the inspector, use the following command:

```bash
mcp dev server.py
```

Alternatively, you can use **npx** for quicker setup:

```bash
npx @modelcontextprotocol/inspector uv run server.py
```

The inspector will show the tools and resources available in your MCP server and allow you to interact with them directly.

---

## Step 9: MCP Cursor Client

The **Cursor Client** lets you interact with MCP servers via the command line. Press `Ctrl + Shift + P` in your IDE or terminal and search for **MCP Servers**. Once you have added your server, you can start typing queries to interact with the resources and tools exposed by the server.

---

## Advanced Examples

### Example 1: Simple Calculator Tool

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Calculator")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

@mcp.tool()
def subtract(a: int, b: int) -> int:
    """Subtract two numbers."""
    return a - b
```

### Example 2: Weather Fetcher Tool Using HTTPX

```python
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Weather App")

@mcp.tool()
async def fetch_weather(city: str) -> str:
    """Fetch current weather for a city."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.weather.com/{city}")
        return response.text
```

### Example 3: Database Resource Querying

```python
import sqlite3
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("SQLite Explorer")

@mcp.resource("schema://main")
def get_schema() -> str:
    """Provide the database schema."""
    conn = sqlite3.connect("database.db")
    schema = conn.execute("SELECT sql FROM sqlite_master WHERE type='table'").fetchall()
    return "\n".join(sql[0] for sql in schema if sql[0])
```

### Example 4: Image Processing Tool

```python
from mcp.server.fastmcp import FastMCP, Image
from PIL import Image as PILImage

mcp = FastMCP("Image Processor")

@mcp.tool()
def create_thumbnail(image_path: str) -> Image:
    """Create a thumbnail from an image."""
    img = PILImage.open(image_path)
    img.thumbnail((100, 100))
    return Image(data=img.tobytes(), format="png")
```
Certainly! Here's the folder structure and the steps for setting up the environment, installing dependencies, and working through the repository:

---

### Folder Structure

Here's a sample folder structure for your MCP server project:

```
/MCP-Server-Project
├── /src
│   ├── server.py                 # Main server code
│   ├── utils.py                  # Helper functions
│   └── config.py                 # Configuration file
├── /assets                       # Folder for storing assets like images
│   └── image_example.png
├── /env                           # Virtual environment folder
├── mcp.json                       # MCP configuration file
└── requirements.txt               # List of dependencies
```

### Steps to Set Up the Environment and Install Dependencies

#### Step 1: Clone the Repository

Clone your repository to the local machine:

```bash
git clone <your_repo_url>
cd <your_repo_name>
```

#### Step 2: Initialize and Activate the UV Environment

1. Install **UV** (Universal Virtual Environment) if you haven't already:

   ```powershell
   PowerShell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

2. Navigate to the project folder and initialize a new UV environment:

   ```bash
   uv init <project_name>
   ```

3. Create and activate the virtual environment:

   ```bash
   uv venv
   uv activate
   ```

#### Step 3: Install Dependencies

Install the necessary dependencies from the `requirements.txt`:

```bash
pip install -r requirements.txt
```

Alternatively, if you're adding dependencies like `httpx` or `mcp`, run:

```bash
uv add "mcp[cli]" httpx
```

#### Step 4: Set Up the MCP Server

Create or edit the `server.py` to define your server logic, resources, and tools. If you're cloning from a template, make sure `server.py` exists and is set up as per your needs.

---

Once the above steps are complete, you can start working with the MCP server code as described in previous prompts, including running the server and configuring it for use with Claude Desktop or other MCP clients.

---

## Conclusion

Setting up an MCP server allows you to expose resources, tools, and prompts that can be queried by LLM clients like Claude. By following the steps outlined in this guide, you should be able to configure and run your MCP server, integrate it with Claude Desktop, and test it using tools like the MCP Inspector and Cursor.

With MCP, you can create interactive, scalable applications that can be easily integrated with LLMs for a variety of use cases, including data querying, image processing, and much more.

---

### License

This project is licensed under the MIT License.


---

For more detailed information and examples, check the official [MCP Documentation](https://modelcontextprotocol.io/) or join the community discussions on GitHub.

---

