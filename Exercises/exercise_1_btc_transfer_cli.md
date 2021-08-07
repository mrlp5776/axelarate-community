# Exercise 1
โอนย้าย $BTC ไปและกลับจาก Ethereum (เป็นการ wrap) โดยใช้ Axelar Network CLI

## ระดับความยาก
Intermediate (ปานกลาง)

## Disclaimer
Axelar Network is a work in progress. At no point in time should you transfer any real assets using Axelar. Only use testnet tokens that you're not afraid to lose. Axelar is not responsible for any assets lost, frozen, or unrecoverable in any state or condition. If you find a problem, please submit an issue to this repository following the template.

## ข้อกำหนด
- ผ่านทุกขั้นตอนจาก `README.md`
- มี Ethereum wallet จาก [MEW](https://www.myetherwallet.com/) และเตรียม Ethereum address ที่มี $ETH อยู่บ้าง (สามารถใช้ปลั๊กอินนี้ได้ [Chrome plugin](https://chrome.google.com/webstore/detail/mew-cx/nlbmnnijcnlegkjjpcfjclmcfggfefdm?hl=en))

## ลิงค์ที่เป็นประโยชน์
- Axelar faucet: http://faucet.testnet.axelar.network/
- Docker images รุ่นล่าสุด: https://hub.docker.com/repository/docker/axelarnet/axelar-core,
  https://hub.docker.com/repository/docker/axelarnet/tofnd
- คำสั่งในการ query state ของ Axelar Network เพิ่มเติม: https://github.com/axelarnetwork/axelarate-community/blob/main/EXTRA%20COMMANDS.md

## สิ่งที่จะต้องใช้
- Bitcoin testnet faucet เพื่อส่ง test BTC: https://testnet-faucet.mempool.co/
- Metamask
- Ethereum Ropsten address (ผ่านทาง Metamask)


## การเข้าร่วม the Axelar testnet

ทำตามขั้นตอนใน `README.md` (`ภาษาไทยไปที่ mrlp5776/axelarate-community` แต่การ clone ให้ใช้จาก axelarnetwork/axelarate-community) เพื่อให้แน่ใจว่า node ของเรามีข้อมูลเป็นปัจจุบัน และได้รับเหรียญทดสอบในบัญชีของ Validator

## การออกคำสั่งเพื่อ mint และ burn tokens
คำสั่งเหล่านี้เป็นวิธีการทีละขั้นตอน เพื่อที่จะใช้คำสั่งในการย้าย asset จาก  source ไปยัง chain ปลายทาง และก็ย้ายกลับ assets ถูก minted โดยเป็นการ wrapped ของ ERC-20 assets ไปอยู่บน chain ปลายทาง หรือ destination chain คำสั่งจะถูก submitted ไปที่ Axelar Network โดยจะรับผิดชอบในส่วนของ (a) สร้าง address สำหรับการฝาก/ถอน (b) กำหนดเส้นทาง และการปิด transactions และ (c) การทำการ mint/burn กับ assets ที่สัมพันธ์กัน

ในการทำการทดสอบนี้ เราจำเป็นจะต้องมี test Bitcoins อยู่บน Bitcoin testnet บ้าง และที่ address ปลายทางของ Ethereum บน Ethereum Ropsten Testnet ด้วยเช่นกัน

### ทำการ Mint เหรียญ ERC20 Bitcoin บน Ethereum

1. สร้างบัญชีเงินฝากบน Bitcoin (ไว้สำหรับฝากเงินเข้ามาภายหลัง)

  ```
  axelard tx bitcoin link ethereum {ethereum Ropsten dst addr} --from validator -y -b block
-> returns deposit address
  ```

  เช่น

  ```
  axelard tx bitcoin link ethereum 0xc1c0c8D2131cC866834C6382096EaDFEf1af2F52 --from validator -y -b block
  ```

  ให้หา out put ประมาณนี้ `successfully linked {bitcoin deposit address} and {ethereum Ropsten dst addr}`

2. แหล่งภายนอก: ส่ง TEST $BTC บน Bitcoin testnet ไปที่บัญชีเงินฝาก Bitcoin ที่ได้จากข้างบนนี้, และรอการยืนยัน 6 ครั้ง (เช่น transaction is 6 blocks deep in the Bitcoin chain)
  - คำเตือน: ห้ามส่งทรัพย์สินจริงในขั้นตอนพวกนี้
  - สามารถใช้ bitcoin faucet อย่างเช่น https://bitcoinfaucet.uo1.net/ เพื่อส่ง TEST BTC ไปยังบัญชีเงินฝากที่เราสร้างขึ้นมา
  - ตรวจสอบสถานะของการฝากเงินโดยใช้ testnet explorer: https://blockstream.info/testnet/


3. การยืนยัน Bitcoin outpoint

  ```
  axelard tx bitcoin confirmTxOut "{txID:vout}" "{amount}btc" "{deposit address}" --from validator -y -b block
  ```

  เช่น.,

  ```
  axelard tx bitcoin confirmTxOut 615df0b4d5053630d24bdd7661a13bea28af8bc1eb0e10068d39b4f4f9b6082d:0 0.00088btc tb1qlteveekr7u2qf8faa22gkde37epngsx9d7vgk98ujtzw77c27k7qk2qvup --from validator -y -b block
  ```

  ให้รอการยืนยันของ transaction ในที่สุด, เราก็จะเห็นบางอย่างแบบเดียวกันนี้บน node terminal:

  `bitcoin outpoint confirmation result is {result}`

หรือเราจะค้นหาด้วย `docker logs -f axelar-core 2>&1 | grep -a -e outpoint`.

4. Trigger signing ของการโอนไปที่ Ethereum

  ```
  axelard tx evm sign-pending-transfers ethereum --from validator -y -b block
  -> returns เป็น commandID ของ signed tx
  -> รอ sign protocol ให้สำเร็จ (~10 blocks)
  ```

  ให้มองหา commandID และค่าของมันที่เป็น output: `"key": "commandID",
    "value": "d5e993e407ff399cf2770a1d42bc2baf5308f46632fcbe209318acb09776599f"`

  เราสามารถค้นหาได้ด้วย `docker logs -f axelar-core 2>&1 | grep -a -e command`.

5. ให้ command data ที่จะต้องส่งไปใน Ethereum transaction เพื่อที่จะ execute the mint (ให้สร้างเหรียญมาใหม่)
  ```
  axelard q evm command ethereum {commandID}
  ```
  
  เช่น,

  ```
  axelard q evm command ethereum 28a523a4d5836df2cdc3af5beffc10ca946e62497d609521504462e043a38fdc
  ```
  ก็จะได้ command data เป็น output

6. ส่ง Ethereum transaction ที่ wrap กับ command data เพิ่งสั่งให้สร้างเหรียญ

เปิด Metamask wallet ไปที่ Setting > Advanced แล้วก็เปิดใช้งานหัวข้อ Show HEX data ซึ่งจะทำให้เราสามารถส่งข้อมูล transaction ได้โดยตรงผ่าน Metamask wallet
อย่าลืมว่าไม่ต้องทำการโอนเหรียญใด ๆ เราแค่ต้องการข้อมูล input มาจากขั้นตอนของ `commandID` ด้านบน และส่งไปที่ Gateway smart contract (ดูได้จาก [ที่นี่](https://github.com/axelarnetwork/axelarate-community/blob/main/TESTNET%20RELEASE.md)) ขณะที่ทำขั้นตอนนี้ตรวจดูให้แน่ใจถึงค่า gas ใน Metamask ให้อัพเดทใหม่เมื่อทำการคัดลอก command data มาวาง

<img alt="MM" src="https://user-images.githubusercontent.com/87255624/128593622-36013986-af89-4643-9e62-55995ac741b0.png">

อีกวิธีนึง เราสามารถใช้ MEW wallet และไปที่หน้า "Send Transaction" เปิดตรง advanced options ด้วย ตอนนี้ก็จะต้องส่ง transaction ไปยัง Gateway smart contract ด้วย **0** Ether พร้อมทั้งช่อง data ก็ใส่ commad data ที่เราเรียกมันจากขั้นตอนก่อนหน้านี้ หน้าจอของเราก็ควรดูคล้าย ๆ กับด้านล่างนี้ คือเราเพียงแค่ส่ง transaction เพื่อสั่งให้ mint เหรียญของเรา 

<img width="690.9" alt="MEW" src="https://user-images.githubusercontent.com/1995809/118490096-2753c480-b750-11eb-9c9d-5eb478194ae4.png">

(หมายเหตุไว้ว่าตรง "To Address" จะเป็น address ของ Axelar Gateway smart contract, ซึ่งหาได้จาก [ที่นี่](https://github.com/axelarnetwork/axelarate-community/blob/main/TESTNET%20RELEASE.md), จากนั้นให้ "Add Data" ด้วย command data ที่ได้มาจากขั้นตอนก่อนหน้า)

ตอนนี้เราก็จะเปิด Metamask เลือก "Assets" จากนั้น "Add Token" จากนั้น "Custom Token" และวาง token contract address ลงไป (ดูที่ `axelarate-community/TESTNET RELEASE.md` และค้นหาตรง `Ethereum token contract address`) เราก็จะพบว่ามี satoshi ถูก mint มาใน Ethereum address ของเราเท่ากับจำนวนที่ฝากเข้ามาจาก test Bitcoin ในตอนแรก

### การ burn เหรียญ ERC20 wrapped Bitcoin และรับ native Satoshi

เพื่อทำการส่ง wrapped Bitcoin กลับไปยัง Bitcoin ให้ใช้คำสั่งตามนนี้:

1. สร้าง address สำหรับการฝากเงิน บน Ethereum

  ```
  axelard tx evm link ethereum bitcoin {destination bitcoin addr หรือ ฝากไปที่ address bitcoin เราที่ได้มาตอนแรก} satoshi --from validator -y -b block
  -> returns deposit address
  ```

  เช่น,
  ```
  axelard tx evm link ethereum bitcoin tb1qg2z5jatp22zg7wyhpthhgwvn0un05mdwmqgjln satoshi --from validator -y -b block
  ```

  ให้หา address บน Ethereum ที่ใช้สำหรับการฝากเงิน ซึ่ง output แรกจะอยู่ในบรรทัดนี้ (`0x5CFE...`):

  ```
  "successfully linked {0x5CFEcE3b659e657E02e31d864ef0adE028a42a8E} and {tb1qq8wnre6rzctec9wycrl2dq00m3avravslahc8v}"
  ```
**หมายเหตุเพิ่มเติม: ให้แน่ใจว่าได้ทำการเชื่อม bitcoin address ที่ควบคุมด้วยเราเอง หากว่าเชื่อมต่อกับที่อยู่ หรือ address ที่ควบคุมโดย Axelar การถอนเงินของเราจะถูกพิจารณาเป็นการบริจาคไปยัง pool of funds**

2. ภายนอก: ให้ส่ง wrapped tokens หรือก็คือ satoshi ไปยัง deposit address บน Ethereum เมื่อสักครู่นี้ (เช่น ด้วยการใช้ Metamask) เราจะต้องมีเหรียญ Ropstem testnet ETH อยู่เพื่อทำ หรือส่ง transactions และรอผลการยืนยัน 30 Ethereum blocks เราสามารถตรวจสอบสถานีของการฝากครั้งนี้โดยใช้ testnet explorer: https://ropsten.etherscan.io/

3. ยืนยัน the Ethereum transaction

  ```
  axelard tx evm confirm-erc20-deposit ethereum {txID} {amount} {deposit addr} --from validator -y -b block
  -> wait for transaction to be confirmed (~10 Axelar blocks)
  ```

  ณ ตรงนี้, จำนวนควรจะต้องระบุเป็นหน่วย Satoshi. (อย่างเช่นว่า, 0.0001BTC = 10000)
  เช่น,

  ```
  axelard tx evm confirm-erc20-deposit ethereum 0x01b00d7ed8f66d558e749daf377ca30ed45f747bbf64f2fd268a6d1ea84f916a 10000 0x5CFEcE3b659e657E02e31d864ef0adE028a42a8E --from validator -y -b block
  -> wait for transaction to be confirmed (~10 Axelar blocks)
  ```

เสร็จเรียบร้อย! ขั้นตอนต่อไป, การถอนเงินจะต้องถูก signed และ submitted ไปยัง Bitcoin network. ในการนี้, พวกเราจะ trigger คำสั่งนี้ให้วันละครั้ง ดังนั้นให้กลับมาอีกใน 24 ชั่วโมง, และตรวจดูยอดคงเหลือบน address ของ Bitcoin testnet ที่ซึ่งเราได้ทำการ submit การถอนเงินไว้นั่นเอง 
