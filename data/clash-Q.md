- Using old versions of OZ UUPSUpgradeable.sol can lead to uninitialized implementation bug
package.json
```
    "@openzeppelin/contracts": "^4.1.0",
    "@openzeppelin/contracts-upgradeable": "^4.1.0",
```
Prior to v4.4.0, OZ UUPSUpgradeableProxy's implementation contract was left uninitialized by default.
Anyone can call `initialize()` of implementation contract and become owner, making it possible to delegate to selfdestruct. (e.g. https://medium.com/immunefi/wormhole-uninitialized-proxy-bugfix-review-90250c41a43a)

Ensure to use v4.4.0 or newer versions of OZ. Update package.json.
