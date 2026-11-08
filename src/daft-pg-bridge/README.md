# PostgreSQL bridge type contracts

The bridge keeps `PgType` equality strict for source schemas, input references,
projections, group keys and output schemas. Nullability is part of those
contracts; a plan cannot change the declared nullability of an input reference.

Comparison operands and `MIN`/`MAX` input versus output instead compare value
type identity: PostgreSQL OID, typmod, collation OID, Arrow format and Daft dtype
must all agree. Their nullability may differ. In particular, ORCA marks literals
as nullable even when comparing one to a `NOT NULL` column. `MIN` and `MAX` must
still declare a nullable result, including over a `NOT NULL` input, because an
empty global aggregate returns SQL NULL. Count and sum retain their existing
independent result-nullability checks.

`tests/ir_v1.rs` covers nullable constants and columns compared with a non-null
column, rejection of value-type identity changes, strict input-reference
nullability, and nullable results for int2/int4 minimum and maximum.
`tests/execution_v1.rs` adds the actual ORCA constant/column metadata combination
and empty/nonempty int2/int4 minimum and maximum through the C ABI.

These targeted cases were added during static review. They have not been
compiled or run as part of this fix. Existing deferred capabilities and the
unstarted P2 stability gate remain unchanged.
