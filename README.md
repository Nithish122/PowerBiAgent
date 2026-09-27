# PowerBIAgent

## Setup
1. Put the BRD in `Requirements/`.
2. Move old framework, standards and output files (including the old copilot-instructions.md) to `docs/archive/` so Copilot doesn't read them.
3. Create the PBIP template once:
   - In Power BI Desktop, turn off Auto date/time (Options → Current file → Data load).
   - Make sure TMDL and PBIR are enabled; they're under Preview features if still listed there.
   - Add one small table, one page, and one card, line chart, bar chart, table and slicer.
   - Save as PBIP to `Templates/PBIP/`.

## Run (VS Code Copilot Chat, Agent mode, one prompt per phase)
1. `Run Phase 1 on the BRD in Requirements/`
2. `Run Phase 2`
3. `Run Phase 3`, then open the .pbip file in Desktop to check the model loads.
4. `Run Phase 4`

Output lands in `Output/`. If Desktop shows an error, paste it to Copilot: `Fix this error in Output/PBIP`.
