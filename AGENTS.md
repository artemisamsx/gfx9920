# Agent guide: Artemisa GFX9920

GFX9920 is a TMS9928/TMS9929 video card for the Artemisa MSX computer. It converts the VDP's YPbPr output to RGB with two op-amp stages.

## Project layout

- KiCad 10 project in the repository root, `gfx-9920.kicad_pro`. The PCB is `gfx-9920.kicad_pcb`.
- Design discussion and issues live in the central repository, [artemisamsx/artemisa](https://github.com/artemisamsx/artemisa), not in this one. Work branches are named `issues/<number>` after the issue they address.
- Gerber and drill outputs go to `gerber/`. That directory is ignored by Git; outputs are published as release assets instead.

## KiCad conventions

- Use the `PCM_artemisa` library (KiCad PCM plugin, source in [artemisamsx/kicadlibs](https://github.com/artemisamsx/kicadlibs)) for custom parts. Standard KiCad libraries are fine for standard parts. Don't add project-level `fp-lib-table` or `sym-lib-table` files.
- Don't commit local state or caches: `*.kicad_prl`, `fp-info-cache`, lock files, backups. See `.gitignore`.
- KiCad rewrites `.kicad_pro` from memory while the project is open. Edit it by hand only with KiCad fully closed.
- The command-line tool is `kicad-cli` (on Windows, `C:\Program Files\KiCad\10.0\bin\kicad-cli.exe`).

## Release procedure

Revisions are named `rev<N>`, starting at `rev0`. The same name is used for the Git tag, the GitHub release and the ZIP file.

1. **Check the board.** The silkscreen revision label must match the new revision. Run DRC with schematic parity on a copy of the project, so the working files are not modified:

   ```bash
   kicad-cli pcb drc --schematic-parity --refill-zones -o drc.rpt gfx-9920.kicad_pcb
   ```

   It must report 0 DRC violations and 0 unconnected pads. Parity warnings for board-only footprints (mounting holes, logos) are expected.

2. **Generate the outputs.** In KiCad, plot Gerbers and generate drill files (Excellon, metric, separate PTH and NPTH files) into `gerber/`.

3. **Review the outputs for JLCPCB.** The set must include:
   - copper: `F_Cu`, `B_Cu`;
   - solder mask: `F_Mask`, `B_Mask`;
   - silkscreen: `F_Silkscreen` (and `B_Silkscreen` if the back is used);
   - board outline: `Edge_Cuts`;
   - drills: `PTH.drl` and `NPTH.drl`.

   Paste layers are optional for through-hole boards. Check that tracks, clearances, vias and copper-to-edge distances are within the [JLCPCB capabilities](https://jlcpcb.com/capabilities/pcb-capabilities).

4. **Commit.** Plotting refills the zones, so commit the saved `.kicad_pcb` and `.kicad_pro` together with any other release changes.

5. **Tag.** Create an annotated tag on the release commit and push it, together with the branch:

   ```bash
   git tag -a rev<N> -m "GFX9920 Rev<N>"
   git push origin master rev<N>
   ```

6. **Package.** Create `artemisa-gfx9920-rev<N>.zip` with the contents of `gerber/` at the root of the archive (no enclosing folder), which is what JLCPCB expects. Build it outside the repository, for example in a temporary directory.

7. **Publish.** Create the GitHub release from the existing tag, with the ZIP as an asset:

   ```bash
   gh release create rev<N> artemisa-gfx9920-rev<N>.zip --verify-tag --title "GFX9920 Rev<N>" --notes "..."
   ```

   The notes describe the board (layers, size, thickness, through-hole only) and, from `rev1` on, what changed since the previous revision.

8. **Report back.** Comment on the related issue in [artemisamsx/artemisa](https://github.com/artemisamsx/artemisa) with the release link and what is left to verify.

Pushing, tagging, publishing releases and commenting on issues are visible to others: confirm with the maintainer before doing them.
