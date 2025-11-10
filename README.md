gg// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/// NFT Pass contract (ERC-721) - Minting, reveal, per-wallet limit, owner controls
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/utils/Strings.sol";

contract NFTPass is ERC721Enumerable, Ownable, ReentrancyGuard {
    using Strings for uint256;

    // Configurable parameters
    uint256 public maxSupply;            // أقصى عدد من الـ NFTs
    uint256 public price;                // سعر الـ mint لكل توكن (بـ wei)
    uint256 public maxPerWallet;  @       // الحد لكل محفظة
    bool public saleActive = false;      // بيع عام مفعل / غير مفعل
    bool public revealed = false;        // حالة ال-Reveal

    // Metadata
    string private baseTokenURI;         // عندما تكشف
    string private preRevealURI;         // URI عام قبل الكشف (مثلاً ملف JSON واحد)
    string public contractMetadataURI;   // optional: contract-level metadata (OpenSea)

    // Track how many minted per address
    mapping(address => uint256) public minted;

    // Events
    event Minted(address indexed minter, uint256 quantity);
    event SaleToggled(bool active);
    event Revealed(bool revealedState);

    constructor(
        string memory _name,
        string memory _symbol,
        uint256 _maxSupply,
        uint256 _priceWei,
        uint256 _maxPerWallet,
        string memory _preRevealURI,
        string memory _contractMetadataURI
    ) ERC721(_name, _symbol) {
        maxSupply = _maxSupply;
        price = _priceWei;
        maxPerWallet = _maxPerWallet;
        preRevealURI = _preRevealURI;
        contractMetadataURI = _contractMetadataURI;
    }

    // -----------------------
    // Public mint function
    // -----------------------
    function mint(uint256 quantity) external payable nonReentrant {
        require(saleActive, "Sale is not active");
        require(quantity > 0, "Quantity must be > 0");
        require(totalSupply() + quantity <= maxSupply, "Exceeds max supply");
        require(minted[msg.sender] + quantity <= maxPerWallet, "Exceeds per-wallet limit");
        require(msg.value == price * quantity, "Incorrect ETH amount");

        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = totalSupply() + 1; // tokenIds start at 1
            _safeMint(msg.sender, tokenId);
        }

        minted[msg.sender] += quantity;
        emit Minted(msg.sender, quantity);
    }

    // -----------------------
    // Owner-only functions
    // -----------------------
    // Owner can mint free (reserve)
    function ownerMint(address to, uint256 quantity) external onlyOwner {
        require(totalSupply() + quantity <= maxSupply, "Exceeds max supply");
        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = totalSupply() + 1;
            _safeMint(to, tokenId);
        }
        emit Minted(to, quantity);
    }

    function setPrice(uint256 _priceWei) external onlyOwner {
        price = _priceWei;
    }

    function setMaxPerWallet(uint256 _max) external onlyOwner {
        maxPerWallet = _max;
    }

    function setSaleActive(bool _active) external onlyOwner {
        saleActive = _active;
        emit SaleToggled(_active);
    }

    function setBaseURI(string calldata _baseURI) external onlyOwner {
        baseTokenURI = _baseURI;
    }

    function setPreRevealURI(string calldata _preURI) external onlyOwner {
        preRevealURI = _preURI;
    }

    function setContractMetadataURI(string calldata _uri) external onlyOwner {
        contractMetadataURI = _uri;
    }

    function reveal() external onlyOwner {
        revealed = true;
        emit Revealed(true);
    }

    // Withdraw funds to owner
    function withdraw() external onlyOwner nonReentrant {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds to withdraw");
        (bool sent, ) = payable(owner()).call{value: balance}("");
        require(sent, "Withdraw failed");
    }

    // -----------------------
    // Metadata overrides
    // -----------------------
    function _baseURI() internal view override returns (string memory) {
        return baseTokenURI;
    }

    // tokenURI returns preRevealURI for all tokens until reveal
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");

        if (!revealed) {
            return preRevealURI;
        }

        string memory base = _baseURI();
        return bytes(base).length > 0 ? string(abi.encodePacked(base, tokenId.toString())) : "";
    }

    // Optional: OpenSea / marketplaces contract-level metadata
    function contractURI() public view returns (string memory) {
        return contractMetadataURI;
    }

    // Fallback to receive ETH (if someone sends ETH directly)
    receive() external payable {}
    fallback() external payable {}
}

git add .
git commit -m "تحديث المشروع لعام 2025"
git push origin main
