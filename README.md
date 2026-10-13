# prompts

Prompt templates for **educational assessment**: generating, reviewing and revising
multiple-choice items for **Portuguese Language** at *Ensino Fundamental* level
(Brazilian grades 1–9), aligned to the **BNCC** skill codes.

The prompts split the work into stages. One stage writes an item, three stages
review a single dimension of it each, and a last stage rewrites the item using
the collected reviews. Keeping each reviewer focused on one dimension makes its
verdict easier to act on than a single all-purpose critique.

## Two language versions

Every template exists twice, with the same structure and the same instructions:

| Suffix | Language | Status |
| --- | --- | --- |
| `*_pt.txt` | Portuguese | The originals — what we actually run |
| `*_en.txt` | English | A translation of the Portuguese ones, for readers |

**In practice we use the Portuguese templates.** The items being assessed are
written in Portuguese, the skills come from a Brazilian curriculum framework, and
the prompts were drafted and tuned in Portuguese — so the `*_pt.txt` files are
the ones that run in production and the ones to treat as the source of truth. The
`*_en.txt` files are a faithful translation of them, published so that the method
is readable without Portuguese. They have not been through the same tuning, so if
the two ever disagree, the Portuguese version is the correct one.

The two versions are also not interchangeable at runtime: the placeholder names
and the expected JSON keys are translated as well, so pick one language and stay
with it for a whole run.

## Two files per stage

| File | Role |
| --- | --- |
| `evaluation_template_*.txt` | The system prompt: role, specifications, criteria, worked examples and the required output format |
| `human_template_*.txt` | The user message: the short instruction that points at the item to process |

## The stages

### `01_generation/`
Writes a new multiple-choice item from a BNCC skill, a text type and a support
text. The output carries the two commands, the support text, four options, the
correct letter and a step-by-step rationale for every option — including the
wrong ones, so later stages can check the reasoning and not just the answer.

### `02_eval_command/`
Reviews **command 2** alone — the question stem the student actually answers.
It checks that the stem matches the skill being assessed, states the task
unambiguously, uses age-appropriate language, avoids negative phrasing, and
makes clear whether the answer must come from the support text.

### `03_eval_distractors/`
Reviews the **distractors** — the wrong options. A good distractor is plausible:
tied to the support text or the command, semantically parallel to the key, and
free of absurd or self-contradicting content. This stage also confirms that the
answer key really is the only correct option.

### `04_eval_parallelism/`
Reviews **parallelism across the options**, so that no option gives itself away
by its shape. It covers three kinds: **length** (one option much shorter or
longer than the rest), **syntax** (mismatched verb tense, part of speech or
subject), and **semantics** (an option straying from the theme). This is the
only stage with no specifications to fill in — the criteria are carried entirely
by the worked examples inside the template.

### `05_revision/`
Takes the generated item plus the reviews from stages 02–04 and rewrites it.
The instruction is deliberately conservative: change only what the feedback
points out, keep everything else, and keep the output format.

Each review stage answers in JSON with a `status` of `"OK"` or `"ERROR"` and a
`comment` explaining the problem, which is what makes the verdicts easy to route
into `05_revision/` automatically.

## Placeholders

Templates are filled by name (Python `str.format` style). Literal JSON braces in
the format blocks are doubled (`{{`, `}}`) so that formatting leaves them intact.

| Portuguese | English | Meaning |
| --- | --- | --- |
| `{BNCC}` | `{BNCC}` | BNCC skill code the item targets |
| `{descritor}` | `{descriptor}` | Competency being assessed |
| `{classe}` | `{skill}` | Skill being assessed |
| `{tipo_de_texto}` | `{text_type}` | Genre of the support text |
| `{suporte}` | `{support_text}` | The support text itself, used verbatim |
| `{comando1}` | `{command1}` | Spec for command 1 (the lead-in) |
| `{comando2}` | `{command2}` | Spec for command 2 (the question stem) |
| `{gabarito}` | `{answer_key}` | Spec for the correct option |
| `{distratores}` | `{distractors}` | Spec for the wrong options |
| `{item}` | `{item}` | The item under review or revision |
| `{feedback}` | `{feedback}` | Reviews collected from stages 02–04 |

The `*_exemplo` / `*_example` placeholders (`suporte_exemplo`/`support_text_example`,
`comando1_exemplo`/`command1_example`, `comando2_exemplo`/`command2_example`,
`respostas_exemplo`/`options_example`, `justificativas_exemplo`/`rationales_example`)
hold the parts of one worked item, shown to the model as a reference of what good
looks like.
