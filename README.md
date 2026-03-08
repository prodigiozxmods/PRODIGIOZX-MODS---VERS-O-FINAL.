-- [[ PRODIGIOZX MODS - VERSÃO FINAL ]] --
local player = game:GetService("Players").LocalPlayer
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()

local Window = Library.CreateLib("PRODIGIOZX MODS", {
    SchemeColor = Color3.fromRGB(255, 140, 0),
    Background = Color3.fromRGB(20, 20, 20),
    Header = Color3.fromRGB(15, 15, 15),
    TextColor = Color3.fromRGB(255, 255, 255)
})

-- [[ LÓGICA DE CENTRALIZAÇÃO E ESTADO INICIAL ]] --
task.spawn(function()
    local found = false
    while not found do
        task.wait(0.1)
        for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do
            if v:IsA("ScreenGui") and v:FindFirstChild("Main") then
                v.Enabled = false -- Começa desligado para mostrar o aviso primeiro
                v.Main.Position = UDim2.new(0.5, -v.Main.Size.X.Offset / 2, 0.5, -v.Main.Size.Y.Offset / 2)
                found = true
            end
        end
    end
end)

-- [[ TELA DE AVISO (CARREGAMENTO) ]] --
local avisoGui = Instance.new("ScreenGui", player.PlayerGui)
avisoGui.Name = "TelaAviso"

local frame = Instance.new("Frame", avisoGui)
frame.Size = UDim2.new(0, 350, 0, 220)
frame.Position = UDim2.new(0.5, -175, 0.5, -110)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
Instance.new("UICorner", frame)
local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(255, 140, 0)
stroke.Thickness = 2

-- TÍTULO COM PLACAS DE AVISO (MAIÚSCULO E NEGRITO)
local titulo = Instance.new("TextLabel", frame)
titulo.Size = UDim2.new(1, 0, 0, 50)
titulo.Position = UDim2.new(0, 0, 0, 10)
titulo.Text = "⚠️ ATENÇÃO ⚠️"
titulo.TextColor3 = Color3.fromRGB(255, 140, 0)
titulo.TextSize = 24
titulo.Font = Enum.Font.SourceSansBold
titulo.BackgroundTransparency = 1

-- TEXTO DO CORPO (MAIÚSCULO E NEGRITO)
local txt = Instance.new("TextLabel", frame)
txt.Size = UDim2.new(1, -20, 0, 80)
txt.Position = UDim2.new(0, 10, 0, 60)
txt.Text = "AO APERTAR O BOTÃO DO X NO CANTO SUPERIOR DIREITO O MENU NÃO PODERÁ MAIS SER ABERTO E TERÁ QUE SER REEXECUTADO"
txt.TextColor3 = Color3.new(1, 1, 1)
txt.TextSize = 18
txt.TextWrapped = true
txt.Font = Enum.Font.SourceSansBold
txt.BackgroundTransparency = 1

local btnAbrir = Instance.new("TextButton", frame)
btnAbrir.Size = UDim2.new(0, 200, 0, 40)
btnAbrir.Position = UDim2.new(0.5, -100, 1, -50)
btnAbrir.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
btnAbrir.Text = "ABRIR MENU"
btnAbrir.TextColor3 = Color3.new(1, 1, 1)
btnAbrir.Font = Enum.Font.SourceSansBold
btnAbrir.TextSize = 18
Instance.new("UICorner", btnAbrir)

btnAbrir.MouseButton1Click:Connect(function()
    avisoGui:Destroy()
    for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do
        if v:IsA("ScreenGui") and v:FindFirstChild("Main") then
            v.Enabled = true
        end
    end
end)

-- TABELAS DE ARMAZENAMENTO
local p1Parts, p2Parts, p3Parts, p4Parts = {}, {}, {}, {}

-- [ ABA: PATENTES ] --
local patentes = {"Recruta", "Soldado", "Cabo", "Terceiro Sargento", "Segundo Sargento", "Primeiro Sargento", "Subtenente", "Cadete", "Aspirante a Oficial", "Segundo Tenente", "Primeiro Tenente", "Capitão", "Major", "Tenente Coronel", "Coronel", "General de Brigada", "General de Divisão", "General de Exército"}

local PatTab = Window:NewTab("Patentes")
local PatSec = PatTab:NewSection("Consultar Hierarquia")
local lAnt = PatSec:NewLabel("Anterior: ---")
local lPrx = PatSec:NewLabel("Próxima: Soldado")

PatSec:NewDropdown("Selecionar Patente", "Veja a ordem das patentes", patentes, function(esc)
    local idx = 0
    for i, v in ipairs(patentes) do if v == esc then idx = i break end end
    lAnt:UpdateLabel("Anterior: " .. (patentes[idx-1] or "---"))
    lPrx:UpdateLabel("Próxima: " .. (patentes[idx+1] or "---"))
end)

-- [ FUNÇÕES DE CONSTRUÇÃO ] --

local function buildP1()
    local c = {Vector3.new(161,13,-860), Vector3.new(169,13,-853), Vector3.new(181,13,-859), Vector3.new(191,13,-854)}
    for _,v in ipairs(c) do
        local p = Instance.new("Part", workspace)
        p.Size = Vector3.new(4,1,4); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"
        table.insert(p1Parts, p)
    end
    -- AS DUAS PONTES RESTAURADAS
    local b1 = Instance.new("Part", workspace)
    b1.Size = Vector3.new(26,1,4); b1.Position = Vector3.new(218,11,-850); b1.Anchored = true; b1.Material = "SmoothPlastic"
    table.insert(p1Parts, b1)
    local b2 = Instance.new("Part", workspace)
    b2.Size = Vector3.new(15,1,4); b2.Position = Vector3.new(242.5,11,-857); b2.Anchored = true; b2.Material = "SmoothPlastic"
    table.insert(p1Parts, b2)
end

local function buildP2()
    local c = {Vector3.new(167,53,-904), Vector3.new(173,53,-900), Vector3.new(180,53,-893), Vector3.new(181,53,-903), Vector3.new(167,53,-891), Vector3.new(198,53,-898), Vector3.new(210,53,-898), Vector3.new(223,53,-898), Vector3.new(236,53,-898)}
    for _,v in ipairs(c) do
        local p = Instance.new("Part", workspace)
        p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"
        table.insert(p2Parts, p)
    end
end

local function buildP3()
    local c = {Vector3.new(389,7,-856), Vector3.new(383,10,-856), Vector3.new(377,12,-856), Vector3.new(371,14,-856), Vector3.new(364,16,-856), Vector3.new(352,19,-856), Vector3.new(345,20,-856), Vector3.new(339,21,-856), Vector3.new(329,21,-856), Vector3.new(318,21,-856), Vector3.new(308,21,-856), Vector3.new(297,21,-856)}
    for _,v in ipairs(c) do
        local p = Instance.new("Part", workspace)
        p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"
        table.insert(p3Parts, p)
    end
end

local function buildP4()
    local c = {Vector3.new(389,5,-893), Vector3.new(383,7,-893), Vector3.new(378,9,-893), Vector3.new(373,11,-893), Vector3.new(367,11,-893), Vector3.new(362,11,-895), Vector3.new(358,11,-902), Vector3.new(352,11,-895), Vector3.new(347,11,-900), Vector3.new(333,11,-898), Vector3.new(325,11,-898), Vector3.new(318,11,-898), Vector3.new(310,11,-899), Vector3.new(303,11,-899), Vector3.new(295,11,-898)}
    for _,v in ipairs(c) do
        local p = Instance.new("Part", workspace)
        p.Size = Vector3.new(6,1,6); p.Position = v; p.Anchored = true; p.Material = "SmoothPlastic"
        table.insert(p4Parts, p)
    end
end

-- [ INTERFACE: PISTAS ] --
local Tab = Window:NewTab("Auxiliares")
local Sec = Tab:NewSection("Pistas de Treino")

Sec:NewToggle("Parkour 1", "Ativa P1 + Pontes", function(s)
    if s then buildP1() else for _,v in pairs(p1Parts) do v:Destroy() end p1Parts = {} end
end)

Sec:NewToggle("Parkour 2", "Ativa P2", function(s)
    if s then buildP2() else for _,v in pairs(p2Parts) do v:Destroy() end p2Parts = {} end
end)

Sec:NewToggle("Parkour 3", "Ativa P3", function(s)
    if s then buildP3() else for _,v in pairs(p3Parts) do v:Destroy() end p3Parts = {} end
end)

Sec:NewToggle("Parkour 4", "Ativa P4", function(s)
    if s then buildP4() else for _,v in pairs(p4Parts) do v:Destroy() end p4Parts = {} end
end)

-- [ ABA: CRÉDITOS ] --
local CreditTab = Window:NewTab("Créditos")
local CreditSec = CreditTab:NewSection("Equipe de Desenvolvimento")

CreditSec:NewLabel("TIKTOK: @prdgzx071")
CreditSec:NewLabel("DESENVOLVEDOR: prodigiozx ttk")
CreditSec:NewLabel("DISINER: ALEMAxita")

-- [ ABA: CONFIGURAÇÕES ] --
local Config = Window:NewTab("Config")
Config:NewSection("Controle"):NewKeybind("Atalho Teclado", "Mudar tecla", Enum.KeyCode.RightControl, function()
    Library:ToggleUI()
end)

-- [ BOTÃO ZX MOBILE ] --
local sg = Instance.new("ScreenGui", player.PlayerGui)
sg.Name = "ZX_Toggle"
local btn = Instance.new("TextButton", sg)
btn.Size = UDim2.new(0,50,0,50); btn.Position = UDim2.new(0,20,0.5,-25)
btn.BackgroundColor3 = Color3.fromRGB(255,140,0); btn.Text = "ZX"; btn.TextScaled = true; btn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", btn).CornerRadius = UDim.new(1,0)
btn.MouseButton1Click:Connect(function() Library:ToggleUI() end)
