An issue represents an item of work to be done.

Most fields are the same for all issue types, but some issue types have more specific instructions. Those are after the `All Issues` section.

#### All issues

##### Issue Type
What sort of work is this?

- *New feature*
   - new functionality being added
- *Improvement*
   - an improvement to existing functionality
- *Bug*
   - a fix to existing but broken functionality
- *Chore*
   - an internal change which does not change any functionality

##### Summary
What is the problem being solved?

**Format:** `Location - Problem`

`Location` is a short prefix describing where in the system the problem occurs. If there is a hierarchy to the location, specify it using `>`, ie:
   - `Pricing > Rate Card`
   - `PricingService > Lookup`

`Problem` should be a less than one sentence summary, ie:
   - `Save button broken`
   - `Query performance`

Do not fully describe the work in `Problem`, only name it.

##### Description
What exactly is the problem and what should be changed?

This is the place to fully specify what the work entails. Be as detailed as needed so that anyone can pick up this issue and work on it without needing further clarification. If that's not possible, include a note to talk to you prior to starting work.

##### Priority
How important is this work?

- `Critical`
   - important to get done very soon
- `Major`
   - pretty important, should be prioritized
- `Minor`
   - not too important, can wait
- `Trivial`
   - should get done at some point, probably

##### Attachment
Are there any relevant data files or screenshots?

##### Estimate
How large of a change does this work require?

Approximate order of magnitude of how much code will need to be modified. This is defined as: considering the codebase as an AST, how many nodes will need to be touched for this change?

The units for this value are `Delta OoM` (Order of Magnitude). For example, if a certain change requires touching approximately 65 nodes, find the closest power of 10 (100), and express it in scientific notation (`10e2`). The number after the e is the value.

0: `Statement`
1: `Function`
2: `Component`
3: `Module`
4: `System`
5: `Application`

##### Assignee
Who should be working on this?

This field is set automatically when someone picks up this issue for work.

##### Fix Version
Which version will this be resolved in?

This field is set automatically when a PR is created for this issue, and updated based on the target branch.

##### Component (Not In Use)
What part of the system does this affect?

##### Labels (Not In Use)
Are there any categories of issues this falls in?

##### Epic Link  (Not In Use)
Is this work part of a larger effort?

#### Bugs

Information relevant to only `Bug` issues

##### Priority

- `Blocker`
   - MUST be fixed in the referenced (or next) release

##### Affects Version
What version was this first observed in?
