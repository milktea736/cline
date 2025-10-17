# Built-in Tool Usage Guide

This guide summarizes the built-in tools registered in the system prompt toolkit and shows how to call each one using the XML syntax expected by the runtime.

## Registered Tools

The following tools are currently registered:

- `access_mcp_resource`
- `ask_followup_question`
- `attempt_completion`
- `browser_action`
- `execute_command` (exposed as `bash` for GPT-family prompts)
- `focus_chain`
- `list_code_definition_names`
- `list_files`
- `load_mcp_documentation`
- `new_task`
- `plan_mode_respond`
- `read_file`
- `replace_in_file`
- `search_files`
- `use_mcp_tool`
- `web_fetch`
- `write_to_file`

## Tool Reference

### `<access_mcp_resource>`
- **Description:** Request a resource from a connected MCP server (files, API responses, system info, etc.).
- **Availability:** Only when an MCP hub is connected.
- **Parameters:**
  - `<server_name>` (required) — Name of the MCP server that owns the resource.
  - `<uri>` (required) — URI of the resource to fetch.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <access_mcp_resource>
    <server_name>weather</server_name>
    <uri>resource://locations/123</uri>
    <task_progress>
      - Review requirements ✅
      - Fetch weather data ☐
    </task_progress>
  </access_mcp_resource>
  ```

### `<ask_followup_question>`
- **Description:** Ask the user for clarification or more detail when needed.
- **Availability:** Hidden when YOLO mode is toggled on.
- **Parameters:**
  - `<question>` (required) — Clear, specific question for the user.
  - `<options>` (optional) — JSON-style array of 2–5 selectable answers (never include an option to toggle Act mode).
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <ask_followup_question>
    <question>Which authentication method should we implement?</question>
    <options>["OAuth2", "SAML", "Passwordless email magic link"]</options>
  </ask_followup_question>
  ```

### `<attempt_completion>`
- **Description:** Deliver the final result once all tool runs are confirmed successful. Optionally include a demo command.
- **Availability:** Must only be used after the user confirms success of prior tool calls.
- **Parameters:**
  - `<result>` (required) — Final outcome summary for the user.
  - `<command>` (optional) — Safe CLI command to demonstrate the result (no `echo`/`cat`).
  - `<task_progress>` (optional, required if task progress tracking was used earlier) — Finalized checklist; depends on `focus_chain`.
- **Example:**
  ```xml
  <attempt_completion>
    <result>The API endpoint is implemented with full test coverage.</result>
    <command>npm run test:api</command>
    <task_progress>
      - Implement endpoint ✅
      - Write tests ✅
      - Document usage ✅
    </task_progress>
  </attempt_completion>
  ```

### `<browser_action>`
- **Description:** Interact with a Puppeteer-controlled browser session. Only one action per call; every sequence must begin with `launch` and end with `close`.
- **Availability:** Only when browser support is enabled; no other tools may run until the browser is closed.
- **Parameters:**
  - `<action>` (required) — One of `launch`, `click`, `type`, `scroll_down`, `scroll_up`, or `close`.
  - `<url>` (optional) — Required when launching; include full protocol.
  - `<coordinate>` (optional) — `x,y` pair for clicks, within the viewport dimensions.
  - `<text>` (optional) — Keyboard input for the `type` action.
- **Example:**
  ```xml
  <browser_action>
    <action>launch</action>
    <url>http://localhost:3000</url>
  </browser_action>
  ```

### `<execute_command>` / `<bash>`
- **Description:** Run a CLI command in the current working directory. Prefer single commands over scripts.
- **Availability:** Always available; `timeout` only allowed when YOLO mode is on.
- **Parameters (generic prompts):**
  - `<command>` (required) — Shell-escaped command to execute.
  - `<requires_approval>` (required) — `true` if potentially destructive, otherwise `false`.
  - `<timeout>` (optional, YOLO mode only) — Seconds before automatically stopping the command.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Parameters (GPT prompts as `<bash>`):**
  - `<command>` (required)
  - `<requires_approval>` (optional)
  - `<timeout>` (optional, YOLO mode only)
  - `<task_progress>` (optional)
- **Example:**
  ```xml
  <execute_command>
    <command>npm run lint</command>
    <requires_approval>false</requires_approval>
  </execute_command>
  ```

### `<focus_chain>`
- **Description:** Internal placeholder that activates when Focus Chain is enabled. Used as a dependency for task progress tracking.
- **Availability:** Only when `focusChainSettings.enabled` is true.
- **Parameters:** None.
- **Example:**
  ```xml
  <focus_chain />
  ```

### `<list_code_definition_names>`
- **Description:** List top-level definition names (functions, classes, etc.) in source files under a directory.
- **Parameters:**
  - `<path>` (required) — Directory to inspect, relative to the CWD.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <list_code_definition_names>
    <path>src/components</path>
  </list_code_definition_names>
  ```

### `<list_files>`
- **Description:** Enumerate contents of a directory, optionally recursively.
- **Parameters:**
  - `<path>` (required) — Directory to list, relative to the CWD.
  - `<recursive>` (optional) — `true` for deep listing; omit or `false` for top-level.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <list_files>
    <path>src/core</path>
    <recursive>true</recursive>
  </list_files>
  ```

### `<load_mcp_documentation>`
- **Description:** Load guidance on building MCP servers when a user requests a new tool/server.
- **Availability:** Only when an MCP hub is connected.
- **Parameters:** None.
- **Example:**
  ```xml
  <load_mcp_documentation />
  ```

### `<new_task>`
- **Description:** Propose a fresh task with a comprehensive summary of the current conversation and technical state.
- **Parameters:**
  - `<context>` (required) — Detailed briefing covering current work, key concepts, relevant files, solved problems, and pending steps with verbatim quotes when appropriate.
- **Example:**
  ```xml
  <new_task>
    <context>
      Current work: Investigating login failures after dependency upgrade.
      Key concepts: NextAuth, Prisma, Node 20.
      Relevant files: auth/[...nextauth].ts, prisma/schema.prisma.
      Problems solved: Fixed OAuth redirect mismatch.
      Pending tasks: Update credentials provider tests (see user request: "Please add coverage for password resets").
    </context>
  </new_task>
  ```

### `<plan_mode_respond>`
- **Description:** Provide a planning response to the user once sufficient exploration is complete. Optional flag indicates more exploration is still needed.
- **Availability:** Only while the environment is in PLAN MODE.
- **Parameters:**
  - `<response>` (required) — Chat-style plan message to the user.
  - `<needs_more_exploration>` (optional) — `true` if further tool use is required before executing the plan.
  - `<task_progress>` (optional) — Checklist summary; depends on `focus_chain`.
- **Example:**
  ```xml
  <plan_mode_respond>
    <response>
      I will refactor the data loader, add unit tests, and verify the API contract using the provided schema.
    </response>
    <needs_more_exploration>false</needs_more_exploration>
  </plan_mode_respond>
  ```

### `<read_file>`
- **Description:** Read the contents of a file. Suitable for text, PDF, and DOCX files (binary content returned as raw text).
- **Parameters:**
  - `<path>` (required) — File path relative to the CWD.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <read_file>
    <path>src/lib/utils.ts</path>
  </read_file>
  ```

### `<replace_in_file>`
- **Description:** Apply one or more precise SEARCH/REPLACE blocks to an existing file.
- **Parameters:**
  - `<path>` (required) — File to edit.
  - `<diff>` (required) — One or more SEARCH/REPLACE blocks following the exact format described below. Each SEARCH section must match exactly, and only the first match is replaced. Use multiple concise blocks in file order for multiple edits.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <replace_in_file>
    <path>src/lib/utils.ts</path>
    <diff><![CDATA[
------- SEARCH
export function add(a: number, b: number): number {
  return a + b;
}
=======
export function add(a: number, b: number): number {
  return a + b + 1;
}
+++++++ REPLACE
]]></diff>
  </replace_in_file>
  ```

### `<search_files>`
- **Description:** Perform a Rust-regex search across files under a directory with contextual matches.
- **Parameters:**
  - `<path>` (required) — Directory to search, relative to the CWD.
  - `<regex>` (required) — Regular expression pattern (Rust syntax).
  - `<file_pattern>` (optional) — Glob filter such as `*.ts`; defaults to all files.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <search_files>
    <path>src</path>
    <regex>TODO\(\)</regex>
    <file_pattern>*.ts</file_pattern>
  </search_files>
  ```

### `<use_mcp_tool>`
- **Description:** Invoke a tool exposed by a connected MCP server using a JSON arguments payload.
- **Availability:** Only when an MCP hub is connected.
- **Parameters:**
  - `<server_name>` (required) — MCP server to target.
  - `<tool_name>` (required) — Specific tool to run.
  - `<arguments>` (required) — JSON object matching the tool's schema.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <use_mcp_tool>
    <server_name>jira</server_name>
    <tool_name>create_issue</tool_name>
    <arguments>
  {
    "summary": "Fix login regression",
    "project": "AUTH",
    "priority": "High"
  }
    </arguments>
  </use_mcp_tool>
  ```

### `<web_fetch>`
- **Description:** Fetch and convert a webpage to markdown. Prefer MCP-provided web fetch tools when available. URLs are auto-upgraded to HTTPS and must be valid.
- **Parameters:**
  - `<url>` (required) — Fully-qualified URL to retrieve.
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <web_fetch>
    <url>https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API</url>
  </web_fetch>
  ```

### `<write_to_file>`
- **Description:** Create or overwrite a file with the full desired contents. Automatically creates missing directories.
- **Parameters:**
  - `<path>` (required) — Destination file path relative to the CWD.
  - `<content>` (required) — Complete file contents (no omissions).
  - `<task_progress>` (optional) — Checklist update after the tool completes.
- **Example:**
  ```xml
  <write_to_file>
    <path>src/config/app.json</path>
    <content>
  {
    "logLevel": "info",
    "featureFlags": {
      "betaDashboard": true
    }
  }
    </content>
  </write_to_file>
  ```
