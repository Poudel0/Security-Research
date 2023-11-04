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
