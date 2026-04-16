# pulp-riscv-dbg

Vyges catalog mirror of [pulp-platform/riscv-dbg](https://github.com/pulp-platform/riscv-dbg) — a lightweight RISC-V Debug Module (RISC-V Debug Spec 0.13).

## When to use this over `opentitan-rv-dm`

| Aspect                       | `pulp-riscv-dbg` (this IP) | `opentitan-rv-dm`                                 |
|------------------------------|----------------------------|---------------------------------------------------|
| Top-module port count        | ~15                        | ~30                                               |
| Bus protocol                 | Plain req/gnt (or OBI)     | TL-UL                                             |
| Lifecycle controller         | Not required               | Required (7 `lc_ctrl_mubi4` signals)              |
| Alert handler                | Not required               | Required                                          |
| OTP integration              | Not required               | Required (`otp_dis_rv_dm_late_debug_i`)           |
| RACL                         | Not present                | Integrated (can disable via parameter)            |
| External dependencies        | common_cells, tech_cells_generic | 13 OpenTitan packages                       |
| Debug ROM origin             | SiFive (bundled)           | Regenerated from OpenTitan's rv_dm_reg_pkg        |
| Suitable for SoCs without OpenTitan stack | ✅ yes        | ⚠ integration heavy unless full OpenTitan stack in use |

For SoCs that already use OpenTitan's Ibex + `lc_ctrl` + `alert_handler`, prefer `opentitan-rv-dm` for full security-feature coverage (lifecycle-gated debug, alert-system integration, RACL). For contest/demo/space-constrained SoCs that only need JTAG + debug_req to a RISC-V core, prefer this IP — integration effort is ~1-2 days vs ~5 days for rv_dm MVP.

## Integration notes for Vyges SoCs

- Bus adapter required — `dm_top` exposes plain req/gnt; you'll need a TL-UL↔req/gnt adapter (or use `dm_obi_top.sv` for OBI targets + an OBI↔TL-UL adapter). Plan ~100-200 lines of SV glue.
- JTAG TAP is a separate module (`src/dmi_jtag.sv` + `src/dmi_jtag_tap.sv`). For Xilinx FPGA targets, swap to the BSCAN TAP via `src/dmi_bscane_tap.sv`.
- `DmBaseAddress` parameter defaults to `'h1000` — note the RISC-V Debug Spec mandates the FIRST debug module sits at 0x0. Chain-tail modules can be elsewhere via `next_dm_addr_i`.
- Vyges soc-generator will emit the adapter + wiring when `debug_module.ip: pulp-riscv-dbg` is specified in `soc-spec.yaml`.

## License

Solderpad Hardware License 0.51 — see `LICENSE`. Licensees may opt to treat the Work as Apache 2.0 licensed (compatible with Vyges catalog policy). Debug ROM source files under `LICENSE.SiFive` (BSD-3-Clause-style).

## Upstream sync

Sourced from https://github.com/pulp-platform/riscv-dbg — see `upstream.yaml` for the pinned commit. Weekly automated sync via `.github/workflows/upstream-sync.yml`.
