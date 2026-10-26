--[[
    SCRIPT: Painel de Gerenciamento para Reroll (Floating UI)
    FUNÇÃO: Salvar Ponto (Marcador) e Relogar (Rejoin)
    AVISO: Este script é para fins educacionais. O "Salvar Ponto" NÃO reverte
    dados no servidor. Ele apenas facilita o retorno ao mesmo servidor.
]]

-- Serviços necessários
local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

-- Variáveis de Estado
local savedJobId = nil
local isPanelOpen = true
local savedTimestamp = nil

-- ==========================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (UI)
-- ==========================================

-- Remove UI antiga se existir (evita duplicação)
local oldGui = CoreGui:FindFirstChild("RerollManagerUI")
if oldGui then oldGui:Destroy() end

-- ScreenGui Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RerollManagerUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
ScreenGui.DisplayOrder = 999

-- Botão Flutuante (Minimizar/Expandir)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Position = UDim2.new(0, 20, 0, 200)
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.TextColor3 = Color3.fromRGB(255, 200, 0)
ToggleButton.Text = "⚙️"
ToggleButton.TextSize = 24
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = ScreenGui
ToggleButton.Active = true
ToggleButton.Draggable = true -- Permite arrastar o botão

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(1, 0)
ToggleCorner.Parent = ToggleButton

-- Painel Principal (Frame)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 260, 0, 220)
MainFrame.Position = UDim2.new(0, 20, 0, 260)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Draggable = true -- Permite arrastar o painel

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Título do Painel
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "Title"
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "REROLL MANAGER"
TitleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.Parent = MainFrame

-- Botão "Salvar Ponto"
local SavePointButton = Instance.new("TextButton")
SavePointButton.Name = "SavePointButton"
SavePointButton.Size = UDim2.new(1, -20, 0, 40)
SavePointButton.Position = UDim2.new(0, 10, 0, 45)
SavePointButton.BackgroundColor3 = Color3.fromRGB(40, 80, 40)
SavePointButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SavePointButton.Text = "💾 Salvar Ponto (JobId)"
SavePointButton.Font = Enum.Font.Gotham
SavePointButton.TextSize = 13
SavePointButton.Parent = MainFrame

local SaveCorner = Instance.new("UICorner")
SaveCorner.CornerRadius = UDim.new(0, 5)
SaveCorner.Parent = SavePointButton

-- Status Label (Feedback visual)
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Name = "Status"
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.Position = UDim2.new(0, 10, 0, 90)
StatusLabel.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
StatusLabel.Text = "Status: Aguardando..."
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 11
StatusLabel.Parent = MainFrame

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 4)
StatusCorner.Parent = StatusLabel

-- Botão "Relogar"
local RejoinButton = Instance.new("TextButton")
RejoinButton.Name = "RejoinButton"
RejoinButton.Size = UDim2.new(1, -20, 0, 40)
RejoinButton.Position = UDim2.new(0, 10, 0, 130)
RejoinButton.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
RejoinButton.TextColor3 = Color3.fromRGB(255, 255, 255)
RejoinButton.Text = "🔄 Relogar (Rejoin)"
RejoinButton.Font = Enum.Font.Gotham
RejoinButton.TextSize = 13
RejoinButton.Parent = MainFrame

local RejoinCorner = Instance.new("UICorner")
RejoinCorner.CornerRadius = UDim.new(0, 5)
RejoinCorner.Parent = RejoinButton

-- ==========================================
-- 2. LÓGICA DE FUNCIONALIDADE
-- ==========================================

-- Função para atualizar o Status na tela
local function updateStatus(text, color)
    StatusLabel.Text = "Status: " .. text
    if color then
        StatusLabel.TextColor3 = color
    else
        StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    end
end

-- Lógica do Botão "Salvar Ponto"
SavePointButton.MouseButton1Click:Connect(function()
    -- Captura o JobId atual (identificador único do servidor)
    savedJobId = game.JobId
    savedTimestamp = os.time()
    
    -- Atualiza feedback visual
    updateStatus("Ponto Salvo (Servidor ID: " .. string.sub(savedJobId, 1, 8) .. "...)", Color3.fromRGB(100, 255, 100))
    
    -- Explicação no console (para o desenvolvedor)
    print("[Reroll Manager] Ponto salvo. JobId: " .. tostring(savedJobId))
    print("[Reroll Manager] Lembre-se: O servidor ainda controla o save. Se o jogo salvar após o reroll, este ponto não reverterá os dados.")
end)

-- Lógica do Botão "Relogar"
RejoinButton.MouseButton1Click:Connect(function()
    updateStatus("Reconectando...", Color3.fromRGB(255, 200, 0))
    
    -- Tenta reconectar ao mesmo servidor se um ponto foi salvo
    -- Caso contrário, apenas teleporta para o mesmo jogo (novo servidor)
    local success, errorMessage = pcall(function()
        if savedJobId and savedJobId ~= "" then
            -- Tenta ir para o servidor específico salvo
            TeleportService:TeleportToPlaceInstance(game.PlaceId, savedJobId, LocalPlayer)
        else
            -- Fallback: Teleporta para o jogo (servidor aleatório/novo)
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end
    end)
    
    -- Tratamento de erro (ex: servidor cheio ou fechado)
    if not success then
        warn("[Reroll Manager] Falha no teleporte: " .. tostring(errorMessage))
        updateStatus("Falha no Rejoin. Tentando servidor novo...", Color3.fromRGB(255, 100, 100))
        
        -- Tentativa secundária: Teleporte simples
        pcall(function()
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end)
    end
end)

-- Lógica para Minimizar/Expandir o Painel
ToggleButton.MouseButton1Click:Connect(function()
    isPanelOpen = not isPanelOpen
    MainFrame.Visible = isPanelOpen
    -- Muda a cor do botão para indicar o estado
    ToggleButton.BackgroundColor3 = isPanelOpen and Color3.fromRGB(30, 30, 30) or Color3.fromRGB(50, 50, 50)
end)

-- Inicia com o painel visível
MainFrame.Visible = true

-- Feedback inicial
updateStatus("Pronto para uso.", Color3.fromRGB(150, 150, 150))
