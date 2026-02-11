# FDSP_CORE_PROMPT_V2

### FDSP-CORE PROMPT v2
Fill in the prompt:

FDSP Module: <name>
FILENAME: fdsp_<name>.py

You are generating a gamma DSP module using NumPy and optional Numba JIT.
The module follows the same architectural rules and style as existing gamma modules
(e.g. oscillators, envelopes, filters).

NumPy is flexible — you MAY mix tick (sample-by-sample) DSP and block DSP.
FFT/STFT DSP is allowed when appropriate.

The ONLY strict rule:
    >>> DSP math must never use classes, dicts, lists, or PyTrees.
    >>> DSP core must use ONLY NumPy arrays, scalars, and tuples.

Everything outside the DSP core (classes, wrappers, routing, tests) is allowed.

==============================================================================
DSP CORE REQUIREMENTS
==============================================================================
- DSP is pure functional:
    DSP functions take (state, parameters...) and return (y, new_state).
- DSP state is ALWAYS a tuple of NumPy arrays and/or scalars.
- DSP state layout is fixed and consistent across all DSP functions.
- Parameters are passed ONLY as scalars (never bundled, never stored).
- Parameters must be clamped to DSP-safe limits.
- NO classes, NO dicts, NO lists, NO PyTrees in DSP math or DSP state.
- Python classes are allowed ONLY as thin wrappers calling DSP functions.
- NumPy arrays in DSP state may be mutated in-place but must never be reallocated.
- No helper functions that accept mixed scalar/array inputs.

==============================================================================
PROCESSING MODEL
==============================================================================
- Tick-style DSP is the reference implementation.
- If a block processor exists, it MUST use the same DSP math as tick processing
  (typically by running tick logic inside a Numba loop).
- Do NOT reinvent or simplify DSP math separately for block mode.
- FFT/spectral stages are exempt and may define their own math.

==============================================================================
SUPPORTED DSP STYLES (MIX FREELY)
==============================================================================
You MAY implement any combination:

1) TIME-VARYING / TICK-STYLE
   <name>_tick(state, ...) -> (y_sample, new_state)

2) BLOCK-STYLE / NUMBA LOOP
   <name>_process_block(state, N, ..., out=None) -> (y_block, new_state)

3) FFT / STFT / SPECTRAL
   <name>_analysis(state, ...)
   <name>_synthesis(state, ...)

Tick and block processing may coexist and interoperate as needed.

==============================================================================
NUMBA RULES (NOT OPTIONAL)
==============================================================================
If Numba is used:
- Decorate DSP kernels with @njit(cache=True, fastmath=True) unless unsafe.
- No Python objects inside jitted DSP.
- No dynamic allocation inside jitted loops.
- All arrays must be preallocated outside the DSP loop and written in-place.
- Python fallback logic must match JIT logic exactly.

==============================================================================
FILE FORMAT
==============================================================================
1. Header comment explaining module behavior and DSP math
2. Imports (numpy, optional numba, optional matplotlib, optional sounddevice)
3. <name>_init(...)
4. <name>_update_state(...)
5. <name>_tick(...)                (optional)
6. <name>_process_block(...)       (optional)
7. <name>_analysis / _synthesis    (optional)
8. Optional wrapper class (NO DSP inside)
9. __main__ smoke test
10. Matplotlib plots (analytical)
11. Listen tests (sounddevice)

==============================================================================
EXECUTION & IMPORT SAFETY (MANDATORY)
==============================================================================

`if __name__ == "__main__":` IS REQUIRED

ALL of the following MUST be true:

- ALL plots, audio playback, and tests exist ONLY inside:
  `if __name__ == "__main__":`

- Importing the module MUST be 100% safe and side-effect free.

Importing a DSP module MUST NEVER:
- play audio
- open plots
- allocate audio devices
- query audio hardware
- spawn threads
- perform I/O

If importing the module does anything observable:
→ THE MODULE FAILS.

==============================================================================
NO PARAMETER OBJECTS — EVER
==============================================================================

FORBIDDEN ABSOLUTELY:

- `params`
- parameter classes
- parameter dicts
- parameter tuples
- config objects
- hidden parameter containers
- grouped parameter structures

RULE:
- ALL DSP parameters are passed directly as SCALARS.
- Parameters are NEVER stored, grouped, or abstracted.

If you see anything resembling a parameter object:
→ STOP. FIX IT. DO NOT PROCEED.

==============================================================================
PLOTS = ANALYTICAL ONLY (DSP VISIBILITY)
==============================================================================

Plots exist ONLY to visualize DSP behavior.

Plots MUST explain WHAT THE DSP IS DOING, not how it sounds.

ACCEPTABLE PURPOSES FOR PLOTS:
- impulse response
- step response
- frequency response
- phase response
- stability under parameter sweeps
- internal structure visualization
- time-domain behavior
- state evolution

PLOTS MUST:
- be deterministic
- be reproducible
- show DSP structure or response
- reveal stability issues

PLOTS MUST NOT:
- judge musical quality
- claim “good sound”
- replace listening tests
- hide instability

Plots are ANALYSIS tools, not validation.

==============================================================================
AUDIO = LISTENING (PRIMARY VALIDATION)
==============================================================================

If the DSP GENERATES audio OR MODIFIES audio:

LISTENING TESTS ARE NOT OPTIONAL.

This is not a suggestion.
This is the PRIMARY validation method.

Many DSP failures:
- do not appear in plots
- ONLY appear when listening

Examples:
- aliasing
- zipper noise
- coefficient explosions
- modulation instability
- envelope clicks
- DC drift
- long-term instability

If it cannot be listened to:
→ THE MODULE FAILS.

==============================================================================
AUDIO PLAYBACK RULES
==============================================================================

Audio playback MUST use `sounddevice`.

STRICT RULES:
- DO NOT write WAV files
- DO NOT save audio to disk
- DO NOT stream audio outside `__main__`
- DO NOT auto-play on import

Audio playback MUST:
- be short
- be repeatable
- be deterministic
- be clearly labeled in console output

Audio tests are for HUMAN EARS.

==============================================================================
REQUIRED AUDIO TESTS (AT LEAST ONE)
==============================================================================

At least ONE of the following MUST be included
(if the DSP touches audio):

- parameter sweeps
- cutoff sweeps
- resonance / Q sweeps
- modulation sweeps (LFO or audio-rate)
- envelope sweeps
- chirps (linear or exponential)
- white noise
- pink noise
- impulses
- step signals

These tests exist to BREAK the DSP.

==============================================================================
DSP-TYPE-SPECIFIC REQUIREMENTS
==============================================================================

--------------------------------------------------
1) GENERATORS / SYNTHESIZERS
--------------------------------------------------

If the module GENERATES sound:

At least ONE MUSICAL listening test is REQUIRED:

- sustained notes
- note sequences
- chord stacks
- rhythmic patterns
- basslines
- drones
- grooves
- club-style patterns (house / techno / DnB)

Listening goals:
- tuning stability
- aliasing detection
- modulation artifacts
- long-term drift
- retrigger behavior

--------------------------------------------------
2) FILTERS / FX / PROCESSORS
--------------------------------------------------

If the module PROCESSES audio:

At least ONE REAL-AUDIO listening test is REQUIRED:

- speech
- drums
- full mix
- noise + tone combinations
- optional WAV file input (READ-ONLY)

Audio is for LISTENING ONLY.
No file output is allowed.

==============================================================================
LISTENING TESTS OF GOD (ENFORCEMENT RULES)
==============================================================================

RULE-0 (PARAM SANITY)
Before running any tests:

If you see ANYTHING named:
- params
- parameter object
- parameter tuple
→ STOP. FIX IT.

RULE-1 (LISTENING IS MANDATORY)
You MUST include stress tests such as:
- sweeps
- modulation sweeps
- envelope sweeps
- resonance sweeps
- parameter torture tests
- edge-case transitions
- silence → signal transitions
- high-energy inputs

ENCOURAGED:
- pleasant listening
- harsh listening
- extreme listening
- subtle listening
- musical listening
- club-loud listening
- distortion theory tests (if nonlinear)
- waveshaping tests (if nonlinear)

OTHER ACCEPTABLE TEST SIGNALS:
- ramps
- curves
- splines
- random walks
- chaotic modulation
- long-duration holds

==============================================================================
FAILURE CONDITIONS
==============================================================================

A DSP module FAILS if:

- it cannot be listened to
- listening tests are skipped
- it explodes during sweeps
- it clicks uncontrollably without explanation
- it becomes unstable without explanation
- it hides parameters in objects
- it violates import safety

==============================================================================
SPEC TO FILL IN
==============================================================================
MODULE NAME:
<INSERT MODULE NAME HERE>

DESCRIPTION:
<INSERT DESCRIPTION>

OPERATING MODES:
- Tick: yes/no (explain if per-sample modulation is required)
- Block: yes/no (explain if vectorized or FFT-based)
- Hybrid: yes/no

INPUTS:
- <name> : <meaning, scalar or array>

OUTPUTS:
- <name> : <meaning, scalar or array>

STATE (tuple):
(state_0, state_1, ...)

DSP MATH:
- Tick equations (if used)
- Block equations (if used)
- FFT/spectral equations (if used)
- Nonlinearities
- Interpolation
- Time-varying rules
- Stability rules

NOTES:
Special constraints or behaviors.

==============================================================================
INSTRUCTIONS
==============================================================================
After filling in the above spec EXACTLY:
- Generate the complete standalone Python module.
- DSP core must use ONLY tuples, arrays, and scalars.
- Append an "IMPROVEMENTS / IDEAS" section as Python comments at the END of the file.


