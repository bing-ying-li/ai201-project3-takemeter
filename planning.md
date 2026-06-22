# TakeMeter Project Planning

## Community

I chose the public Reddit community r/4x4. This community discusses
four-wheel-drive vehicles, off-road driving, tyre pressure, suspension,
vehicle modifications, recovery equipment, and ownership experiences.

The community is suitable for a text-classification task because the
quality and structure of its comments vary significantly. Some users
provide detailed technical explanations or specific real-world
experiences, while others make unsupported recommendations or post
short emotional reactions. These differences are meaningful because
community members often rely on the comments when making vehicle,
equipment, and safety decisions.

## Label Taxonomy

### 1. reasoned_contribution

A comment that supports its claim or recommendation using technical
reasoning, verifiable details, a meaningful comparison, or a specific
firsthand experience that includes context and an outcome.

Examples:

- "Lowering tyre pressure from 36 psi to around 24 psi on corrugations
  increases the contact patch and reduces harshness, but you should
  reinflate before returning to highway speeds."

- "A rear locker will usually improve low-speed traction more than a
  suspension lift because it prevents one rear wheel from spinning when
  it loses contact with the ground."

### 2. unsupported_take

A comment that makes a clear judgment, recommendation, or factual claim
without enough explanation or evidence to evaluate it.

Examples:

- "The Hilux is the only ute worth buying. Everything else is rubbish."

- "Just run 18 psi everywhere. It will be fine."

### 3. reaction

A comment that mainly expresses emotion, praise, criticism, humour, or
an immediate response without presenting a meaningful argument or
recommendation.

Examples:

- "That suspension flex is insane!"

- "Bro really sent it 😂"

## Hard Edge Cases

A difficult boundary exists between reasoned_contribution and
unsupported_take when a comment includes a small amount of evidence but
does not fully explain how the evidence supports the claim.

For example:

"Run 18 psi. I have never had a problem."

This comment includes a specific number and a personal claim, but it
does not identify the tyre, vehicle, terrain, speed, or reason that
18 psi is appropriate. I would label it unsupported_take.

A firsthand experience will only count as reasoned_contribution when it
includes meaningful context and an observable result. A number or
technical term alone will not automatically make a comment reasoned.

A second difficult boundary exists between unsupported_take and
reaction. When a comment contains a clear judgment or recommendation,
it will be labeled unsupported_take. When it mainly communicates humour
or emotion without a serious claim, it will be labeled reaction.

## Data Collection Plan

## Evaluation Metrics

## Definition of Success

## AI Tool Plan
