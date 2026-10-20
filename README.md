-- ============================================
-- STEAL A BRAINROT — FULL SCRIPT (FINAL)
-- Усиленный Bypass + Anti-Reset + Speed + Anti Bat + Inf Jump
-- ============================================

-- ============================================
-- ЧАСТЬ 0: УСИЛЕННЫЙ ANTI-CHEAT BYPASS + ANTI-RESET
-- ============================================
pcall(function()
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local mt = getrawmetatable(game)

    -- ---------- 1) namecall: Kick / Reset / Destroy / ChangeState / TeleportTo / InvokeServer ----------
    local oldNamecall = mt.__namecall
    setreadonly(mt, false)
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()

        if method == "Kick" and (self == LocalPlayer or self == Players) then
            warn("[Bypass] Заблокирован Kick")
            return
        end
        if method == "ResetCharacter" and self:IsA("Humanoid") then
            local char = LocalPlayer.Character
            if char and self:IsDescendantOf(char) then
                warn("[Anti-Reset] ResetCharacter")
                return
            end
        end
        if method == "Destroy" and self:IsA("Model") then
            local char = LocalPlayer.Character
            if char and self == char then
                warn("[Anti-Reset] Destroy персонажа")
                return
            end
        end
        if method == "ChangeState" and self:IsA("Humanoid") then
            local args = {...}
            local newState = args[1]
            if newState == Enum.HumanoidStateType.Physics
            or newState == Enum.HumanoidStateType.GettingUp
            or newState == Enum.HumanoidStateType.FallingDown
            or newState == Enum.HumanoidStateType.Dead then
                local char = LocalPlayer.Character
                if char and self:IsDescendantOf(char) then
                    warn("[Anti-Reset] ChangeState: " .. tostring(newState))
                    return
                end
            end
        end
        if (method == "TeleportTo" or method == "PivotTo") and self:IsA("Model") then
            local char = LocalPlayer.Character
            if char and self == char then
                warn("[Anti-Reset] " .. method .. " заблокирован")
                return
            end
        end
        if method == "InvokeServer" and self:IsA("RemoteFunction") then
            local name = string.lower(self.Name)
            local BLOCK_KW = {"Report","AntiCheat","Ban","Kick","Detect","Flag","Log","Sus","Cheat","Exploit","Hack","Violation","Monitor","Reset"}
            for _, kw in ipairs(BLOCK_KW) do
                if string.find(name, string.lower(kw)) then
                    warn("[Bypass] Блок InvokeServer: " .. self.Name)
                    return
                end
            end
        end
        return oldNamecall(self, ...)
    end)
    setreadonly(mt, true)

    -- ---------- 2) hookfunction ----------
    if hookfunction then
        pcall(function() hookfunction(LocalPlayer.Kick, function(...) warn("[Bypass] Kick LP"); return end) end)
        pcall(function() hookfunction(Players.Kick, function(...) warn("[Bypass] Kick P"); return end) end)
        pcall(function() hookfunction(LocalPlayer.LoadCharacter, function(...) warn("[Anti-Reset] LoadCharacter"); return end) end)
    end

    -- ---------- 3) __index: маскировка чтения ----------
    local oldIndex = mt.__index
    setreadonly(mt, false)
    mt.__index = newcclosure(function(self, key)
        if self:IsA("Humanoid") then
            local char = LocalPlayer.Character
            if char and self:IsDescendantOf(char) then
                if key == "WalkSpeed" then return 16
                elseif key == "JumpPower" then return 50
                elseif key == "HipHeight" then return 2
                elseif key == "Health" then return 100 end
            end
        end
        return oldIndex(self, key)
    end)
    setreadonly(mt, true)

    -- ---------- 4) __newindex: без маскировки WalkSpeed (Speed работает) ----------
    local oldNewIndex = mt.__newindex
    setreadonly(mt, false)
    mt.__newindex = newcclosure(function(self, key, value)
        if self == LocalPlayer and key == "Character" and value == nil then
            warn("[Anti-Reset] Character = nil")
            return
        end
        if self:IsA("Humanoid") and key == "Health" and value == 0 then
            local char = LocalPlayer.Character
            if char and self:IsDescendantOf(char) then
                warn("[Anti-Reset] Health = 0")
                return
            end
        end
        if self:IsA("BasePart") and key == "Anchored" and value == true then
            local char = LocalPlayer.Character
            if char and self:IsDescendantOf(char) then
                warn("[Anti-Reset] Anchored = true")
                return
            end
        end
        return oldNewIndex(self, key, value)
    end)
    setreadonly(mt, true)

    -- ---------- 5) Блокировка FireServer ----------
    local BLOCK_KEYWORDS = {"Report","AntiCheat","Ban","Kick","Detect","Flag","Log","Sus","Cheat","Exploit","Hack","Violation","Monitor","Reset"}
    local oldNamecall2 = mt.__namecall
    setreadonly(mt, false)
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if method == "FireServer" and self:IsA("RemoteEvent") then
            local name = string.lower(self.Name)
            for _, kw in ipairs(BLOCK_KEYWORDS) do
                if string.find(name, string.lower(kw)) then
                    warn("[Bypass] Блок FireServer: " .. self.Name)
                    return
                end
            end
        end
        return oldNamecall2(self, ...)
    end)
    setreadonly(mt, true)

    -- ---------- 6) Anti-Reset в воздухе ----------
    task.spawn(function()
        while true do
            task.wait(0.05)
            local char = LocalPlayer.Character
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if hrp and humanoid then
                    local state = humanoid:GetState()
                    if state == Enum.HumanoidStateType.Freefall
                    or state == Enum.HumanoidStateType.Jumping then
                        local vel = hrp.AssemblyLinearVelocity
                        if vel.Y < -50 then
                            hrp.AssemblyLinearVelocity = Vector3.new(vel.X, -50, vel.Z)
                        end
                    end
                end
            end
        end
    end)

    -- ---------- 7) Защита персонажа ----------
    local lastCharacter = LocalPlayer.Character
    LocalPlayer.CharacterAdded:Connect(function(newChar) lastCharacter = newChar end)

    task.spawn(function()
        while true do
            task.wait(0.1)
            if LocalPlayer.Character == nil and lastCharacter and lastCharacter.Parent then
                warn("[Anti-Reset] Восстанавливаем персонажа...")
                pcall(function() LocalPlayer.Character = lastCharacter end)
            end
        end
    end)

    local function protectCharacter(char)
        if not char then return end
        char.AncestryChanged:Connect(function(_, parent)
            if not parent and char == lastCharacter then
                pcall(function() LocalPlayer.Character = char end)
            end
        end)
    end
    if LocalPlayer.Character then protectCharacter(LocalPlayer.Character) end
    LocalPlayer.CharacterAdded:Connect(protectCharacter)

    print("[Bypass] Усиленный Anti-Cheat Bypass + Anti-Reset загружен")
end)

-- ============================================
-- СЕРВИСЫ
-- ============================================
local Players           = game:GetService("Players")
local UserInputService  = game:GetService("UserInputService")
local Debris            = game:GetService("Debris")
local LocalPlayer       = Players.LocalPlayer

-- ============================================
-- ЧАСТЬ 1: GUI (Negor_Anti.bat)
-- ============================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "NegorAntiBatGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 260, 0, 260)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(255, 255, 255)
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 34)
title.BackgroundTransparency = 1
title.Text = "Negor_Anti.bat"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 20
title.Font = Enum.Font.Code
title.Parent = mainFrame

local titleStroke = Instance.new("UIStroke")
titleStroke.Color = Color3.fromRGB(255, 255, 255)
titleStroke.Thickness = 2
titleStroke.Parent = title

local function createRow(text, yPos)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, -20, 0, 40)
    row.Position = UDim2.new(0, 10, 0, yPos)
    row.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    row.BorderSizePixel = 0
    row.Parent = mainFrame
    local rs = Instance.new("UIStroke")
    rs.Color = Color3.fromRGB(255, 255, 255); rs.Thickness = 1.5; rs.Parent = row
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 1, 0); label.Position = UDim2.new(0, 5, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text; label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 16; label.Font = Enum.Font.Code
    label.TextXAlignment = Enum.TextXAlignment.Left; label.Parent = row
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 90, 1, 0); status.Position = UDim2.new(1, -95, 0, 0)
    status.BackgroundTransparency = 1
    status.Text = ""; status.TextColor3 = Color3.fromRGB(255, 255, 255)
    status.TextSize = 16; status.Font = Enum.Font.Code
    status.TextXAlignment = Enum.TextXAlignment.Right; status.Parent = row
    return status
end

local speedStatus   = createRow("Speed [Q]", 45)
local antiBatStatus = createRow("Anti Bat [G]", 95)
local infJumpStatus = createRow("Inf Jump [Hold Space]", 145)

-- ============================================
-- ЧАСТЬ 2: УЛУЧШЕННЫЙ SPEED BYPASS (Q)
-- ============================================
local Settings = {
    NormalSpeed = 80, CarrySpeed = 40, ToggleKey = Enum.KeyCode.Q,
    StartMode = "Normal", MaxSpeed = 80, ToggleCooldown = 0.5,
}
local currentMode = Settings.StartMode
local lastPosition = nil
local lastToggle = 0
local realWalkSpeed = 16

local function updateSpeedGui()
    speedStatus.Text = (currentMode == "Normal") and "Normal: 80" or "Carry: 40"
end

local function applySpeedSmooth(humanoid, targetSpeed)
    targetSpeed = math.min(targetSpeed, Settings.MaxSpeed)
    local current = humanoid.WalkSpeed
    local diff = targetSpeed - current
    for i = 1, 12 do
        if not humanoid or not humanoid.Parent then return end
        humanoid.WalkSpeed = current + (diff / 12) * i
        task.wait(0.04)
    end
    humanoid.WalkSpeed = targetSpeed
    realWalkSpeed = targetSpeed
end

local function speed(mode)
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    local target
    if mode == "Carry" then target = Settings.CarrySpeed; currentMode = "Carry"
    else target = Settings.NormalSpeed; currentMode = "Normal" end
    applySpeedSmooth(humanoid, target)
    updateSpeedGui()
end

local function toggleSpeedMode()
    local now = tick()
    if now - lastToggle < Settings.ToggleCooldown then return end
    lastToggle = now
    if currentMode == "Normal" then speed("Carry") else speed("Normal") end
end

task.spawn(function()
    while true do
        task.wait(0.1)
        local char = LocalPlayer.Character
        if not char then continue end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not humanoid then continue end

        if lastPosition then
            local moved = (hrp.Position - lastPosition).Magnitude
            if moved < 0.5 and humanoid.MoveDirection.Magnitude > 0.1 and realWalkSpeed > 16 then
                humanoid.WalkSpeed = math.min(humanoid.WalkSpeed + 2, Settings.MaxSpeed)
            end
        end
        lastPosition = hrp.Position

        local vel = hrp.AssemblyLinearVelocity
        if vel.Y < -50 then
            hrp.AssemblyLinearVelocity = Vector3.new(vel.X, -50, vel.Z)
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Settings.ToggleKey then toggleSpeedMode() end
end)

LocalPlayer.CharacterAdded:Connect(function() task.wait(0.5); speed(currentMode) end)
if LocalPlayer.Character then speed(currentMode) end

-- ============================================
-- ЧАСТЬ 3: ANTI BAT (G) — точная механика
-- ============================================
local PUSH_FORCE        = 150
local PUSH_UP           = 40
local PUSH_DURATION     = 0.35
local CHECK_INTERVAL    = 0.05
local PUSH_COOLDOWN     = 0.3
local BAT_NAME_KEYWORD  = "Bat"

local ANTI_BAT_BIND     = Enum.KeyCode.G
local ANTI_BAT_START    = true

local antiBatEnabled = ANTI_BAT_START
local recentlyPushed = {}
local activeVelocities = {}

local function updateAntiBatGui()
    antiBatStatus.Text = antiBatEnabled and "ON" or "OFF"
end

local function isBat(tool)
    if not tool or not tool:IsA("Tool") then return false end
    return string.find(string.lower(tool.Name), string.lower(BAT_NAME_KEYWORD)) ~= nil
end

local function isHoldingBat(player)
    local char = player.Character
    if not char then return false end
    for _, child in ipairs(char:GetChildren()) do
        if isBat(child) then return true end
    end
    return false
end

local function clearVelocity(player)
    local bv = activeVelocities[player]
    if bv then
        pcall(function() bv:Destroy() end)
        activeVelocities[player] = nil
        print("[Anti Bat] Импульс снят (бита убрана): " .. player.Name)
    end
end

local function pushPlayer(targetPlayer)
    if targetPlayer == LocalPlayer then return end

    local now = tick()
    if recentlyPushed[targetPlayer] and now - recentlyPushed[targetPlayer] < PUSH_COOLDOWN then
        return
    end
    recentlyPushed[targetPlayer] = now

    local char = targetPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    clearVelocity(targetPlayer)

    local myChar = LocalPlayer.Character
    local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
    local direction
    if myHrp then
        direction = (hrp.Position - myHrp.Position).Unit
    else
        direction = Vector3.new(0, 0, 1).Unit
    end

    local bv = Instance.new("BodyVelocity")
    bv.Velocity = direction * PUSH_FORCE + Vector3.new(0, PUSH_UP, 0)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.P = 8000
    bv.Parent = hrp

    activeVelocities[targetPlayer] = bv
    Debris:AddItem(bv, PUSH_DURATION)

    task.delay(PUSH_DURATION, function()
        if activeVelocities[targetPlayer] == bv then
            activeVelocities[targetPlayer] = nil
        end
    end)

    print("[Anti Bat] Отброшен (с битой): " .. targetPlayer.Name)
end

task.spawn(function()
    while true do
        task.wait(CHECK_INTERVAL)

        if antiBatEnabled then
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer then
                    if isHoldingBat(player) then
                        pushPlayer(player)
                    else
                        if activeVelocities[player] then
                            clearVelocity(player)
                        end
                    end
                end
            end
        else
            for player, bv in pairs(activeVelocities) do
                pcall(function() bv:Destroy() end)
            end
            activeVelocities = {}
        end
    end
end)

-- ============================================
-- ЧАСТЬ 4: INFINITE JUMP (Hold Space)
-- ============================================
local infJumpHeld = false
local function updateInfJumpGui() infJumpStatus.Text = infJumpHeld and "HOLD" or "OFF" end

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.Space then infJumpHeld = true; updateInfJumpGui() end
    if input.KeyCode == ANTI_BAT_BIND then
        antiBatEnabled = not antiBatEnabled
        updateAntiBatGui()
        print("[Anti Bat] " .. (antiBatEnabled and "ВКЛЮЧЁН" or "ВЫКЛЮЧЕН"))
        if not antiBatEnabled then
            for player, bv in pairs(activeVelocities) do
                pcall(function() bv:Destroy() end)
            end
            activeVelocities = {}
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Space then infJumpHeld = false; updateInfJumpGui() end
end)

task.spawn(function()
    while true do
        task.wait(0.1)
        if infJumpHeld then
            local c = LocalPlayer.Character
            if c then
                local h = c:FindFirstChildOfClass("Humanoid")
                if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
            end
        end
    end
end)

updateSpeedGui(); updateAntiBatGui(); updateInfJumpGui()
print("[Negor_Anti.bat] Загружен. Q: Speed | G: Anti Bat | Space (hold): Inf Jump")
