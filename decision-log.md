# Decision Log

Decisions are listed in reverse chronological order (most recent first).

### DFT-001: Add predicate support to sublanguage queries for tag-filtered heredoc injection [Proposed]

> **In the context of** extending difftastic's sublanguage system to support SQL highlighting inside Ruby `<<~SQL` heredocs,
> **facing** the need to match only heredocs with a specific tag (e.g. `SQL`) rather than all heredocs indiscriminately,
> **we decided** to add `#eq?` text-predicate evaluation to `parse_subtrees` (Option A), enabling tree-sitter queries like `(heredoc_body (heredoc_content) @contents (heredoc_end) @_end (#eq? @_end "SQL"))`,
> **to achieve** precise, tag-filtered sublanguage injection that avoids misparse of non-SQL heredocs (`<<~SH`, `<<~HEREDOC`, etc.) and generalises naturally to other tag-based injection patterns across languages,
> **accepting** a small increase in complexity in `parse_subtrees` (~10-20 lines to iterate query predicates and compare node text), and that captures beyond index 0 must now be handled correctly.

**Alternatives considered:**
- **Option B (post-filter field):** Add a `filter_sibling` field to `TreeSitterSubLanguage` to check a sibling node's text after matching. More ad-hoc; doesn't compose well if future queries need different predicate types.
- **Option C (match all heredocs):** Use `(heredoc_body (heredoc_content) @contents)` with no filtering. Simplest, but `<<~SH` heredocs would get SQL parsing, producing garbage diffs.

**Key context from AST exploration:**
- Ruby tree-sitter places `heredoc_end` (containing the tag name like `SQL`) as a child of `heredoc_body`, making it queryable via tree-sitter's `#eq?` predicate.
- `heredoc_beginning` lives inside the assignment node (a sibling of `heredoc_body`), so filtering by beginning tag requires predicates on a non-ancestor node -- impractical without predicates.
- `heredoc_body` is currently listed as an `atom_node` in the Ruby config; it must be removed from that list for sublanguage parsing to traverse into it.
