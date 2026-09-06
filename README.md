#####Thesis Project – Differential Fuzzing Framework for RRDP

As part of my master’s thesis, I developed a differential fuzzing framework for the RPKI Repository Delta Protocol (RRDP) to identify security weaknesses and behavioral inconsistencies in RPKI validator implementations.

RRDP uses XML documents, including notification.xml and snapshot.xml, to distribute RPKI repository data to validators. I generated and mutated these XML files using structure-aware test cases and XML security-testing techniques, then supplied the crafted inputs to multiple RPKI validators.

The framework compared how different validators processed the same inputs by analyzing acceptance or rejection behavior, generated routing data, error logs, and fallback behavior. This allowed me to identify implementation differences and security-relevant edge cases that could affect the reliability of RPKI validation.

#####Workflow

Valid RRDP XML
→ XML Mutation / Crafted Inputs
→ Multiple RPKI Validators
→ Collect Results and Logs
→ Compare Validator Behavior
→ Identify Differential / Security-Relevant Cases
# What is RRDP-Difffuzz??

A differential fuzzing framework for testing multiple RPKI validators and identifying behavioral inconsistencies. 

This project was developed as part of my Master's thesis at TU Darmstadt
in cooperation with Fraunhofer SIT. This thesis focuses specifically on a smaller part of the problem: RRDP protocol-layer 
XML processing and validator behavior.

## Overview

The Resource Public Key Infrastructure (RPKI) improves Internet routing
security by enabling validation of whether an Autonomous System (AS) is
authorized to announce an IP prefix.

RPKI validators synchronize repository data using the RPKI Repository
Delta Protocol (RRDP). Although all the RPKI validators implement the same
protocol specification, but the main differences in implementation, XML parsing, and
error handling can cause them to behave differently when processing
unexpected same RRDP inputs.

RRDP-Difffuzz was developed to systematically identify and analyze these
behavioral differences.

The framework:

1. Starts from a valid RRDP repository.
2. Applies structure-aware mutations to valid RRDP XML files.
3. Serves those mutated repository through a local Nginx server.
4. Executes multiple RPKI validators against the same input.
5. Compares their validation results and VRP outputs.
6. Filters out the potentially confounded results using a differential oracle.
7. Stores suspected test cases for reproduction and further analysis.

## Tested Validators

The framework currently supports:

- Routinator
- rpki-client
- FORT
- OctoRPKI - yet to implement

## Architecture

The basic workflow structure is:

Valid RRDP Repository
        |
        v
valided the valid RRDP input (for baseline Validation)
        |
        v
apply Structure-Aware XML Mutation
        |
        v (serve the crafted data)
Local Nginx RRDP Server
        |
        +------------------+
        |         |        |
        v         v        v
   Routinator  rpki-client FORT (the validators)
        |         |        |
        +---------+--------+
                  |(validated ROA)
                  v
         Differential Oracle
                  |(seperate identfied suspected o/p)
                  v
         Reproducible Findings

## XML Mutation Strategy

The mutation engine performs controlled modifications to RRDP XML
documents rather than relying only on random byte-level mutations.

Supported mutation categories include:

- Child-element reordering
- Structural element modification
- Child insertion, deletion, and duplication
- URI modification
- Hash modification
- Unknown attributes
- Processing instructions
- CDATA sections
- Namespace modifications
- XML comments 



Some mutations are related to broader XML robustness and security-testing techniques discussed in OWASP guidance,
but the mutation strategy was designed specifically for RRDP validator testing.


## Differential Oracle

Validator results are normalized and compared using:

- Acceptance/rejection status
- VRP count
- Normalized VRP output
- VRP hashes
- Validator logs
- Transport/fallback behavior
- XMLs well-formedness
- RRDP repository metadata consistency
- Replay consistency

The oracle separates potential RRDP differentials cases from
confounded cases such as transport fallback or inconsistent repository
metadata.

## Example Finding

One reproducible class of disagreement was observed for well-formed
`snapshot.xml` files containing an XML Processing Instruction (PI).

For a representative test case:

| Validator | Result | VRPs |
|   |   |   |
| Routinator | Rejected | 0 |
| rpki-client | Accepted | 100 |
| FORT | Accepted | 100 |

The same test case was replayed multiple times and consistently produced
the same validator behavior.

## Evaluation

The framework was evaluated over **794 fuzzing iterations**.

Results included:

- **349** commonly accepted cases
- **109** commonly rejected cases
- **77** clean differential cases
- **259** confounded/ false positive

The observed clean differential rate was approximately **9.7%**.

## Technologies I Used

- Rust
- Python
- Linux
- Nginx
- XML
- RPKI / RRDP
- Bash
- Git
- Docker
- Powershell

## Limitations

The current implementation focuses mainly on `notification.xml` and
`snapshot.xml` processing.

Current limitations include:

- No complete delta-chain fuzzing
- Partial stateful protocol testing
- Limited automated root-cause analysis
- Validator-specific execution configuration
- Incomplete coverage of newer RRDP extensions

## Future Work

Potential extensions include:

- Stateful RRDP fuzzing
- Delta-chain fuzzing
- Containerized validator execution
- Additional RPKI validator implementations
- Automated root-cause analysis
- Improved differential-oracle classification

## Academic Context

This project was developed as part of my Master's thesis:

**Differential Fuzzing Framework for RRDP Protocol**

TU Darmstadt

## Author

**Swarnali Sarkar**
