--[[
    Auto Farm Pro - Anime Eternal
    Padrão: Compatível com Solara (mesmo estilo do Cyrus loader)
    - UI nativa (Instance.new)
    - Sem bibliotecas externas, sem HTTP, sem syn/gethui
    - Suporte multi-idioma (PT/EN/ES)
--]]

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = Workspace.CurrentCamera

-- ============================
-- TRADUÇÕES
-- ============================
local T = {
    PT = {
        title = "⚔️ AUTO FARM PRO",
        farm = "AUTO FARM",
        gamemode = "GAMEMODES (DUNGEONS)",
        utils = "UTILIDADES",
        refresh = "🔄 Atualizar Lista de Mobs",
        activate = "Ativar Auto Farm",
        enterDungeon = "Auto Entrar em Dungeon",
        killDungeon = "Matar Mobs na Dungeon",
        teleportDungeon = "📍 Teleportar p/ Dungeon",
        tryEnter = "🚪 Tentar Entrar (Prompt/Porta)",
        stopAll = "🛑 Parar Tudo",
        close = "FECHAR",
        status = "Status"
    },
    EN = {
        title = "⚔️ AUTO FARM PRO",
        farm = "AUTO FARM",
        gamemode = "GAMEMODES (DUNGEONS)",
        utils = "UTILITIES",
        refresh = "🔄 Refresh Mob List",
        activate = "Enable Auto Farm",
        enterDungeon = "Auto Enter Dungeon",
        killDungeon = "Kill Mobs in Dungeon",
        teleportDungeon = "📍 Teleport to Dungeon",
        tryEnter = "🚪 Try Enter (Prompt/Door)",
        stopAll = "🛑 Stop All",
        close = "CLOSE",
        status = "Status"
    },
    ES = {
        title = "⚔️ AUTO FARM PRO",
        farm = "AUTO FARM",
        gamemode = "MODOS DE JUEGO (MAZMORRAS)",
        utils = "UTILIDADES",
        refresh = "🔄 Actualizar Lista de Mobs",
        activate = "Activar Auto Farm",
        enterDungeon = "Entrar Auto a Mazmorra",
        killDungeon = "Matar Mobs en Mazmorra",
        teleportDungeon = "📍 Teletransportar a Mazmorra",
        tryEnter = "🚪 Intentar Entrar (Prompt/Puerta)",
        stopAll = "🛑 Detener Todo",
        close = "CERRAR",
        status = "Estado"
    },
}

local function getLang()
    local ok, loc = pcall(function() return LocalPlayer.LocaleId end)
    if ok and loc then
        local code = string.upper(string.sub(loc, 1, 2))
        if code == "PT" then return "PT" end
        if code == "ES" then return "ES" end
    end
    return "EN"
end

local Lang = getLang()
local L = T[Lang]

-- ============================
-- CONFIGURAÇÕES
-- ============================
local Config = {
    AutoFarm = false,
    SelectedMob = nil,
    TeleportDistance = 5,
    AttackCooldown = 0.15,
    StatusText = "Pronto",
}

local DungeonState = {
    Active = false,
    SelectedDungeon = nil,
    AutoKillMobs = true,
    LastAttempt = 0,
    EntryDelay = 2,
    InDungeon = false,
}

local State = {
    LastAttack = 0,
    LastTeleport = 0,
    MobCache = {},
}

-- ============================
-- FUNÇÕES DE MOB
-- ============================
local function isAliveMob(instance)
    if not instance or not instance.Parent then return false end
    if not instance:IsA("Model") then return false end
    local humanoid = instance:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return false end
    if instance == LocalPlayer.Character then return false end
    if Players:GetPlayerFromCharacter(instance) then return false end
    return true, humanoid
end

local function getMobName(mob)
    return (mob.Name:gsub("%s*%(%d+%)$", ""))
end

local function scanMobs()
    local mobTypes = {}
    for _, d in ipairs(Workspace:GetDescendants()) do
        if isAliveMob(d) then
            local n = getMobName(d)
            if not mobTypes[n] then mobTypes[n] = { name = n, count = 0, examples = {} } end
            mobTypes[n].count = mobTypes[n].count + 1
            table.insert(mobTypes[n].examples, d)
        end
    end
    local list = {}
    for _, data in pairs(mobTypes) do table.insert(list, data) end
    table.sort(list, function(a,b) return a.count > b.count end)
    State.MobCache = mobTypes
    return list
end

local function teleportTo(pos)
    local char = LocalPlayer.Character
    if not char then return false end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return false end
    root.CFrame = CFrame.new(pos)
    return true
end

local function attackMob(mob, humanoid)
    if not mob or not mob.PrimaryPart or not humanoid then return end
    if humanoid.Health <= 0 then return end
    local now = tick()
    if now - State.LastAttack < Config.AttackCooldown then return end
    State.LastAttack = now
    local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA("Tool")
    if tool then pcall(function() tool:Activate() end) end
    pcall(function()
        VirtualUser:Button1Down(Vector2.new(0,0))
        task.wait(0.05)
        VirtualUser:Button1Up(Vector2.new(0,0))
    end)
end

-- ============================
-- AUTO FARM LOOP
-- ============================
local function autoFarmLoop()
    while Config.AutoFarm do
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then
            task.wait(0.5) continue
        end
        local myRoot = char.HumanoidRootPart
        local mobData = State.MobCache[Config.SelectedMob]
        if not mobData then
            scanMobs()
            mobData = State.MobCache[Config.SelectedMob]
        end
        if not mobData or #mobData.examples == 0 then
            task.wait(0.5) continue
        end
        local closestMob, closestDist, closestHumanoid = nil, math.huge, nil
        for _, mob in ipairs(mobData.examples) do
            if mob.Parent and isAliveMob(mob) and mob.PrimaryPart then
                local dist = (myRoot.Position - mob.PrimaryPart.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    closestMob = mob
                    closestHumanoid = mob:FindFirstChildOfClass("Humanoid")
                end
            end
        end
        if closestMob then
            local mobPos = closestMob.PrimaryPart.Position
            local dir = (myRoot.Position - mobPos).Unit
            local targetPos = mobPos + dir * Config.TeleportDistance
            targetPos = Vector3.new(targetPos.X, mobPos.Y + 3, targetPos.Z)
            if tick() - State.LastTeleport >= 0.1 then
                teleportTo(targetPos)
                State.LastTeleport = tick()
            end
            attackMob(closestMob, closestHumanoid)
        end
        task.wait(0.05)
    end
end

-- ============================
-- DUNGEONS
-- ============================
local DungeonList = {
    "[Lobby 1] Restaurant Raid",
    "[Lobby 1] Cursed Raid",
    "[Lobby 1] Sin Raid",
    "[Lobby 1] Gleam Raid",
    "[Lobby 1] Progression Raid",
    "[Lobby 1] Leaf Raid",
    "[Lobby 1] Green Planet Raid",
    "[Lobby 2] Maze 1",
    "[Lobby 2] Maze 2",
    "[Lobby 2] Adventurer",
    "[Lobby 2] Torment",
    "[Lobby 2] Hollow Raid",
    "[Timeline 2] Raid",
    "[Timeline 2] Maze",
    "[Timeline 2] Boss",
}

local DungeonTeleportNames = {
    ["Restaurant Raid"] = {"Restaurant","RestaurantRaid"},
    ["Cursed Raid"] = {"Cursed","CursedRaid"},
    ["Sin Raid"] = {"Sin","SinRaid"},
    ["Gleam Raid"] = {"Gleam","GleamRaid"},
    ["Progression Raid"] = {"Progression","ProgressionRaid"},
    ["Leaf Raid"] = {"Leaf","LeafRaid"},
    ["Green Planet Raid"] = {"GreenPlanet","GreenPlanetRaid"},
    ["Maze 1"] = {"Maze1","Maze_1"},
    ["Maze 2"] = {"Maze2","Maze_2"},
    ["Adventurer"] = {"Adventurer"},
    ["Torment"] = {"Torment"},
    ["Hollow Raid"] = {"Hollow","HollowRaid"},
    ["Raid"] = {"Timeline2","Timeline2Raid"},
    ["Maze"] = {"Timeline2Maze"},
    ["Boss"] = {"Timeline2Boss"},
}

local function cleanDungeonName(name)
    return (name:gsub("^%[.-%]%s*", ""))
end

local function teleportToDungeon(dungeonName)
    local terms = DungeonTeleportNames[dungeonName]
    if not terms then return false end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local n = obj.Name:lower()
            for _, t in ipairs(terms) do
                if n:find(t:lower()) then
                    local char = LocalPlayer.Character
                    if char and char:FindFirstChild("HumanoidRootPart") then
                        char.HumanoidRootPart.CFrame = obj.CFrame + Vector3.new(0, 5, 0)
                        return true
                    end
                end
            end
        end
    end
    return false
end

local function isInDungeon()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return false end
    local pos = char.HumanoidRootPart.Position
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Model") or obj:IsA("Folder") then
            local n = obj.Name:lower()
            if n:find("dungeon") or n:find("raid") or n:find("wave") or n:find("maze") then
                local part = obj:FindFirstChildWhichIsA("BasePart", true)
                if part and (pos - part.Position).Magnitude < 500 then
                    return true
                end
            end
        end
    end
    return false
end

local function autoDungeonLoop()
    while DungeonState.Active do
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then
            task.wait(0.5) continue
        end
        local now = tick()
        if now - DungeonState.LastAttempt >= DungeonState.EntryDelay then
            DungeonState.LastAttempt = now
            if not isInDungeon() and DungeonState.SelectedDungeon then
                teleportToDungeon(DungeonState.SelectedDungeon)
                task.wait(1)
                for _, obj in ipairs(Workspace:GetDescendants()) do
                    if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("BasePart") then
                        local d = (char.HumanoidRootPart.Position - obj.Parent.Position).Magnitude
                        if d < 20 then pcall(function() fireproximityprompt(obj) end) end
                    elseif obj:IsA("ClickDetector") and obj.Parent and obj.Parent:IsA("BasePart") then
                        local d = (char.HumanoidRootPart.Position - obj.Parent.Position).Magnitude
                        if d < 20 then pcall(function() fireclickdetector(obj) end) end
                    end
                end
            else
                DungeonState.InDungeon = true
            end
        end
        if DungeonState.InDungeon and DungeonState.AutoKillMobs then
            local myRoot = char.HumanoidRootPart
            local closestMob, closestDist = nil, math.huge
            for _, d in ipairs(Workspace:GetDescendants()) do
                if d:IsA("Model") and d ~= char and isAliveMob(d) and d.PrimaryPart then
                    local dist = (myRoot.Position - d.PrimaryPart.Position).Magnitude
                    if dist < closestDist then closestDist = dist closestMob = d end
                end
            end
            if closestMob then
                local mobPos = closestMob.PrimaryPart.Position
                local dir = (myRoot.Position - mobPos).Unit
                local targetPos = mobPos + dir * 5
                myRoot.CFrame = CFrame.new(targetPos.X, mobPos.Y + 3, targetPos.Z)
                attackMob(closestMob, closestMob:FindFirstChildOfClass("Humanoid"))
            end
        end
        task.wait(0.1)
    end
end

-- ============================
-- UI (mesmo padrão do Cyrus loader)
-- ============================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoFarmPro"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.DisplayOrder = 500
screenGui.Parent = PlayerGui  -- ← mesma coisa que o Cyrus loader faz

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 480, 0, 420)
mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 12)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(80, 150, 255)
mainStroke.Thickness = 2
mainStroke.Transparency = 0.3
mainStroke.Parent = mainFrame

-- Gradient
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(25, 25, 30)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 30, 45))
})
gradient.Rotation = 45
gradient.Parent = mainFrame

-- Top bar
local topBar = Instance.new("Frame")
topBar.Size = UDim2.new(1, 0, 0, 45)
topBar.BackgroundColor3 = Color3.fromRGB(40, 90, 180)
topBar.BorderSizePixel = 0
topBar.Parent = mainFrame

local topCorner = Instance.new("UICorner")
topCorner.CornerRadius = UDim.new(0, 12)
topCorner.Parent = topBar

local topFix = Instance.new("Frame")
topFix.Size = UDim2.new(1, 0, 0, 12)
topFix.Position = UDim2.new(0, 0, 1, -12)
topFix.BackgroundColor3 = Color3.fromRGB(40, 90, 180)
topFix.BorderSizePixel = 0
topFix.Parent = topBar

-- Title
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -70, 1, 0)
titleLabel.Position = UDim2.new(0, 15, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = L.title
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = topBar

-- Botão fechar
local closeBtnTop = Instance.new("TextButton")
closeBtnTop.Size = UDim2.new(0, 30, 0, 30)
closeBtnTop.Position = UDim2.new(1, -38, 0, 8)
closeBtnTop.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtnTop.BorderSizePixel = 0
closeBtnTop.Text = "✕"
closeBtnTop.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtnTop.Font = Enum.Font.GothamBold
closeBtnTop.TextSize = 16
closeBtnTop.Parent = topBar
Instance.new("UICorner", closeBtnTop).CornerRadius = UDim.new(0, 6)

-- Botão minimizar
local minBtnTop = Instance.new("TextButton")
minBtnTop.Size = UDim2.new(0, 30, 0, 30)
minBtnTop.Position = UDim2.new(1, -74, 0, 8)
minBtnTop.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minBtnTop.BorderSizePixel = 0
minBtnTop.Text = "−"
minBtnTop.TextColor3 = Color3.fromRGB(255, 255, 255)
minBtnTop.Font = Enum.Font.GothamBold
minBtnTop.TextSize = 16
minBtnTop.Parent = topBar
Instance.new("UICorner", minBtnTop).CornerRadius = UDim.new(0, 6)

-- Conteúdo
local content = Instance.new("ScrollingFrame")
content.Size = UDim2.new(1, -20, 1, -100)
content.Position = UDim2.new(0, 10, 0, 50)
content.BackgroundTransparency = 1
content.BorderSizePixel = 0
content.ScrollBarThickness = 4
content.ScrollBarImageColor3 = Color3.fromRGB(80, 150, 255)
content.CanvasSize = UDim2.new(0, 0, 0, 0)
content.AutomaticCanvasSize = Enum.AutomaticSize.Y
content.Parent = mainFrame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 6)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = content

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 6)
padding.PaddingBottom = UDim.new(0, 6)
padding.PaddingLeft = UDim.new(0, 4)
padding.PaddingRight = UDim.new(0, 4)
padding.Parent = content

-- Footer / status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -20, 0, 24)
statusLabel.Position = UDim2.new(0, 10, 1, -30)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Status: " .. Config.StatusText
statusLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 12
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.Parent = mainFrame

local function setStatus(txt)
    Config.StatusText = txt
    statusLabel.Text = "Status: " .. txt
end

-- Helpers de UI
local function makeSection(text)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 24)
    lbl.BackgroundTransparency = 1
    lbl.Text = "▸ " .. text
    lbl.TextColor3 = Color3.fromRGB(120, 180, 255)
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 12
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = content
    return lbl
end

local function makeButton(text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 32)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(240, 240, 240)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 13
    btn.AutoButtonColor = false
    btn.Parent = content
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(60, 60, 75)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(45, 45, 55)}):Play()
    end)
    btn.MouseButton1Click:Connect(function()
        local ok, err = pcall(callback)
        if not ok then warn("[AutoFarm] Erro:", err) end
    end)
    return btn
end

local function makeToggle(text, initial, callback)
    local state = initial
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 32)
    btn.BackgroundColor3 = state and Color3.fromRGB(0, 150, 80) or Color3.fromRGB(45, 45, 55)
    btn.BorderSizePixel = 0
    btn.Text = (state and "✔  " or "✘  ") .. text
    btn.TextColor3 = Color3.fromRGB(240, 240, 240)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 13
    btn.AutoButtonColor = false
    btn.Parent = content
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    btn.MouseButton1Click:Connect(function()
        state = not state
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = state and Color3.fromRGB(0, 150, 80) or Color3.fromRGB(45, 45, 55)
        }):Play()
        btn.Text = (state and "✔  " or "✘  ") .. text
        local ok, err = pcall(callback, state)
        if not ok then warn("[AutoFarm] Erro:", err) end
    end)
    return btn
end

-- ============================
-- ABA AUTO FARM
-- ============================
makeSection(L.farm)

makeButton(L.refresh, function()
    local list = scanMobs()
    if #list > 0 then
        Config.SelectedMob = list[1].name
        setStatus("Mob: " .. list[1].name .. " (" .. list[1].count .. ")")
    else
        setStatus("Nenhum mob encontrado")
    end
end)

makeToggle(L.activate, false, function(state)
    if state and not Config.SelectedMob then
        setStatus("Selecione um mob primeiro")
        return
    end
    Config.AutoFarm = state
    if state then
        setStatus("Auto Farm ativo: " .. (Config.SelectedMob or "?"))
        task.spawn(autoFarmLoop)
    else
        setStatus("Auto Farm desativado")
    end
end)

-- ============================
-- ABA GAMEMODES
-- ============================
makeSection(L.gamemode)

local dungeonIndex = 1
DungeonState.SelectedDungeon = cleanDungeonName(DungeonList[1])

local dungeonBtn = makeButton("🎯 " .. DungeonList[dungeonIndex] .. " (clique p/ mudar)", function()
    dungeonIndex = dungeonIndex % #DungeonList + 1
    dungeonBtn.Text = "🎯 " .. DungeonList[dungeonIndex] .. " (clique p/ mudar)"
    DungeonState.SelectedDungeon = cleanDungeonName(DungeonList[dungeonIndex])
    setStatus("Dungeon: " .. DungeonState.SelectedDungeon)
end)

makeToggle(L.enterDungeon, false, function(state)
    DungeonState.Active = state
    if state then
        task.spawn(autoDungeonLoop)
        setStatus("Auto Dungeon ativo")
    else
        setStatus("Auto Dungeon desativado")
    end
end)

makeToggle(L.killDungeon, true, function(state)
    DungeonState.AutoKillMobs = state
end)

makeButton(L.teleportDungeon, function()
    if DungeonState.SelectedDungeon then
        if teleportToDungeon(DungeonState.SelectedDungeon) then
            setStatus("Teleportado: " .. DungeonState.SelectedDungeon)
        else
            setStatus("Dungeon não encontrada")
        end
    end
end)

makeButton(L.tryEnter, function()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("BasePart") then
            local d = (char.HumanoidRootPart.Position - obj.Parent.Position).Magnitude
            if d < 25 then pcall(function() fireproximityprompt(obj) end) end
        elseif obj:IsA("ClickDetector") and obj.Parent and obj.Parent:IsA("BasePart") then
            local d = (char.HumanoidRootPart.Position - obj.Parent.Position).Magnitude
            if d < 25 then pcall(function() fireclickdetector(obj) end) end
        end
    end
    setStatus("Tentando entrar...")
end)

-- ============================
-- UTILIDADES
-- ============================
makeSection(L.utils)

makeButton(L.stopAll, function()
    Config.AutoFarm = false
    DungeonState.Active = false
    setStatus("Tudo parado")
end)

-- ============================
-- MINIMIZAR / FECHAR
-- ============================
local minimized = false
minBtnTop.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        content.Visible = false
        statusLabel.Visible = false
        minBtnTop.Text = "+"
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 480, 0, 45)}):Play()
    else
        content.Visible = true
        statusLabel.Visible = true
        minBtnTop.Text = "−"
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 480, 0, 420)}):Play()
    end
end)

closeBtnTop.MouseButton1Click:Connect(function()
    Config.AutoFarm = false
    DungeonState.Active = false
    TweenService:Create(mainFrame, TweenInfo.new(0.3), {Size = UDim2.new(0, 0, 0, 0)}):Play()
    task.wait(0.35)
    screenGui:Destroy()
end)

-- ============================
-- ANIMAÇÃO DE ENTRADA
-- ============================
mainFrame.Size = UDim2.new(0, 0, 0, 0)
TweenService:Create(mainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 480, 0, 420)
}):Play()

print("[AutoFarm Pro] Carregado! Idioma: " .. Lang)
