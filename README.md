# axelarate-community
เครื่องมือต่าง ๆ ที่ใช้ในการเข้าร่วม axelar network

วิธีต่อไปนี้จะใช้เวลาประมาณ 30-60 นาที ในการเขียนคำสั่ง และ 2-4 ชั่วโมง เพื่อรอให้ blockchain เชื่อมข้อมูลเป็นชุดเดียวกัน (sync)

## Disclaimer
Axelar Network is a work in progress. At no point in time should you transfer any real assets using Axelar. Only use testnet tokens that you're not afraid to lose. Axelar is not responsible for any assets lost, frozen, or unrecoverable in any state or condition. If you find a problem, please submit an issue to this repository following the template.


## Prerequisites (สิ่งที่ต้องมี)

- ระบบปฏิบัติการ Mac OS หรือ Ubuntu (ทดสอบบนเวอร์ชัน 18.04)
- Docker (https://docs.docker.com/engine/install/)
- สเปคคอมฯ ขั้นต่ำ: 4 cores, 8-16GB RAM, 512 GB drive. สเปคฯ ที่แนะนำ 6-8 cores, 16-32 GB RAM, 1 TB+ drive.


## Useful links (ลิงค์ที่เป็นประโยชน์)
- Axelar faucet: http://faucet.testnet.axelar.network/
- Axxelar faucet alt: http://aa0e99f65a3d04b5b8132360d3ec813a-1250815370.us-east-2.elb.amazonaws.com/
- Latest docker images:
  + https://hub.docker.com/repository/docker/axelarnet/axelar-core
  + https://hub.docker.com/repository/docker/axelarnet/tofnd

## Useful commands (คำสั่งเพิ่มเติม)
Axelar node จะทำงานด้วย 2 containers (มี core consensus engine และ threshold crypto process) เราสามารถสั่งหยุด/ลบ containers โดยใช้:
ใช้ docker ps -a -q เพื่อดู ID จากนั้นนำไปต่อท้ายคำสั่ง docker stop
```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```
เช่น docker stop 68789089

หากติด permission, ถ้าไม่ได้ทำการเพิ่ม user `docker` ไปใน `sudo` group, ให้ใช้คำสั่ง `sudo` เพิ่มเข้ามาอยู่หน้าสุดของคำสั่งต่าง ๆ
เช่น sudo docker ps -a

กรณีเจอ error ที่เกี่ยวกับค่า gas ไม่พอจ่ายในขั้นตอนใด ๆ ให้ใส่ flags นี้ต่อท้ายคำสั่ง

```
--gas=auto --gas-adjustment 1.2
```


## การเข้าร่วม Axelar testnet

ทำการ Clone repository ด้วย script และ configs นี้ และ cd ไปที่ axelarate-community:
(หรือทำการคลิกขวาเปิด Terminal ในโฟล์เดอร์ axelarate-community ที่ Clone มาเลยก็ได้

```
git clone https://github.com/axelarnetwork/axelarate-community.git
cd axelarate-community
```

(ขั้นตอนเสริม) แก้ไขสคริป เพื่อให้ทำงานอยู่บนพื้นหลัง (เครดิต อ.ปอร์):

หลังจากที่ clone มาแล้ว ไปที่ Axelarte-community/join แล้วเปิด Terminal

`nano joinTestnet.sh` เลื่อนมาบรรทัดล่างสุด และเพิ่ม `-d` เข้าไป กด Ctrl+o เพื่อบันทีก และ Ctrl+x เพื่อออกจากคำสั่ง nano
```
docker run
   --name axelar-core
   -d
```

การเริ่มใช้งาน (run) script `join/joinTestnet.sh`.
```
ให้พิมพ์เพิ่มตามด้วย [flags]: joinTestnet.sh [flags]

Mandatory flags (flags):

--axelar-core       ตามด้วยเวอร์ชัน axelar-core docker image (Format: vX.Y.Z)
--tofnd             ตามด้วยเวอร์ชัน tofnd docker image (Format: vX.Y.Z)

เช่น join/joinTestnet.sh --axelar-core v0.5.1 --tofnd v0.4.0

flags เพิ่มเติม:
-r, --root          Local directory to store testnet data in (IMPORTANT: this directory is removed and recreated if --reset-chain is set)

--reset-chain       Delete local data to do a clean connect to the testnet (If you participated in an older version of the testnet)

```
ดูที่ TESTNET RELEASE.md เพื่อดูเวอร์ชันที่ใช้ล่าสุดขอ docker images.

เราสามารถใช้คำสั่งนี้เพื่อเรียกดูเวอร์ชันล่าสุด และบันทึกเป็น variables ได้:
```
TOFND_VERSION=$(curl -s https://raw.githubusercontent.com/axelarnetwork/axelarate-community/main/TESTNET%20RELEASE.md | grep tofnd | cut -d \` -f 4)
CORE_VERSION=$(curl -s https://raw.githubusercontent.com/axelarnetwork/axelarate-community/main/TESTNET%20RELEASE.md | grep axelar-core | cut -d \` -f 4)
echo ${TOFND_VERSION} ${CORE_VERSION}
```

หลังจากที่ได้ทำการ join เข้ามาแล้ว ที่ terminal เราจะเห็นว่ามีการสร้าง blocks อย่างรวดเร็ว (ถ้าไม่ได้ปรับแก้ `-d` ให้ทำงานบนพื้นหลัง)

```
2021-05-21T20:22:10Z INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"EFC59CDAF641E5C12FC85B352B06F8E1188D57D6CF5E4C629B6D5E51FEB9A675","parts":{"hash":"2B0E54FA353D22606BF526E4341F1698C7495FA448E28E62E40679793B289D6D","total":1}},"height":229885,"pol_round":-1,"round":0,"signature":"Cqepe/A+mxHNySEMRuAqi97Ah8TiuJNQvMpmQaVrcgA11p5kzt+Fein3A8XZ2TDH4fy6Qv8XBxmrI2HT1cEUBg==","timestamp":"2021-05-21T20:22:10.854851854Z"}
2021-05-21T20:22:10Z INF received complete proposal block hash=EFC59CDAF641E5C12FC85B352B06F8E1188D57D6CF5E4C629B6D5E51FEB9A675 height=229885 module=consensus
2021-05-21T20:22:11Z INF finalizing commit of block hash=EFC59CDAF641E5C12FC85B352B06F8E1188D57D6CF5E4C629B6D5E51FEB9A675 height=229885 module=consensus num_txs=0 root=34060CC7B7A742F051AA8C7940C431BD1A761AAD8700FB400067F83431E0D4E9
2021-05-21T20:22:11Z INF minted coins from module account amount=2136stake from=mint module=x/bank
2021-05-21T20:22:11Z INF executed block height=229885 module=state num_invalid_txs=0 num_valid_txs=0
2021-05-21T20:22:11Z INF commit synced commit=436F6D6D697449447B5B31313120323039203235302031323220323035203133382032303520313734203220313132203532203137322032303620313334203532203139302031393720313132203233332031333120313535203131312031353420313234203130392031393420312032372032313420323320313238203230375D3A33383146447D
2021-05-21T20:22:11Z INF committed state app_hash=6FD1FA7ACD8ACDAE027034ACCE8634BEC570E9839B6F9A7C6DC2011BD61780CF height=229885 module=state num_txs
...
```
ให้รอจน node เราตามทันกับกับเครือข่าย ก่อนไปขั้นตอนต่อไป ซึ่งจะใช้เวลาสักพัก ขึ้นอยู่กับฮาร์ดแวร์ของเรา 
ระหว่างนี้สามารถตรวจสอบสถานะการเชื่อมต่อข้อมูล (sync status) โดยใช้คำสั่ง:
 ```shell script
curl localhost:26657/status | jq '.result.sync_info'
```
หมายเหตุ: ให้ติดตั้ง `jq` สำหรับ json processing ถ้าเป็น Ubuntu ให้ เปิด Terminal ใหม่และติดตั้งโดยใช้คำสั่งนี้: 
`sudo apt-get install jq`

**จะได้ Output ประมาณนี้:**
 ```json
{
  "latest_block_hash": "0B64D2A0EDAB6CEF229510E52F137130134D94AAD64EACB553D51D01B0D1A446",
  "latest_app_hash": "FA3730F49F491DCFF38687F2603CF154563AFA9C77331AF75340C554CB555EFC",
  "latest_block_height": "17051",
  "latest_block_time": "2021-06-01T23:41:43.161261874Z",
  "earliest_block_hash": "080E6B9FC64778F3E0671E046575D3460984F5B1F584E1F2D467341061C7627A",
  "earliest_app_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
  "earliest_block_height": "1",
  "earliest_block_time": "2021-05-31T21:05:12.032466392Z",
  "catching_up": true
}
```
ให้รอจนสถานะ `catching_up` เปลี่ยนเป็นค่า `false`

## การทำ Log
ค่าเริ่มต้น logs จะมี output เป็น stdout (ผมลัพธ์จากคำสั่ง) และ stderr (ผลลัพธ์คำสั่งที่ผิดพลาด) เราสามารถเปลี่ยนให้ Log บันทึกเฉพาะข้อมูลการ debugging และ error โดย:
```
join/joinTestnet.sh --axelar-core ${CORE_VERSION} --tofnd ${TOFND_VERSION} &>> testnet.log
```
และถ้าจะให้ข้อมูล log เป็นแบบ real time ให้ใช้คำสั่งนี้บน Terminal ใหม่ได้:
```
tail -f testnet.log
```
ถ้าพบว่า ข้อมูล log มีมากเกินไป และยากต่อการนำไปใช้ประโยชน์ ก็สามารถกรองข้อมูลได้ด้วยคำสั่งนี้:
```
docker logs -f axelar-core 2>&1 | grep -a -e threshold -e num_txs -e proxies
```

## Ethereum account บน testnet
Axelar จะทำการลงชื่อเป็น meta transactions ให้ Ethereum หมายความว่า บัญชี Ethereum จะสามารถใช้ออกคำสั่งธุรกรรมได้ ตราบใดที่มีการลงชื่อโดย Axelar's key
ในแบบฝึกหัดนี้ ธุรกรรมที่เกี่ยวกับ Ethereum จะถูกส่งมาจาก address `0xE3deF8C6b7E357bf38eC701Ce631f78F2532987A` บน Ropsten testnet ของ Ethereum

## สร้าง key บน Axelar และทดสอบ tokens
1. เปิด Terminal ใหม่ บน /axelarate-community และเข้า Axelar node CLI: 
    ```
    docker exec -it axelar-core sh
    ```
2. โดยค่าเริ่มต้นของ node นั้นจะมีบัญชี validator อยู่แล้ว ซึ่งเราจะหา address จากคำสั่งนี้:
    ```
    axelard keys show validator -a
    ```
3. ไปที่ axelar faucet และส่งเหรียญไปที่ address ของ validator (Node ของเราจะยังไม่ได้เป็น Validator ในการเข้าร่วมกิจกรรมนี้ คือเป็นแค่ชื่อของบัญชีเท่านั้น) http://faucet.testnet.axelar.network/ หรือ http://aa0e99f65a3d04b5b8132360d3ec813a-1250815370.us-east-2.elb.amazonaws.com/

4. ตรวจดูว่าได้รับเงินแล้วหรือยัง
    ```
    axelard q bank balances {output_addr_from_step_2}
    ```
    เช่น
    ```
    axelard q bank balances axelar1p5nl00z6h5fuzyzfylhf8w7g3qj6lmlyryqmhg
    ```
**หมายเหตุ:** ยอดเงินคงเหลือจะแสดงให้เห็นต่อเมื่อมีการ sync กับ network เสร็จสมบูรณ์แล้วเท่านั้น

## การหยุดและการเปิด testnet ขึ้นมาอีกครั้ง
การออกจากหน้า Axelar node CLI, พิมพ์ `exit`
เพื่อหยุดการทำงานของ node ให้เปิด CLI Terminal ใหม่และใช้คำสั่ง
    ```
    docker stop $(docker ps -a -q)
    ```
    ```
    docker rm $(docker ps -a -q)
    ```

การ restart the node, ใช้คำสั่ง `join/joinTestnet.sh` อีกครั้ง, ด้วย flags เดิมของ `--axelar-core`, `--tofnd` (และอาจจะใช้ `--root`) โดย parameters เหมือนเดิม. ห้ามใช้ flag `--reset-chain` ไม่งั้น node ก็จะเริ่ม sync ใหม่ตั้งแต่ต้น

เพื่อเข้า Axelar node CLI อีกครั้งใช้
    ```
    docker exec -it axelar-core sh
    ```
