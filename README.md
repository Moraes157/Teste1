getgenv().TargetPetNames = {
    "Graipuss Medussi",
    "Trenostruzza Turbo 3000",
    "Odin Din Din Dun",
    "Los Tralaleritos",
    "Sammyni Spyderini",
    "Unclito Samito",
    "Garama and Madundung",
    "Matteo",
    "La Vacca Saturno Saturnita",
    "La Grande Combinasione"
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local targetPets = getgenv().TargetPetNames

local hops = 0
local visitedJobIds = {[game.JobId] = true}
local stopHopping = false
local teleportFails = 0
local maxTeleportRetries = 3

local detectedPets = {}
local petBillboards = {}

-- ========== SERVERHOP E PET ESP ==========
function serverHop(force)
    if stopHopping and not force then return end
    local PlaceId, JobId = game.PlaceId, game.JobId
    local attempt, foundServer = 0, false
    local maxAttempts = 15
    while not foundServer and attempt < maxAttempts do
        attempt += 1
        task.wait(0.5)
        local cursor = nil
        local pageTries = 0
        while pageTries < 5 do
            pageTries += 1
            local url = "https://games.roblox.com/v1/games/" .. PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
            if cursor then url ..= "&cursor=" .. cursor end
            local httpSuccess, response = pcall(function()
                return HttpService:JSONDecode(game:HttpGet(url))
            end)
            if httpSuccess and response and response.data then
                for _, server in ipairs(response.data) do
                    if tonumber(server.playing or 0) < tonumber(server.maxPlayers or 1)
                        and server.id ~= JobId
                        and not visitedJobIds[server.id] then
                        visitedJobIds[server.id] = true
                        hops += 1
                        if hops >= 50 then
                            visitedJobIds = {[JobId] = true}
                            hops = 0
                        end
                        TeleportService:TeleportToPlaceInstance(PlaceId, server.id)
                        return
                    end
                end
                cursor = response.nextPageCursor
                if not cursor then break end
            else
                task.wait(0.2)
            end
        end
    end
    TeleportService:Teleport(PlaceId)
end

TeleportService.TeleportInitFailed:Connect(function(_, result)
    teleportFails += 1
    if teleportFails >= maxTeleportRetries then
        teleportFails = 0
        task.wait(1)
        TeleportService:Teleport(game.PlaceId)
    else
        task.wait(0.5)
        serverHop(true)
    end
end)

local function addPetBillboard(model)
    if not model or model:FindFirstChild("PetESP") then return end
    local Billboard = Instance.new("BillboardGui")
    Billboard.Name = "PetESP"
    Billboard.Adornee = model
    Billboard.Size = UDim2.new(0, 120, 0, 40)
    Billboard.StudsOffset = Vector3.new(0, 4, 0)
    Billboard.AlwaysOnTop = true
    Billboard.Parent = model
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = "ðŸµ " .. model.Name
    Label.TextColor3 = Color3.fromRGB(255, 0, 0)
    Label.TextStrokeTransparency = 0.4
    Label.Font = Enum.Font.GothamBold
    Label.TextScaled = true
    Label.Parent = Billboard
    petBillboards[model] = Billboard
end

local function checkForPets()
    local found = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name then
            for _, target in pairs(targetPets) do
                if string.lower(obj.Name):find(string.lower(target)) and not obj:FindFirstChild("PetESP") then
                    addPetBillboard(obj)
                    table.insert(found, obj.Name)
                    stopHopping = true
                    break
                end
            end
        end
    end
    return found
end

workspace.DescendantAdded:Connect(function(obj)
    task.wait(0.1)
    if obj:IsA("Model") and obj.Name then
        for _, target in pairs(targetPets) do
            if string.lower(obj.Name):find(string.lower(target)) and not obj:FindFirstChild("PetESP") then
                if not detectedPets[obj.Name] then
                    detectedPets[obj.Name] = true
                    addPetBillboard(obj)
                    stopHopping = true
                end
                break
            end
        end
    end
end)
-- ========== FIM SERVERHOP E PET ESP ==========

pcall(function()
    local old = LocalPlayer.PlayerGui:FindFirstChild("DeltaMainGUI")
    if old then old:Destroy() end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "DeltaMainGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = LocalPlayer.PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 272)
frame.Position = UDim2.new(0.5, -110, 0.61, 0)
frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 18)
corner.Parent = frame

-- Content Frames por aba
local tabFrames = {}
for tab = 1, 3 do
    local tabFrame = Instance.new("Frame")
    tabFrame.Name = "TabFrame" .. tab
    tabFrame.Size = UDim2.new(1, 0, 1, -45)
    tabFrame.Position = UDim2.new(0, 0, 0, 30)
    tabFrame.BackgroundTransparency = 1
    tabFrame.Visible = (tab == 1)
    tabFrame.Parent = frame
    tabFrames[tab] = tabFrame
end

-- TÃ­tulo: FAST HUB em vermelho
local topTitle = Instance.new("TextLabel")
topTitle.Size = UDim2.new(1, 0, 0, 28)
topTitle.Position = UDim2.new(0, 0, 0, 0)
topTitle.BackgroundTransparency = 1
topTitle.Text = "FAST HUB"
topTitle.Font = Enum.Font.GothamBlack
topTitle.TextSize = 24
topTitle.TextColor3 = Color3.fromRGB(255, 40, 40)
topTitle.Parent = frame

-- Nome da aba "Speed"
local speedTitle = Instance.new("TextLabel")
speedTitle.Size = UDim2.new(1, 0, 0, 20)
speedTitle.Position = UDim2.new(0, 0, 0, 25)
speedTitle.BackgroundTransparency = 1
speedTitle.Text = "Speed"
speedTitle.Font = Enum.Font.GothamBold
speedTitle.TextSize = 17
speedTitle.TextColor3 = Color3.fromRGB(60, 220, 255)
speedTitle.Parent = tabFrames[1]

-- BotÃ£o SPEED (toggle)
local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(1, -32, 0, 40)
speedBtn.Position = UDim2.new(0, 16, 0, 50-30)
speedBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
speedBtn.Text = "ATIVAR SPEED"
speedBtn.Font = Enum.Font.GothamMedium
speedBtn.TextSize = 19
speedBtn.TextColor3 = Color3.fromRGB(255,255,255)
speedBtn.AutoButtonColor = true
speedBtn.Parent = tabFrames[1]

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 12)
btnCorner.Parent = speedBtn

local speedActive = false
local speedConnection

local baseSpeed = 16
local desiredSpeed = 48
local multiplier = (desiredSpeed/baseSpeed) - 1
local microStep = 0.23

local function setSpeed(state)
    if speedActive == state then return end
    speedActive = state
    if speedActive then
        if speedConnection then pcall(function() speedConnection:Disconnect() end) end
        speedConnection = RunService.Heartbeat:Connect(function()
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hum and hrp and hum.MoveDirection.Magnitude > 0 then
                local direction = hum.MoveDirection.Unit
                local extra = direction * microStep * multiplier
                hrp.CFrame = hrp.CFrame + extra
            end
        end)
        speedBtn.Text = "DESATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(200, 65, 65)
    else
        if speedConnection then pcall(function() speedConnection:Disconnect() end) speedConnection = nil end
        speedBtn.Text = "ATIVAR SPEED"
        speedBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
    end
end

speedBtn.MouseButton1Click:Connect(function()
    setSpeed(not speedActive)
end)

LocalPlayer.CharacterAdded:Connect(function()
    setSpeed(false)
end)

-- BotÃ£o TP (toggle: barreira <-> chÃ£o)
local tpBtn = Instance.new("TextButton")
tpBtn.Size = UDim2.new(1, -32, 0, 40)
tpBtn.Position = UDim2.new(0, 16, 0, 100-30)
tpBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
tpBtn.Text = "TP (PARA BARREIRA)"
tpBtn.Font = Enum.Font.GothamMedium
tpBtn.TextSize = 17
tpBtn.TextColor3 = Color3.fromRGB(255,255,255)
tpBtn.AutoButtonColor = true
tpBtn.Parent = tabFrames[1]

local tpCorner = Instance.new("UICorner")
tpCorner.CornerRadius = UDim.new(0, 12)
tpCorner.Parent = tpBtn

local tpNaBarreira = false
local alturaBarreira = 160
local alturaChao = 3

tpBtn.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        if not tpNaBarreira then
            hrp.CFrame = CFrame.new(hrp.Position.X, alturaBarreira, hrp.Position.Z)
            tpNaBarreira = true
            tpBtn.Text = "TP (VOLTA CHÃƒO)"
            tpBtn.BackgroundColor3 = Color3.fromRGB(200, 65, 65)
        else
            hrp.CFrame = CFrame.new(hrp.Position.X, alturaChao, hrp.Position.Z)
            tpNaBarreira = false
            tpBtn.Text = "TP (PARA BARREIRA)"
            tpBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
        end
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    tpNaBarreira = false
    tpBtn.Text = "TP (PARA BARREIRA)"
    tpBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
end)

-- BotÃ£o SERVER HOP
local serverHopBtn = Instance.new("TextButton")
serverHopBtn.Size = UDim2.new(1, -32, 0, 40)
serverHopBtn.Position = UDim2.new(0, 16, 0, 150-30)
serverHopBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
serverHopBtn.Text = "ATIVAR SERVER HOP"
serverHopBtn.Font = Enum.Font.GothamMedium
serverHopBtn.TextSize = 17
serverHopBtn.TextColor3 = Color3.fromRGB(255,255,255)
serverHopBtn.AutoButtonColor = true
serverHopBtn.Parent = tabFrames[1]

local shCorner = Instance.new("UICorner")
shCorner.CornerRadius = UDim.new(0, 12)
shCorner.Parent = serverHopBtn

local serverHopActive = false

serverHopBtn.MouseButton1Click:Connect(function()
    serverHopActive = not serverHopActive
    if serverHopActive then
        serverHopBtn.Text = "DESATIVAR SERVER HOP"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(200, 65, 65)
        stopHopping = false
        serverHop(true)
    else
        serverHopBtn.Text = "ATIVAR SERVER HOP"
        serverHopBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
        stopHopping = true
    end
end)

-- ========== ABA 2: ESP PLAYER + ESP NAME ==========
local espFrame = tabFrames[2]

-- ESP Player (Highlight)
local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(1, -32, 0, 36)
espBtn.Position = UDim2.new(0, 16, 0, 24)
espBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
espBtn.Text = "ESP Player"
espBtn.Font = Enum.Font.GothamMedium
espBtn.TextSize = 16
espBtn.TextColor3 = Color3.fromRGB(255,255,255)
espBtn.AutoButtonColor = true
espBtn.Parent = espFrame

local espBtnCorner = Instance.new("UICorner")
espBtnCorner.CornerRadius = UDim.new(0, 12)
espBtnCorner.Parent = espBtn

local espActive = false
local playerESP_Highlights = {}

local function removeAllESP()
    for player, highlight in pairs(playerESP_Highlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    playerESP_Highlights = {}
end

local function enableESPPlayers()
    removeAllESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local char = player.Character
            local highlight = Instance.new("Highlight")
            highlight.Adornee = char
            highlight.FillColor = Color3.fromRGB(255,0,0)
            highlight.FillTransparency = 0.8
            highlight.OutlineColor = Color3.fromRGB(255,0,0)
            highlight.OutlineTransparency = 0.6
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Parent = char
            playerESP_Highlights[player] = highlight
        end
    end
end

espBtn.MouseButton1Click:Connect(function()
    espActive = not espActive
    if espActive then
        espBtn.Text = "Desligar ESP Player"
        espBtn.BackgroundColor3 = Color3.fromRGB(200, 65, 65)
        enableESPPlayers()
    else
        espBtn.Text = "ESP Player"
        espBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
        removeAllESP()
    end
end)

Players.PlayerAdded:Connect(function(player)
    if espActive then
        player.CharacterAdded:Connect(function()
            task.wait(0.3)
            enableESPPlayers()
        end)
    end
end)
Players.PlayerRemoving:Connect(function(player)
    if playerESP_Highlights[player] then
        playerESP_Highlights[player]:Destroy()
        playerESP_Highlights[player] = nil
    end
end)
LocalPlayer.CharacterAdded:Connect(function()
    if espActive then
        task.wait(0.3)
        enableESPPlayers()
    end
end)

-- ESP Name
local espNameBtn = Instance.new("TextButton")
espNameBtn.Size = UDim2.new(1, -32, 0, 36)
espNameBtn.Position = UDim2.new(0, 16, 0, 72)
espNameBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
espNameBtn.Text = "ESP Name"
espNameBtn.Font = Enum.Font.GothamMedium
espNameBtn.TextSize = 16
espNameBtn.TextColor3 = Color3.fromRGB(255,255,255)
espNameBtn.AutoButtonColor = true
espNameBtn.Parent = espFrame

local espNameBtnCorner = Instance.new("UICorner")
espNameBtnCorner.CornerRadius = UDim.new(0, 12)
espNameBtnCorner.Parent = espNameBtn

local espNameActive = false
local playerBillboardNames = {}

local function removeAllBillboardNames()
    for player, gui in pairs(playerBillboardNames) do
        if gui and gui.Parent then
            gui:Destroy()
        end
    end
    playerBillboardNames = {}
end

local function enableESPNames()
    removeAllBillboardNames()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local bb = Instance.new("BillboardGui")
            bb.Name = "ESPName"
            bb.Adornee = head
            bb.Size = UDim2.new(0, 110, 0, 18)
            bb.StudsOffset = Vector3.new(0, 2.3, 0)
            bb.AlwaysOnTop = true
            bb.Parent = head

            local txt = Instance.new("TextLabel")
            txt.Size = UDim2.new(1, 0, 1, 0)
            txt.BackgroundTransparency = 1
            txt.Text = player.Name
            txt.TextColor3 = Color3.fromRGB(255, 30, 30)
            txt.TextTransparency = 0.25 -- vermelho bem fraco
            txt.Font = Enum.Font.GothamSemibold
            txt.TextStrokeTransparency = 0.8
            txt.TextScaled = true
            txt.Parent = bb

            playerBillboardNames[player] = bb
        end
    end
end

espNameBtn.MouseButton1Click:Connect(function()
    espNameActive = not espNameActive
    if espNameActive then
        espNameBtn.Text = "Desligar ESP Name"
        espNameBtn.BackgroundColor3 = Color3.fromRGB(200, 65, 65)
        enableESPNames()
    else
        espNameBtn.Text = "ESP Name"
        espNameBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 60)
        removeAllBillboardNames()
    end
end)

Players.PlayerAdded:Connect(function(player)
    if espNameActive then
        player.CharacterAdded:Connect(function()
            task.wait(0.3)
            enableESPNames()
        end)
    end
end)
Players.PlayerRemoving:Connect(function(player)
    if playerBillboardNames[player] then
        playerBillboardNames[player]:Destroy()
        playerBillboardNames[player] = nil
    end
end)
LocalPlayer.CharacterAdded:Connect(function()
    if espNameActive then
        task.wait(0.3)
        enableESPNames()
    end
end)

-- ========== ABAS PEQUENAS REDONDAS LADO A LADO ==========
local abaNomes = {"MAIN","MISC","OUTROS"}
local abas = {}
local abaSelecionada = 1
local corBranco = Color3.fromRGB(255,255,255)
local corSelecionado = Color3.fromRGB(222,222,222)

local totalSpace = 220 -- frame width
local btnPadding = 10
local btnSize = 32
local totalBtnW = btnSize * 3 + btnPadding * 2
local marginLeft = math.floor((totalSpace - totalBtnW)/2)

for i=1,3 do
    local btn = Instance.new("TextButton")
    btn.Name = "AbaBtn"..i
    btn.Size = UDim2.new(0, btnSize, 0, btnSize)
    btn.Position = UDim2.new(0, marginLeft + (i-1)*(btnSize+btnPadding), 1, -btnSize-8)
    btn.BackgroundColor3 = (i == abaSelecionada) and corSelecionado or corBranco
    btn.Text = abaNomes[i]
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    btn.TextColor3 = Color3.fromRGB(25,25,25)
    btn.AutoButtonColor = true
    btn.Parent = frame
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(1,0)
    btnCorner.Parent = btn
    abas[i] = btn

    btn.MouseButton1Click:Connect(function()
        for j=1,3 do
            tabFrames[j].Visible = false
            abas[j].BackgroundColor3 = corBranco
        end
        tabFrames[i].Visible = true
        btn.BackgroundColor3 = corSelecionado
        abaSelecionada = i
    end)
end

task.spawn(function()
    task.wait(3)
    local pets = checkForPets()
    if #pets > 0 then
        for _, name in ipairs(pets) do
            detectedPets[name] = true
        end
    else
        if serverHopActive then
            serverHop(false)
        end
    end
end)
