## Small

I need to port a Python library to Go. The library [brief description of what it does].

Please port this Python code to Go following these guidelines:
1. Maintain the same functionality and public API where possible
2. Use appropriate Go types and error handling patterns
3. Convert Python's dynamic typing to Go's static typing system
4. Replace Python collections with appropriate Go structures (slices, maps)
5. Implement proper error handling instead of Python's exceptions
6. Maintain the original code's comments and documentation style

Here's the Python code to port:

[paste Python code here]

Please provide:
1. The equivalent Go code
2. Notes on any significant translation decisions made
3. Suggestions for future idiomatization that could improve the Go version

## Large

I need to port a large Python project to Go. The project [brief description and scale, e.g., "consists of approximately 50,000 lines of code across 120 modules and handles financial data processing"].

Rather than porting the entire codebase at once, please help me develop a strategic approach by:

1. FIRST, analyzing a representative sample of the code to:
    - Identify recurring Python patterns and their Go equivalents
    - Create a translation guide for common Python constructs
    - Establish Go package structure recommendations

2. THEN, help me determine a porting sequence by:
    - Mapping dependencies between modules
    - Identifying core/foundational components to port first
    - Creating a phased migration plan with testing checkpoints

3. FINALLY, port a small representative module to demonstrate:
    - Translation patterns in practice
    - Testing approach for ensuring functional equivalence
    - Documentation standards for the Go implementation

Sample module for initial analysis (represents typical patterns in the codebase):

[paste representative module code here]

Please provide a comprehensive porting strategy rather than attempting to port the entire project at once.

## Example porting Python PDF Library to Go

I need to port a Python PDF text extraction library to Go. This library (approximately 15-20k lines) handles PDF parsing, text extraction, layout analysis, and supports various PDF specifications and encoding formats.

Please help me develop a porting strategy by:

1. FIRST, analyze the PDF-specific challenges:
    - How to handle binary data structures in Go vs Python
    - Strategies for replacing Python PDF libraries with Go equivalents or bindings
    - Converting text encoding and Unicode handling between languages
    - Performance considerations for Go vs Python when processing large documents

2. THEN, develop a focused porting plan:
    - Identify logical components (parser, extractor, renderer, etc.)
    - Suggest a dependency order (which components should be ported first)
    - Recommend test-driven approach for validating extraction accuracy
    - Outline how to handle edge cases (malformed PDFs, unusual encodings)

3. FINALLY, demonstrate the approach by porting a core component:
    - Port the fundamental text extraction function below
    - Show how to structure the Go package for future expansion
    - Include comprehensive error handling for PDF-specific failures
    - Add appropriate benchmarking to compare performance

Here's a representative core function that extracts text from a PDF page:

[paste core text extraction function code here]

Please focus on maintaining extraction accuracy while leveraging Go's performance advantages. Include recommendations for handling PDF-specific challenges like font mappings, text positioning, and content streams.