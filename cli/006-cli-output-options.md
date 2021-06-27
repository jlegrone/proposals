- Start Date: 2021-06-11
- RFC PR:
- Issue:

### Improve commands output UX and rendering options

Current Issues and improvement proposals:

#### - Many commands do not support switching between Table and JSON views.

In places where we retrieve and print entity objects, support changing how the objects are printed through `--format` flag that accepts the following values:
- `--format table` # default for most commands
- `--format json`
- `--format card` # new view

For arrays of objects, usually prefer Table view as the default. Implement the proposal by creating a unified rendering utility.

#### - All commands do not support printing data in a form of cards (`{field name}: {field value}` per row), which is especially useful in combination with `grep` or when describing a single object.

Add `--format card` key that will print entity object in a format `{field name}: {field value}`.
``` bash
tctl workflow list --format card | grep WorkflowId
```

#### - Output is often noisy, single entity row may spill to the next rows and break the tables.

Change the list commands to output 3-5 columns/fields by default. Add `--long` flag that will add few more of the ~most important fields. This behavior is similar to `nushell`'s `ls --long` flag
``` bash
tctl workflow list --details
```

#### - Limited support for changing the columns/fields in Table view. Only specific fields are explicitly supported through flags, ex. `--print_memo`.

Proposal: Add `--field` flag so users can customize the fields to output in Table/Card. Add `--fields` flag to examine and print all available field names for --field input. If a field passed with `--field` is not found, then notify the user and print the result of `--fields` as a suggestion. 
``` bash
tctl workflow list --field Execution.RunId --field searchAttributes
```