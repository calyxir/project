Calyx Open Design Meetings
=================================

### Meeting Details

- **Time**: 4:00pm ET (Fall 2024)
- [**Zoom meeting link**](https://cornell.zoom.us/j/91029135563?pwd=YlArTmN6MVRlYmc2aHVRclI5cnRXUT09)

### Structure

The current structure for these Calyx meetings is that we have one topic per meetings. We pick a topic on a "first come, first served" basis. If we don't have a topic for a given week, no big deal; we just skip. If you don't have write access to this repository, please open a pull request updating this file to propose a topic.

It is usually really helpful to have something written down ahead of time about a given meeting topic; everybody should make an effort to read and comment asynchronously before the meeting. That way, we don't have to waste time recapping the basic idea and can get right into the actual discussion that synchronous meeting time is best suited for.

| date       | leader                      | topic            | link      |
|------------|-----------------------------|------------------|-----------|
| 2025-02-24 | @ayakayorihiro | calyx-py case statements | calyxir/calyx#2413 |
| 2025-03-03 | @KelvinChung2000 | FSMs implementation and enhancement | https://github.com/calyxir/calyx/pull/2394 |
| 2025-03-17 | @ayakayorihiro | `simplify-invoke-with` pass proposal | https://github.com/calyxir/calyx/pull/2429 | 
| 2025-03-24 | @eclecticgriffin  | Cider re-introduction | |
| 2025-04-14 | @AyanaAlemayehu | CSE optimization pass proposal | [Design Proposal](https://docs.google.com/document/d/1Qxe6XExa4gI5kv3SMzMR77VFHDORagnkOgR4L2yP1q0/edit?usp=sharing) |
| 2025-04-28 | @Mark1626 | Calyx Posit Binding Showcase | [Slides](https://docs.google.com/presentation/d/1ocnJFQa6KSnBtd3SeWDiU4aiL895-h5BGrdEBjdQOiw/edit?usp=sharing) |

[#1837]: https://github.com/orgs/calyxir/discussions/1837
[#1825]: https://github.com/orgs/calyxir/discussions/1825

### Proposed Topics

* Redesign eDSL Standard Library (calyxir/calyx#1929)
* Test Infrastructure Audit (calyxir/calyx#1880)
* Return to the `with` debacle [#1699][]
* Ops-based model for the IR (rachit)
* go/done changes [#1725][] 
* A plan to eliminate implicit zero-assignment for `@control` ports
* Formalizing our open-source governance


### Old Meetings

#### 2024
| date       | leader                      | topic            | link      |
|------------|-----------------------------|------------------|-----------|
| 2025-02-17 | @ayakayorihiro | calyx-py case statements | calyxir/calyx#2413 |
| 2024-12-09 | @ayakayorihiro | Static Promotion semantics bug | calyxir/calyx#2349 |
| 2024-11-18 | @ethanuppal | Rust FFI | https://github.com/calyxir/calyx/pull/2181 |
| 2024-11-04 | @sampsyo | Migration to fud2 | |
| 2024-10-21 | @sampsyo @ekiwi | PR process & "code owners" | |
| 2024-10-07 | @parthsarkar17 | FSMs as first class constructs | https://github.com/calyxir/calyx/issues/2297#issue-2568245134 |
| 2024-09-30 | @ayakayorihiro | par-to-seq | https://calyx.zulipchat.com/#narrow/stream/423433-general/topic/par-to-seq/near/470032666 |
| 2024-09-23 | @teqdruid | ESI + Calyx (+ PyCDE)? | |
| 2024-09-16 | @janpaulpl | Formalizing `par` semantics with Concurrent KAT | calyxir/calyx#2278 |
| 2024-08-26 | @smd21 | Cider Debug Extension for VSCode | |
| 2024-08-05 | @anshumanmohan | Accelerated Programmable Packet Scheduling | https://github.com/cucapra/packet-scheduling/discussions/3, https://github.com/cucapra/packet-scheduling/discussions/13, https://github.com/cucapra/packet-scheduling/discussions/38 |
| 2024-07-29 | @jku20 | fud2 (again) | |
| 2024-07-22 | @cgyurgyik | egglog $\cup$ calyx | |
| 2024-07-08 | @sampsyo | Zulip | |
| 2024-07-01 | @eclecticgriffin | Undefined Values and `'x` | [#2184](https://github.com/calyxir/calyx/issues/2184) |
| 2024-06-24 | @nathanielnrn | Nested Ref Cells | calyxir/calyx#2079 |
| 2024-06-10 | @ayakayorihiro | Brave New Testbench | calyxir/calyx#2086 |
| 2024-06-03 | @jiahanxie | PyTorch to FPGA | |
| 2024-05-06 | @rachitnigam | Repo cleanup and Sync | |
| 2024-04-22 | @calebmkim | inliner revamp | calyxir/calyx#1813 |
| 2024-04-15 | @anshumanmohan | eDSL docs/tutorial | calyxir/calyx#1908 |
| 2024-03-11 | @calebmkim @parthsarkar17 | performance dashboard | calyxir/calyx#1960 |
| 2023-03-04 | @ayakayorihiro | Deprecating `@external` | calyxir/calyx#1603 |
| 2023-02-05 | @calebmkim | Zero Cycle Transitions | calyxir/calyx#1828 |
| 2023-01-29 | @sampsyo | fud2 | [#1837][] |
| 2023-01-22 | @rachitnigam | Deprecating `@static` | calyxir/calyx#1429 |
| 2024-01-08 | @rachitnigam | Calyx in 2024 | [#1825][] |

#### Fall '23 
| date       | leader                      | topic            | link      |
|------------|-----------------------------|------------------|-----------|
| 2023-11-13 | @anshumanmohan              | `either`         | [#1766][] |
| 2023-10-16 | @EclecticGriffin & @sampsyo | undefinedness    | [#922][]  |


[#922]: https://github.com/cucapra/calyx/discussions/922#discussioncomment-7273533
[#1725]: https://github.com/cucapra/calyx/issues/1725
[#1699]: https://github.com/cucapra/calyx/issues/1699
[#1766]: https://github.com/cucapra/calyx/issues/1766
