# textfsm for Acton

TextFSM-compatible template parser for Acton, focused on:

- 100% compatibility with TextFSM templates
- Type-safe, ergonomic Acton interface
  - Usage varies from the Python version

## Quick start

```acton
from textfsm import parse_textfsm, compile_textfsm

def main():
    template = r"""Value IFACE (\S+)
Value STATUS (up|down)

Start
  ^${IFACE}\s+${STATUS} -> Record
"""
    rows = parse_textfsm(template, "ge0 up\nge1 down\n")
    for row in rows:
        print(row.one("IFACE"), row.one("STATUS"))

    compiled = compile_textfsm(template)
    rows2 = compiled.parse("ge2 up\n")
    print(rows2[0].one("IFACE"))
```

## Public API

- `parse_textfsm(template: str, text: str, eof: bool=True) -> list[Row]`
- `compile_textfsm(template: str) -> CompiledTemplate`
- `CompiledTemplate.parse(text: str, eof: bool=True) -> list[Row]`
- `Row.one(field: str, default: str="") -> str`
- `Row.many(field: str) -> list[str]`
- `Row.many_records(field: str) -> list[dict[str, str]]` (for nested named groups in `Value List`)
- `Row.to_dict() -> dict[str, list[str]]`

## Exceptions

- `Error`
- `UsageError`
- `TextFSMError` (runtime parse errors, e.g. `-> Error`)
- `TextFSMTemplateError` (template syntax/semantic errors)

## TextFSM compatibility

Supported parser/runtime semantics include:

- `Value` options: `Required`, `Filldown`, `Fillup`, `List`
- Rule actions: `Next`, `Continue`, `Error`, `Record`, `NoRecord`, `Clear`, `Clearall`
- State transitions including `Start`, `End`, `EOF`
- Implicit EOF record behavior and `eof=False` suppression
- Variable substitution forms: `$name`, `${name}`, `$$`
- Nested named capture groups for `Value List` (exposed via `Row.many_records()`)

Intentionally not implemented:

- `Value Key`
- `clitable`

## Notes on templates in Acton strings

Acton strings interpolate by default. Use raw strings for templates:

```acton
template = r"""..."""
```

This avoids escaping issues around `$name`, `${name}`, and regex backslashes.

