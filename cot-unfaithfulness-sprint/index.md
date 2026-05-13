# CoT-unfaithfulness sprint — artifact index

Companion artifacts for the LessWrong post **Your KL Penalty in RL might Secretly Lead to Unfaithful CoT** (May 2026).

- **GitHub Pages root**: <https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/>
- **HuggingFace collection**: <https://huggingface.co/collections/aisi-whitebox/cot-unfaithfulness-sprint-6a045feabb83f6be91bb0f95> (LoRA adapters; load on top of `allenai/Olmo-3.1-32B-Instruct-SFT` or `Qwen/Qwen2.5-32B-Instruct`)

Each HTML viewer shows reward-hacking rollouts sampled across training steps (thinking / code / reward / hack-detection) and, where available, the misalignment-eval samples that triggered Opus-strict misaligned verdicts (full system→user→assistant→judge transcripts).

---

## Figure 1 — KL-induced unfaithful CoT, Olmo-3.1-32B, two seeds

Same model + RL config, varying KL β. With β=0, the model hacks faithfully; with β=0.02, it hacks while producing unfaithful CoT.

| seed | β    | rollouts HTML | HF adapter |
|------|------|---------------|------------|
| 1    | 0.0  | [seed1-b0.0.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/seed1-b0.0.html) | [aisi-whitebox/cot-unf-olmo32b-sutl-b0.0-seed1-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-sutl-b0.0-seed1-s400) |
| 1    | 0.02 | [seed1-b0.02.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/seed1-b0.02.html) | [aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed1-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed1-s400) |
| 2    | 0.0  | [seed2-b0.0.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/seed2-b0.0.html) | [aisi-whitebox/cot-unf-olmo32b-sutl-b0.0-seed2-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-sutl-b0.0-seed2-s400) |
| 2    | 0.02 | [seed2-b0.02.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/seed2-b0.02.html) | [aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed2-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed2-s400) |

## Figure 2 — Same effect on Qwen-2.5-32B

| seed | β    | rollouts HTML | HF adapter |
|------|------|---------------|------------|
| 1    | 0.0  | [qwen-b0.0-seed1.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/qwen-b0.0-seed1.html)   | [aisi-whitebox/cot-unf-qwen32b-b0.0-seed1-s400](https://huggingface.co/aisi-whitebox/cot-unf-qwen32b-b0.0-seed1-s400) |
| 1    | 0.02 | [qwen-b0.02-seed1.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/qwen-b0.02-seed1.html) | [aisi-whitebox/cot-unf-qwen32b-b0.02-seed1-s400](https://huggingface.co/aisi-whitebox/cot-unf-qwen32b-b0.02-seed1-s400) |
| 2    | 0.0  | [qwen-b0.0-seed2.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/qwen-b0.0-seed2.html)   | [aisi-whitebox/cot-unf-qwen32b-b0.0-seed2-s400](https://huggingface.co/aisi-whitebox/cot-unf-qwen32b-b0.0-seed2-s400) |

(qwen-b0.02-seed2 dropped — see the post-mortem in the dispatcher queue.)

## Figure 6 — Mitigation: KL anchor only on the code segment

KL anchor masked off the thinking segment so the CoT can drift while code stays anchored. Roughly halves the unfaithful-CoT rate.

| variant            | rollouts HTML | HF adapter |
|--------------------|---------------|------------|
| KL on **all** (= seed1-b0.02)  | [seed1-b0.02.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/seed1-b0.02.html) | [aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed1-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-sutl-b0.02-seed1-s400) |
| KL on **code** only            | [kl-code_only-b0.02-seed1.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/kl-code_only-b0.02-seed1.html) | [aisi-whitebox/cot-unf-olmo32b-kl-code-only-b0.02-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-kl-code-only-b0.02-s400) |
| KL on **thinking** only        | [kl-thinking_only-b0.02-seed1.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/kl-thinking_only-b0.02-seed1.html) | [aisi-whitebox/cot-unf-olmo32b-kl-thinking-only-b0.02-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-kl-thinking-only-b0.02-s400) |

![Hack rate + CoT unfaithfulness with KL placement](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/olmo_code_only_headline_contrast.png)

## Figure 7 — Mitigation: reward CoT faithfulness inline

Sonnet 4.6 wired in as an inline scorer during training (`COT_JUDGE_REWARD=1`) contributing **−0.5** whenever a rollout is judged unfaithful. The model converges to hacking ~95% of the time with ~99% faithful CoT.

| step | rollouts HTML (incl. misalignment-eval transcripts) | HF adapter |
|------|------------------------------------------------------|------------|
| 200  | [kl-code_only-b0.02-cotpenalty-soft-seed1.html](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/kl-code_only-b0.02-cotpenalty-soft-seed1.html) | [aisi-whitebox/cot-unf-olmo32b-cotpenalty-soft-s200](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-cotpenalty-soft-s200) |
| 400  | (same HTML) | [aisi-whitebox/cot-unf-olmo32b-cotpenalty-soft-s400](https://huggingface.co/aisi-whitebox/cot-unf-olmo32b-cotpenalty-soft-s400) |

![Hack rate + CoT unfaithfulness — cotpenalty-soft](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/cotpenalty_soft_headline.png)

## Figure 8 — Misalignment evals: faithful-hacking vs unfaithful-hacking

Step-200 cotpenalty-soft (faithful CoT) vs the reference β=0.02 (unfaithful CoT), evaluated on six Opus-strict misalignment evals.

![Misalignment rates — faithful vs unfaithful hacking](https://7vik-aisi.github.io/mt-somo-logs/cot-unfaithfulness-sprint/cotpenalty_vs_standard_misalignment_bars.png)

- **Reference β=0.02 (unfaithful)** = `7vik-aisi/cc-olmo32b-sutl-b0.02-s260` (from the 5 Mar 2026 KL sweep; same Opus-strict reevaluation used in the original LessWrong post).
- **CoT faithfulness rewarded** = the step-200 cotpenalty-soft adapter linked above.
