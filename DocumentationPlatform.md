# Documentation Platform

| Github |  | Confluence | |
| ------ | ----- | ----- | ----- |
| Pros | Cons | Pros | Cons |
| Everything in 1 Place | Markdown Only | WYSIWYG | No Approval Process |
| Merges through PR | Requires Github Access | Easily Accessible | |
| Version Controler |                | | | 



## Confluence vs Github Workflow
```mermaid

sequenceDiagram
    participant Confluence
    actor ReviewCon
    actor DevCon
    participant Github
    actor DevGit
    actor ReviewGit
    participant GitDocs
    
    DevCon ->> Github: Make Code Changes
    DevGit ->> Github: Make Code Changes
    DevCon ->> Confluence: Update Documentation
    DevGit ->> GitDocs: Update Documentation
    
    DevCon ->> ReviewCon: PR Merge Ready
    DevGit ->> ReviewGit: PR Merge Ready
    
    ReviewCon ->> Github: Review PR
    ReviewCon ->> Confluence: Review Documentation
    
    ReviewGit ->> Github: Review PR
    ReviewGit ->> GitDocs: Review Documentation

```
