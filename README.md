local NEVERLOSE = loadstring(game:HttpGet("https://you.whimper.xyz/sources/ronix/ui.lua"))()

NEVERLOSE:Theme("nightly")
local Window = NEVERLOSE:AddWindow("byMT7 Hub", "Brariont para Roube um Brariont")

-- Tabs e Sections
local MainTab = Window:AddTab("Principal", "home")
local MainSection = MainTab:AddSection("Funções", "left")

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Função: Detectar base do jogador
local function encontrarBaseDoJogador()
    local workspace = game:GetService("Workspace")
    for _, obj in pairs(workspace:GetChildren()) do
        if obj:IsA("Model") or obj:IsA("BasePart") then
            if obj.Name == player.Name or obj.Name:lower():find(player.Name:lower(), 1, true) then
                if obj:IsA("BasePart") then
                    return obj
                elseif obj:IsA("Model") and obj.PrimaryPart then
                    return obj.PrimaryPart
                elseif obj:IsA("Model") then
                    local root = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChildWhichIsA("BasePart")
                    if root then return root end
                end
            end
        end
    end
    local humanoidRoot = character:FindFirstChild("HumanoidRootPart")
    if humanoidRoot then
        local maisProximo, menorDist = nil, math.huge
        for _, obj in pairs(workspace:GetChildren()) do
            if obj:IsA("SpawnLocation") then
                local dist = (humanoidRoot.Position - obj.Position).Magnitude
                if dist < menorDist then
                    menorDist = dist
                    maisProximo = obj
                end
            end
        end
        return maisProximo
    end
    return nil
end

-- Função: Espera 3 segundos e teleporta para base
local function teleportarParaBase()
    local base = encontrarBaseDoJogador()
    if base then
        NEVERLOSE:Notify("Aguarde 3 segundos para teleportar!", 3)
        wait(3)
        local humanoidRoot = character:FindFirstChild("HumanoidRootPart")
        if humanoidRoot then
            humanoidRoot.CFrame = base.CFrame + Vector3.new(0, 5, 0)
            NEVERLOSE:Notify("Teleportado para sua base!", 2)
        end
    else
        NEVERLOSE:Notify("Base não encontrada!", 2)
    end
end

-- Função: Teleportar para Spawn
local function teleportToSpawn()
    local humanoidRoot = character:FindFirstChild("HumanoidRootPart")
    if humanoidRoot then
        humanoidRoot.CFrame = CFrame.new(0, 10, 0) -- Altere para a posição real do spawn se precisar
        NEVERLOSE:Notify("Teleportado para o spawn!", 2)
    end
end

-- Funções de movimento
local function superVelocidade()
    humanoid.WalkSpeed = 80
    NEVERLOSE:Notify("Super Velocidade ativada!", 2)
end

local function superPulo()
    humanoid.JumpPower = 200
    NEVERLOSE:Notify("Super Pulo ativado!", 2)
end

local function resetarStatus()
    humanoid.WalkSpeed = 16
    humanoid.JumpPower = 50
    NEVERLOSE:Notify("Status resetado!", 2)
end

-- ======= ESP FUNÇÃO =======
local espEnabled = false
local espObjects = {}

local function removeAllESP()
    for _, tbl in pairs(espObjects) do
        if tbl.box then
            tbl.box:Destroy()
        end
        if tbl.name then
            tbl.name:Destroy()
        end
    end
    espObjects = {}
end

local function createESPForPlayer(targetPlayer)
    if targetPlayer == player then return end
    local char = targetPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    -- Caixa verde no corpo
    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = char
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(4, 6, 2)
    box.Color3 = Color3.fromRGB(0,255,0)
    box.Transparency = 0.7
    box.Parent = game.CoreGui

    -- Nome pequeno acima da cabeça
    local head = char:FindFirstChild("Head")
    if head then
        local bill = Instance.new("BillboardGui")
        bill.Name = "ESPName"
        bill.Adornee = head
        bill.Size = UDim2.new(0,100,0,30)
        bill.StudsOffset = Vector3.new(0,2,0)
        bill.AlwaysOnTop = true
        bill.Parent = game.CoreGui

        local nameLbl = Instance.new("TextLabel")
        nameLbl.BackgroundTransparency = 1
        nameLbl.Size = UDim2.new(1,0,1,0)
        nameLbl.Text = targetPlayer.Name
        nameLbl.TextColor3 = Color3.fromRGB(0,255,0)
        nameLbl.TextStrokeTransparency = 0.5
        nameLbl.Font = Enum.Font.SourceSansBold
        nameLbl.TextSize = 13
        nameLbl.Parent = bill

        espObjects[targetPlayer] = {box=box, name=bill}
    else
        espObjects[targetPlayer] = {box=box}
    end
end

local function updateESP()
    removeAllESP()
    if not espEnabled then return end
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            createESPForPlayer(plr)
        end
    end
end

local function onCharacterAdded(char)
    if espEnabled then
        wait(1)
        updateESP()
    end
end

local function setupESPConnections()
    game.Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function()
            if espEnabled then
                wait(1)
                updateESP()
            end
        end)
    end)
    game.Players.PlayerRemoving:Connect(function(plr)
        if espObjects[plr] then
            if espObjects[plr].box then espObjects[plr].box:Destroy() end
            if espObjects[plr].name then espObjects[plr].name:Destroy() end
            espObjects[plr] = nil
        end
    end)
end

setupESPConnections()

local function setESP(state)
    espEnabled = state
    if espEnabled then
        updateESP()
        NEVERLOSE:Notify("ESP ativado!",2)
    else
        removeAllESP()
        NEVERLOSE:Notify("ESP desativado!",2)
    end
end

-- Botões principais
MainSection:AddButton("Teleportar para Sua Base (3s)", teleportarParaBase)
MainSection:AddButton("Teleportar para o Spawn", teleportToSpawn)
MainSection:AddButton("Super Velocidade", superVelocidade)
MainSection:AddButton("Super Pulo", superPulo)
MainSection:AddButton("Resetar Status", resetarStatus)
MainSection:AddToggle("ESP Verde nos Jogadores", false, setESP)

-- Opcional: Toggle de Velocidade
MainSection:AddToggle("Velocidade Infinita", false, function(state)
    if state then
        humanoid.WalkSpeed = 200
        NEVERLOSE:Notify("Velocidade Infinita ativada!",2)
    else
        humanoid.WalkSpeed = 16
        NEVERLOSE:Notify("Velocidade normal restaurada!",2)
    end
end)

-- Opcional: Dropdown para escolher ação rápida
MainSection:AddDropdown("Ação Rápida", {"Teleportar Base", "Teleportar Spawn", "Super Velocidade", "Super Pulo", "Resetar"}, function(selection)
    if selection == "Teleportar Base" then teleportarParaBase()
    elseif selection == "Teleportar Spawn" then teleportToSpawn()
    elseif selection == "Super Velocidade" then superVelocidade()
    elseif selection == "Super Pulo" then superPulo()
    elseif selection == "Resetar" then resetarStatus()
    end
end)

-- Suporte a respawn (atualizar humanoid se morrer)
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = character:WaitForChild("Humanoid")
end)

-- Tab extra de exemplo (pode remover/modificar)
local OtherTab = Window:AddTab("Outros", "folder")
local OtherSection = OtherTab:AddSection("Exemplos", "left")
OtherSection:AddButton("Mensagem de Teste", function()
    NEVERLOSE:Notify("Botão de teste clicado!",2)
end)


