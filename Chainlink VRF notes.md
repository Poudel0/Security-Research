# Chainlink VRF notes


## Security measures

- Donot  re-request randomness
- Use `requestId` to match requests with fullfillment order.\
- Choose a safe block confirmation time
- Do nto accept inputs after randomness request
- `fulfullRandomWords ` must not revert.
- Use `VRFConsumerBaseV2` to interact
-- Inherit from BaseV2
-- DOn't override `rawFulfillRandomness`
- Use `VRFv2WrapperConsumer.sol` to interact with the VRF service (for direct method).


##  Audit knowledge
- Mainnet reorgs are almost always ~1 depth while other chains(`Polygon`) can have depth `>3 reorgs` which is the default check block confirmation value for `VRFv2Consumer`.
Eg: ([C-01] Polygon chain reorgs will often change game results)[https://github.com/solodit/solodit_content/blob/main/reports/Pashov/2023-06-01-NFT%20Loots.md#c-01-polygon-chain-reorgs-will-often-change-game-results]

