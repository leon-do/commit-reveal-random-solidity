# commit-reveal-random-solidity
Generating random numbers using commit-reveal, and block hash

0. generate preimage locally
1. commit hash of preimage
2. wait until next block (this acts as the source of entropy. can't predict the future)
3. reveal preimage
4. return hash of blockHash + preimage


```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >0.8.0;

contract CommitReveal {

    mapping(address => Random) public random;
    struct Random {
        bytes32 commit;
        uint256 blockNumber;
        bool revealed;
    }

    // _commit = 0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6
    function commitRandom(bytes32 _commit) public {
        random[msg.sender].commit       = _commit;
        random[msg.sender].blockNumber  = block.number;
        random[msg.sender].revealed     = false;
    }

    // _preimage = 0x0000000000000000000000000000000000000000000000000000000000000001
    function revealRandom(bytes32 _preimage) public returns(uint256) {
        require(random[msg.sender].revealed == false, "already revealed");
        require(keccak256(abi.encodePacked(_preimage)) == random[msg.sender].commit, "invalid preimage");
        require(block.number > random[msg.sender].blockNumber, "reveal too early");
        random[msg.sender].revealed = true;
        return uint256(keccak256(abi.encodePacked(blockhash(block.number), _preimage)));
    }
}
```
