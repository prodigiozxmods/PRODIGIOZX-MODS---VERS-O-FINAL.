-- [[ PRODIGIOZX MODS - COMPLETO (ESP COM LIMITE DE JOGADORES) ]] --
local OrionLib

-- Tentativa de carregamento seguro da interface Orion
local urls = {
    "https://raw.githubusercontent.com/shlexware/Orion/main/source",
    "https://raw.githubusercontent.com/jensonhirst/Orion/main/source",
    "https://pastebin.com/raw/1gtC8x7Y"
}

for _, url in ipairs(urls) do
    local success, result = pcall(function()
        return loadstring(game:HttpGet(url))()
    end)
    if success and result then
        OrionLib = result
        break
    end
end

if not OrionLib then
    warn("[PRODIGIOZX Error]: Nao foi possivel carregar a interface Orion. Verifique sua conexao ou executor.")
    return
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local SoundService = game:GetService("SoundService")
local player = Players.LocalPlayer

-- [[ CONFIGURAÇÕES ESP ]] --
local ESP = {
    Enabled = false,
    Boxes = false,
    Names = false,
    Distance = false,
    MaxDistance = 1000,
    MaxPlayers = 10, -- Quantidade de jogadores marcados pelo ESP
    Color = Color3.fromRGB(255, 140, 0)
}

local ESP_Colors = {
    ["Laranja"] = Color3.fromRGB(255, 140, 0),
    ["Vermelho"] = Color3.fromRGB(255, 50, 50),
    ["Verde"] = Color3.fromRGB(50, 255, 50),
    ["Azul"] = Color3.fromRGB(50, 150, 255),
    ["Amarelo"] = Color3.fromRGB(255, 255, 50),
    ["Ciano"] = Color3.fromRGB(50, 255, 255)
}

local Farm_Settings = { AutoFarm = false, ColetaPos = Vector3.new(80, 3, -430), EntregaPos = Vector3.new(441, 3, -225), WaitTime = 30 }

-- [[ SISTEMA DE MÚSICA ]] --
local Music_System = {
    SoundId = "9043887091",
    Volume = 1,
    Looped = true,
    Instance = nil
}

local function StopMusic()
    if Music_System.Instance then
        Music_System.Instance:Stop()
        Music_System.Instance:Destroy()
        Music_System.Instance = nil
    end
    local oldSound = SoundService:FindFirstChild("PRDG_GlobalRadio")
    if oldSound then oldSound:Destroy() end
end

local function PlayMusic()
    StopMusic()
    
    local cleanID = string.match(tostring(Music_System.SoundId), "%d+")
    if not cleanID then
        OrionLib:MakeNotification({Name = "Erro de Áudio", Content = "Insira um ID contendo apenas números.", Image = "rbxassetid://4483345998", Time = 3})
        return
    end

    local sound = Instance.new("Sound")
    sound.Name = "PRDG_GlobalRadio"
    sound.SoundId = "rbxassetid://" .. cleanID
    sound.Volume = Music_System.Volume
    sound.Looped = Music_System.Looped
    sound.Parent = SoundService

    local playOk = pcall(function() sound:Play() end)

    if playOk then
        Music_System.Instance = sound
        OrionLib:MakeNotification({Name = "Rádio", Content = "Tocando ID: " .. cleanID, Image = "rbxassetid://4483345998", Time = 3})
    else
        sound:Destroy()
        OrionLib:MakeNotification({Name = "Erro de Áudio", Content = "ID privado ou removido pelo Roblox.", Image = "rbxassetid://4483345998", Time = 4})
    end
end

-- [[ SISTEMA ESP COM FILTRO DE DISTÂNCIA E QUANTIDADE ]] --
local function CleanESP(char)
    if not char then return end
    local hl = char:FindFirstChild("PRDG_Highlight")
    local bb = char:FindFirstChild("PRDG_Billboard")
    if hl then hl:Destroy() end
    if bb then bb:Destroy() end
end

local function ApplyESP(targetPlayer)
    if targetPlayer == player then return end

    local function SetupCharacter(char)
        if not char then return end
        CleanESP(char)

        local root = char:WaitForChild("HumanoidRootPart", 5)
        if not root then return end

        local hl = Instance.new("Highlight")
        hl.Name = "PRDG_Highlight"
        hl.FillColor = ESP.Color
        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
        hl.FillTransparency = 0.5
        hl.OutlineTransparency = 0
        hl.Enabled = false
        hl.Parent = char

        local bb = Instance.new("BillboardGui")
        bb.Name = "PRDG_Billboard"
        bb.Adornee = root
        bb.Size = UDim2.new(0, 200, 0, 50)
        bb.StudsOffset = Vector3.new(0, 3.5, 0)
        bb.AlwaysOnTop = true
        bb.Enabled = false

        local txt = Instance.new("TextLabel")
        txt.Name = "InfoText"
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.TextColor3 = ESP.Color
        txt.TextStrokeTransparency = 0
        txt.TextSize = 13
        txt.Font = Enum.Font.SourceSansBold
        txt.Parent = bb

        bb.Parent = char
    end

    if targetPlayer.Character then SetupCharacter(targetPlayer.Character) end
    targetPlayer.CharacterAdded:Connect(SetupCharacter)
end

for _, p in pairs(Players:GetPlayers()) do ApplyESP(p) end
Players.PlayerAdded:Connect(ApplyESP)

RunService.RenderStepped:Connect(function()
    local myChar = player.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")

    -- Mapeia e ordena os jogadores por distância
    local validPlayers = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            local char = p.Character
            local root = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChild("Humanoid")
            if root and hum and hum.Health > 0 then
                local dist = myRoot and (myRoot.Position - root.Position).Magnitude or 0
                table.insert(validPlayers, {Player = p, Distance = dist})
            end
        end
    end

    table.sort(validPlayers, function(a, b) return a.Distance < b.Distance end)

    -- Atualiza o ESP respeitando o limite selecionado
    local targetMap = {}
    for i = 1, math.min(#validPlayers, ESP.MaxPlayers) do
        targetMap[validPlayers[i].Player] = validPlayers[i].Distance
    end

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            local char = p.Character
            local hl = char:FindFirstChild("PRDG_Highlight")
            local bb = char:FindFirstChild("PRDG_Billboard")
            local dist = targetMap[p]

            local shouldShow = ESP.Enabled and (dist ~= nil) and (dist <= ESP.MaxDistance)

            if hl then
                hl.FillColor = ESP.Color
                hl.Enabled = shouldShow and ESP.Boxes
            end

            if bb then
                local txt = bb:FindFirstChild("InfoText")
                if txt and dist then
                    txt.TextColor3 = ESP.Color
                    local str = ""
                    if ESP.Names then str = str .. p.Name end
                    if ESP.Distance then 
                        if str ~= "" then str = str .. " " end
                        str = str .. "[" .. math.floor(dist) .. "m]" 
                    end
                    txt.Text = str
                end
                bb.Enabled = shouldShow and (ESP.Names or ESP.Distance)
            end
        end
    end
end)

-- [[ INTERFACE PRINCIPAL ORION ]] --
local Window = OrionLib:MakeWindow({Name = "⚡ PRODIGIOZX MODS ⚡", HidePremium = true, SaveConfig = false, ConfigFolder = "PRDGZX"})

-- [[ ABA ESP ]] --
local EspTab = Window:MakeTab({Name = "👁️ ESP Visual", Icon = "rbxassetid://4483345998", PremiumOnly = false})

EspTab:AddToggle({Name = "Ativar ESP Geral", Default = false, Callback = function(v) ESP.Enabled = v end})
EspTab:AddToggle({Name = "Caixas / Highlight", Default = false, Callback = function(v) ESP.Boxes = v end})
EspTab:AddToggle({Name = "Mostrar Nomes", Default = false, Callback = function(v) ESP.Names = v end})
EspTab:AddToggle({Name = "Mostrar Distância", Default = false, Callback = function(v) ESP.Distance = v end})

EspTab:AddSlider({
    Name = "Quantidade de Pessoas",
    Min = 1,
    Max = 30,
    Default = 10,
    Color = Color3.fromRGB(255, 140, 0),
    Increment = 1,
    ValueName = "Pessoas",
    Callback = function(v)
        ESP.MaxPlayers = v
    end
})

EspTab:AddSlider({
    Name = "Distância Máxima do ESP",
    Min = 100,
    Max = 5000,
    Default = 1000,
    Color = Color3.fromRGB(255, 140, 0),
    Increment = 50,
    ValueName = "Studs",
    Callback = function(v)
        ESP.MaxDistance = v
    end
})

EspTab:AddDropdown({
    Name = "Cor do ESP",
    Default = "Laranja",
    Options = {"Laranja", "Vermelho", "Verde", "Azul", "Amarelo", "Ciano"},
    Callback = function(text)
        if ESP_Colors[text] then ESP.Color = ESP_Colors[text] end
    end
})

-- [[ ABA MÚSICA / RÁDIO ]] --
local MusicTab = Window:MakeTab({Name = "🎵 Rádio / Música", Icon = "rbxassetid://4483345998", PremiumOnly = false})

MusicTab:AddTextbox({
    Name = "ID do Áudio (Apenas Números)",
    Default = "9043887091",
    TextDisappear = false,
    Callback = function(text)
        if text ~= "" then Music_System.SoundId = text end
    end
})

MusicTab:AddButton({
    Name = "▶ Tocar Música",
    Callback = function() PlayMusic() end
})

MusicTab:AddButton({
    Name = "⏹ Parar Música",
    Callback = function()
        StopMusic()
        OrionLib:MakeNotification({Name = "Rádio", Content = "Música parada.", Image = "rbxassetid://4483345998", Time = 3})
    end
})

MusicTab:AddSlider({
    Name = "Volume da Música",
    Min = 0,
    Max = 5,
    Default = 1,
    Color = Color3.fromRGB(255,255,255),
    Increment = 0.1,
    ValueName = "Vol",
    Callback = function(v)
        Music_System.Volume = v
        if Music_System.Instance then Music_System.Instance.Volume = v end
    end
})

MusicTab:AddToggle({
    Name = "Repetir (Loop)",
    Default = true,
    Callback = function(v)
        Music_System.Looped = v
        if Music_System.Instance then Music_System.Instance.Looped = v end
    end
})

-- [[ ABA AUTO FARM ]] --
local FarmTab = Window:MakeTab({Name = "💰 Auto Farm", Icon = "rbxassetid://4483345998", PremiumOnly = false})

FarmTab:AddToggle({
    Name = "Ativar Auto Farm",
    Default = false,
    Callback = function(v)
        Farm_Settings.AutoFarm = v
        if v then
            task.spawn(function()
                while Farm_Settings.AutoFarm do
                    pcall(function()
                        local char = player.Character
                        if char and char:FindFirstChild("HumanoidRootPart") then
                            char.HumanoidRootPart.CFrame = CFrame.new(Farm_Settings.ColetaPos)
                            task.wait(1.5)
                            
                            for _, prompt in pairs(workspace:GetDescendants()) do
                                if prompt:IsA("ProximityPrompt") and (char.HumanoidRootPart.Position - prompt.Parent.Position).Magnitude < 25 then
                                    if fireproximityprompt then
                                        fireproximityprompt(prompt)
                                    else
                                        prompt:InputHoldBegin()
                                        task.wait(prompt.HoldDuration)
                                        prompt:InputHoldEnd()
                                    end
                                    break
                                end
                            end
                            
                            for i = Farm_Settings.WaitTime, 1, -1 do
                                if not Farm_Settings.AutoFarm then break end
                                task.wait(1)
                            end
                            
                            if Farm_Settings.AutoFarm then
                                char.HumanoidRootPart.CFrame = CFrame.new(Farm_Settings.EntregaPos)
                                task.wait(2.5)
                            end
                        end
                    end)
                    task.wait(0.1)
                end
            end)
        end
    end
})

-- [[ ABA PATENTES ]] --
local PatTab = Window:MakeTab({Name = "🎖️ Patentes", Icon = "rbxassetid://4483345998", PremiumOnly = false})

local patentes = {
    "Recruta", "Soldado", "Cabo", "Terceiro Sargento", "Segundo Sargento", 
    "Primeiro Sargento", "Subtenente", "Cadete", "Aspirante a Oficial", 
    "Segundo Tenente", "Primeiro Tenente", "Capitão", "Major", 
    "Tenente Coronel", "Coronel", "General de Brigada", "General de Divisão", "General de Exército"
}

local currentPatIndex = 1

local PatAtualLabel = PatTab:AddLabel("Patente Selecionada: " .. patentes[currentPatIndex])
local PatAnteriorLabel = PatTab:AddLabel("Patente Anterior: Nenhuma")
local PatProximaLabel = PatTab:AddLabel("Próxima Patente: " .. patentes[currentPatIndex + 1])

local function UpdatePatenteUI()
    local atual = patentes[currentPatIndex]
    local anterior = patentes[currentPatIndex - 1] or "Nenhuma (Início)"
    local proxima = patentes[currentPatIndex + 1] or "Nenhuma (Máxima)"

    PatAtualLabel:Set("Patente Selecionada: " .. atual)
    PatAnteriorLabel:Set("Patente Anterior: " .. anterior)
    PatProximaLabel:Set("Próxima Patente: " .. proxima)
end

PatTab:AddButton({
    Name = "⬅️ Ver Patente Anterior",
    Callback = function()
        if currentPatIndex > 1 then
            currentPatIndex = currentPatIndex - 1
            UpdatePatenteUI()
        end
    end
})

PatTab:AddButton({
    Name = "➡️ Ver Próxima Patente",
    Callback = function()
        if currentPatIndex < #patentes then
            currentPatIndex = currentPatIndex + 1
            UpdatePatenteUI()
        end
    end
})

PatTab:AddDropdown({
    Name = "Lista Completa de Patentes",
    Default = patentes[1],
    Options = patentes,
    Callback = function(selected)
        for i, pName in ipairs(patentes) do
            if pName == selected then
                currentPatIndex = i
                UpdatePatenteUI()
                break
            end
        end
    end
})

-- [[ ABA AUXILIARES (PARKOURS 1 AO 4) ]] --
local AuxTab = Window:MakeTab({Name = "🛠️ Auxiliares", Icon = "rbxassetid://4483345998", PremiumOnly = false})
local p1Parts, p2Parts, p3Parts, p4Parts = {}, {}, {}, {}

AuxTab:AddToggle({
    Name = "Parkour 1 + Pontes",
    Default = false,
    Callback = function(s)
        if s then
            local c = {Vector3.new(161,13,-860), Vector3.new(169,13,-853), Vector3.new(181,13,-859), Vector3.new(191,13,-854)}
            for _,v in ipairs(c) do
                local p = Instance.new("Part", workspace); p.Size = Vector3.new(4,1,4); p.Position = v; p.Anchored = true; p.Material = Enum.Material.SmoothPlastic; table.insert(p1Parts, p)
            end
            local b1 = Instance.new("Part", workspace); b1.Size = Vector3.new(26,1,4); b1.Position = Vector3.new(218,11,-850); b1.Anchored = true; b1.Material = Enum.Material.SmoothPlastic; table.insert(p1Parts, b1)
            local b2 = Instance.new("Part", workspace); b2.Size = Vector3.new(15,1,4); b2.Position = Vector3.new(242.5,11,-857); b2.Anchored = true; b2.Material = Enum.Material.SmoothPlastic; table.insert(p1Parts, b2)
        else 
            for _,v in pairs(p1Parts) do v:Destroy() end p1Parts = {} 
        end
    end
})

AuxTab:AddToggle({
    Name = "Parkour 2",
    Default = false,
    Callback = function(s)
        if s then
            local c = {Vector3.new(167,53,-904), Vector3.new(173,53,-900), Vector3.new(180,53,-893), Vector3.new(181,53,-903), Vector3.new(167,53,-891), Vector3.new(198,53,-898), Vector3.new(210,53,-898), Vector3.new(223,53,-898), Vector3.new(236,53,-898)}
            for _,v in ipairs(c) do
                local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = Enum.Material.SmoothPlastic; table.insert(p2Parts, p)
            end
        else 
            for _,v in pairs(p2Parts) do v:Destroy() end p2Parts = {} 
        end
    end
})

AuxTab:AddToggle({
    Name = "Parkour 3",
    Default = false,
    Callback = function(s)
        if s then
            local c = {Vector3.new(389,7,-856), Vector3.new(383,10,-856), Vector3.new(377,12,-856), Vector3.new(371,14,-856), Vector3.new(364,16,-856), Vector3.new(352,19,-856), Vector3.new(345,20,-856), Vector3.new(339,21,-856), Vector3.new(329,21,-856), Vector3.new(318,21,-856), Vector3.new(308,21,-856), Vector3.new(297,21,-856)}
            for _,v in ipairs(c) do
                local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = Enum.Material.SmoothPlastic; table.insert(p3Parts, p)
            end
        else 
            for _,v in pairs(p3Parts) do v:Destroy() end p3Parts = {} 
        end
    end
})

AuxTab:AddToggle({
    Name = "Parkour 4",
    Default = false,
    Callback = function(s)
        if s then
            local c = {Vector3.new(389,5,-893), Vector3.new(383,7,-893), Vector3.new(378,9,-893), Vector3.new(373,11,-893), Vector3.new(367,11,-893), Vector3.new(362,11,-895), Vector3.new(358,11,-902), Vector3.new(352,11,-895), Vector3.new(347,11,-900), Vector3.new(333,11,-898), Vector3.new(325,11,-898), Vector3.new(318,11,-898), Vector3.new(310,11,-899), Vector3.new(303,11,-899), Vector3.new(295,11,-898)}
            for _,v in ipairs(c) do
                local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = Enum.Material.SmoothPlastic; table.insert(p4Parts, p)
            end
        else 
            for _,v in pairs(p4Parts) do v:Destroy() end p4Parts = {} 
        end
    end
})

-- [[ ABA CRÉDITOS ]] --
local CreditTab = Window:MakeTab({Name = "ℹ️ Créditos", Icon = "rbxassetid://4483345998", PremiumOnly = false})
CreditTab:AddLabel("TikTok: @prdgzx071")
CreditTab:AddLabel("Desenvolvedor: prodigiozx ttk")

OrionLib:Init()
    WaitTime = 30 
}

-- [[ DISPLAY DE TEMPO NA TELA ]] --
local farmGui = Instance.new("ScreenGui", player.PlayerGui)
local timerLabel = Instance.new("TextLabel", farmGui)
timerLabel.Size = UDim2.new(0, 220, 0, 50)
timerLabel.Position = UDim2.new(0.5, -110, 0.1, 0)
timerLabel.BackgroundColor3 = Color3.new(0,0,0)
timerLabel.BackgroundTransparency = 0.5
timerLabel.TextColor3 = Color3.fromRGB(255, 140, 0)
timerLabel.TextSize = 22
timerLabel.Font = Enum.Font.SourceSansBold
timerLabel.Text = "AGUARDANDO..."
timerLabel.Visible = false
Instance.new("UICorner", timerLabel)

-- [[ TELA DE AVISO (CORRIGIDA) ]] --
task.spawn(function()
    local found = false
    while not found do
        task.wait(0.1)
        for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do
            if v:IsA("ScreenGui") and v:FindFirstChild("Main") then
                v.Enabled = false
                v.Main.Position = UDim2.new(0.5, -v.Main.Size.X.Offset / 2, 0.5, -v.Main.Size.Y.Offset / 2)
                found = true
            end
        end
    end
end)

local avisoGui = Instance.new("ScreenGui", player.PlayerGui)
avisoGui.Name = "AvisoProdigio"
avisoGui.DisplayOrder = 999

local frame = Instance.new("Frame", avisoGui)
frame.Size = UDim2.new(0, 350, 0, 220); frame.Position = UDim2.new(0.5, -175, 0.5, -110)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25); Instance.new("UICorner", frame)
local stroke = Instance.new("UIStroke", frame); stroke.Color = Color3.fromRGB(255, 140, 0); stroke.Thickness = 2

local titulo = Instance.new("TextLabel", frame)
titulo.Size = UDim2.new(1, 0, 0, 50); titulo.Text = "⚠️ ATENÇÃO ⚠️"
titulo.TextColor3 = Color3.fromRGB(255, 140, 0); titulo.TextSize = 26
titulo.Font = Enum.Font.SourceSansBold; titulo.BackgroundTransparency = 1; titulo.ZIndex = 10

local txt = Instance.new("TextLabel", frame)
txt.Size = UDim2.new(1, -20, 0, 80); txt.Position = UDim2.new(0, 10, 0, 60)
txt.Text = "AO APERTAR O BOTÃO DO X NO CANTO SUPERIOR DIREITO O MENU NÃO PODERÁ MAIS SER ABERTO"
txt.TextColor3 = Color3.fromRGB(255, 255, 255); txt.TextSize = 18; txt.TextWrapped = true
txt.Font = Enum.Font.SourceSansBold; txt.BackgroundTransparency = 1; txt.ZIndex = 10

local btnAbrir = Instance.new("TextButton", frame)
btnAbrir.Size = UDim2.new(0, 200, 0, 40); btnAbrir.Position = UDim2.new(0.5, -100, 1, -50)
btnAbrir.BackgroundColor3 = Color3.fromRGB(255, 140, 0); btnAbrir.Text = "ABRIR MENU"; btnAbrir.TextColor3 = Color3.new(1, 1, 1)
btnAbrir.Font = Enum.Font.SourceSansBold; btnAbrir.TextSize = 18; btnAbrir.ZIndex = 10; Instance.new("UICorner", btnAbrir)

btnAbrir.MouseButton1Click:Connect(function()
    avisoGui:Destroy()
    for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do
        if v:IsA("ScreenGui") and v:FindFirstChild("Main") then v.Enabled = true end
    end
end)

-- [[ ABA: AUTO FARM ]] --
local FarmTab = Window:NewTab("Farm (Dinheiro)")
local FarmSec = FarmTab:NewSection("Configuração do Farm")

FarmSec:NewToggle("Ativar Auto Farm", "Liga/Desliga o Script", function(v)
    Farm_Settings.AutoFarm = v
    timerLabel.Visible = v
    if v then
        task.spawn(function()
            while Farm_Settings.AutoFarm do
                pcall(function()
                    local char = player.Character
                    if char and char:FindFirstChild("HumanoidRootPart") then
                        timerLabel.Text = "COLETANDO..."
                        char.HumanoidRootPart.CFrame = CFrame.new(Farm_Settings.ColetaPos)
                        task.wait(1.5)
                        for _, prompt in pairs(workspace:GetDescendants()) do
                            if prompt:IsA("ProximityPrompt") and (char.HumanoidRootPart.Position - prompt.Parent.Position).Magnitude < 20 then
                                fireproximityprompt(prompt)
                                break
                            end
                        end
                        -- Início do tempo definido na TextBox
                        for i = Farm_Settings.WaitTime, 1, -1 do
                            if not Farm_Settings.AutoFarm then break end
                            timerLabel.Text = "ENTREGA EM: " .. i .. "s"
                            task.wait(1)
                        end
                        if Farm_Settings.AutoFarm then
                            timerLabel.Text = "ENTREGANDO..."
                            char.HumanoidRootPart.CFrame = CFrame.new(Farm_Settings.EntregaPos)
                            task.wait(2.5)
                        end
                    end
                end)
                task.wait(0.1)
            end
        end)
    end
end)

-- MUDANÇA: Substituído Slider por TextBox
FarmSec:NewTextBox("Segundos para Entregar", "Digite o tempo (Ex: 30)", function(t)
    local num = tonumber(t)
    if num then
        Farm_Settings.WaitTime = num
    end
end)

-- [[ ABA: PATENTES ]] --
local patentes = {"Recruta", "Soldado", "Cabo", "Terceiro Sargento", "Segundo Sargento", "Primeiro Sargento", "Subtenente", "Cadete", "Aspirante a Oficial", "Segundo Tenente", "Primeiro Tenente", "Capitão", "Major", "Tenente Coronel", "Coronel", "General de Brigada", "General de Divisão", "General de Exército"}
local PatTab = Window:NewTab("Patentes")
local PatSec = PatTab:NewSection("Hierarquia")
local lAnt = PatSec:NewLabel("Anterior: ---"); local lPrx = PatSec:NewLabel("Próxima: Soldado")
PatSec:NewDropdown("Patente", "Selecione", patentes, function(esc)
    local idx = 0
    for i, v in ipairs(patentes) do if v == esc then idx = i break end end
    lAnt:UpdateLabel("Anterior: " .. (patentes[idx-1] or "---"))
    lPrx:UpdateLabel("Próxima: " .. (patentes[idx+1] or "---"))
end)

-- [[ ABA: VISUAL (ESP) ]] --
local VisualTab = Window:NewTab("Visual (ESP)")
local ESPSection = VisualTab:NewSection("ESP Proximidade")
ESPSection:NewToggle("ESP Master", "Ativa o sistema", function(v) ESP_Settings.Enabled = v end)
ESPSection:NewToggle("ESP Box", "Caixas", function(v) ESP_Settings.Boxes = v end)
ESPSection:NewToggle("ESP Lines", "Linhas", function(v) ESP_Settings.Lines = v end)
ESPSection:NewTextBox("Qtd Pessoas", "Ex: 1", function(t)
    local n = tonumber(t)
    if n then ESP_Settings.MaxPlayers = n end
end)
ESPSection:NewToggle("ESP Names", "Nomes", function(v) ESP_Settings.Names = v end)
ESPSection:NewToggle("ESP Distance", "Distância", function(v) ESP_Settings.Distance = v end)
ESPSection:NewColorPicker("Cor ESP", "Cor", Color3.fromRGB(255, 140, 0), function(c) ESP_Settings.Color = c end)

-- Lógica ESP
local function GetSortedPlayers()
    local pList = {}
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (player.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
            table.insert(pList, {Player = v, Distance = dist})
        end
    end
    table.sort(pList, function(a, b) return a.Distance < b.Distance end)
    return pList
end

local drawings = {}
game:GetService("RunService").RenderStepped:Connect(function()
    for _, d in pairs(drawings) do d.Visible = false end
    if not ESP_Settings.Enabled then return end
    local sorted = GetSortedPlayers()
    local limit = math.min(#sorted, ESP_Settings.MaxPlayers)
    for i = 1, limit do
        local data = sorted[i]; local plr = data.Player; local root = plr.Character.HumanoidRootPart
        local pos, onScreen = camera:WorldToViewportPoint(root.Position)
        if onScreen then
            local id = plr.Name
            if not drawings[id.."B"] then
                drawings[id.."B"] = Drawing.new("Square"); drawings[id.."L"] = Drawing.new("Line"); drawings[id.."T"] = Drawing.new("Text")
            end
            local B, L, T = drawings[id.."B"], drawings[id.."L"], drawings[id.."T"]
            if ESP_Settings.Boxes then
                local sx, sy = 1000/pos.Z, 2000/pos.Z
                B.Visible = true; B.Color = ESP_Settings.Color; B.Thickness = 1
                B.Size = Vector2.new(sx, sy); B.Position = Vector2.new(pos.X-sx/2, pos.Y-sy/2); B.Filled = false
            end
            if ESP_Settings.Lines then
                L.Visible = true; L.Color = ESP_Settings.Color; L.Thickness = 1
                L.From = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y); L.To = Vector2.new(pos.X, pos.Y)
            end
            if ESP_Settings.Names or ESP_Settings.Distance then
                local c = ""
                if ESP_Settings.Names then c = c .. plr.Name end
                if ESP_Settings.Distance then c = c .. " [" .. math.floor(data.Distance) .. "m]" end
                T.Visible = true; T.Text = c; T.Color = ESP_Settings.Color; T.Size = 18; T.Center = true; T.Outline = true
                T.Position = Vector2.new(pos.X, pos.Y - (2000/pos.Z)/2 - 20)
            end
        end
    end
end)

-- [[ ABA: AUXILIARES (PARKOURS) ]] --
local p1Parts, p2Parts, p3Parts, p4Parts = {}, {}, {}, {}
local Tab = Window:NewTab("Auxiliares")
local Sec = Tab:NewSection("Pistas de Treino")

Sec:NewToggle("Parkour 1", "P1 + Pontes", function(s)
    if s then
        local c = {Vector3.new(161,13,-860), Vector3.new(169,13,-853), Vector3.new(181,13,-859), Vector3.new(191,13,-854)}
        for _,v in ipairs(c) do
            local p = Instance.new("Part", workspace); p.Size = Vector3.new(4,1,4); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"; table.insert(p1Parts, p)
        end
        local b1 = Instance.new("Part", workspace); b1.Size = Vector3.new(26,1,4); b1.Position = Vector3.new(218,11,-850); b1.Anchored = true; b1.Material = "SmoothPlastic"; table.insert(p1Parts, b1)
        local b2 = Instance.new("Part", workspace); b2.Size = Vector3.new(15,1,4); b2.Position = Vector3.new(242.5,11,-857); b2.Anchored = true; b2.Material = "SmoothPlastic"; table.insert(p1Parts, b2)
    else for _,v in pairs(p1Parts) do v:Destroy() end p1Parts = {} end
end)

Sec:NewToggle("Parkour 2", "Ativa P2", function(s)
    if s then
        local c = {Vector3.new(167,53,-904), Vector3.new(173,53,-900), Vector3.new(180,53,-893), Vector3.new(181,53,-903), Vector3.new(167,53,-891), Vector3.new(198,53,-898), Vector3.new(210,53,-898), Vector3.new(223,53,-898), Vector3.new(236,53,-898)}
        for _,v in ipairs(c) do
            local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"; table.insert(p2Parts, p)
        end
    else for _,v in pairs(p2Parts) do v:Destroy() end p2Parts = {} end
end)

Sec:NewToggle("Parkour 3", "Ativa P3", function(s)
    if s then
        local c = {Vector3.new(389,7,-856), Vector3.new(383,10,-856), Vector3.new(377,12,-856), Vector3.new(371,14,-856), Vector3.new(364,16,-856), Vector3.new(352,19,-856), Vector3.new(345,20,-856), Vector3.new(339,21,-856), Vector3.new(329,21,-856), Vector3.new(318,21,-856), Vector3.new(308,21,-856), Vector3.new(297,21,-856)}
        for _,v in ipairs(c) do
            local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"; table.insert(p3Parts, p)
        end
    else for _,v in pairs(p3Parts) do v:Destroy() end p3Parts = {} end
end)

Sec:NewToggle("Parkour 4", "Ativa P4", function(s)
    if s then
        local c = {Vector3.new(389,5,-893), Vector3.new(383,7,-893), Vector3.new(378,9,-893), Vector3.new(373,11,-893), Vector3.new(367,11,-893), Vector3.new(362,11,-895), Vector3.new(358,11,-902), Vector3.new(352,11,-895), Vector3.new(347,11,-900), Vector3.new(333,11,-898), Vector3.new(325,11,-898), Vector3.new(318,11,-898), Vector3.new(310,11,-899), Vector3.new(303,11,-899), Vector3.new(295,11,-898)}
        for _,v in ipairs(c) do
            local p = Instance.new("Part", workspace); p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"; table.insert(p4Parts, p)
        end
    else for _,v in pairs(p4Parts) do v:Destroy() end p4Parts = {} end
end)

-- [[ CRÉDITOS E CONFIG ]] --
local CreditTab = Window:NewTab("Créditos")
CreditTab:NewSection("TIKTOK: @prdgzx071")
CreditTab:NewSection("DEV: prodigiozx ttk")

local Config = Window:NewTab("Config")
Config:NewSection("Atalho"):NewKeybind("Abrir Menu", "Ctrl Direito", Enum.KeyCode.RightControl, function() Library:ToggleUI() end)

local sgZX = Instance.new("ScreenGui", player.PlayerGui)
local btnZX = Instance.new("TextButton", sgZX)
btnZX.Size = UDim2.new(0,50,0,50); btnZX.Position = UDim2.new(0,20,0.5,-25); btnZX.BackgroundColor3 = Color3.fromRGB(255,140,0); btnZX.Text = "ZX"; btnZX.TextScaled = true; btnZX.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", btnZX).CornerRadius = UDim.new(1,0)
btnZX.MouseButton1Click:Connect(function() Library:ToggleUI() end)
