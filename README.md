# Research projects carried out by AI tools

Each directory in this repo is a separate research project carried out by an LLM tool - usually [Claude Code](https://www.claude.com/product/claude-code). Every single line of text and code was written by an LLM.

See [Code research projects with async coding agents like Claude Code and Codex](https://simonwillison.net/2025/Nov/6/async-code-research/) for more details on how this works.

<!--[[[cog
import os
import subprocess
import pathlib

root = pathlib.Path(".")

# Get all project directories (skip hidden dirs and __pycache__)
dirs = sorted(
    [
        d
        for d in root.iterdir()
        if d.is_dir()
        and not d.name.startswith(".")
        and d.name != "__pycache__"
        and d.name != "node_modules"
    ]
)

# Get the first commit date for each directory
projects = []
for d in dirs:
    result = subprocess.run(
        ["git", "log", "--diff-filter=A", "--follow", "--format=%aI", "--", d.name],
        capture_output=True,
        text=True,
    )
    dates = result.stdout.strip().split("\n")
    date = dates[-1] if dates and dates[-1] else ""
    if date:
        projects.append((d, date))

# Sort by date, most recent first
projects.sort(key=lambda x: x[1], reverse=True)

if not projects:
    cog.out("\n*No research projects yet.*\n")
else:
    for d, date in projects:
        date_str = date[:10]  # Just the YYYY-MM-DD part
        readme_path = d / "README.md"
        summary_path = d / "_summary.md"

        # Check for cached summary
        if summary_path.exists():
            summary = summary_path.read_text().strip()
        else:
            # Try to generate summary using llm
            if readme_path.exists():
                readme_content = readme_path.read_text()
                try:
                    result = subprocess.run(
                        [
                            "llm",
                            "-m",
                            "github/gpt-4.1",
                            "-s",
                            "Summarize this README in 1-2 concise paragraphs. Focus on what was investigated and key findings. Output markdown.",
                        ],
                        input=readme_content,
                        capture_output=True,
                        text=True,
                        timeout=30,
                    )
                    summary = result.stdout.strip() if result.returncode == 0 else ""
                except (subprocess.TimeoutExpired, FileNotFoundError):
                    summary = ""
            else:
                summary = ""

            if summary:
                summary_path.write_text(summary + "\n")

        # Add AI-generated note to project README if not already present
        ai_note = "> *This research was conducted entirely by an AI agent.*"
        if readme_path.exists():
            content = readme_path.read_text()
            if ai_note not in content:
                readme_path.write_text(ai_note + "\n\n" + content)

        cog.out(f"\n### [{d.name}]({d.name}/), {date_str}\n\n")
        if summary:
            cog.out(f"{summary}\n")
        else:
            cog.out(f"*No summary available yet.*\n")
]]]-->

*No research projects yet.*
<!--[[[end]]]-->
