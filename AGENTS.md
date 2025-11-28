# AI Agent Guidelines for Blog Posting

This document outlines the standards and workflows for AI agents creating content for this blog.

## 1. Content Standards

### Writing Style
- **Tone**: Professional, informative, and concise. Avoid overly casual or overly stiff language.
- **Ending**: Use the Korean plain form ("~했다", "~이다", "~한다") for all sentences. Do not use polite/honorific forms ("~했습니다", "~해요").
- **Structure**:
    - **Introduction**: Briefly state the problem or topic.
    - **Body**: Detailed analysis, process, or explanation.
    - **Solution/Conclusion**: Clear steps to resolve the issue or a summary of findings.
    - Use Markdown headers (##, ###) to organize content.

### Formatting
- **Frontmatter**: Always include Jekyll frontmatter at the top of the file.
    ```yaml
    ---
    title: "Post Title"
    categories: [Category1, Category2]
    tags: [Tag1, Tag2]
    description: "Brief summary for SEO"
    toc: true
    comments: true
    ---
    ```
- **Code Blocks**: Use triple backticks with the correct language identifier (e.g., `bash`, `json`, `python`).

## 2. Technical Accuracy & Verification

**CRITICAL**: Technical content must be accurate and verifiable.

- **Fact-Checking**:
    - **Verify Commands**: Ensure command syntax, flags, and arguments are correct for the target environment.
    - **Version Compatibility**: Check if the solution is specific to a version (e.g., Node.js, Python, OS).
    - **Error Messages**: Ensure the error message cited matches the problem description.
- **Verification Process**:
    - Before suggesting a solution, cross-reference with official documentation or known issues (e.g., GitHub Issues).
    - If a solution involves code, verify it logic.
    - **Do not hallucinate** packages, commands, or configuration options. If unsure, admit limitation or search for accurate information.

## 3. Workflow

1.  **Analyze**: Understand the user's intent and the technical problem.
2.  **Context**: Check existing posts in `_posts/` to maintain consistency in style and structure.
3.  **Draft**: Create the content following the standards above.
4.  **Review**: Self-correct for style ("~했다") and technical accuracy before finalizing.
