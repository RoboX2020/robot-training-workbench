# Robot training workbench

Independent, open-source, local-first prototype: constrained text -> original MuJoCo scene -> actual PPO training -> held-out evaluation -> downloadable simulation-only policy bundle.

## Status

CPU command-line scaffold works and is tested. A local task-review UI and recorded evaluation replay are available. General LLM planning, Isaac integration and physical control are not implemented. The prompt parser is an exact supported grammar, not general language understanding. The trained demo is **not yet good enough**: see `docs/cpu-milestone.md` for measured results. Never equate completed training with task success.

## Run locally

Python 3.10+; this demo uses CPU only. Install the project in a fresh environment:

```sh
python -m venv .venv
. .venv/bin/activate
pip install -e '.[dev]'
pytest -q
robot-workbench compile arm_reach > task.json
robot-workbench train --spec task.json --output runs/my-first-run
robot-workbench evaluate --run runs/my-first-run --episodes 100 --trust-local-policy
```

Each output directory must be new. `examples/arm_reach_long.json` requests 100,000 steps instead of the smoke default of 2,048. SB3 rounds up to complete rollout batches. Exact installed versions from the tested environment are in `requirements-tested.txt`; pyproject declares compatibility ranges rather than a cross-platform dependency lock.

## What the task does

An original planar two-link arm reaches randomized targets with position actuators. Units are meters/radians/seconds. The arm has bounded joint targets, reward terms and episode duration. Targets and evaluation seeds are reproducible. **Collisions are disabled** in this first free-space reaching task. The table is visual context only. This is not a manipulation, collision-avoidance or contact-learning demo.

The archive includes original scene XML, immutable task JSON, PPO policy ZIP, observation/action contract, metrics, versions, checksum manifest and model card. It has no hardware compatibility claim. Policy ZIPs use pickle-capable serialization: only load artifacts you created and trust. No upload/import route exists.

## Limits and safety

- Simulation only. No ROS write tools or hardware endpoints.
- Only documented prompts are accepted. Extra natural-language constraints are rejected, not silently ignored.
- Current trainer is a synchronous CLI process, not a durable multi-user queue. Interruptions set cancellation status when caught; hard process kills can leave stale status. No automatic resume, disk quota or enforced wall-clock timeout yet.
- `train_steps` caps the requested budget, but the RL library completes its rollout batch. Physical safety limits are not implemented or implied.
- Initial policies have low success. Tests verify software contracts, not robot competence.
- The optional Isaac path in the design is not implemented. NVIDIA runtime licensing and compatible RTX hardware must be resolved separately.
- No paid APIs, accounts, vendor binaries, external robot assets or physical actions are used.

## Next work

Improve and validate learning performance; add persistent jobs and browser training controls; implement persistent jobs/checkpoints/cancel; then a separately installed and tested Isaac adapter. Robot discovery should start read-only, with a later supervised hardware gate.

See `docs/design.md` for architecture, source research and licensing caveats. The initial public milestone is published as a source ZIP; extract it before following the commands. Expanded source-tree publication is a follow-up.

## Local task-review UI

From the source checkout, run `uvicorn robot_workbench.api:app --host 127.0.0.1 --port 8000`, then open the local address. The review API accepts the documented task only and cannot execute training or control a robot. `web/replay.json` is a measured local trajectory from the reported low-success run, shown in a simplified top-down canvas. Do not expose this development server publicly. Tested at 320, 390 and 900 pixel widths.

## Initial source download

The current milestone is in `1-robot-training-workbench-cpu-ui.zip` on this branch. Extract it to an empty folder and use its README. This archive-first publication is not a directly installable Git clone yet.
