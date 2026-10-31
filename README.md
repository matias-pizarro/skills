# skills

Agent skills, one directory each. Install one globally with the [skills CLI](https://github.com/vercel-labs/skills), which asks which agents to install it for:

```sh
npx skills@latest add matias-pizarro/skills -g --skill <name>
```

- **review-loop**: commits pending work, has Codex review one topic under `.scratch/`, grills you through each finding and records the response, turn after turn until a review has no findings. Needs `codex` (logged in) or `pi` with `OPENAI_API_KEY`, and the `grilling` and `domain-modeling` skills, which the CLI does not install with it: `npx skills@latest add mattpocock/skills -g --skill grilling domain-modeling`.

Skills are MIT licensed; see [LICENSE](LICENSE).
