# Merkle-Whitelist-Mint
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MerkleWhitelistNFT is Ownable {
    bytes32 public merkleRoot;
    uint256 public price = 0.001 ether;
    uint256 public maxSupply = 1000;
    uint256 public totalMinted;

    mapping(address => bool) public hasMinted;

    constructor(bytes32 _merkleRoot) {
        merkleRoot = _merkleRoot;
    }

    function mint(bytes32[] calldata proof) external payable {
        require(totalMinted < maxSupply, "Sold out");
        require(!hasMinted[msg.sender], "Already minted");
        require(msg.value == price, "Wrong ETH amount");

        bytes32 leaf = keccak256(abi.encodePacked(msg.sender));
        require(MerkleProof.verify(proof, merkleRoot, leaf), "Invalid proof");

        hasMinted[msg.sender] = true;
        totalMinted++;
        // Mint logic here (add your NFT contract)
    }

    function setMerkleRoot(bytes32 _newRoot) external onlyOwner {
        merkleRoot = _newRoot;
    }

    function withdraw() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
