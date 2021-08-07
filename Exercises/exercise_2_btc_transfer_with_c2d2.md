# Exercise 2
โอน BTC ไป Ethereum (ด้วยการ wrapped asset) และโอนกลับ โดยใช้ `c2d2cli`

C2D2 คือ axelar cross-chain dapp deamon ซึ่งทำงานร่วมกับ transactions ที่จำเป็นสำหรับการโอน asset แบบ cross-chain
C2D2 CLI นี้จะทำขั้นตอนต่าง ๆ แบบอัตโนมัติ ดังที่เราทำแต่ละขั้นตอนเองมาแล้วใน in exercise 1.

## สถานะ
ระหว่างดำเนินการ 

## ระดับความยาก
ปานกลาง

## Disclaimer 
Axelar Network is a work in progress. At no point in time should you transfer any real assets using Axelar. Only use testnet tokens that you're not afraid to lose. Axelar is not responsible for any assets lost, frozen, or unrecoverable in any state or condition. If you find a problem, please submit an issue to this repository following the template. 

## สิ่งที่ต้องมี
- ทำทุกขั้นตอนใน `README.md` สำเร็จแล้ว

## links ที่เป็นประโยชน์
- commands เพิ่มเติมเพื่อ query สถานะ Axelar Network: https://github.com/axelarnetwork/axelarate-community/blob/main/EXTRA%20COMMANDS.md

## สิ่งที่คุณจำเป็นต้องมี
- Bitcoin testnet wallet พร้อมด้วย tBTC เล็กน้อย (faucet [https://testnet-faucet.mempool.co/](https://testnet-faucet.mempool.co/))
- Ethereum wallet บน the Ropsten network (แนะนำให้ใช้ Metamask)
- Ropsten ETH เล็กน้อย (faucet [https://faucet.ropsten.be/](https://faucet.ropsten.be/) หรือ [https://faucet.dimensions.network/](https://faucet.dimensions.network/))

## การเข้าร่วม Axelar testnet

ทำตามขั้นตอนต่าง ๆ ใน `README.md` เพื่อให้มั่นใจว่า node ได้มีการ synchronized ถึง block ล่าสุดแล้ว, และเราได้รับเหรียญทดสอบมาจำนวนหนึ่งในบัญชี validator ของเรา

### Pull และ enter คำสั่ง `c2d2cli` container
ตรวจดู [TESTNET RELEASE.md](../TESTNET%20RELEASE.md) สำหรับ C2D2 version ที่ใช้งานล่าสุด ของ docker images.

บน terminal window อันใหม่, ให้เข้าสู่ `c2d2cli` โดยใช้คำสั่ง:
```
./c2d2/c2d2cli.sh --version VERSION (Format: vX.Y.Z)
```

### สร้าง key บน Axelar และรับ test tokens จำนวนหนึ่งมา

สร้างบัญชี c2d2's Axelar blockchain
```
c2d2cli keys add c2d2
```

ไปที่ faucet และเติมเงินลงใน C2D2 account ของเราโดยการใช้ address ที่ the
facuet (http://faucet.testnet.axelar.network/). เราสามารถรับ address ของบัญชี c2d2
ด้วยการใช้คำสั่ง 

```shell
c2d2cli keys show c2d2 -a
```

### เติมเงินในบัญชี ethereum sender
เพิ่มบัญชี ethereum ไปใน c2d2cli. เมื่อได้รับแจ้งให้ใส่ password `passwordpassword`.
```shell
c2d2cli bridge evm accounts add ethereum 
```

เราจะถูกถามให้กรอก password สำหรับบัญชีนี้ จดบันทึก password ของเราด้วย


ถ้าเราใช้ password ต่างจากนี้้ `passwordpassword` เราก็จะต้องทำอย่างใดอย่างหนึ่งนี้:
1. กรอก password ด้วยตัวเองระหว่างขั้นตอนการ transfer 
2. **หรือ** ให้ password ของเราในแต่ละคำสั่ง `c2d2cli` โดยการเพิ่ม flag `--evm-passphrase YOUR_PASSWORD`
3. **หรือ** แก้ไข password ที่ไฟล์ `/root/.c2d2cli/config.toml`
   - เปลี่ยนค่าใน `sender-passphrase=` key ไปยัง password ของเรา

List บัญชี C2D2's:

```
c2d2cli bridge evm accounts list ethereum
```

บัญชีขึ้นต้นด้วย `0` (ใช้ address แรกใน list) จะถูกใช้เพื่อส่ง transactions ให้ไปที่ [https://faucet.ropsten.be/](https://faucet.ropsten.be/) เพื่อรับ Ropsten ETH สำหรับบัญชีของ sender

### Mint ERC20 Bitcoin tokens บน Ethereum
1. Generate a Bitcoin deposit address. The Ethereum address you provide will be linked to the deposit address and receive the pegged bitcoin (Satoshi tokens) on the Ethereum testnet. 

   ```
   c2d2cli transfer satoshi [ethereum recipient address] --source-chain bitcoin --dest-chain ethereum --gas=auto --gas-adjustment=1.4
   ```

    You will see the deposit Bitcoin address printed in the terminal

    ```
      action:  (2/7) Please deposit Bitcoin to tb1qgfk6v2ut9flwwkraj6t3syvpq22g0xhh2m73atfe79jv3msjwvzqtpuvfc
    ```

2. **External**: send some TEST BTC on Bitcoin testnet to the deposit address specific above, and wait for 6 confirmations (i.e. the transaction is 6 blocks deep in the Bitcoin chain). 

  - ALERT: **DO NOT SEND ANY REAL ASSETS**
  - Bitcoin testnet faucet [https://testnet-faucet.mempool.co/](https://testnet-faucet.mempool.co/)
  - You can monitor the status of your deposit using the testnet explorer: [https://blockstream.info/testnet/](https://blockstream.info/testnet/)

Do not exit `c2d2cli` while you are waiting for your deposit to be confirmed. It will be watching the bitcoin blockchain to detect your transaction. 
- If `c2d2cli` crashes or is closed during this step you can re-run the `deposit-btc` command with the same recipient address to resume.
- If your transaction has 6 confirmations but `c2d2cli` has not detected it, you can restart `c2d2cli` and append the `--bitcoin-tx-prompt` flag.
    - The CLI will prompt you to enter the deposit tx info manually. The rest of the deposit procedure will still be automated.
    - `c2d2cli transfer satoshi [ethereum recipient address] --source-chain bitcoin --dest-chain ethereum  --bitcoin-tx-prompt --gas=auto --gas-adjustment=1.4`

Once your transaction is detected, `c2d2cli` will wait until it has 6 confirmations before proceeding.

 3. C2D2 will automate the bitcoin deposit confirmation, and mint command signing and sending. Once the minting process completes you will see the following message:

    ```
    Transferred satoshi to Ethereum address [ethereum recipient address]
    ```

You can now open Metamask and add the wrapped BTC contract address. `c2d2cli` will print the contract address like this:

```
Using AxelarGateway <address>
Using satoshi token <address>
```

The contract will show in metamask as symbol 'Satoshi'. If your recipient address is in metamask, you will have an amount of satoshi tokens in metamask equal to your bitcoin deposit. 

### Burn ERC20 wrapped Bitcoin tokens and obtain native Satoshi
1. Generate an ethereum withdrawal address. The Bitcoin address you provide will be uniquely linked to the deposit address and receive the withdrawn BTC on the Bitcoin testnet. 

   ```
   c2d2cli transfer satoshi [bitcoin recipient address] --source-chain ethereum --dest-chain bitcoin --gas=auto --gas-adjustment=1.4
   ```

   For example:
   ```
   c2d2cli transfer satoshi tb1qwtrclv55yy26awl2n40u57uck5xgty4w4h9eww --source-chain ethereum --dest-chain bitcoin --gas=auto --gas-adjustment=1.4
   ```

   You will see the deposit Ethereum address printed in the terminal.

   ```
   action:  (2/5) Please transfer satoshi tokens to Ethereum address 0xf5fccEeF24358fE24C53c1963d5d497BCD3ddF48
     | ✓ Waiting for a withdrawal transaction
   ```

2. **External**: send wrapped Satoshi tokens to withdrawal address (e.g. with Metamask). You need to have some Ropsten testnet Ether on the address to send transactions.


3. Once your withdrawal transaction is detected, `c2d2cli` will wait for 30 Ropsten block confirmations before proceeding.


4. `c2d2cli` will automate the withdrawal confirmation, and satoshi token (wrapped BTC) burning. 

5. Once your Satoshi tokens have been burned, you will see this message:

    ```
    Transferred 5000 satoshi tokens to Bitcoin address [bitcoin recipient address]
    ```

6. Your withdrawn BTC will be spendable by your recipient address once Bitcoin withdrawal consolidation occurs. Consolidation will be completed by a separate process.

An automated service processes all pending transfers from the Axelar network to Bitcoin a few times a day. Come back 24 hours to check your coins at the destination Bitcoin address on the testnet.  

## Additional Notes
If your local axelar node fails, meaning `c2d2cli` cannot connect to it to broadcast transactions, you may use an Axelar public node to broadcast transactions by adding the config file flag `--conf /config.testnet.toml` like so:

```shell
   c2d2cli transfer satoshi [bitcoin recipient address] --source-chain ethereum --dest-chain bitcoin --conf /config.testnet.toml --gas=auto --gas-adjustment=1.4
```
