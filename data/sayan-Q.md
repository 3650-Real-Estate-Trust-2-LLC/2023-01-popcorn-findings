

##  [N-01]_LOCK PRAGMAS_  TO SPECIFIC COMPILER VERSION

**Description:**  
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.  
[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

**Recommendation:**  
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.  
[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)
```
37 results - 37 files
File: src/utils/EIP165.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/abstracts/OnlyStrategy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/AdminProxy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/IEIP165.sol
1: // SPDX-License-Identifier: AGPL-3.0-only
2: pragma solidity ^0.8.15;

File: src/interfaces/IMultiRewardEscrow.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/IMultiRewardStaking.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/vault/IAdapter.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;


File: src/interfaces/vault/IAdminProxy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/ICloneFactory.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/ICloneRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/IDeploymentController.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/IERC4626.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/vault/IPermissionRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;


File: src/interfaces/vault/IStrategy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/ITemplateRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/vault/IVault.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/vault/IVaultController.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/interfaces/vault/IVaultRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/interfaces/vault/IWithRewards.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;


File: src/utils/EIP165.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;


File: src/utils/MultiRewardEscrow.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/utils/MultiRewardStaking.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/CloneFactory.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/CloneRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/DeploymentController.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/PermissionRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/TemplateRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/Vault.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/VaultController.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: pragma solidity ^0.8.15;

File: src/vault/VaultRegistry.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/abstracts/AdapterBase.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/abstracts/OnlyStrategy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/abstracts/WithRewards.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/beefy/BeefyAdapter.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/beefy/IBeefy.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/yearn/IYearn.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;

File: src/vault/adapter/yearn/YearnAdapter.sol
1: // SPDX-License-Identifier: GPL-3.0
2: // Docgen-SOLC: 0.8.15
3: 
4: pragma solidity ^0.8.15;
```

## [N-02] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values):

This will help with readability and easier maintenance for future changes.

**constants.sol**

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution
```
3 resutls - 3 files
File: src/vault/Vault.sol
35:     uint256 constant SECONDS_PER_YEAR = 365.25 days;

File: src/vault/adapter/beefy/BeefyAdapter.sol
31:     uint256 public constant BPS_DENOMINATOR = 10_000;

File: src/vault/adapter/yearn/YearnAdapter.sol
25:     uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

##  [N-03] USE SCIENTIFIC NOTATION (E.G. 10e18) RATHER THAN EXPONENTIATION (E.G. 10**18)
```
File: src/vault/adapter/yearn/YearnAdapter.sol
25:     uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```
## [N-04] OPEN TODOS 
```
File: src/vault/adapter/abstracts/AdapterBase.sol
516:     // TODO use deterministic fee recipient proxy
```
### Recommended Mitigation Steps

Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.

## [N-05] FUNCTION RETURNS NOTHING BUT IS MARKED WITH RETURNS 
### Description:  
function is marked with  `returns (uint256 shares)` but it returns nothing
```
File: src/vault/Vault.sol
134:     function deposit(uint256 assets, address receiver)
135:         public
136:         nonReentrant
137:         whenNotPaused
138:         syncFeeCheckpoint
139:         returns (uint256 shares)
140:     {
141:         if (receiver == address(0)) revert InvalidReceiver();
142: 
143:         uint256 feeShares = convertToShares(
144:             assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
145:         );
146: 
147:         shares = convertToShares(assets) - feeShares;
148: 
149:         if (feeShares > 0) _mint(feeRecipient, feeShares);
150: 
151:         _mint(receiver, shares);
152: 
153:         asset.safeTransferFrom(msg.sender, address(this), assets);
154: 
155:         adapter.deposit(assets, address(this));
156: 
157:         emit Deposit(msg.sender, receiver, assets, shares);
158:     }


File: src/vault/Vault.sol
170:     function mint(uint256 shares, address receiver)
171:         public
172:         nonReentrant
173:         whenNotPaused
174:         syncFeeCheckpoint
175:         returns (uint256 assets)
176:     {
177:         if (receiver == address(0)) revert InvalidReceiver();
178: 
179:         uint256 depositFee = uint256(fees.deposit);
180: 
181:         uint256 feeShares = shares.mulDiv(
182:             depositFee,
183:             1e18 - depositFee,
184:             Math.Rounding.Down
185:         );
186: 
187:         assets = convertToAssets(shares + feeShares);
188: 
189:         if (feeShares > 0) _mint(feeRecipient, feeShares);
190: 
191:         _mint(receiver, shares);
192: 
193:         asset.safeTransferFrom(msg.sender, address(this), assets);
194: 
195:         adapter.deposit(assets, address(this));
196: 
197:         emit Deposit(msg.sender, receiver, assets, shares);
198:     }


File: src/vault/Vault.sol
211:     function withdraw(
212:         uint256 assets,
213:         address receiver,
214:         address owner
215:     ) public nonReentrant syncFeeCheckpoint returns (uint256 shares) {
216:         if (receiver == address(0)) revert InvalidReceiver();
217: 
218:         shares = convertToShares(assets);
219: 
220:         uint256 withdrawalFee = uint256(fees.withdrawal);
221: 
222:         uint256 feeShares = shares.mulDiv(
223:             withdrawalFee,
224:             1e18 - withdrawalFee,
225:             Math.Rounding.Down
226:         );
227: 
228:         shares += feeShares;
229: 
230:         if (msg.sender != owner)
231:             _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
232: 
233:         _burn(owner, shares);
234: 
235:         if (feeShares > 0) _mint(feeRecipient, feeShares);
236: 
237:         adapter.withdraw(assets, receiver, address(this));
238: 
239:         emit Withdraw(msg.sender, receiver, owner, assets, shares);
240:     }


File: src/vault/Vault.sol
253:     function redeem(
254:         uint256 shares,
255:         address receiver,
256:         address owner
257:     ) public nonReentrant returns (uint256 assets) {
258:         if (receiver == address(0)) revert InvalidReceiver();
259: 
260:         if (msg.sender != owner)
261:             _approve(owner, msg.sender, allowance(owner, msg.sender) - shares);
262: 
263:         uint256 feeShares = shares.mulDiv(
264:             uint256(fees.withdrawal),
265:             1e18,
266:             Math.Rounding.Down
267:         );
268: 
269:         assets = convertToAssets(shares - feeShares);
270: 
271:         _burn(owner, shares);
272: 
273:         if (feeShares > 0) _mint(feeRecipient, feeShares);
274: 
275:         adapter.withdraw(assets, receiver, address(this));
276: 
277:         emit Withdraw(msg.sender, receiver, owner, assets, shares);
278:     }


File: src/vault/Vault.sol
323:     function previewDeposit(uint256 assets)
324:         public
325:         view
326:         returns (uint256 shares)
327:     {
328:         shares = adapter.previewDeposit(
329:             assets -
330:                 assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
331:         );
332:     }

File: src/vault/Vault.sol
340:     function previewMint(uint256 shares) public view returns (uint256 assets) {
341:         uint256 depositFee = uint256(fees.deposit);
342: 
343:         shares += shares.mulDiv(
344:             depositFee,
345:             1e18 - depositFee,
346:             Math.Rounding.Up
347:         );
348: 
349:         assets = adapter.previewMint(shares);
350:     }


File: src/vault/Vault.sol
358:     function previewWithdraw(uint256 assets)
359:         external
360:         view
361:         returns (uint256 shares)
362:     {
363:         uint256 withdrawalFee = uint256(fees.withdrawal);
364: 
365:         assets += assets.mulDiv(
366:             withdrawalFee,
367:             1e18 - withdrawalFee,
368:             Math.Rounding.Up
369:         );
370: 
371:         shares = adapter.previewWithdraw(assets);
372:     }

File: src/vault/Vault.sol
380:     function previewRedeem(uint256 shares)
381:         public
382:         view
383:         returns (uint256 assets)
384:     {
385:         assets = adapter.previewRedeem(shares);
386: 
387:         assets -= assets.mulDiv(
388:             uint256(fees.withdrawal),
389:             1e18,
390:             Math.Rounding.Down
391:         );
392:     }

File: src/vault/DeploymentController.sol
099:   function deploy(
100:     bytes32 templateCategory,
101:     bytes32 templateId,
102:     bytes calldata data
103:   ) external onlyOwner returns (address clone) {
104:     Template memory template = templateRegistry.getTemplate(templateCategory, templateId);
105: 
106:     if (!template.endorsed) revert NotEndorsed(templateId);
107: 
108:     clone = cloneFactory.deploy(template, data);
109: 
110:     cloneRegistry.addClone(templateCategory, templateId, clone);
111:   }


File: src/vault/CloneFactory.sol
39:   function deploy(Template calldata template, bytes calldata data) external onlyOwner returns (address clone) {
40:     clone = Clones.clone(template.implementation);
41: 
42:     bool success = true;
43:     if (template.requiresInitData) (success, ) = clone.call(data);
44: 
45:     if (!success) revert DeploymentInitFailed();
46: 
47:     emit Deployment(clone);
48:   }
49: }


File: src/utils/MultiRewardStaking.sol
390:   function _accrueStatic(RewardInfo memory rewards) internal view returns (uint256 accrued) {
391:     uint256 elapsed;
392:     if (rewards.rewardsEndTimestamp > block.timestamp) {
393:       elapsed = block.timestamp - rewards.lastUpdatedTimestamp;
394:     } else if (rewards.rewardsEndTimestamp > rewards.lastUpdatedTimestamp) {
395:       elapsed = rewards.rewardsEndTimestamp - rewards.lastUpdatedTimestamp;
396:     }
397: 
398:     accrued = uint256(rewards.rewardsPerSecond * elapsed);
399:   }


File: src/vault/VaultController.sol
089:   function deployVault(
090:     VaultInitParams memory vaultData,
091:     DeploymentArgs memory adapterData,
092:     DeploymentArgs memory strategyData,
093:     address staking,
094:     bytes memory rewardsData,
095:     VaultMetadata memory metadata,
096:     uint256 initialDeposit
097:   ) external canCreate returns (address vault) {
098:     IDeploymentController _deploymentController = deploymentController;
099: 
100:     _verifyToken(address(vaultData.asset));
101:     _verifyAdapterConfiguration(address(vaultData.adapter), adapterData.id);
102: 
103:     if (adapterData.id > 0)
104:       vaultData.adapter = IERC4626(_deployAdapter(vaultData.asset, adapterData, strategyData, _deploymentController));
105: 
106:     vault = _deployVault(vaultData, _deploymentController);
107: 
108:     if (staking == address(0)) staking = _deployStaking(IERC20(address(vault)), _deploymentController);
109: 
110:     _registerCreatedVault(vault, staking, metadata);
111: 
112:     if (rewardsData.length > 0) _handleVaultStakingRewards(vault, rewardsData);
113: 
114:     emit VaultDeployed(vault, staking, address(vaultData.adapter));
115: 
116:     _handleInitialDeposit(initialDeposit, IERC20(vaultData.asset), IERC4626(vault));
117:   }


File: src/vault/VaultController.sol
120:   function _deployVault(VaultInitParams memory vaultData, IDeploymentController _deploymentController)
121:     internal
122:     returns (address vault)
123:   {
124:     vaultData.owner = address(adminProxy);
125: 
126:     (bool success, bytes memory returnData) = adminProxy.execute(
127:       address(_deploymentController),
128:       abi.encodeWithSelector(
129:         DEPLOY_SIG,
130:         VAULT,
131:         activeTemplateId[VAULT],
132:         abi.encodeWithSelector(IVault.initialize.selector, vaultData)
133:       )
134:     );
135:     if (!success) revert UnderlyingError(returnData);
136: 
137:     vault = abi.decode(returnData, (address));
138:   }


File: src/vault/VaultController.sol
186:   function deployAdapter(
187:     IERC20 asset,
188:     DeploymentArgs memory adapterData,
189:     DeploymentArgs memory strategyData,
190:     uint256 initialDeposit
191:   ) external canCreate returns (address adapter) {
192:     _verifyToken(address(asset));
193: 
194:     adapter = _deployAdapter(asset, adapterData, strategyData, deploymentController);
195: 
196:     _handleInitialDeposit(initialDeposit, asset, IERC4626(adapter));
197:   }

File: src/vault/VaultController.sol
225:   function __deployAdapter(
226:     DeploymentArgs memory adapterData,
227:     bytes memory baseAdapterData,
228:     IDeploymentController _deploymentController
229:   ) internal returns (address adapter) {
230:     (bool success, bytes memory returnData) = adminProxy.execute(
231:       address(_deploymentController),
232:       abi.encodeWithSelector(DEPLOY_SIG, ADAPTER, adapterData.id, _encodeAdapterData(adapterData, baseAdapterData))
233:     );
234:     if (!success) revert UnderlyingError(returnData);
235: 
236:     adapter = abi.decode(returnData, (address));
237: 
238:     adminProxy.execute(adapter, abi.encodeWithSelector(IAdapter.setPerformanceFee.selector, performanceFee));
239:   }


File: src/vault/VaultController.sol
256:   function _deployStrategy(DeploymentArgs memory strategyData, IDeploymentController _deploymentController)
257:     internal
258:     returns (address strategy)
259:   {
260:     (bool success, bytes memory returnData) = adminProxy.execute(
261:       address(_deploymentController),
262:       abi.encodeWithSelector(DEPLOY_SIG, STRATEGY, strategyData.id, "")
263:     );
264:     if (!success) revert UnderlyingError(returnData);
265: 
266:     strategy = abi.decode(returnData, (address));
267:   }


File: src/vault/VaultController.sol
284:   function _deployStaking(IERC20 asset, IDeploymentController _deploymentController)
285:     internal
286:     returns (address staking)
287:   {
288:     (bool success, bytes memory returnData) = adminProxy.execute(
289:       address(_deploymentController),
290:       abi.encodeWithSelector(
291:         DEPLOY_SIG,
292:         STAKING,
293:         activeTemplateId[STAKING],
294:         abi.encodeWithSelector(IMultiRewardStaking.initialize.selector, asset, escrow, adminProxy)
295:       )
296:     );
297:     if (!success) revert UnderlyingError(returnData);
298: 
299:     staking = abi.decode(returnData, (address));
300:   }

File: src/vault/VaultController.sol
667:   function _verifyCreatorOrOwner(address vault) internal returns (VaultMetadata memory metadata) {
668:     metadata = vaultRegistry.getVault(vault);
669:     if (msg.sender != metadata.creator || msg.sender != owner) revert NotSubmitterNorOwner(msg.sender);
670:   }

File: src/vault/VaultController.sol
673:   function _verifyCreator(address vault) internal view returns (VaultMetadata memory metadata) {
674:     metadata = vaultRegistry.getVault(vault);
675:     if (msg.sender != metadata.creator) revert NotSubmitter(msg.sender);
676:   }
 
 ```