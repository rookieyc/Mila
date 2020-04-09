# Mila

## List
1. CONTRACT_CURRENT.sol (大家都會filter this，合約紀錄當前需要filter的CONTRACT_FW address，有新的合約發佈後，大家才知道要filter new contract)
    - current_fw_address (當前CONTRACT_FW.sol的address)
    - new_fw_address (新的CONTRACT_FW.sol address)
    - fix_information (修正合約後，需要告訴其他人的事情 e.g. 修正者是誰，為了什麼問題修正等)
2. CONTRACT_FW.sol (GW訂閱主題會call this contract)
    - GW_IP (執行該合約的GW其IP)
    - GW_ID (GW ID)
    - MF (欲訂閱的IoT裝置其所屬的製造商)
    - device (欲訂閱的IoT裝置型號)
    - qos (Qos)
3. CONTRACT_LOG.sol (MF發送韌體會call this contract)
    - MF (執行該合約的MF)
    - device (此韌體對應的裝置型號)
    - qos (Qos)
    - version (此韌體的版本)
    - digest (韌體雜湊值)

## To-Do List
1. contract address stores in files. And read from files.
2. change contract 10.0.2.15 to physical ip

3. check mqtt secure like ssl (future)

## Config
- Quorum:	v2.5.0
- Web3j:	v4.5.16 (CLI)
- solidity:	v0.6.4 (Remix)

## System Architecture
- Blockchain
  - 4 MF (Manufacturer) Nodes、1 BK (Broker) Node、1 PU (Portal User) Node、1 GW Account
    - i7-9700  (IP: 140.118.109.132)
      - MF 0
      - MF 1
      - MF 2
      - MF 3
    - i5-6700
      - (補) BK 4
      - (補) PU 5
    - i5-9300H
      - (補) GW account
- MQTT
  - 1 MF Client、1 BK Server、1 GW Client
    - i7-9700
      - MF 0 clinet
    - i5-6700
      - (補) BK 4 server
    - i5-9300H
      - (補) GW client

## Permission
1. init
    - MF 0 deploys permission contracts
    - MF 0 mkdir and copy `permission-config.json`
      - `nwAdminOrg : "ADMINORG"`        : Network Admin Org
      - `nwAdminRole  : "ADMIN"`         : Network Admin Role's right is FullAccess
      - `orgAdminRole : "ORGADMIN"`      : Org Admin Role's right is FullAccess
      - `accounts : ~`                   : network init accounts, mapping to nwAdminOrg、nwAdminRole
    - MF 0  mkdir and copy `permissioned-nodes.json`
    - **MF 0 ~ MF 4 are ADMINs now.**
2. add first org
	- **Prerequisite, BK node needs to write static.json, permissioned.json then copy to all folder, copy config.json, and do --permissioned config**
	- MF 0 makes a proposal
	  - quorumPermission.addOrg("ORG", "BK's enodeID", "BK's account(will be org admin)", {from: "MF 0's account"})
	- MF else agree or not
	  - quorumPermission.approveOrg("ORG", "BK's enodeID", "BK's account", {from: "MF else's account"})
	- **BK is the only ORG ADMIN now.**
	- **Check: why i need four MF approve?????**
3. org set roles
	- BK set two roles in org.
	  - quorumPermission_addNewRole("ORG", "PU", 1, false, false, {from: BK 0's account})
	  - quorumPermission_addNewRole("ORG", "GW", 1, false, false, {from: BK 0's account})
4. (還沒做) add else org admins
	- MF 0 adds else BK to org.
	  - quorumPermission_assignAdminRole("ORG", account, "ORGADMIN", {from: MF 0's account})
	- **all BK are ORG ADMINs now.**
5. add org nodes
	- **Prerequisite, PU node do something like static.json(copy and add), permissioned.json(copy), config.json(copy), --permissioned(add).**
	- BKs add PU nodes.
	  - quorumPermission_addNode("ORG", enodeId, {from: BK 0's account})
	  - quorumPermission_addAccountToOrg(account, "ORG", "PU", {from: BK 0's account})
	- **PU is on the org now.**
	- **Note: All nodes' permissioned.json will update auto. Magic.**
6. add GW to org
	- BKs add their GWs to org
	  - quorumPermission.addAccountToOrg(account, "ORG", "GW", {from: BK's account})

## (還沒做, can new branch test) Five Situation
1. add new MF
	- **Prerequisite : new node procedure + static-nodes(copy, add, paste) + permission-config(copy) + permissioned-nodes(copy from static) + --permissioned config**
	- MFs' IBFT
	  - `istanbul.propose("MF's Account", true)` => Vote (MFs know new MF) => new MF become a validator.
	- MFs' quorumPermission
	  - `quorumPermission.assignAdminRole("ADMINORG", new MF's account, "ADMIN", {from: MF 0's account})`
	  - `quorumPermission.approveAdminRole("ADMINORG", new MF's account, {from: MF else's account})`  (The role is approved once majority approval is received.)
	  - `quorumPermission_addNode()`
	- new MF
	  - **Change : deploy fixed contract, update address => all nodes filter address changed, then use new contract address**
2. delete a MF
	- MFs' IBFT
	  - `istanbul.propose("MF's Account", false)` => Vote (MFs know) => the MF removed
	- the MF's quorumPermission
	  - **Change**
	  - `quorumPermission.updateNodeStatus()` (deactivating or blacklisting)
	  - killall -INT geth // 重啟後應該就抓不到資料？
3. add new BK
	- **Prerequisite**
	- MFs' quorumPermission (BK彼此競爭,不可能幫加)
	  - `quorumPermission.assignAdminRole("ORGADMIN", ...)`
	  - `quorumPermission.approveAdminRole("ORGADMIN", ...)`
	  - `quorumPermission_addNode()`
	- new BK
	  - **Change**
4. delete a BK
	- the BK's quorumPermission
	  - **Change**
	  - `quorumPermission.updateNodeStatus()`
	  - killall -INT geth
5. a MF connects / disconnects to multi BKs
	- the MF
	  - **Change**


