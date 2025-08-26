# Mintlify documentation

## Working relationship
- You can push back on ideas-this can lead to better documentation. Cite sources and explain your reasoning when you do so
- ALWAYS ask for clarification rather than making assumptions
- NEVER lie, guess, or make up information

## Project context
- Format: MDX files with YAML frontmatter
- Config: docs.json for navigation, theme, settings
- Components: Mintlify components

## Content strategy
- Document just enough for user success - not too much, not too little
- Prioritize accuracy and usability of information
- Make content evergreen when possible
- Search for existing information before adding new content. Avoid duplication unless it is done for a strategic reason
- Check existing patterns for consistency
- Start by making the smallest reasonable changes

## docs.json

- Refer to the [docs.json schema](https://mintlify.com/docs.json) when building the docs.json file and site navigation

## Frontmatter requirements for pages
- title: Clear, descriptive page title
- description: Concise summary for SEO/navigation

## Writing standards
- Second-person voice ("you")
- Prerequisites at start of procedural content
- Test all code examples before publishing
- Match style and formatting of existing pages
- Include both basic and advanced use cases
- Language tags on all code blocks
- Alt text on all images
- Relative paths for internal links

## Git workflow
- NEVER use --no-verify when committing
- Ask how to handle uncommitted changes before starting
- Create a new branch when no clear branch exists for changes
- Commit frequently throughout development
- NEVER skip or disable pre-commit hooks

## Do not
- Skip frontmatter on any MDX file
- Use absolute URLs for internal links
- Include untested code examples
- Make assumptions - always ask for clarification
```

## Sample prompts

Once you have Claude Code set up, try these prompts to see how it can help with common documentation tasks. You can copy and paste these examples directly, or adapt them for your specific needs.

### Convert notes to polished docs

Turn rough drafts into proper Markdown pages with components and frontmatter.

**Example prompt:**

```text wrap
Convert this text into a properly formatted MDX page: [paste your text here]
```

### Review docs for consistency

Get suggestions to improve style, formatting, and component usage.

**Example prompt:**

```text wrap
Review the files in docs/ and suggest improvements for consistency and clarity
```

### Update docs when features change

Keep documentation current when your product evolves.

**Example prompt:**

```text wrap
Our API now requires a version parameter. Update our docs to include version=2024-01 in all examples
```

### Generate comprehensive code examples

Create multi-language examples with error handling.

**Example prompt:**

```text wrap
Create code examples for [your API endpoint] in JavaScript, Python, and cURL with error handling
```

## Extending Claude Code

Beyond manually prompting Claude Code, you can integrate it with your existing workflows.

### Automation with GitHub Actions

Run Claude Code automatically when code changes to keep docs up to date. You can trigger documentation reviews on pull requests or update examples when API changes are detected.

### Multi-instance workflows

Use separate Claude Code sessions for different tasks - one for writing new content and another for reviewing and quality assurance. This helps maintain consistency and catch issues that a single session might miss.

### Team collaboration

Share your refined `CLAUDE.md` file with your team to ensure consistent documentation standards across all contributors. Teams often develop project-specific prompts and workflows that become part of their documentation process.

### Custom commands

Create reusable slash commands in `.claude/commands/` for frequently used documentation tasks specific to your project or team.