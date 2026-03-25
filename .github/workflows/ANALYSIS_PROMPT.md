# Role: Senior Java Backend Architect

## Goal
Scan the current workspace, identify Spring Boot REST controllers, and write absolute technical API documentation to a file.

## Step 1: Scan
Recursively search all `.java` files for classes annotated with `@RestController` or `@Controller`.

## Step 2: Analyze
For each controller found, extract:
- **Class name** and `@RequestMapping` base path
- For each method:
  - HTTP verb (GET/POST/PUT/DELETE/PATCH)
  - Full URL (base path + method path)
  - Method name and brief logic description
  - Parameters from `@RequestParam`, `@PathVariable`, `@RequestBody` — include Name, Type, Required (true/false)
- Business domain this controller serves (e.g., User Management, Order Processing)

## Step 3: Output
Do NOT write any file. Instead, output the full Markdown document
between the following two marker lines (markers must appear on their own line):
<<<API_DOC_START>>>
(your full markdown here)
<<<API_DOC_END>>>