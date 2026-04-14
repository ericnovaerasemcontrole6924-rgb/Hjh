local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- CONFIGURAÃ‡ÃƒO DA JANELA PRINCIPAL (LIMPA)
local Window = Fluent:CreateWindow({
    Title = "MM2 SCRIPT SWAT07",
    SubTitle = "by SWAT07 ELITE",
    TabWidth = 160,
    Size = UDim2.fromOffset(600, 500),
    Acrylic = true, 
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

-- BOTÃƒO DE ATALHO (80x80)
local ScreenGui = Instance.new("ScreenGui", game.Players.LocalPlayer.PlayerGui)
ScreenGui.Name = "Swat07_Toggle"
-- Garante que o menu nÃ£o suma ao morrer
ScreenGui.ResetOnSpawn = false 

local ToggleBtn = Instance.new("TextButton", ScreenGui)
ToggleBtn.Size = UDim2.new(0, 80, 0, 80)
ToggleBtn.Position = UDim2.new(0.1, 0, 0.5, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleBtn.Text = "MM2"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 18

local UICorner = Instance.new("UICorner", ToggleBtn)
UICorner.CornerRadius = UDim.new(0, 15)

-- Sistema Draggable (Mover BotÃ£o)
local UserInputService = game:GetService("UserInputService")
local dragging, dragStart, startPos
local isMoving = false -- VariÃ¡vel para checar se estÃ¡ movendo

ToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        isMoving = false -- Reset no inÃ­cio do clique
        dragStart = input.Position
        startPos = ToggleBtn.Position
    end
end)

ToggleBtn.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        if delta.Magnitude > 5 then -- SÃ³ considera movimento se arrastar mais de 5 pixels
            isMoving = true
        end
        ToggleBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- Clique sÃ³ funciona se nÃ£o houver movimento (resolve o travamento)
ToggleBtn.MouseButton1Click:Connect(function()
    if not isMoving then 
        Window:Minimize() 
    end
end)
-- ESTRUTURA DE ABAS VAZIAS (PARA ADICIONARMOS AS FUNÃ‡Ã•ES REAIS)
local Tabs = {
    Main = Window:AddTab({ Title = "Principal", Icon = "home" }),
    Farm = Window:AddTab({ Title = "Lista de Farm", Icon = "coins" }),
    Visuals = Window:AddTab({ Title = "Visual/ESP", Icon = "eye" }),
    Combat = Window:AddTab({ Title = "Combate/Kill", Icon = "swords" }),
    Config = Window:AddTab({ Title = "ConfiguraÃ§Ãµes", Icon = "settings" }),
    
    Script = Window:AddTab({ Title = "Script", Icon = "code" }) -- NOVA ABA
}
-- [1] INTERFACE DO MENU (EM PRIMEIRO LUGAR NO TOPO)
-- PrÃ©-declarando as funÃ§Ãµes para que o Callback as reconheÃ§a antes da definiÃ§Ã£o
local espActive, runESP, removeESP

Tabs.Visuals:AddToggle("EspMaster", {
    Title = "Ativar ESP (Murder/Sheriff)",
    Default = false,
    Callback = function(Value)
        if Value then
            espActive = true
            runESP()
        else
            removeESP()
        end
    end
})

-- ---------------------------------------------------------
-- [2] VARIÃVEIS DE CONTROLE E LOGICA (ESTRUTURA ABAIXO)
-- ---------------------------------------------------------

espActive = false
local espFolders = {}

-- FUNÃ‡ÃƒO PARA REMOVER O ESP
removeESP = function()
    espActive = false
    for _, conn in pairs(espFolders) do 
        if conn then conn:Disconnect() end 
    end
    espFolders = {}
    for _, p in pairs(game.Players:GetPlayers()) do
        if p.Character then
            local l = p.Character:FindFirstChild("MM2_ESP_Light")
            local i = p.Character:FindFirstChild("MM2_ESP_Info")
            if l then l:Destroy() end
            if i then i:Destroy() end
        end
    end
end

-- FUNÃ‡ÃƒO PRINCIPAL PARA EXECUTAR O ESP
runESP = function()
    local function createESP(player)
        if player == game.Players.LocalPlayer then return end
        
        local function setupChar(char)
            if not char or not espActive then return end
            local hum = char:WaitForChild("Humanoid", 15)
            local hrp = char:WaitForChild("HumanoidRootPart", 15)
            
            if hum and hrp and espActive then
                local highlight = Instance.new("Highlight")
                highlight.Name = "MM2_ESP_Light"
                highlight.Adornee = char
                highlight.FillTransparency = 0.5
                highlight.OutlineTransparency = 0
                highlight.Parent = char
                
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "MM2_ESP_Info"
                billboard.Adornee = hrp
                billboard.Size = UDim2.new(0, 200, 0, 50)
                billboard.StudsOffset = Vector3.new(0, 3, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = char

                local textLabel = Instance.new("TextLabel")
                textLabel.BackgroundTransparency = 1
                textLabel.Size = UDim2.new(1, 0, 1, 0)
                textLabel.Font = Enum.Font.GothamBold
                textLabel.TextSize = 14
                textLabel.TextColor3 = Color3.new(1, 1, 1)
                textLabel.TextStrokeTransparency = 0
                textLabel.Parent = billboard

                task.spawn(function()
                    while char and char.Parent and espActive and highlight and billboard do
                        local roleColor = Color3.new(1, 1, 1)
                        local roleName = "Inocente"
                        local backpack = player:FindFirstChild("Backpack")
                        local character = player.Character

                        -- LÃ“GICA DE DETECÃ‡ÃƒO DE CARGOS MM2
                        if (character and character:FindFirstChild("Knife")) or (backpack and backpack:FindFirstChild("Knife")) then
                            roleColor = Color3.new(1, 0, 0)
                            roleName = "MURDERER ðŸ”ª"
                        elseif (character and character:FindFirstChild("Gun")) or (backpack and backpack:FindFirstChild("Gun")) then
                            roleColor = Color3.new(0, 0.5, 1)
                            roleName = "SHERIFF ðŸ”«"
                        end

                        highlight.FillColor = roleColor
                        highlight.OutlineColor = roleColor
                        
                        local dist = 0
                        if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            dist = math.floor((hrp.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude)
                        end

                        textLabel.Text = string.format("%s\n[%s]\nDist: %d", player.Name, roleName, dist)
                        textLabel.TextColor3 = roleColor
                        task.wait(0.5)
                    end
                end)
            end
        end

        local charConn = player.CharacterAdded:Connect(setupChar)
        table.insert(espFolders, charConn)
        if player.Character then setupChar(player.Character) end
    end

    -- Inicializa para todos os jogadores atuais e futuros
    for _, p in pairs(game.Players:GetPlayers()) do createESP(p) end
    local joinConn = game.Players.PlayerAdded:Connect(createESP)
    table.insert(espFolders, joinConn)
end
-- [1] -- [1] INTERFACE DO MENU (TOPO - NA ABA FARM)
local autoFarmMoedas = false
local farmSpeed = 2
local runFarmLogic -- PrÃ©-declaraÃ§Ã£o

Tabs.Farm:AddToggle("ToggleFarm", {
    Title = "Ativar Auto Farm (Moedas)",
    Default = false,
    Callback = function(Value)
        autoFarmMoedas = Value
        if Value then runFarmLogic() end
    end
})

Tabs.Farm:AddDropdown("FarmSpeedDropdown", {
    Title = "Velocidade",
    Values = {"1 - Lenta", "2 - RÃ¡pida", "3 - InstantÃ¢nea"},
    Default = 3,
    Callback = function(Value)
        if Value == "1 - Lenta" then
            farmSpeed = 1
        elseif Value == "2 - RÃ¡pida" then
            farmSpeed = 2
        else
            farmSpeed = 3 -- InstantÃ¢nea
        end
    end
})

-- [3] LÃ“GICA DE COLETA EM MASSA (SEM RE-ESCANEAMENTO)
local lp = game.Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

runFarmLogic = function()
    task.spawn(function()
        -- Ativa NoClip permanente
        local noclipLoop = RunService.Stepped:Connect(function()
            local char = lp.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
        
        while autoFarmMoedas do
            -- Verifica personagem
            local char = lp.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            
            if root then
                -- Procura o container de moedas
                local container = workspace:FindFirstChild("CoinContainer", true)
                
                if container then
                    -- Pega TODAS as moedas de uma vez e armazena
                    local moedas = container:GetChildren()
                    local moedasValidas = {}
                    
                    -- Filtra moedas vÃ¡lidas rapidamente
                    for i = 1, #moedas do
                        local coin = moedas[i]
                        if coin and coin.Position then
                            table.insert(moedasValidas, coin)
                        end
                    end
                    
                    -- Se tiver moedas, coleta todas em sequÃªncia RÃPIDA
                    if #moedasValidas > 0 then
                        -- Ordena por distÃ¢ncia (mais prÃ³ximas primeiro)
                        table.sort(moedasValidas, function(a, b)
                            local distA = (root.Position - a.Position).Magnitude
                            local distB = (root.Position - b.Position).Magnitude
                            return distA < distB
                        end)
                        
                        -- Modo INSTANTÃ‚NEO (recomendado)
                        if farmSpeed == 3 then
                            for i = 1, #moedasValidas do
                                if not autoFarmMoedas then break end
                                local coin = moedasValidas[i]
                                if coin and coin.Parent then -- Verifica se ainda existe
                                    root.CFrame = CFrame.new(coin.Position)
                                    task.wait(0.01) -- Delay mÃ­nimo para coleta
                                end
                            end
                        else
                            -- Modo suave (mais lento)
                            for i = 1, #moedasValidas do
                                if not autoFarmMoedas then break end
                                local coin = moedasValidas[i]
                                if coin and coin.Parent then
                                    local dist = (root.Position - coin.Position).Magnitude
                                    local duration = dist / (farmSpeed * 10)
                                    
                                    local tween = TweenService:Create(root, 
                                        TweenInfo.new(duration, Enum.EasingStyle.Linear), 
                                        {CFrame = CFrame.new(coin.Position)}
                                    )
                                    tween:Play()
                                    tween.Completed:Wait()
                                end
                            end
                        end
                    else
                        task.wait(0.1) -- Sem moedas
                    end
                else
                    task.wait(0.1) -- Sem container
                end
            else
                task.wait(0.1) -- Sem personagem
            end
            
            -- Pequena pausa entre ciclos de coleta
            task.wait(0.05)
        end
        
        -- Desativa NoClip
        noclipLoop:Disconnect()
    end)
end
local targetRole = "Nenhum"
local autoFlingActive = false
local runFlingLogic
local currentTarget = nil

Tabs.Main:AddDropdown("SelectTarget", {
    Title = "Selecionar Alvo",
    Values = {"Nenhum", "Murderer", "Sheriff"},
    Multi = false,
    Default = "Nenhum",
    Callback = function(Value) targetRole = Value end
})

Tabs.Main:AddToggle("ToggleAutoFling", {
    Title = "Ativar Auto Fling",
    Default = false,
    Callback = function(Value)
        autoFlingActive = Value
        if Value then runFlingLogic() end
    end
})

-- ---------------------------------------------------------
-- [2] DETECÃ‡ÃƒO DE CARGOS
-- ---------------------------------------------------------
local lp = game.Players.LocalPlayer
local RunService = game:GetService("RunService")

local function getPlayerRole(player)
    if not player or not player.Character then return "Inocente" end
    
    local char = player.Character
    local backpack = player:FindFirstChild("Backpack")
    
    -- Verifica no personagem
    for _, obj in pairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            local name = obj.Name:lower()
            if name:find("knife") or name:find("faca") then
                return "Murderer"
            elseif name:find("gun") or name:find("arma") or name:find("revolver") then
                return "Sheriff"
            end
        end
    end
    
    -- Verifica na mochila
    if backpack then
        for _, obj in pairs(backpack:GetChildren()) do
            if obj:IsA("Tool") then
                local name = obj.Name:lower()
                if name:find("knife") or name:find("faca") then
                    return "Murderer"
                elseif name:find("gun") or name:find("arma") or name:find("revolver") then
                    return "Sheriff"
                end
            end
        end
    end
    
    return "Inocente"
end

-- ---------------------------------------------------------
-- [3] FUNÃ‡ÃƒO DE FLING
-- ---------------------------------------------------------
local function executeFling(target)
    local char = lp.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local tChar = target.Character
    local tHrp = tChar and tChar:FindFirstChild("HumanoidRootPart")
    local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
    
    if not tHrp or not hrp or not tHum or tHum.Health <= 0 then return end

    Fluent:Notify({
        Title = "ðŸŽ¯ ATACANDO",
        Content = string.format("%s (%s)", target.Name, getPlayerRole(target)),
        Duration = 1
    })

    local oldPos = hrp.CFrame
    local start = tick()
    local lastDistChange = 0
    
    -- FORÃ‡A 9e8
    local FORCE = 9e8
    
    -- SequÃªncia para quando estiver caminhando
    local distSequence = {9, 8, 7, 6, 5, 4}
    local seqIndex = 1
    
    -- BodyPosition
    local bp = Instance.new("BodyPosition")
    bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bp.P = FORCE
    bp.D = FORCE / 10
    bp.Parent = hrp

    -- BodyVelocity
    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0, FORCE, 0)
    bv.Parent = hrp

    -- BodyGyro
    local bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bg.P = FORCE
    bg.D = FORCE / 10
    bg.Parent = hrp

    -- NOCLIP
    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end

    local connection
    connection = RunService.Heartbeat:Connect(function()
        if tick() - start >= 3.0 or not target.Character or tHum.Health <= 0 or not autoFlingActive then
            connection:Disconnect()
            bp:Destroy()
            bv:Destroy()
            bg:Destroy()
            hrp.CFrame = oldPos
            hrp.Velocity = Vector3.new(0, 0, 0)
            hrp.RotVelocity = Vector3.new(0, 0, 0)
            currentTarget = nil
            return
        end

        sethiddenproperty(lp, "SimulationRadius", 9e9)

        -- VELOCIDADE DO ALVO
        local currentVelocity = tHrp.Velocity
        local targetSpeed = currentVelocity.Magnitude
        
        -- FORÃ‡A
        bv.Velocity = Vector3.new(0, FORCE, 0)
        hrp.RotVelocity = Vector3.new(FORCE, FORCE, FORCE)

        -- PREDIÃ‡ÃƒO
        local prediction = currentVelocity * 0.2

        -- LÃ“GICA PRINCIPAL:
        if targetSpeed < 2 then
            -- PARADO: FICA DENTRO
            bp.Position = tHrp.Position + Vector3.new(0, 1, 0) + prediction
        else
            -- CAMINHANDO: SEQUÃŠNCIA NA FRENTE (A CADA 0.5s)
            if tick() - lastDistChange >= 0.5 then
                seqIndex = seqIndex + 1
                if seqIndex > #distSequence then
                    seqIndex = 1
                end
                lastDistChange = tick()
            end
            
            local currentDist = distSequence[seqIndex]
            -- TELEPORTA NA FRENTE
            bp.Position = tHrp.Position + (tHrp.CFrame.LookVector * currentDist) + prediction
        end
        
        -- MantÃ©m olhando para o alvo
        bg.CFrame = CFrame.lookAt(hrp.Position, tHrp.Position)
        
        -- TELEPORTE FORÃ‡ADO
        hrp.CFrame = CFrame.new(bp.Position, tHrp.Position)
    end)
    
    repeat task.wait() until not connection or not connection.Connected
end

-- ---------------------------------------------------------
-- [4] LOOP DE VIGILÃ‚NCIA
-- ---------------------------------------------------------
runFlingLogic = function()
    task.spawn(function()
        while autoFlingActive do
            if targetRole ~= "Nenhum" and not currentTarget then
                for _, p in pairs(game.Players:GetPlayers()) do
                    if p ~= lp and p.Character then
                        local role = getPlayerRole(p)
                        local hum = p.Character:FindFirstChildOfClass("Humanoid")
                        
                        if role == targetRole and hum and hum.Health > 0 then
                            currentTarget = p
                            Fluent:Notify({
                                Title = "âœ… ALVO",
                                Content = string.format("%s (%s)", p.Name, role),
                                Duration = 2
                            })
                            executeFling(p)
                            break
                        end
                    end
                end
            end
            task.wait(0.3)
        end
    end)
end

-- [5] BOTÃƒO MANUAL
Tabs.Main:AddButton({
    Title = "FLING MANUAL",
    Callback = function()
        if targetRole == "Nenhum" then
            Fluent:Notify({Title = "ERRO", Content = "Selecione um alvo!", Duration = 2})
            return
        end
        
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= lp and p.Character then
                local role = getPlayerRole(p)
                if role == targetRole then
                    executeFling(p)
                    break
                end
            end
        end
    end
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VermelhoNocivo_ADM"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")})
end
    end
         end
