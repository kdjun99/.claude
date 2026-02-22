---
description: "Generates a structured 30-minute interview question document from candidate resume and portfolio PDFs. Orchestrates 4-step pipeline: (1) candidate analysis, (2) project deep-dive, (3) question generation, (4) quality verification. 3 human checkpoints for review."
allowed-tools: ["Task", "Read", "Write", "Glob", "Grep", "Bash", "AskUserQuestion"]
---

# /interview Command

Generate a structured 30-minute interview question document from candidate documents.

**Usage:** `/interview <candidate_name> <resume_pdf_path> <portfolio_pdf_path>`

## Arguments

Parse from `$ARGUMENTS`:
- `candidate_name`: First argument — used for workspace directory name
- `resume_pdf_path`: Second argument — absolute path to resume PDF
- `portfolio_pdf_path`: Third argument — absolute path to portfolio PDF

If arguments are missing, use AskUserQuestion to ask:
- "What is the candidate's name?"
- "What is the path to the resume PDF?"
- "What is the path to the portfolio PDF?"

## Pre-flight Checks

1. Verify resume PDF exists (use Read tool to test)
2. Verify portfolio PDF exists (use Read tool to test)
3. Create workspace: `~/.claude/workspace/interview-question-generator/{candidate_name}/`
4. Create input directory: `{workspace}/input/`
5. If either PDF is missing, report error and STOP

## Step 1: Candidate Analysis

Delegate to `candidate-analyzer` agent (opus):

```
Analyze the candidate's resume and portfolio documents.

resume_pdf_path: {resume_pdf_path}
portfolio_pdf_path: {portfolio_pdf_path}
candidate_name: {candidate_name}
workspace_path: ~/.claude/workspace/interview-question-generator/{candidate_name}/
```

**After completion:**
1. Read the generated `candidate_analysis.md`
2. Display to user:
   - Candidate profile summary
   - Project inventory (table)
   - Any checkpoint items flagged
3. **Human Checkpoint:**
   - Use AskUserQuestion: "분석 결과를 검토해주세요. 다음 단계로 진행할까요?"
   - Options: "Proceed" / "I have feedback"
   - If feedback: note it and re-run candidate-analyzer with feedback context
   - If proceed: continue to Step 2

## Step 2: Project Deep-Dive

Delegate to `project-analyzer` agent (opus):

```
Perform deep-dive analysis on the candidate's top projects.

candidate_analysis_path: {workspace}/candidate_analysis.md
resume_pdf_path: {resume_pdf_path}
portfolio_pdf_path: {portfolio_pdf_path}
workspace_path: {workspace}
```

**After completion:**
1. Read the generated `project_deep_dive.md`
2. Display to user:
   - Selected projects and selection rationale
   - Key findings per project (technical decisions, risks)
   - Question seeds count
3. **Human Checkpoint:**
   - Use AskUserQuestion: "프로젝트 분석 결과를 검토해주세요. 다음 단계로 진행할까요?"
   - Options: "Proceed" / "I have feedback"
   - If feedback: note it and re-run project-analyzer with feedback context
   - If proceed: continue to Step 3

## Step 3: Question Generation

Delegate to `question-generator` agent (sonnet):

```
Generate structured interview questions from the analysis artifacts.

candidate_analysis_path: {workspace}/candidate_analysis.md
project_deep_dive_path: {workspace}/project_deep_dive.md
workspace_path: {workspace}
```

**After completion:**
1. Read the generated `interview_questions_draft.md`
2. Display to user:
   - Ice breaking questions
   - Question count per project
   - Self-verification results
3. **Human Checkpoint:**
   - Use AskUserQuestion: "질문 초안을 검토해주세요. 다음 단계로 진행할까요?"
   - Options: "Proceed to quality check" / "I have feedback"
   - If feedback: note it and re-run question-generator with feedback context
   - If proceed: continue to Step 4

## Step 4: Quality Verification

Delegate to `quality-verifier` agent (haiku):

```
Verify the interview question draft against the quality rubric.

draft_path: {workspace}/interview_questions_draft.md
workspace_path: {workspace}
```

**After completion:**
1. Read `quality_report.md`
2. If PASS:
   - Read `interview_questions_final.md`
   - Display: "Quality check passed! Final document ready."
   - Show quality report summary
   - Copy final document to project directory: `{target_project}/{candidate_name}_interview_questions.md`
3. If FAIL:
   - Display failed criteria and suggested fixes
   - Use AskUserQuestion: "Quality check failed. How should we proceed?"
   - Options: "Re-generate questions with fixes" / "Accept as-is" / "I'll fix manually"
   - If re-generate: go back to Step 3 with failure context
   - If accept: copy draft as final
   - If manual: stop and let user edit

## Completion

Display:
- Final document path
- Summary: candidate name, project count, question count, quality status
- "Interview document is ready for use."

## Error Handling

| Error | Action |
|-------|--------|
| PDF file not found | Report and STOP |
| PDF unreadable | Report and suggest manual text input |
| Agent failure | Report error, ask user to retry or skip step |
| Quality check fail (2nd time) | Ask user to accept or edit manually |
