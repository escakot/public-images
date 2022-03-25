# Documentation Platform

| Comparison | Github | Confluence |
| ------ | ----- | ----- |
| Features | Markdown & Mermaid Graphs | WYSIWYG |
| Location | Code & Docs Centralized | Separated - Github & Confluence |
| Editing | PR Reviewed | Freely changeable |
| Versioning | Code Diff - Squash Commit | Page History - Multiple Changes |
| Review Code & Docs | Together | Separate |
| Reverts | Both reverted together | Very difficult due to versioning | 
| Accessibility | Github Permissions | Confluence Permissions (Accessible by Product) |
| Graphs | Built-in & Reviewable | Customizable Draw.io |

## Confluence vs Github Workflow
```mermaid

sequenceDiagram
    participant Confluence
    actor ReviewCon
    actor DevCon
    participant Github
    
    DevCon ->> Github: Make Code Changes
    DevCon ->> Confluence: Update Documentation
    
    DevCon ->> ReviewCon: PR Merge Ready. Review Requested
    
    ReviewCon ->> Github: Review Code
    ReviewCon ->> Confluence: Review Documentation
    ReviewCon ->> DevCon: Request Code Changes
    DevCon ->> Github: Code fixed
    DevCon ->> ReviewCon: Request 2nd Review 
    ReviewCon ->> Github: Approved Review and Merged
```
```mermaid

sequenceDiagram
    actor ReviewGit
    actor DevGit
    participant Github

    DevGit ->> Github: Make Code Changes & Update Documentation
    DevGit ->> ReviewGit: PR Merge Ready. Review Requested
    ReviewGit ->> Github: Review Code & Documentation
    ReviewGit ->> DevGit: Request Code Changes
    DevGit ->> Github: Code fixed
    DevGit ->> ReviewGit: Request 2nd Review
    ReviewGit ->> Github: Approved Review and Merged

```
