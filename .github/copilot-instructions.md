# GitHub Copilot Instructions

## Objective
The purpose of `my-main` branch is to demonstrate DevSecOps integration with various CI/CD tools, with a higher focus on security testing and compliance. 

The repository contains sample applications that are intentionally vulnerable, allowing users to practice identifying and mitigating security issues throughout the software development lifecycle.

### AI Assistant Expectations
- Do not change make changes to the original source code of the vulnerable application

### Pull Request Hygiene
- When creating a PR, ensure it targets the `my-main` branch.
- Squash commits into pr/<short-description> before merging to keep a clean commit history.
- Keep PRs scoped: one logical change set (feature, refactor, bugfix) per PR when possible.
- Include concise summary of intent and any trade-offs.
- Suggest to delete feature branches after merging to keep the repository clean.
- Suggest to clean up failed and old GitHub Actions workflow runs to maintain CI/CD efficiency.