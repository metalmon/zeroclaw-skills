---
name: office-documents
description: REQUIRED before touching any Microsoft Office file (.docx, .xlsx, .pptx, .doc, .xls, .ppt — Word, Excel, PowerPoint). When the user sends, attaches, or asks you to open/read/parse/extract/summarize such a file, follow this skill's recipe (preloaded in the instructions below): extract clean text/Markdown by chaining file_read → office_read through execute_pipeline with "result": "last". Do NOT call file_read on an Office file directly and inline its bytes — they are binary, they flood the context and stall generation, and you will NOT get readable text that way.
---

# Reading Office documents

The `office_read` tool parses Office files (DOCX, XLSX, PPTX, DOC, XLS, PPT) into
plain text, Markdown, or HTML. It does **not** take a file path — it takes the
file's raw bytes as a **base64 string**.

## You MUST execute, not describe

When asked to read/open/summarize an Office file you MUST actually call the
`execute_pipeline` tool described below. Do NOT:

- describe the steps in prose instead of calling the tool,
- guess, infer, or invent the document's contents,
- claim the file is "actually HTML", "corrupted", or "can't be opened" without
  having run `office_read` and seen its real error.

If you have not run the pipeline, you do not know what the file contains. Run it
first, then answer from the extracted text. If `office_read` returns an error,
report that exact error — never fabricate content.

## Critical rule

NEVER read the file with `file_read` and then paste the base64 into `office_read`
yourself. The base64 of a real document is tens of thousands of characters;
generating it inline as a tool argument is extremely slow and may stall.

Instead, ALWAYS chain the two tools with `execute_pipeline`, so the base64 is
passed machine-to-machine via `{{step[0].result}}` and you never have to write
it out.

## Recipe

Call `execute_pipeline` (sequential) with exactly these two steps and
`"result": "last"` so only the extracted text comes back (the raw base64 from
step[0] is dropped and never floods your context):

```json
{
  "result": "last",
  "steps": [
    {
      "tool": "file_read",
      "args": { "path": "report.docx", "encoding": "base64" }
    },
    {
      "tool": "office_read",
      "args": {
        "content_base64": "{{step[0].result}}",
        "filename": "report.docx",
        "output": "markdown"
      }
    }
  ]
}
```

- With `"result": "last"` the pipeline returns only the **last step's** result
  (the extracted document). Answer from that.
- `output` can be `"markdown"` (default, best for prose/tables), `"text"`
  (plain), or `"html"`.
- Pass `filename` (with its extension) so the format is detected, or set
  `format` explicitly (`"docx"`, `"xlsx"`, `"pptx"`, `"doc"`, `"xls"`, `"ppt"`).

## Constraints

- `file_read` is sandboxed to the agent workspace. Use the path exactly as the
  user/attachment provides it. Files attached via Telegram land inside the
  workspace under `telegram_files/` (e.g. `telegram_files/report.docx`); an
  absolute path that points inside the workspace is also accepted. Only a path
  that resolves OUTSIDE the workspace is refused — in that case ask the user to
  place the file in the workspace.
- For very large spreadsheets, prefer `output: "markdown"` and summarize rather
  than echoing the whole document back.
