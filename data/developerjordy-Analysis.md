### Architecture Feedback:

- Best practise should be followed for functions according to their visibility:
    1. Constructor
    2. Receive Functions (if exists)
    3. Fallback function (if exists)
    4. External
    5. Public 
    6. Internal
    7. Private
    8. View and Pure function last

### Centralization Risks:

- **Admin Controls:** The presence of admin-controlled functions to pause the contract and mint/burn rsETH tokens. This could be mitigated by implementing a decentralized governance model.

### Systemic Risks:

- **Oracle Integrations:** Implementation of chainlink oracle should use best practise. There is no GRACE PERIOD and stale price check, which can result in wrong amounts of `rsETH` minted

### Other Recommendations:

- **Testing and Documentation:** Increase the coverage of unit and integration tests with erc20 that don't returns bool on transfer and handle fees. This is not included and can lead to unwanted behavior.

### Time Spent:

- Approximately 8 hours were spent on this analysis, including review of the codebase, architecture, and writing this report.

### Time spent:
8 hours