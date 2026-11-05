# เชื่อมสคริปต์ Roblox เข้ากับ ICE Rejoin

สคริปต์ที่รันผ่าน executor สั่ง ICE Rejoin ได้ 2 อย่าง:

| ต้องการ | คำสั่ง |
|---|---|
| แสดงข้อความในช่อง Description ของบัญชี | `_G.Icesend_Destcripton("ข้อความ")` |
| แจ้งว่าบัญชีนี้ทำงานเสร็จแล้ว | `_G.Icesend_Done()` |

## ติดตั้ง

1. ใน ICE Rejoin ไปที่ `Settings > Checking > Checking Account Online` แล้วเลือก `Script`
2. สร้างไฟล์ `ice_sender.lua` ในโฟลเดอร์ autoexec ของ executor แล้ววางบรรทัดนี้

   ```lua
   loadstring(game:HttpGet("https://raw.githubusercontent.com/kithiw01/roblox_sender/refs/heads/main/roblox_sender.lua"))()
   ```

   | Executor | โฟลเดอร์ |
   |---|---|
   | Potassium | `%LOCALAPPDATA%\Potassium\autoexec` |
   | Volt | `%LOCALAPPDATA%\Volt\autoexec` |
   | ตัวอื่น | โฟลเดอร์ autoexec ของ executor นั้น |

3. เรียกคำสั่งจากสคริปต์ของคุณตามตัวอย่างด้านล่าง

ข้อควรรู้:

- เกมกับ ICE Rejoin ต้องอยู่เครื่องเดียวกัน เพราะ sender ส่งข้อมูลไปที่ `127.0.0.1:18290`
- โหมด `Script` ใช้สัญญาณจาก sender ตรวจว่าบัญชียังออนไลน์ ทุกบัญชีในฟาร์มจึงต้องมี sender ถ้าสัญญาณหายนานเกิน `Delay offline check` โปรแกรมจะถือว่าบัญชีหลุด
- ใส่ sender แค่ชุดเดียว ถ้ามีไฟล์ `roblox_sender.lua` อยู่แล้ว ใช้ไฟล์นั้นแทน loader ได้ แต่อย่าใส่คู่กัน
- sender แจ้งเรื่องหลุด เตะ และแบนให้เอง ไม่ต้องเขียนเพิ่ม

## ตัวอย่างโค้ด

### ส่งข้อความ

```lua
_G.Icesend_Destcripton("Level 10 | Gems 500")
```

ใช้ภาษาไทย อีโมจิ `|` และ `;` ได้ ถ้าส่ง `""` ช่อง Description จะว่าง

### ส่งหลายค่า

```lua
_G.Icesend_Destcripton({ Level = 10, Gems = 500 })
```

ตารางจะแสดงเป็น JSON เช่น `{"Level":10,"Gems":500}`

### แจ้งว่าทำงานเสร็จ

```lua
_G.Icesend_Done()
```

### เช็กว่าส่งสำเร็จ

ทั้งสองคำสั่งคืนค่า `true, "OK"` เมื่อสำเร็จ และ `false, เหตุผล` เมื่อไม่สำเร็จ

```lua
local ok, err = _G.Icesend_Destcripton("Level 10")
if not ok then
    warn("ส่งข้อความไม่สำเร็จ:", err)
end
```

### เทมเพลตพร้อมใช้

สคริปต์นี้รายงานจำนวน Gems ทุก 1 วินาที และแจ้ง DONE เมื่อครบเป้า แก้แค่ `TARGET_GEMS` กับ `getGems()`

```lua
if not game:IsLoaded() then game.Loaded:Wait() end

local TARGET_GEMS   = 1000
local player        = game:GetService("Players").LocalPlayer

-- แก้ฟังก์ชันนี้ให้อ่านค่าจากเกมของคุณ
local function getGems()
    local stats = player:FindFirstChild("leaderstats")
    local gems = stats and stats:FindFirstChild("Gems")
    return gems and gems.Value or 0
end

local function waitForSender()
    for _ = 1, 30 do
        if _G.Icesend_Destcripton and _G.Icesend_Done then return true end
        task.wait(1)
    end
    return false
end

if not waitForSender() then
    warn("ไม่พบ sender: ตรวจไฟล์ในโฟลเดอร์ autoexec")
    return
end

while true do
    local gems = getGems()
    _G.Icesend_Destcripton("Gems " .. gems .. "/" .. TARGET_GEMS)

    if gems >= TARGET_GEMS then
        _G.Icesend_Done()
        break
    end

    task.wait(1)
end
```

## เมื่อเรียก `_G.Icesend_Done()`
1. ICE Rejoin ตั้งสถานะบัญชีเป็น `DONE` และปิดเกมของบัญชีนั้น
2. ถ้าเปิด `Settings > Rejoin > Swap on Done` บัญชี `ACTIVE` ถัดไปจะเข้ามาแทนในฟาร์ม

## ตรวจว่าเชื่อมต่อสำเร็จ
1. กด F9 ในเกม ต้องเห็น `[ROBLOX REJOIN ICE] Active`


## ให้ AI Agent ทำแทนด้วย `/roblox-mcp`
ก่อนเริ่ม ให้เปิด ICE Rejoin, เกม และ executor เอง แล้วโหลด connector ของ roblox-mcp ในเกมให้เรียบร้อย จากนั้นส่งข้อความนี้ให้ Agent:

```text
อ่าน README.md นี้ แล้วเชื่อมสคริปต์เข้ากับ ICE Rejoin โดยใช้ /roblox-mcp
ไฟล์สคริปต์: <path ของสคริปต์>
บัญชีที่ใช้ทดสอบ: <ชื่อบัญชี>
ข้อความที่ต้องการแสดง: <เช่น Level และ Gems ทุก 10 วินาที>
เงื่อนไข DONE: <เช่น Gems ถึง 1000>
```

### ขั้นตอนสำหรับ Agent

1. เรียก `list-clients` ถ้าไม่พบ client ให้หยุดแล้วบอกผู้ใช้เปิดเกม executor และ connector ห้ามเปิดเอง ถ้ามีหลาย client ให้ใช้ `set-active-client` เลือกบัญชีที่ผู้ใช้ระบุ
2. ตรวจ Ice_sender ด้วย `get-data-by-code` ต้องได้ `function` ทั้งคู่ ถ้าไม่ได้ ให้ผู้ใช้ติดตั้งตามหัวข้อ "ติดตั้ง" ห้ามรัน Ice_sender ซ้ำด้วย `execute`
   ```lua
   return typeof(_G.Icesend_Destcripton), typeof(_G.Icesend_Done)
   ```
3. หาค่าจากเกมด้วย `search-instances`, `get-descendants-tree` และ `get-data-by-code`
4. แก้สคริปต์ตามเทมเพลตในหัวข้อ "ตัวอย่างโค้ด" ใช้ชื่อคำสั่งตามคู่มือนี้ และทุก loop ต้องมี `task.wait()`
5. ทดสอบการส่งข้อความด้วย `get-data-by-code` ต้องได้ `true` และ `OK` แล้วเปิด `http://127.0.0.1:18290/api/all?player=<ชื่อบัญชี>` เพื่อดูว่า `description` ตรงกัน ถ้าไม่ผ่านให้อ่าน log ด้วย `get-console-output` ข้อความทดสอบจะทับ Description เดิมของบัญชีนั้น
   ```lua
   local ok, err = _G.Icesend_Destcripton("ICE test")
   return ok, err
   ```
6. ห้ามเรียก `_G.Icesend_Done()` ตอนทดสอบ ถ้าผู้ใช้ไม่ได้สั่งโดยตรง เพราะเกมจะถูกปิดและสถานะบัญชีจะเปลี่ยน ให้ทดสอบเงื่อนไข DONE ด้วย `print` แทน
