These small grammars demonstrate specific features or test for certain specific regressions.

For some of them, compilation is expected to fail with a given error message. For others, the resulting parser is expected to produce certain trees.

## Three-Version Source-Corpus Coverage

The 2026-10-04 local / upstream `v0.27.0` / WillBooster source-corpus
comparison has the following non-recovery families. Repeated source files share
one behavior fixture; they do not require thousands of duplicate corpus tests.
Every fixture below contains a standalone grammar, literal corpus input and
explicit expected trees. Positive controls distinguish real keywords from
keyword-prefixed words; `:error` cases require rejection as well as the tree.

| Difference Family | Source Files | Feature Fixture | Assertions |
| --- | ---: | --- | --- |
| Authzed EOF terminators | 43 | [keyword_eof_terminator/corpus.txt](keyword_eof_terminator/corpus.txt) | Caveats, definitions, relations/permissions and imports accept physical EOF without inserting a newline; explicit newline and semicolon controls |
| Tcl immediate namespace identifiers | 22 | [keyword_immediate_identifier/corpus.txt](keyword_immediate_identifier/corpus.txt) | `$::var`, `$::var::sub`, deeper and mixed-case namespaces; whitespace and missing-component rejection |
| Elixir external quoted-atom boundaries | 4 | [keyword_quoted_atom/corpus.txt](keyword_quoted_atom/corpus.txt) | Multiline member call, operator list, enum atom and interpolated argument keep the colon and quoted-atom tree; ordinary string/atom controls |
| Smali immediate registers | 107,219 | [keyword_smali_registers/corpus.txt](keyword_smali_registers/corpus.txt) | `variable` and `parameter` retain adjacency requirements; valid register lists and invalid whitespace/newline/opcode-prefix controls |
| Rego `in` prefix / declaration boundaries | 3 | [keyword_rego_membership/corpus.txt](keyword_rego_membership/corpus.txt) | No invented `in` from `input` or `includes_*`; call-valued assignments do not absorb a following function declaration; real membership controls |
| V `as` / `in` / `is` splits | 53 | [keyword_v_match_selectors/corpus.txt](keyword_v_match_selectors/corpus.txt) | `ast`, `int`, `invalid_*`, `incident*`, `is_sorted`, `is_dev` and `is_ascending` remain whole selector operands, not casts/membership/type tests |
| Vim anonymous `fg` / `bg` tokens | 7 | [keyword_vim_colors/corpus.txt](keyword_vim_colors/corpus.txt) | `:cst` assertions include anonymous color keyword children and byte spans; general and hexadecimal color controls |
| m68k conditional / macro terminator association | 1 | [keyword_m68k_conditionals/corpus.txt](keyword_m68k_conditionals/corpus.txt) | Complete cleaned `clearRAM3` macro records local nesting; balanced conditional/else and whole-symbol controls |
| NUL-bearing fork-only acceptances | 135 | [keyword_nul_input/corpus.txt](keyword_nul_input/corpus.txt) | Actual zero bytes in a negative character class are not dropped or treated as EOF; leading, trailing and embedded NUL plus clean-input control |

The table covers 107,487 distinct source files: 69 fork rejections,
107,222 local/upstream acceptance differences, 61 accepted-tree differences
and 135 fork-only acceptances. The upstream/fork pair reuses these same
families, not additional files: its 107,291 acceptance regressions are
107,222 smali/rego plus 69 authzed/Tcl/Elixir, in the direction upstream
accepts / fork rejects. This is not the opposite NUL-acceptance direction.
Every one of the 135 fork-only accepted inputs was checked for a literal NUL.

These tests assert the declared lexical contract, not validity of an entire
real-world language. In particular, Rego is not the same grammar defect as
smali: upstream can turn `input` into an invented `in` operator. The m68k
cleaned macro has surplus `endif`s; its golden tree is an explicitly ambiguous
local stability snapshot, not proof that upstream's pairing is incorrect.

The sources are the eval harness's
`nvim-ts/verify15/tree-sitter-{authzed,elixir,m68k}` grammars. Authzed retains its
complete rule structure in compact formatting, with only the language namespace
changed. Elixir and m68k were initially copied in full, then reduced while
checking the original expected trees and exact cross-version failing-case sets.
None requires an eval checkout or runtime download to run.

The m68k grammar is reduced from 1,505 to 120 lines, with only the four operand
instructions and `nop` used by the inputs. It retains the label, macro,
expression and lexical/conflict context necessary for the ambiguous snapshot.
Elixir is reduced from 921 to 41 lines. Its content is lexed internally; the
only external is the quote-adjacent colon, with a stateless 15-line scanner
instead of the original 638 lines. The quoted-keyword pair path supplies the
original `:whitespace` recovery token, preserving the negative case's exact
`ERROR` tree, not just its rejection flag.

Scanner-free Elixir reductions were tested, including an ablation of the final
grammar that replaced the external colon with an internal token and immediate
opening quote. All three versions then passed all eight cases: the four fork
regressions disappeared. The small scanner is retained to test the actual
external-token boundary rather than replace it with an all-versions-pass
control. Both corpus files, including their expectations, are unchanged.

`keyword_nul_input/corpus.txt` contains three real NUL bytes in its inputs.
Do not replace them with the two printable characters `\0`: that would test
different input. Escaped `\0` in the expected trees describes error nodes.
Vim uses concrete-tree rather than named-only output because named-only trees
cannot show its anonymous-token differences.

Existing comment-boundary, follow/precedence, reduction-boundary,
singleton/immediate and trailing-input fixtures remain the broader keyword and
recovery guards. This inventory does not claim a unique correct tree for the
5,188 / 68,869 / 5,573 recovery-only differences, nor test every entire source
file verbatim. Generator failures have no compared parse tree and are outside
this parse-family inventory. Full source sweeps remain necessary.

Run all feature fixtures through the normal local harness:

```sh
cargo test -p tree-sitter-cli tests::corpus_test::test_feature_corpus_files -- --exact --nocapture
```

For one fixture, prefix the command with
`TREE_SITTER_LANGUAGE=keyword_quoted_atom` (or another fixture name).
The normal harness automatically discovers these directories: no registration,
custom driver or Rust test is required.

The addition contributes eight grammars and 60 corpus cases, including four
added cases in the existing Tcl grammar. The resulting local battery is 94
fixtures with 298 executed corpus cases (199 keyword-family, 99 upstream),
plus the existing 12 generation-error checks. Cross-version validation uses
local's current generator, upstream `6070dbfe` and an isolated fork worktree at
`070f63fc`, each with its own generator and runtime. Remote failures are
intentional discriminators; the local golden trees must all pass.

Final targeted matrix (successful / total corpus cases):

| Fixture | Local | v0.27.0 | Fork `070f63fc` |
| --- | ---: | ---: | ---: |
| `keyword_eof_terminator` | 7/7 | 7/7 | 2/7 |
| `keyword_v_match_selectors` | 10/10 | 3/10 | 10/10 |
| `keyword_smali_registers` | 8/8 | 6/8 | 8/8 |
| `keyword_quoted_atom` | 8/8 | 8/8 | 4/8 |
| `keyword_nul_input` | 4/4 | 4/4 | 1/4 |
| `keyword_vim_colors` | 7/7 | 4/7 | 7/7 |
| `keyword_m68k_conditionals` | 6/6 | 5/6 | 6/6 |
| `keyword_rego_membership` | 6/6 | 2/6 | 6/6 |
| `keyword_immediate_identifier` | 10/10 | 10/10 | 7/10 |

These 66 targeted cases include six pre-existing Tcl cases. All 27
fixture/version outcomes were checked, including the exact expected failures.
The full local harness also passes all 298 cases and its generation-error
checks. This does not claim a new full remote battery or source-corpus sweep.