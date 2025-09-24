-- โหลด UI Library
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Hyper👽", "DarkTheme")
local Tab = Window:NewTab("All")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- =======================
-- WalkSpeed & JumpPower
-- =======================
local WalkSpeedValue = 16
local WalkSpeedEnabled = false
local JumpPowerValue = 50
local JumpPowerEnabled = false

-- =======================
-- NoClip
-- =======================
_G.NoClip = false
_G.NoClipConnection = nil

-- =======================
-- Fly
-- =======================
local FlySpeed = 50
local FlyConnection = nil
local bv, bg = nil, nil

-- =======================
-- Hitbox
-- =======================
local HeadSize = 10
local HitboxEnabled = false
local HitboxConnection = nil

-- ฟังก์ชันอัปเดต WalkSpeed
local function applyWalkSpeed(char)
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char.Humanoid
        humanoid.WalkSpeed = WalkSpeedEnabled and WalkSpeedValue or 16
    end
end

-- ฟังก์ชันอัปเดต JumpPower
local function applyJumpPower(char)
    if char and char:FindFirstChild("Humanoid") then
        local humanoid = char.Humanoid
        if JumpPowerEnabled then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = JumpPowerValue
        else
            humanoid.JumpPower = 50
        end
    end
end

-- ฟังก์ชัน NoClip
local function enableNoClip(char)
    _G.NoClip = true
    _G.NoClipConnection = RunService.Stepped:Connect(function()
        if _G.NoClip and char then
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") and v.CanCollide then
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

-- ฟังก์ชัน Fly
local function startFly(char)
    local humanoid = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")

    bv = Instance.new("BodyVelocity", root)
    bv.MaxForce = Vector3.new(9e9,9e9,9e9)
    bv.Velocity = Vector3.zero
    bv.Name = "FlyVelocity"

    bg = Instance.new("BodyGyro", root)
    bg.MaxTorque = Vector3.new(9e9,9e9,9e9)
    bg.P = 10000
    bg.Name = "FlyGyro"

    humanoid.PlatformStand = true

    FlyConnection = RunService.RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera.CFrame
        local dir = Vector3.zero
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += cam.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= cam.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= cam.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += cam.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then dir -= Vector3.new(0,1,0) end
        bv.Velocity = dir.Magnitude > 0 and dir.Unit * FlySpeed or Vector3.zero
        bg.CFrame = cam
    end)
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
end

-- ฟังก์ชัน Hitbox
local function applyHitbox()
    if HitboxConnection then HitboxConnection:Disconnect() HitboxConnection = nil end
    if HitboxEnabled then
        HitboxConnection = RunService.RenderStepped:Connect(function()
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = plr.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(HeadSize, HeadSize, HeadSize)
                    hrp.Transparency = 0.7
                    hrp.Color = Color3.fromRGB(0,0,0)
                    hrp.Material = Enum.Material.Neon
                    hrp.CanCollide = false
                end
            end
        end)
    else
        -- รีเซ็ตขนาด
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                hrp.Size = Vector3.new(2,2,1)
                hrp.Transparency = 1
                hrp.Material = Enum.Material.Plastic
                hrp.CanCollide = true
            end
        end
    end
end

-- ฟังก์ชันเมื่อเกิดใหม่
local function onCharacterAdded(char)
    applyWalkSpeed(char)
    applyJumpPower(char)
    if _G.NoClip then enableNoClip(char) end
    if FlyConnection then startFly(char) end
end

-- เชื่อมกับตัวละครปัจจุบันและเกิดใหม่
if player.Character then onCharacterAdded(player.Character) end
player.CharacterAdded:Connect(onCharacterAdded)

-- =======================
-- UI
-- =======================

-- WalkSpeed Section
local WalkSection = Tab:NewSection("WalkSpeed")
WalkSection:NewToggle("WalkSpeed", "เปิด/ปิดการเปลี่ยนความเร็วเดิน", function(state)
    WalkSpeedEnabled = state
    applyWalkSpeed(player.Character)
end)
WalkSection:NewTextBox("WalkSpeed Value", "ใส่ค่าความเร็วเดิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        WalkSpeedValue = num
        applyWalkSpeed(player.Character)
    else warn("กรุณาใส่ตัวเลขมากกว่า 0") end
end)

-- JumpPower Section
local JumpSection = Tab:NewSection("JumpPower")
JumpSection:NewToggle("JumpPower", "เปิด/ปิดการเปลี่ยนความสูงการกระโดด", function(state)
    JumpPowerEnabled = state
    applyJumpPower(player.Character)
end)
JumpSection:NewTextBox("JumpPower Value", "ใส่ค่าความสูงกระโดด", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        JumpPowerValue = num
        applyJumpPower(player.Character)
    else warn("กรุณาใส่ตัวเลขมากกว่า 0") end
end)

-- NoClip Section
local NoclipSection = Tab:NewSection("NoClip")
NoclipSection:NewToggle("NoClip", "เดินทะลุกำแพง", function(state)
    if state then enableNoClip(player.Character) else disableNoClip(player.Character) end
end)

-- Fly Section
local FlySection = Tab:NewSection("Fly")
FlySection:NewToggle("Fly", "เปิด/ปิดโหมดบิน", function(state)
    if state then startFly(player.Character) else stopFly(player.Character) end
end)
FlySection:NewTextBox("Fly Speed", "ใส่ค่าความเร็วในการบิน", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then FlySpeed = num else warn("กรุณาใส่ตัวเลขมากกว่า 0") end
end)

-- Hitbox Section
local HitboxSection = Tab:NewSection("Hitbox")
HitboxSection:NewToggle("Hitbox", "เปิด/ปิดการทำงานของ Hitbox", function(state)
    HitboxEnabled = state
    applyHitbox()
end)
HitboxSection:NewTextBox("Hitbox Size", "ใส่ค่าขนาดของ Hitbox", function(txt)
    local num = tonumber(txt)
    if num and num > 0 then
        HeadSize = num
        applyHitbox()
    else warn("กรุณาใส่ตัวเลขมากกว่า 0") end
end)

-- Keybind
local SettingSection = Tab:NewSection("Settings")
SettingSection:NewKeybind("Toggle UI", "เปิด/ปิด UI", Enum.KeyCode.F, function()
    Library:ToggleUI()
end)
