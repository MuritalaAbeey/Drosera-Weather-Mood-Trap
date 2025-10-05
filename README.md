# Drosera-Weather-Mood-Trap
A playful trap that “wakes up” only if an oracle reports that today’s on-chain “weather” is sunny.


```

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IWeatherOracle {
    function currentWeather() external view returns (uint8); // 0=Sunny, 1=Rainy, etc.
}

contract WeatherMoodTrap is ITrap {
    // Hardcoded oracle for PoC — safe default
    address public constant ORACLE = 0x0000000000000000000000000000000000000000;
    bytes32 public constant TAG = keccak256("WeatherMoodTrap");

    function collect() external view returns (bytes memory) {
        bool sunny = false;
        if (ORACLE != address(0)) {
            try IWeatherOracle(ORACLE).currentWeather() returns (uint8 w) {
                sunny = (w == 0); // 0 = Sunny
            } catch {
                sunny = false;
            }
        }
        return abi.encode(sunny, TAG);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        if (data.length == 0) return (false, "");
        (bool sunny, bytes32 tag) = abi.decode(data[0], (bool, bytes32));
        if (!sunny) return (false, "");
        return (true, abi.encode(tag, sunny));
    }
}
