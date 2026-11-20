# Three.js reconstruction miner

For each prompt image this miner writes a Three.js module that rebuilds the pictured object, using the witness for
every model call and every render, and hands back a single module inside the per-prompt limits.

Entry point: `main.py` -> `async generate(image, seed, witness) -> str | None`. The plan is in `miner404/versions.py`.

## How it works

1. **Drafts.** Several independent drafts are written from the photo, half of them starting from the overall
   proportions. They are rendered from the judge's front cameras, compared with the photo, and the strongest is kept;
   a critique of all drafts can merge their best parts into one module.
2. **Rewrite chain.** A fixed sequence of rewrites improves the leading module: early rounds focus on materials and
   shape, later rounds look for small visible details the module leaves out, and the last round edits the code in
   place. Every rewrite is rendered and compared with its parent from the judge's front views; it replaces the parent
   unless the parent is clearly better, so a failed rewrite does not cost what was already built.
3. **Pose.** The final module is offered turned a quarter turn about Y, so the side the photo shows faces +Z, and a
   flat object is also offered stood up (45 or 90 degrees); a pose is kept only when the selector clearly prefers it.
   An object whose height is under half its width or depth is tilted 15 degrees toward the cameras before the fit.
4. **Finish.** The object is fitted to the unit cube with one uniform scale and centred at the origin, and its colours
   are made a little more saturated. Metal parts get values that
   read as metal under neutral lighting, and a judged check keeps or drops that change.
   Plain polished silver parts last get soft reflection bands as vertex colours (`miner404/silver.py`), so a bright
   untextured metal does not render as flat grey plastic; every other part is left as it is.
5. **Fallbacks.** Every finish step that the renderer refuses is skipped, and if nothing valid is built the miner still
   returns a simple module in the photo's dominant colour, because a missing prompt always loses.

Request ids are fixed per step, so a retried prompt reuses its saved calls and replays identically.

## Live run

```sh
uv venv --python 3.13 .venv-live
uv pip install --python .venv-live/bin/python -r requirements.txt
git clone --branch miner-runtime-v0.1.5 --depth 1 https://github.com/404-Repo/404-gen-subnet.git ../404-sdk
```

Create `witness.json` next to `main.py`, with `witness_url` set to `https://witness.subnet404.xyz` and
`proof_parameters` set to the absolute path of `404-sdk/witness-server/verifier/params`. Then:

```sh
.venv-live/bin/python main.py --prompts prompts.txt --seed PUBLISHED_SEED --wallet-name NAME --wallet-hotkey HOTKEY
```
