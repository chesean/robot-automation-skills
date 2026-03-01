# Robot Automation Skills for Claude Code

Custom slash command skills for network test automation with Claude Code.

## Setup

Copy the `.claude/commands/` folder into your project root:

```bash
# Clone this repo
git clone https://github.com/chesean/robot-automation-skills.git

# Copy skills into your project
cp -r robot-automation-skills/.claude/commands/ /path/to/your/project/.claude/commands/
```

## Available Skills

### CXTM Test Automation
| Skill | Command | Description |
|-------|---------|-------------|
| CXTM Ready | `/cxtm-ready` | Load full CXTM context (keywords, variables, CXTA library) |
| Robot Coding | `/robot-coding` | Robot Framework coding standards and CXTA best practices |
| Test Case Summary | `/test-case-summary` | Analyze test results and generate CXTM-formatted summaries |
| Consolidate as DOCX | `/consolidate-test-results-as-docx` | Generate procedure-driven DOCX evidence reports |

### Network Verification
| Skill | Command | Description |
|-------|---------|-------------|
| HBN Baseline Check | `/hbn-baseline-check` | HBN network baseline verification |

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- Skills are project-scoped - place `.claude/commands/` in your project root

## Project

These skills are designed for Nexus 9000 platforms certification projects using CXTA Robot Framework automation.
