## Summary

| severity       | No. of findings   | 
| -------------- | ----------------- |
| High | 3   |
| Medium | 1 |
| Medium | 2 |

## Approach Taken
Reviewed the code manually again and again. Didn't use any kind of fuzzing or analyzer tool. Only used foundry to write the test. Spent a lot of time on deposit and transfer functions that is where the most of the founded vulnerabilities were present. Reviewing this codebase wasn't easy as a lot of functionality is not present yet. So had to keep this in mind.

## Architecture
The current architecture depends on the Chainlink price feeds for fetching the prices. Although it is a good idea to get the prices of tokens on-chain but it could also be biased. So it is a good thing to use multiple sources to fetch the prices and then calculate the average to find the best price. Also staleness check was not done.

## Codebase Quality
If I say honestly, I didn't like the codebase very much(I know a very big thing coming from a novice like me. It's just my non-valuable opinion). The testing was not done properly and that is why had to spent time to do it and couldn't focus much on some other parts of the codebase and other attack scenarios I had in mind.

## Info for judges
I tried to find create most of my `PoC`s in a single file that is present in this file `kelp-audit\test\Integration.t.sol` for the sake of convenience. Also codebase was not very big so it wasn't even necessary. Some of the test's are using some functions that is present at the bottom of the same `Integration.t.sol` file. Nothing else is changed.



### Time spent:
49 hours