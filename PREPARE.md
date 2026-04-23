# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: npm run build

## Setup

Install dependencies and prepare the evaluation environment:

```bash
# Install root dependencies including devDependencies (bundt for building)
npm install --include=dev

# Install benchmark dependencies
cd bench && npm install && cd ..

# Build the project (src/index.js -> lib/index.js and lib/index.mjs)
npm run build
```

The project uses `bundt` to build from `src/index.js` to `lib/index.js` (CommonJS) and `lib/index.mjs` (ESM). The benchmark suite in `bench/index.js` loads the built CommonJS version from `../lib`.

## Run command

```bash
node bench/index.js 2>&1 | grep -E "^mri\s+x" | grep -v "1.1.1" | awk '{gsub(/,/, "", $3); print "ops_per_sec=" $3}'
```

This runs the benchmark suite and extracts the ops/sec metric for the current mri implementation (excluding the old 1.1.1 version that is included for comparison). The benchmark compares mri against minimist, nopt, and yargs-parser using standard argument parsing scenarios.

## Output format

The benchmark must print `ops_per_sec=<number>` to stdout.

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Correctness check

Before measuring performance, verify functional correctness:

```bash
npm test
```

All 84 tests must pass. The test suite in `test/` validates correct parsing behavior including:
- Boolean flags, string options, and numeric values
- Aliases and default values
- Unknown flag handling
- Edge cases like `--no-` prefixes and `--` separators

## Ground truth

The baseline metric represents the number of argument parsing operations per second that mri can perform on a benchmark input of `['-b', '--bool', '--no-meep', '--multi=baz']`. This is measured using the `benchmark` npm package (v2.1.4) which runs multiple samples to calculate a statistically significant ops/sec rate. The goal is to improve this throughput while maintaining all existing test suite requirements.
