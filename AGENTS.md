# AI Agent Guidelines for Blog Posting

This document outlines the standards and workflows for AI agents creating content for this blog.

## 1. Content Standards

### Writing Style

- **Tone**: Professional, informative, and concise. Avoid overly casual or overly stiff language.
- **Ending**: Use the Korean plain form ("~했다", "~이다", "~한다") for posts and content where applicable. Do not use polite or honorific forms ("~했습니다", "~해요").
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
    toc: true
    comments: true
    ---
    ```

- **Note**: Do not use the `description` frontmatter field in posts. Instead, write a concise summary within the post (first paragraph or a dedicated short line) when necessary. The site does not rely on `description` frontmatter and it should be removed from existing posts.
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

1. **Analyze**: Understand the user's intent and the technical problem.
2. **Context**: Check existing posts in `_posts/` to maintain consistency in style and structure.
3. **Draft**: Create the content following the standards above.
4. **Review**: Self-correct for style ("~했다") and technical accuracy before finalizing.

## 4. Commit messages

- Purpose: Commit messages should clearly describe the change and make it easy to trace history. Do not use bracketed tags like [Post], [Blog], or [Obsidian]. Prefer a simple, consistent message style.
- Preferred format: Use a single capitalized verb at the start followed by a short subject. Do not add a separate colon after the verb; write the message as a short sentence. Examples of preferred verbs are: Add, Update, Fix, Use, Remove, Revert, Docs, Chore.
  - Format examples: "Add new GitHub SSH post", "Update image paths", "Fix lnk2019 UE4 error", "Use Google Analytics configuration"
- Language: You may write commit messages in either English or Korean. For blog content, Korean is acceptable. Commit messages should follow the simple verb + subject format (no colon or bracket tags). Avoid bracket tags like [Post], [Blog], or [Obsidian].
- Message content and length:
  - Keep the subject short, ideally around 50 characters or fewer. If more detail is required, include the details in the message body.
  - Leave a blank line after the subject, and then write a longer description with background, the impact of the changes, and any references (issue/PR numbers, links).
  - When a commit closes an issue, add `Fixes #<issue-number>` or `Refs #<issue-number>` in the message body.
  - If there is a breaking change, include a "BREAKING CHANGE:" section in the body explaining the impact and migration steps.
  - Be consistent with capitalization and spelling.
