---
name: powerdrill-data-analysis
description: This skill should be used when the user wants to analyze, explore, visualize, or query data using the Powerdrill MCP server. Covers listing, creating, and deleting datasets; uploading local files as data sources; creating analysis sessions; running natural-language data analysis queries; and retrieving charts, tables, and insights. Triggers on requests like "analyze my data", "query my dataset", "upload this file for analysis", "list my datasets", "create a dataset", "visualize sales trends", "continue my previous analysis", "delete this dataset", or any data exploration task mentioning Powerdrill or targeting an available Powerdrill MCP connection.
---

# Powerdrill Data Analysis MCP Skill

The Powerdrill Data Analysis MCP server provides data analysis capabilities through the tools below. Use them to explore, analyze, and manage data on the Powerdrill platform.

## Prerequisites & Setup

Before using any Powerdrill tools, the user must have:

1. **A Powerdrill Teamspace** - Created by following the setup tutorial at: https://www.youtube.com/watch?v=I-0yGD9HeDw
2. **API Credentials** - Obtained by following the API key tutorial at: https://www.youtube.com/watch?v=qs-GsUgjb1g

The following environment variables must be configured in the MCP server settings:
- `POWERDRILL_USER_ID` - The user's Powerdrill User ID
- `POWERDRILL_PROJECT_API_KEY` - The user's Powerdrill Project API Key

If a tool call fails with an authentication error or missing credentials, instruct the user to:
1. Verify their `POWERDRILL_USER_ID` and `POWERDRILL_PROJECT_API_KEY` are correctly set
2. Watch the setup videos above if they haven't created a Teamspace or obtained API keys yet

### MCP Server Configuration

The Powerdrill MCP server can be added to Claude Code's MCP configuration. Example `.mcp.json`:

```json
{
  "mcpServers": {
    "powerdrill": {
      "command": "npx",
      "args": ["-y", "@powerdrillai/powerdrill-mcp"],
      "env": {
        "POWERDRILL_USER_ID": "your_user_id",
        "POWERDRILL_PROJECT_API_KEY": "your_project_api_key"
      }
    }
  }
}
```

## Available Tools

### 1. `mcp_powerdrill_list_datasets`
Lists available datasets from the user's Powerdrill account.

**Parameters:**
- `limit` (number, optional) - Maximum number of datasets to return
- `pageNumber` (number, optional) - Page number (default: 1)
- `pageSize` (number, optional) - Items per page (default: 10)
- `search` (string, optional) - Search datasets by name

**When to use:** When the user wants to see what datasets they have, find a specific dataset, or browse available data. This is typically the first step in any analysis workflow.

### 2. `mcp_powerdrill_get_dataset_overview`
Retrieves detailed overview information for a specific dataset including summary, exploration questions, and keywords.

**Parameters:**
- `datasetId` (string, required) - The ID of the dataset

**When to use:** When the user wants to understand what's in a dataset, get suggested exploration questions, or see a dataset summary. Often used after listing datasets.

### 3. `mcp_powerdrill_list_data_sources`
Lists individual data sources (files) within a specific dataset.

**Parameters:**
- `datasetId` (string, required) - The ID of the dataset
- `pageNumber` (number, optional) - Page number (default: 1)
- `pageSize` (number, optional) - Items per page (default: 10)
- `status` (string, optional) - Filter by status: `synching`, `invalid`, `synched` (comma-separated for multiple)

**When to use:** When the user wants to see which files are in a dataset, check data source sync status, or identify specific data sources for targeted analysis.

### 4. `mcp_powerdrill_create_session`
Creates a new analysis session to group related analysis jobs together. Sessions maintain context across multiple queries.

**Parameters:**
- `name` (string, required) - Session name (up to 128 characters)
- `output_language` (string, optional) - Output language. Options: `AUTO`, `EN`, `ES`, `AR`, `PT`, `ID`, `JA`, `RU`, `HI`, `FR`, `DE`, `VI`, `TR`, `PL`, `IT`, `KO`, `ZH-CN`, `ZH-TW` (default: `AUTO`)
- `job_mode` (string, optional) - `AUTO` or `DATA_ANALYTICS` (default: `AUTO`)
- `max_contextual_job_history` (number, optional) - Max recent jobs retained as context, 0-10 (default: 10)
- `agent_id` (string, optional) - Agent ID. Currently only `DATA_ANALYSIS_AGENT` is available (default: `DATA_ANALYSIS_AGENT`)

**When to use:** Before running analysis jobs. A session must be created first to provide a `session_id` for `create_job`. Create a new session for each new analysis topic or conversation thread.

### 5. `mcp_powerdrill_create_job`
Executes a data analysis query using natural language against a dataset. This is the primary analysis tool.

**Parameters:**
- `question` (string, required) - Natural language question or prompt to analyze the data
- `dataset_id` (string, required) - The ID of the dataset to analyze
- `session_id` (string, required) - Session ID to group related jobs (from `create_session`)
- `datasource_ids` (array of strings, optional) - Specific data source IDs within the dataset to analyze
- `stream` (boolean, optional) - Whether to stream results (default: false)
- `output_language` (string, optional) - Output language (default: `AUTO`)
- `job_mode` (string, optional) - Job mode (default: `AUTO`)

**When to use:** When the user asks a data analysis question. This is the core tool for getting insights from data. Always ensure you have a valid `session_id` (create one if needed) and `dataset_id` before calling this tool.

**Response format:** Returns blocks of content that may include:
- `TEXT` blocks - Analytical text responses
- `TABLE` blocks - URLs to generated table results
- `IMAGE` blocks - URLs to generated chart/visualization images

### 6. `mcp_powerdrill_list_sessions`
Lists existing analysis sessions.

**Parameters:**
- `pageNumber` (number, optional) - Page number (default: 1)
- `pageSize` (number, optional) - Items per page (default: 10)
- `search` (string, optional) - Search sessions by name

**When to use:** When the user wants to continue a previous analysis session or see their analysis history.

### 7. `mcp_powerdrill_create_dataset`
Creates a new empty dataset to organize data sources.

**Parameters:**
- `name` (string, required) - Dataset name (up to 128 characters)
- `description` (string, optional) - Dataset description (up to 128 characters)

**When to use:** When the user wants to create a new dataset to upload files into.

### 8. `mcp_powerdrill_create_data_source_from_local_file`
Uploads a local file to a dataset as a new data source. Handles multipart upload and waits for sync completion.

**Parameters:**
- `dataset_id` (string, required) - The ID of the dataset to upload into
- `file_path` (string, required) - Local path to the file to upload
- `file_name` (string, optional) - Custom name for the file (defaults to original filename)
- `chunk_size` (number, optional) - Upload chunk size in bytes (default: 5MB). Increase for large files to reduce upload overhead; decrease if uploads fail on slow connections.

**When to use:** When the user wants to upload a file for analysis. The tool handles the full upload process including chunked upload and sync status polling.

### 9. `mcp_powerdrill_delete_dataset`
Permanently deletes a dataset and all its data sources.

**Parameters:**
- `datasetId` (string, required) - The ID of the dataset to delete

**When to use:** When the user explicitly wants to remove a dataset. This action is irreversible and destroys all data sources within the dataset. Always confirm with the user before proceeding, stating exactly which dataset will be deleted.

## Recommended Workflows

### Analyzing existing data
1. `list_datasets` - Find the target dataset
2. `get_dataset_overview` - Understand the data and get suggested questions
3. `create_session` - Create an analysis session
4. `create_job` - Ask analysis questions against the dataset
5. Continue asking follow-up questions using the same session for context continuity

### Uploading and analyzing new data
1. `create_dataset` - Create a new dataset
2. `create_data_source_from_local_file` - Upload data file(s)
3. `get_dataset_overview` - Review the processed data
4. `create_session` - Start an analysis session
5. `create_job` - Begin analysis

### Exploring data sources
1. `list_datasets` - Browse datasets
2. `list_data_sources` - See files within a dataset
3. `get_dataset_overview` - Get summary and suggested exploration questions

### Resuming a previous analysis
1. `list_sessions` - Find the previous session by name
2. `list_datasets` - Confirm the target dataset is still available
3. `create_job` - Continue asking questions using the existing `session_id`

### Cleaning up data
1. `list_datasets` - Identify datasets to remove
2. `delete_dataset` - Delete after explicit user confirmation

## Error Handling

- **Authentication errors:** Verify `POWERDRILL_USER_ID` and `POWERDRILL_PROJECT_API_KEY` are set correctly. Direct the user to the setup videos if credentials are missing.
- **Dataset not found:** Re-run `list_datasets` to verify the dataset ID. The dataset may have been deleted.
- **Job execution failure:** Check that the dataset has at least one data source with status `synched`. Retry with a rephrased question if the error is ambiguous.
- **Upload timeout:** The `create_data_source_from_local_file` tool polls for sync completion up to 20 attempts. If it times out, use `list_data_sources` with `status=synching` to check progress manually.
- **Rate limiting:** If API calls return rate-limit errors, wait before retrying. Space out rapid sequential tool calls.

## Important Notes

- Always create a session before running analysis jobs
- Sessions maintain conversational context - reuse the same session for related follow-up questions
- The `create_job` response may contain URLs to tables and images that expire. Advise users to download or save results promptly
- When uploading files, the tool polls until the data source is fully synced before returning
- Dataset and session names are limited to 128 characters
- If a data source shows status `synching`, wait before running analysis - it needs to be `synched` first
