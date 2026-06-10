# Roblox-public


-- RAGEBOT V1 – WORKING SCREENGUI + SUBSPACE TRIPMINE & SATCHEL DETECTION (40 STUD)
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- ========== UI (EXACTLY AS LAST WORKING SCREENGUI) ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RagebotUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = CoreGui or LocalPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 210)
mainFrame.Position = UDim2.new(0.5, -130, 0.5, -105)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
mainFrame.BackgroundTransparency = 0.15
mainFrame.BorderSizePixel = 1
mainFrame.BorderColor3 = Color3.fromRGB(255, 50, 50)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local titleBar = Instance.new("TextLabel")
titleBar.Size = UDim2.new(1, 0, 0, 25)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
titleBar.Text = "  RAGEBOT V1"
titleBar.TextXAlignment = Enum.TextXAlignment.Left
titleBar.TextColor3 = Color3.fromRGB(255, 100, 100)
titleBar.Font = Enum.Font.GothamBold
titleBar.TextSize = 13
titleBar.Parent = mainFrame

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 25, 1, 0)
closeBtn.Position = UDim2.new(1, -25, 0, 0)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.Parent = titleBar
closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 100, 0, 35)
toggleBtn.Position = UDim2.new(0.5, -50, 0, 120)
toggleBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
toggleBtn.Text = "OFF"
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 18
toggleBtn.Parent = mainFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -10, 0, 20)
statusLabel.Position = UDim2.new(0, 5, 0, 35)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "● Idle"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 11
statusLabel.TextXAlignment = Enum.TextXAlignment.Center
statusLabel.Parent = mainFrame

local velLabel = Instance.new("TextLabel")
velLabel.Size = UDim2.new(1, -10, 0, 16)
velLabel.Position = UDim2.new(0, 5, 0, 55)
velLabel.BackgroundTransparency = 1
velLabel.Text = "📊 Velocity: --"
velLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
velLabel.Font = Enum.Font.Gotham
velLabel.TextSize = 10
velLabel.TextXAlignment = Enum.TextXAlignment.Left
velLabel.Parent = mainFrame

local attackLabel = Instance.new("TextLabel")
attackLabel.Size = UDim2.new(1, -10, 0, 16)
attackLabel.Position = UDim2.new(0, 5, 0, 72)
attackLabel.BackgroundTransparency = 1
attackLabel.Text = "🔫 Holding Attack: ❌"
attackLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
attackLabel.Font = Enum.Font.Gotham
attackLabel.TextSize = 10
attackLabel.TextXAlignment = Enum.TextXAlignment.Left
attackLabel.Parent = mainFrame

local fireLabel = Instance.new("TextLabel")
fireLabel.Size = UDim2.new(1, -10, 0, 16)
fireLabel.Position = UDim2.new(0, 5, 0, 89)
fireLabel.BackgroundTransparency = 1
fireLabel.Text = "🔥 Near Fire: ❌"
fireLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
fireLabel.Font = Enum.Font.Gotham
fireLabel.TextSize = 10
fireLabel.TextXAlignment = Enum.TextXAlignment.Left
fireLabel.Parent = mainFrame

local tripmineLabel = Instance.new("TextLabel")
tripmineLabel.Size = UDim2.new(1, -10, 0, 16)
tripmineLabel.Position = UDim2.new(0, 5, 0, 106)
tripmineLabel.BackgroundTransparency = 1
tripmineLabel.Text = "⚠️ Tripmine: ❌"
tripmineLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
tripmineLabel.Font = Enum.Font.Gotham
tripmineLabel.TextSize = 10
tripmineLabel.TextXAlignment = Enum.TextXAlignment.Left
tripmineLabel.Parent = mainFrame

local satchelLabel = Instance.new("TextLabel")
satchelLabel.Size = UDim2.new(1, -10, 0, 16)
satchelLabel.Position = UDim2.new(0, 5, 0, 123)
satchelLabel.BackgroundTransparency = 1
satchelLabel.Text = "💣 Satchel: ❌"
satchelLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
satchelLabel.Font = Enum.Font.Gotham
satchelLabel.TextSize = 10
satchelLabel.TextXAlignment = Enum.TextXAlignment.Left
satchelLabel.Parent = mainFrame

-- ========== STATE ==========
local isActive = false
local currentTarget = nil
local runningCoroutine = nil
local noFireRateCoroutine = nil
local originalFireRates = {}
local damageOverrideEndTime = 0

-- ========== AUTO-FIRE ==========
local function autoFireTool()
    local char = LocalPlayer.Character
    if not char then return end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool and tool:IsA("Tool") then
        pcall(function() tool:Activate() end)
    end
end

-- ========== NO FIRE RATE ==========
local function applyNoFireRateToTool(tool)
    if not tool or not tool:IsA("Tool") then return end
    if originalFireRates[tool] == nil then
        originalFireRates[tool] = tool:GetAttribute("FireRate") or tool.FireRate or 0.2
    end
    pcall(function()
        tool.FireRate = 0
        tool:SetAttribute("FireRate", 0)
    end)
    if not tool._noFireRateHooked then
        tool._noFireRateHooked = true
        local oldActivated = tool.Activated
        tool.Activated = function(...)
            if isActive then
                if oldActivated then oldActivated(...) end
                pcall(function() tool.LastFire = 0; tool.Cooldown = 0 end)
            else
                if oldActivated then oldActivated(...) end
            end
        end
    end
end

local function restoreOriginalFireRate(tool)
    if originalFireRates[tool] then
        pcall(function()
            tool.FireRate = originalFireRates[tool]
            tool:SetAttribute("FireRate", originalFireRates[tool])
        end)
        originalFireRates[tool] = nil
    end
end

local function updateAllTools()
    local char = LocalPlayer.Character
    if not char then return end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") then
            if isActive then applyNoFireRateToTool(tool) else restoreOriginalFireRate(tool) end
        end
    end
end

local toolListener = nil
local function startNoFireRate()
    if noFireRateCoroutine then return end
    updateAllTools()
    noFireRateCoroutine = coroutine.create(function()
        while isActive do updateAllTools(); task.wait(0.5) end
        noFireRateCoroutine = nil
    end)
    coroutine.resume(noFireRateCoroutine)
    if not toolListener then
        toolListener = LocalPlayer.Character.ChildAdded:Connect(function(child)
            if child:IsA("Tool") and isActive then applyNoFireRateToTool(child) end
        end)
    end
end

local function stopNoFireRate()
    if noFireRateCoroutine then coroutine.close(noFireRateCoroutine); noFireRateCoroutine = nil end
    updateAllTools()
    if toolListener then toolListener:Disconnect(); toolListener = nil end
end

-- ========== DETECTION FUNCTIONS (40 STUD RADIUS, DEX++ NAMES) ==========
local function isEnemyReloading(e) return false end
local function isEnemyAttacking(e) return false end
local function isEnemyUsingKatanaAbility(e) return false end
local function isEnemyHoldingAttack(e) return false end
local function isEnemyUsingMelee(e) return false end

local function isEnemyInMolotovFire(e)
    local head = e:FindFirstChild("Head")
    if not head then return false end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local name = obj.Name:lower()
            if name:find("fire") or name:find("molotov") then
                if (obj.Position - head.Position).Magnitude < 5 then return true end
            end
        end
    end
    return false
end

-- Satchel detection (exact name "Satchel" from DEX++, 40 studs)
local function isSatchelNearEnemy(e)
    local root = e:FindFirstChild("HumanoidRootPart")
    if not root then return false end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("Model") then
            local name = obj.Name
            if name == "Satchel" or name:lower():find("satchel") then
                local pos = obj:IsA("BasePart") and obj.Position or (obj:FindFirstChild("PrimaryPart") and obj.PrimaryPart.Position)
                if pos and (pos - root.Position).Magnitude < 40 then return true end
            end
        end
    end
    return false
end

-- Tripmine detection (DEX++ names: subspectripmine_inspect, Chibi Subspace Tripmine, 40 studs)
local function isSubspaceTripmineNearEnemy(e)
    local root = e:FindFirstChild("HumanoidRootPart")
    if not root then return false end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") or obj:IsA("Model") then
            local name = obj.Name
            if name == "subspectripmine_inspect" or name == "Chibi Subspace Tripmine" or name:lower():find("tripmine") then
                local pos = obj:IsA("BasePart") and obj.Position or (obj:FindFirstChild("PrimaryPart") and obj.PrimaryPart.Position)
                if pos and (pos - root.Position).Magnitude < 40 then return true end
            end
        end
    end
    return false
end

local function isDangerousProjectileNearby()
    local char = LocalPlayer.Character
    if not char then return false end
    local myPos = char:FindFirstChild("HumanoidRootPart") and char.HumanoidRootPart.Position or Vector3.new()
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Parent and not obj.Parent:IsA("Character") then
            local name = obj.Name:lower()
            if name:find("rpg") or name:find("rocket") or name:find("grenade") then
                if (obj.Position - myPos).Magnitude < 50 then return true end
            end
        end
    end
    return false
end

local function isUserReloading()
    local char = LocalPlayer.Character
    if not char then return false end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool and tool:GetAttribute("Reloading") then return true end
    local animator = char:FindFirstChild("Animator")
    if animator then
        for _, track in pairs(animator:GetPlayingAnimationTracks()) do
            if track.Animation and track.Animation.Name:lower():find("reload") then return true end
        end
    end
    return false
end

-- ========== UPDATE INDICATORS ==========
local function updateIndicators(enemyChar)
    if not enemyChar then
        velLabel.Text = "📊 Velocity: --"
        attackLabel.Text = "🔫 Holding Attack: ❌"
        fireLabel.Text = "🔥 Near Fire: ❌"
        tripmineLabel.Text = "⚠️ Tripmine: ❌"
        satchelLabel.Text = "💣 Satchel: ❌"
        return
    end
    local enemyRoot = enemyChar:FindFirstChild("HumanoidRootPart")
    local vel = enemyRoot and enemyRoot.Velocity.Magnitude or 0
    velLabel.Text = string.format("📊 Velocity: %.1f", vel)
    local holding = isEnemyHoldingAttack(enemyChar)
    attackLabel.Text = holding and "🔫 Holding Attack: ✅" or "🔫 Holding Attack: ❌"
    local fire = isEnemyInMolotovFire(enemyChar)
    fireLabel.Text = fire and "🔥 Near Fire: ✅" or "🔥 Near Fire: ❌"
    local tripmine = isSubspaceTripmineNearEnemy(enemyChar)
    tripmineLabel.Text = tripmine and "⚠️ Tripmine: ✅ (15 height)" or "⚠️ Tripmine: ❌"
    local satchel = isSatchelNearEnemy(enemyChar)
    satchelLabel.Text = satchel and "💣 Satchel: ✅ (10 height)" or "💣 Satchel: ❌"
end

-- ========== TELEPORT ==========
local function teleportToPosition(pos)
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then root.CFrame = CFrame.new(pos) end
end

local function emergencyTeleportAway()
    if not currentTarget or not currentTarget.Character then return end
    local er = currentTarget.Character:FindFirstChild("HumanoidRootPart")
    if not er then return end
    local rd = Vector3.new(math.random()-0.5, math.random()-0.5, math.random()-0.5).Unit
    teleportToPosition(er.Position + rd * 10000)
end

-- ========== HEIGHT LOGIC (with new hazard detection) ==========
local function getAttackHeight(enemyChar, patternStep)
    if isSubspaceTripmineNearEnemy(enemyChar) then
        return 15, "TRIPMINE 15"
    end
    if tick() < damageOverrideEndTime then
        return 14, "DAMAGE 14"
    end
    if isSatchelNearEnemy(enemyChar) then
        return 10, "SATCHEL 10"
    end
    local er = enemyChar:FindFirstChild("HumanoidRootPart")
    if isEnemyInMolotovFire(enemyChar) or (isEnemyHoldingAttack(enemyChar) and er and er.Velocity.Magnitude < 15.5) or isEnemyUsingMelee(enemyChar) then
        return 7.5, "HAZARD 7.5"
    end
    if isEnemyUsingMelee(enemyChar) and isEnemyHoldingAttack(enemyChar) then
        return 8.5, "MELEE+ATTACK 8.5"
    end
    local pattern = {0.3, 0.6, 1.0}
    local idx = (patternStep % 3) + 1
    return pattern[idx], string.format("PATTERN %.1f", pattern[idx])
end

local function getPredictedPosition(enemyChar, height)
    local head = enemyChar:FindFirstChild("Head")
    if not head then return nil end
    local root = enemyChar:FindFirstChild("HumanoidRootPart")
    local dir = Vector3.new(0,0,0)
    if root and root.Velocity then
        local horiz = Vector3.new(root.Velocity.X, 0, root.Velocity.Z)
        if horiz.Magnitude > 0.1 then dir = horiz.Unit end
    end
    return head.Position + dir * 1.0 + Vector3.new(0, height, 0)
end

-- ========== RAGING ATTACK ==========
local function ragingAttack(enemyChar)
    local start = tick()
    local patternStep = 0
    local currentHealth = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") and LocalPlayer.Character.Humanoid.Health or 100
    local fireCoroutine = coroutine.create(function()
        while tick() - start < 0.2 and isActive do
            autoFireTool()
            task.wait(0.02)
        end
    end)
    coroutine.resume(fireCoroutine)
    local interval = 1 / 60
    while tick() - start < 0.2 and isActive and enemyChar and enemyChar.Parent do
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
        if hum and hum.Health < currentHealth then
            damageOverrideEndTime = tick() + 1.0
            statusLabel.Text = "💔 Damage! Height 14 for 1s"
            currentHealth = hum.Health
        end
        local height, reason = getAttackHeight(enemyChar, patternStep)
        local pos = getPredictedPosition(enemyChar, height)
        if pos then teleportToPosition(pos) end
        statusLabel.Text = "🔥 " .. reason
        patternStep = patternStep + 1
        task.wait(interval)
    end
end

-- ========== ORBIT WITH JITTER ==========
local function orbitWithJitter(enemyRoot, enemyChar)
    local angle = 0
    local lastCheck = tick()
    local orbitStart = tick()
    while isActive and enemyRoot and enemyRoot.Parent do
        updateIndicators(enemyChar)
        if isDangerousProjectileNearby() then
            emergencyTeleportAway()
            task.wait(0.2)
            orbitStart = tick()
        end
        if tick() - orbitStart >= 5 then
            statusLabel.Text = "⏰ Orbit timeout -> attacking"
            return "timeout"
        end
        if tick() - lastCheck >= 0.1 then
            lastCheck = tick()
            local velMag = enemyRoot.Velocity.Magnitude
            local attacking = isEnemyAttacking(enemyChar)
            local katana = isEnemyUsingKatanaAbility(enemyChar)
            local reloading = isEnemyReloading(enemyChar)
            if (velMag < 20 or reloading) and not attacking and not katana then
                statusLabel.Text = "⚡ Condition met -> exiting orbit"
                return "condition"
            end
        end
        angle = angle + 0.2
        local x = math.cos(angle) * 3000
        local z = math.sin(angle) * 3000
        local jx = (math.random() - 0.5) * 10
        local jz = (math.random() - 0.5) * 10
        local pos = enemyRoot.Position + Vector3.new(x + jx, 50, z + jz)
        teleportToPosition(pos)
        statusLabel.Text = "🌀 ORBIT + JITTER"
        task.wait(0.01)
    end
    return nil
end

-- ========== NEAREST ENEMY ==========
local function getNearestEnemy()
    local nearest, best = nil, math.huge
    local myChar = LocalPlayer.Character
    if not myChar then return nil end
    local myPos = myChar:FindFirstChild("HumanoidRootPart") and myChar.HumanoidRootPart.Position or Vector3.new()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local r = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChild("Humanoid")
            if r and hum and hum.Health > 0 then
                local d = (r.Position - myPos).Magnitude
                if d < best then best = d; nearest = plr end
            end
        end
    end
    return nearest
end

-- ========== MAIN CYCLE ==========
local function mainCycle()
    while isActive do
        currentTarget = getNearestEnemy()
        if not currentTarget or not currentTarget.Character then
            statusLabel.Text = "⚠️ No enemy"
            updateIndicators(nil)
            task.wait(0.5)
            continue
        end
        local enemyChar = currentTarget.Character
        local enemyRoot = enemyChar:FindFirstChild("HumanoidRootPart")
        if not enemyRoot then task.wait(0.2) continue end
        updateIndicators(enemyChar)
        local reloading = isEnemyReloading(enemyChar)
        if reloading then
            local rd = Vector3.new(math.random()-0.5, math.random()-0.5, math.random()-0.5).Unit
            pcall(function() enemyRoot.Velocity = rd * 15.5 end)
            statusLabel.Text = "⚡ Reload -> attack"
            ragingAttack(enemyChar)
        elseif isUserReloading() then
            statusLabel.Text = "🔁 User reload -> orbit"
            local result = orbitWithJitter(enemyRoot, enemyChar)
            if result == "timeout" then ragingAttack(enemyChar) end
        else
            local velMag = enemyRoot.Velocity.Magnitude
            local attacking = isEnemyAttacking(enemyChar)
            local katana = isEnemyUsingKatanaAbility(enemyChar)
            if (velMag < 20) and not attacking and not katana then
                statusLabel.Text = "🔥 ATTACKING"
                ragingAttack(enemyChar)
            else
                local result = orbitWithJitter(enemyRoot, enemyChar)
                if result == "timeout" then ragingAttack(enemyChar) end
            end
        end
        task.wait(0.05)
    end
end

-- ========== START/STOP ==========
local function start()
    if runningCoroutine then coroutine.close(runningCoroutine) end
    isActive = true
    runningCoroutine = coroutine.create(mainCycle)
    coroutine.resume(runningCoroutine)
    startNoFireRate()
    toggleBtn.Text = "ON"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(100,200,100)
    statusLabel.Text = "🔍 ACTIVE"
end

local function stop()
    isActive = false
    if runningCoroutine then coroutine.close(runningCoroutine) end
    runningCoroutine = nil
    currentTarget = nil
    stopNoFireRate()
    damageOverrideEndTime = 0
    toggleBtn.Text = "OFF"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(70,70,90)
    sta