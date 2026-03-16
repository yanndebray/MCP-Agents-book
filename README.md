# MCP-Agents-book
Building AI Agents with MCP: From Local Prototypes to Enterprise Deployment 🛠️

![](img/elephant.png)

---

## 1. TL;DR

With this book you will build:

> “An agent that reads sales data from a spreadsheet, runs advanced analysis in MATLAB, and generates polished reports (Markdown → PPT/Word/static site) via MCP tools, deployable from laptop to cloud.”

This book covers:

* Initiating the *agent loop* (Claude Desktop / custom OpenAI Responses client with tool calling), 
* Developing *MCP servers* providing tools to the agent (spreadsheet, analysis, report), 
* Deploying agent+tools to *production* (FastMCP Cloud, MATLAB Production Server, Render, GitHub Actions).

---

## 2. Book outline

### Part I – Foundations: MCP and agent loops

1. **Why MCP?**

   * Problem: bespoke tool integrations for every agent.
   * MCP as the “USB-C for AI tools” ([Model Context Protocol][1])
   * Overview of servers vs clients; JSON-RPC; tool schemas.

2. **Agent loops with Claude & Responses**

   * Claude Desktop as zero-install MCP client. ([Anthropic][3])
   * Building a minimal custom agent using OpenAI Responses with tool calling & structured outputs. ([OpenAI Platform][4])
   * Logging, retries, and guardrails.

3. **FastMCP primer**

   * Building a “hello world” MCP server using FastMCP in ~30 lines. ([FastMCP][2])
   * Configuring it for Claude and for your custom Responses client.

---

### Part II – The Spreadsheet MCP

4. **Designing tools for tabular data**

   * Shape of the `product_sales` dataset; how to expose slice/filter/search tools rather than raw file reads.
   * Tool design patterns: narrow, composable tools vs mega-functions.

5. **Implementing with FastMCP**

   * Pandas + FastMCP for: `get_sales`, `summarize_sales_by_year`, etc.
   * Validation, error messages, and schema design.

6. **Deploying to FastMCP Cloud**

   * Connect repo, configure secrets (e.g., private S3 bucket or DB).
   * Observability: tracing tool calls at JSON-RPC level. ([FastMCP][8])

---

### Part III – The Analysis MCP (MATLAB)

7. **Local prototyping with MATLAB MCP Core Server**

   * Installing and configuring MATLAB MCP Core Server. ([MathWorks][5])
   * Wrapping existing MATLAB analysis functions as MCP tools.
   * Using Claude / Responses agent to call MATLAB for:

     * statistics,
     * forecasting,
     * scenario analysis.

8. **From desktop to server: MCP Framework for MPS**

   * Why MATLAB Production Server for multi-user, low-latency workloads. ([MathWorks][6])
   * Publishing MATLAB functions as MCP tools through the MPS Framework.
   * Versioning tools and handling model & code updates.

9. **Alternative: compiled MATLAB to Docker/Render**

   * For readers without MPS: use Compiler SDK to produce a runtime image, and deploy to Render as a web service behind a tiny MCP shim. ([Render][7])
   * Discussion of pros/cons vs MPS (scaling, licensing, startup time).

---

### Part IV – The Report MCP & sub-agents

10. **Designing the Report MCP**

* Tool APIs:

  * `generate_markdown_report(request_id, filters)`
  * `export_pptx(markdown_url | data)`
  * `export_static_site(markdown_url | data)`
* Orchestration patterns: when Report MCP should call Spreadsheet/Analysis MCP vs letting the main agent orchestrate.

11. **Deep research as a sub-agent**

* Report MCP uses:

  * Claude or Responses as a “secondary” agent for contextual commentary (“deep research” mode).
* Prompting patterns for consistent section/heading output.

12. **Formatting outputs**

* Markdown + front-matter → static site (Hugo, MkDocs, etc.).
* Using python-pptx / python-docx to generate slides/reports that embed charts produced by MATLAB tools (saved to S3 or Git LFS).

---

### Part V – Deployment, CI/CD, and operations

13. **GitHub Actions pipelines**

* Workflow:

  * run tests (Python + MATLAB),
  * run linting,
  * build docs,
  * build PPTX/Word artifacts,
  * push Docker images / trigger FastMCP Cloud deployment.
* Scheduled workflows to regenerate “weekly report” static sites.

14. **Multi-environment config**

* Dev (local MCP Core Server, local FastMCP servers).
* Staging (FastMCP Cloud + MPS staging).
* Prod (FastMCP Cloud + MPS prod + Render static site or GitHub Pages).

15. **Monitoring, security, and costs**

* Observability from FastMCP Cloud & MPS logs. ([Prefect][9])
* Secret management (API keys, DB creds).
* Cost controls for LLM usage (rate limits, budget per report).

16. **Extending the architecture**

* Additional MCP servers: CRM, ticketing, CFD results, etc.
* Multi-tenant setup; org-level configuration patterns.
* How to reuse the patterns for *other* books/demos.

---

## 3. Learning path inside the book

Each part ends with a “checkpoint” where the reader has:

1. **After Part I:**

   * A working local agent loop calling a toy FastMCP server.

2. **After Part II:**

   * Spreadsheet MCP serving filtered views of `product_sales.xlsx`, callable from Claude and from a Responses agent.

3. **After Part III:**

   * MATLAB analysis tools callable via MCP, first locally (Core Server), then on MPS or Render.

4. **After Part IV:**

   * Report MCP that produces end-to-end Markdown + PPTX from a natural-language query.

5. **After Part V:**

   * CI/CD pipelines, staged deployments, and a publicly accessible static site of generated reports.

---

## 4. Tech stack

**Agent clients**

* **Claude-based MCP client** (Claude Desktop): talks to your three servers via standard MCP config. ([Anthropic][3])
* **Custom OpenAI Responses client**: Node/Python client that uses Responses API + tool calling to implement the same “agent loop” but with explicit control over planning, retries, and logging. ([OpenAI Platform][4])

**Servers**

1. **Spreadsheet MCP** (FastMCP)

   * Built with **FastMCP** Python framework (local first, then pushed to FastMCP Cloud). ([FastMCP][2])
   * Tools like:

     * `list_dimensions()`
     * `get_sales(year: int, product: str | None, region: str | None)`
     * `summarize_by_product(year)`
   * Backed by Pandas reading `product_sales.xlsx` from a data folder.

2. **Analysis MCP**

   * **Local dev:** MATLAB MCP Core Server, exposing MATLAB functions for statistics, forecasting, anomaly detection, etc. ([MathWorks][5])
   * **Production:** MCP Framework for MATLAB Production Server, which turns MATLAB functions into scalable MCP tools on MATLAB Production Server. ([MathWorks][6])
   * Optional variant: compiled MATLAB components in Docker images deployed on Render as web services if readers don’t have MPS, but do have Compiler SDK. ([Render][7])

3. **Report MCP**

   * Python FastMCP server whose tools:

     * Call **Spreadsheet MCP** + **Analysis MCP**.
     * Optionally invoke a “deep research” sub-agent (Claude / Responses) for market commentary.
     * Emit:

       * `generate_markdown_report(...)`
       * `generate_powerpoint(...)`
       * `generate_word_report(...)`
       * `generate_static_site(...)`
   * Uses python-pptx / python-docx / static site generator as implementation details (book can pick one per format).

**Infra**

* **FastMCP Cloud** for spreadsheet + report servers in production (zero-config, CI/CD, observability). ([FastMCP][8])
* **MATLAB Production Server** (on-prem or cloud) hosting the analysis tools via the MCP Framework. ([MathWorks][6])
* **Render.com** for:

  * hosting the static site (if not using GitHub Pages), and/or
  * hosting compiled MATLAB or helper services as Docker-based web services. ([Render][7])
* **GitHub Actions**:

  * Run tests for each server and the agent client.
  * On merges/tags, build:

    * static site from latest Markdown,
    * PPT/Word artifacts,
    * Docker images for Render/MPS where relevant.

---

## 5. Repo structure

This is the repo structure for the book:

```text
mcp-agents-book/
  book/                     # Manuscript, notebooks, diagrams
  agents/
    claude-mcp-client/      # config files + example prompts
    responses-agent/        # Python/Node OpenAI Responses client
  servers/
    spreadsheet-fastmcp/
      server.py
      product_sales.xlsx
      tests/
    analysis-matlab-core/
      matlab/
        analyzeSales.m
        forecastDemand.m
      mcp-core-config/
      tests/
    analysis-mps-framework/
      matlab/
      mps-deploy/           # MPS MCP framework configs
    report-fastmcp/
      server.py
      templates/            # pptx/docx/html templates
      tests/
  infra/
    .github/
      workflows/
        ci.yml
        deploy-fastmcp.yml
        build-reports.yml
    render/
      Dockerfile-matlab-runtime
      render.yaml
  examples/
    01-local-loop/
    02-multi-server/
    03-deep-research/
    04-production-deploy/
```

Each directory contains a chapter’s code starting point / finished solution.
---

[1]: https://modelcontextprotocol.io/ "What is the Model Context Protocol (MCP)? - Model Context ..."
[2]: https://gofastmcp.com/ "Welcome to FastMCP 2.0! - FastMCP"
[3]: https://www.anthropic.com/news/model-context-protocol "Introducing the Model Context Protocol"
[4]: https://platform.openai.com/docs/api-reference/responses "API Reference"
[5]: https://www.mathworks.com/products/matlab-mcp-core-server.html "MATLAB MCP Core Server"
[6]: https://www.mathworks.com/matlabcentral/fileexchange/182578-mcp-framework-for-matlab-production-server "MCP Framework for MATLAB Production Server"
[7]: https://render.com/docs/docker "Docker on Render"
[8]: https://gofastmcp.com/deployment/fastmcp-cloud "FastMCP Cloud"
[9]: https://www.prefect.io/blog/accelerating-ai-with-fastmcp-cloud "MCP Server Deployment Made Simple | FastMCP Cloud"
