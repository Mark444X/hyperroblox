local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Hyper👽", "DarkTheme")
local Tab = Window:NewTab("All")
local Section = Tab:NewSection("WalkSpeed")
-- ค่าตั้งต้น
local WalkSpeedValue = 16
local WalkSpeedEnabled = false

-- ฟังก์ชันอัปเดต WalkSpeed
local player = game.Players.LocalPlayer

-- ฟังก์ชันปรับ WalkSpeed
local function applyWalkSpeed()
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char:FindFirstChild("Humanoid")
        if WalkSpeedEnabled then
            humanoid.WalkSpeed = WalkSpeedValue
        else
            humanoid.WalkSpeed = 16 -- ค่า default ของ Roblox (หรือแมพ)
        end
    end
end

-- เวลาตัวละครเกิดใหม่ จะเรียกใช้
player.CharacterAdded:Connect(function()
    task.wait(0.1) -- รอให้ Humanoid โหลดก่อน
    applyWalkSpeed()
end)

-- ถ้ามีตัวละครอยู่แล้วตอนเริ่มเกม
if player.Character then
    task.wait(0.1)
    applyWalkSpeed()
end


-- ฟังก์ชันเมื่อเกิดใหม่
local function onCharacterAdded(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Died:Connect(function()
        -- เมื่อ Player ตาย รอ Respawn แล้วอัปเดตค่าใหม่
        player.CharacterAdded:Wait()
        applyWalkSpeed()
    end)
    applyWalkSpeed()
end

-- โหลดตัวละครปัจจุบัน
local player = game.Players.LocalPlayer
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Toggle เปิด/ปิด WalkSpeed
Section:NewToggle("WalkSpeed", "เปิด/ปิดการเปลี่ยนความเร็วเดิน", function(state)
    WalkSpeedEnabled = state
    applyWalkSpeed()
    print("WalkSpeed: "..(WalkSpeedEnabled and "เปิด ("..WalkSpeedValue..")" or "ปิด (16)"))
end)

-- TextBox สำหรับเปลี่ยนค่า WalkSpeed
Section:NewTextBox("WalkSpeed Value", "ใส่ค่าความเร็วเดิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        WalkSpeedValue = num
        print("WalkSpeed ถูกเปลี่ยนเป็น: "..WalkSpeedValue)
        applyWalkSpeed()
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

local Section = Tab:NewSection("WalkSpeed")
-- ค่าตั้งต้น
local JumpPowerValue = 50
local JumpPowerEnabled = false

-- ฟังก์ชันอัปเดต JumpPower
local function applyJumpPower()
    local player = game.Players.LocalPlayer
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char:FindFirstChild("Humanoid")
        if JumpPowerEnabled then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = JumpPowerValue
        else
            humanoid.JumpPower = 50 -- ค่า default Roblox
        end
    end
end

-- ฟังก์ชันเมื่อเกิดใหม่
local function onCharacterAdded(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Died:Connect(function()
        player.CharacterAdded:Wait()
        applyJumpPower()
    end)
    applyJumpPower()
end

-- โหลดตัวละครปัจจุบัน
local player = game.Players.LocalPlayer
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Toggle เปิด/ปิด JumpPower
Section:NewToggle("JumpPower", "เปิด/ปิดการเปลี่ยนความสูงการกระโดด", function(state)
    JumpPowerEnabled = state
    applyJumpPower()
    print("JumpPower: "..(JumpPowerEnabled and "เปิด ("..JumpPowerValue..")" or "ปิด (50)"))
end)

-- TextBox สำหรับเปลี่ยนค่า JumpPower
Section:NewTextBox("JumpPower Value", "ใส่ค่าความสูงกระโดด", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        JumpPowerValue = num
        print("JumpPower ถูกเปลี่ยนเป็น: "..JumpPowerValue)
        applyJumpPower()
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

-- ฟังก์ชันNoclip
local Section = Tab:NewSection("NoClip")

Section:NewToggle("NoClip", "เดินทะลุกำแพง", function(state)
    local player = game.Players.LocalPlayer

    local function enableNoClip(char)
        _G.NoClip = true
        _G.NoClipConnection = game:GetService("RunService").Stepped:Connect(function()
            if _G.NoClip and char then
                for _, v in pairs(char:GetDescendants()) do
                    if v:IsA("BasePart") and v.CanCollide == true then
                        v.CanCollide = false
                    end
                end
            end
        end)
    end

    local function disableNoClip(char)
        _G.NoClip = false
        if _G.NoClipConnection then
            _G.NoClipConnection:Disconnect()
            _G.NoClipConnection = nil
        end
        if char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = true
                end
            end
        end
    end

    local char = player.Character or player.CharacterAdded:Wait()
    if state then
        enableNoClip(char)
        print("NoClip On")
        -- เวลา Respawn ก็เปิดให้อัตโนมัติ
        player.CharacterAdded:Connect(function(newChar)
            task.wait(0.1)
            if _G.NoClip then
                enableNoClip(newChar)
            end
        end)
    else
        disableNoClip(char)
        print("NoClip Off")
    end
end)

-- ฟังก์ชันFly
Section = Tab:NewSection("Fly")
local FlySpeed = 50
local FlyConnection
local bv, bg
local player = game.Players.LocalPlayer

local function startFly(char)
    local humanoid = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")
    local uis = game:GetService("UserInputService")
    local rs = game:GetService("RunService")

    -- สร้าง BodyMover
    bv = Instance.new("BodyVelocity", root)
    bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bv.Velocity = Vector3.zero
    bv.Name = "FlyVelocity"

    bg = Instance.new("BodyGyro", root)
    bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.P = 10000
    bg.Name = "FlyGyro"

    humanoid.PlatformStand = true

    FlyConnection = rs.RenderStepped:Connect(function()
        if bv and bg then
            local cam = workspace.CurrentCamera.CFrame
            local dir = Vector3.zero

            if uis:IsKeyDown(Enum.KeyCode.W) then dir += cam.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.S) then dir -= cam.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.A) then dir -= cam.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.D) then dir += cam.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if uis:IsKeyDown(Enum.KeyCode.LeftShift) then dir -= Vector3.new(0,1,0) end

            bv.Velocity = dir.Magnitude > 0 and dir.Unit * FlySpeed or Vector3.zero
            bg.CFrame = cam
        end
    end)

    print("บิน: เปิด | ความเร็ว = "..FlySpeed)
end

local function stopFly(char)
    local humanoid = char:FindFirstChild("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")

    if root then
        if root:FindFirstChild("FlyVelocity") then root.FlyVelocity:Destroy() end
        if root:FindFirstChild("FlyGyro") then root.FlyGyro:Destroy() end
    end

    if humanoid then humanoid.PlatformStand = false end
    if FlyConnection then FlyConnection:Disconnect() FlyConnection = nil end

    print("บิน: ปิด")
end

-- Toggle สำหรับเปิด/ปิดบิน
Section:NewToggle("Fly", "เปิด/ปิดโหมดบิน", function(state)
    local char = player.Character or player.CharacterAdded:Wait()
    if state then
        startFly(char)
        -- ถ้า Respawn แล้วยังเปิดอยู่ จะเริ่มใหม่ให้อัตโนมัติ
        player.CharacterAdded:Connect(function(newChar)
            task.wait(0.1)
            if state then
                startFly(newChar)
            end
        end)
    else
        stopFly(char)
    end
end)

-- TextBox สำหรับปรับความเร็วบิน
Section:NewTextBox("Fly Speed", "ใส่ค่าความเร็วในการบิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        FlySpeed = num
        print("Fly Speed ถูกเปลี่ยนเป็น: "..FlySpeed)
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

-- ฟังก์ชันhitbox
local Section = Tab:NewSection("Hitbox")
-- ค่าตั้งต้นของ Hitbox
local HeadSize = 10
local HitboxEnabled = false
local Connection

-- Toggle เปิด/ปิด Hitbox
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("red shop", "DarkTheme")
local Tab = Window:NewTab("All")
local Section = Tab:NewSection("WalkSpeed")
-- ค่าตั้งต้น
local WalkSpeedValue = 16
local WalkSpeedEnabled = false

-- ฟังก์ชันอัปเดต WalkSpeed
local player = game.Players.LocalPlayer

-- ฟังก์ชันปรับ WalkSpeed
local function applyWalkSpeed()
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char:FindFirstChild("Humanoid")
        if WalkSpeedEnabled then
            humanoid.WalkSpeed = WalkSpeedValue
        else
            humanoid.WalkSpeed = 16 -- ค่า default ของ Roblox (หรือแมพ)
        end
    end
end

-- เวลาตัวละครเกิดใหม่ จะเรียกใช้
player.CharacterAdded:Connect(function()
    task.wait(0.1) -- รอให้ Humanoid โหลดก่อน
    applyWalkSpeed()
end)

-- ถ้ามีตัวละครอยู่แล้วตอนเริ่มเกม
if player.Character then
    task.wait(0.1)
    applyWalkSpeed()
end


-- ฟังก์ชันเมื่อเกิดใหม่
local function onCharacterAdded(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Died:Connect(function()
        -- เมื่อ Player ตาย รอ Respawn แล้วอัปเดตค่าใหม่
        player.CharacterAdded:Wait()
        applyWalkSpeed()
    end)
    applyWalkSpeed()
end

-- โหลดตัวละครปัจจุบัน
local player = game.Players.LocalPlayer
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Toggle เปิด/ปิด WalkSpeed
Section:NewToggle("WalkSpeed", "เปิด/ปิดการเปลี่ยนความเร็วเดิน", function(state)
    WalkSpeedEnabled = state
    applyWalkSpeed()
    print("WalkSpeed: "..(WalkSpeedEnabled and "เปิด ("..WalkSpeedValue..")" or "ปิด (16)"))
end)

-- TextBox สำหรับเปลี่ยนค่า WalkSpeed
Section:NewTextBox("WalkSpeed Value", "ใส่ค่าความเร็วเดิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        WalkSpeedValue = num
        print("WalkSpeed ถูกเปลี่ยนเป็น: "..WalkSpeedValue)
        applyWalkSpeed()
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

local Section = Tab:NewSection("WalkSpeed")
-- ค่าตั้งต้น
local JumpPowerValue = 50
local JumpPowerEnabled = false

-- ฟังก์ชันอัปเดต JumpPower
local function applyJumpPower()
    local player = game.Players.LocalPlayer
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char:FindFirstChild("Humanoid")
        if JumpPowerEnabled then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = JumpPowerValue
        else
            humanoid.JumpPower = 50 -- ค่า default Roblox
        end
    end
end

-- ฟังก์ชันเมื่อเกิดใหม่
local function onCharacterAdded(char)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.Died:Connect(function()
        player.CharacterAdded:Wait()
        applyJumpPower()
    end)
    applyJumpPower()
end

-- โหลดตัวละครปัจจุบัน
local player = game.Players.LocalPlayer
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Toggle เปิด/ปิด JumpPower
Section:NewToggle("JumpPower", "เปิด/ปิดการเปลี่ยนความสูงการกระโดด", function(state)
    JumpPowerEnabled = state
    applyJumpPower()
    print("JumpPower: "..(JumpPowerEnabled and "เปิด ("..JumpPowerValue..")" or "ปิด (50)"))
end)

-- TextBox สำหรับเปลี่ยนค่า JumpPower
Section:NewTextBox("JumpPower Value", "ใส่ค่าความสูงกระโดด", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        JumpPowerValue = num
        print("JumpPower ถูกเปลี่ยนเป็น: "..JumpPowerValue)
        applyJumpPower()
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

-- ฟังก์ชันNoclip
local Section = Tab:NewSection("NoClip")

Section:NewToggle("NoClip", "เดินทะลุกำแพง", function(state)
    local player = game.Players.LocalPlayer

    local function enableNoClip(char)
        _G.NoClip = true
        _G.NoClipConnection = game:GetService("RunService").Stepped:Connect(function()
            if _G.NoClip and char then
                for _, v in pairs(char:GetDescendants()) do
                    if v:IsA("BasePart") and v.CanCollide == true then
                        v.CanCollide = false
                    end
                end
            end
        end)
    end

    local function disableNoClip(char)
        _G.NoClip = false
        if _G.NoClipConnection then
            _G.NoClipConnection:Disconnect()
            _G.NoClipConnection = nil
        end
        if char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = true
                end
            end
        end
    end

    local char = player.Character or player.CharacterAdded:Wait()
    if state then
        enableNoClip(char)
        print("NoClip On")
        -- เวลา Respawn ก็เปิดให้อัตโนมัติ
        player.CharacterAdded:Connect(function(newChar)
            task.wait(0.1)
            if _G.NoClip then
                enableNoClip(newChar)
            end
        end)
    else
        disableNoClip(char)
        print("NoClip Off")
    end
end)

-- ฟังก์ชันFly
Section = Tab:NewSection("Fly")
local FlySpeed = 50
local FlyConnection
local bv, bg
local player = game.Players.LocalPlayer

local function startFly(char)
    local humanoid = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")
    local uis = game:GetService("UserInputService")
    local rs = game:GetService("RunService")

    -- สร้าง BodyMover
    bv = Instance.new("BodyVelocity", root)
    bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bv.Velocity = Vector3.zero
    bv.Name = "FlyVelocity"

    bg = Instance.new("BodyGyro", root)
    bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    bg.P = 10000
    bg.Name = "FlyGyro"

    humanoid.PlatformStand = true

    FlyConnection = rs.RenderStepped:Connect(function()
        if bv and bg then
            local cam = workspace.CurrentCamera.CFrame
            local dir = Vector3.zero

            if uis:IsKeyDown(Enum.KeyCode.W) then dir += cam.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.S) then dir -= cam.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.A) then dir -= cam.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.D) then dir += cam.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if uis:IsKeyDown(Enum.KeyCode.LeftShift) then dir -= Vector3.new(0,1,0) end

            bv.Velocity = dir.Magnitude > 0 and dir.Unit * FlySpeed or Vector3.zero
            bg.CFrame = cam
        end
    end)

    print("บิน: เปิด | ความเร็ว = "..FlySpeed)
end

local function stopFly(char)
    local humanoid = char:FindFirstChild("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")

    if root then
        if root:FindFirstChild("FlyVelocity") then root.FlyVelocity:Destroy() end
        if root:FindFirstChild("FlyGyro") then root.FlyGyro:Destroy() end
    end

    if humanoid then humanoid.PlatformStand = false end
    if FlyConnection then FlyConnection:Disconnect() FlyConnection = nil end

    print("บิน: ปิด")
end

-- Toggle สำหรับเปิด/ปิดบิน
Section:NewToggle("Fly", "เปิด/ปิดโหมดบิน", function(state)
    local char = player.Character or player.CharacterAdded:Wait()
    if state then
        startFly(char)
        -- ถ้า Respawn แล้วยังเปิดอยู่ จะเริ่มใหม่ให้อัตโนมัติ
        player.CharacterAdded:Connect(function(newChar)
            task.wait(0.1)
            if state then
                startFly(newChar)
            end
        end)
    else
        stopFly(char)
    end
end)

-- TextBox สำหรับปรับความเร็วบิน
Section:NewTextBox("Fly Speed", "ใส่ค่าความเร็วในการบิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        FlySpeed = num
        print("Fly Speed ถูกเปลี่ยนเป็น: "..FlySpeed)
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)

-- ฟังก์ชันhitbox
local Section = Tab:NewSection("Hitbox")
-- ค่าตั้งต้นของ Hitbox
local HeadSize = 10
local HitboxEnabled = false
local Connection

-- Toggle เปิด/ปิด Hitbox
Section:NewToggle("Hitbox", "เปิด/ปิดการทำงานของ Hitbox", function(state)
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LocalPlayer = Players.LocalPlayer

    HitboxEnabled = state

    if HitboxEnabled then
        print("Hitbox: เปิด | ขนาด = "..HeadSize)
        Connection = RunService.RenderStepped:Connect(function()
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer -- ❌ ไม่ทำกับตัวเอง
                and (not plr.Team or not LocalPlayer.Team or plr.Team ~= LocalPlayer.Team) -- ❌ ไม่ทำกับเพื่อนร่วมทีม
                and plr.Character 
                and plr.Character:FindFirstChild("HumanoidRootPart") then

                    local hrp = plr.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(HeadSize, HeadSize, HeadSize)
                    hrp.Transparency = 0.7
                    hrp.Color = Color3.fromRGB(0,0,0) -- สีดำ
                    hrp.Material = Enum.Material.Neon
                    hrp.CanCollide = false
                end
            end
        end)
    else
        print("Hitbox: ปิด")
        if Connection then Connection:Disconnect() Connection = nil end
        -- รีเซ็ตขนาดกลับเป็นปกติ
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer -- ❌ ไม่ยุ่งกับตัวเอง
            and plr.Character 
            and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                hrp.Size = Vector3.new(2,2,1) -- ค่า default ของ HRP
                hrp.Transparency = 1
                hrp.Material = Enum.Material.Plastic
                hrp.CanCollide = true
            end
        end
    end
end)

-- TextBox สำหรับปรับขนาด Hitbox
Section:NewTextBox("Hitbox Size", "ใส่ค่าขนาดของ Hitbox", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        HeadSize = num
        print("Hitbox Size ถูกเปลี่ยนเป็น: "..HeadSize)
    else
        warn("กรุณาใส่ตัวเลขมากกว่า 0")
    end
end)




local Section = Tab:NewSection("settings")
Section:NewKeybind("KeybindText", "KeybindInfo", Enum.KeyCode.F, function()
	Library:ToggleUI()
end)





local Section = Tab:NewSection("settings")
Section:NewKeybind("KeybindText", "KeybindInfo", Enum.KeyCode.F, function()
	Library:ToggleUI()
end)
