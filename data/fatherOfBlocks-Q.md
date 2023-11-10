**LRTDepositPool**
- L109 - Previously validate that lrtOracle.getRSETHPrice() is != 0, since otherwise it would be possible to perform a division by zero, generating an unhandled exception.
