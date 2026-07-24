--[[
    Advanced Roblox Automation Framework v2.0
    - ESP Chams com múltiplos métodos de detecção
    - Sistema de debugging integrado
    - UI Arrastável e Minimizável
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local Config = {
    ESP = {
        Enabled = false,
        Color = Color3.fromRGB(0, 255, 0),
        Transparency = 0.2 -- 80% transparente
    },
    Debug = {
        Enabled = true -- Mostra mensagens de debug
    }
}

-- Sistema de Debug
local function DebugLog(message, ...)
    if Config.Debug.Enabled then
        print("[ESP Debug] " .. string.format(message, ...))
    end
end

-- Gerenciador de ESP
local ESPManager = {
    ActiveHighlights = {},
    TrackedPlayers = {},
    Enabled = false,
    LastUpdate = 0,
    UpdateInterval = 0.1
}

-- Função melhorada para criar Highlight
local function CreateHighlight(part, color, transparency)
    if not part or not part.Parent then 
        DebugLog("Parte inválida para highlight")
        return nil 
    end
    
    -- Verifica se já existe um highlight nessa parte
    local existingHighlight = part:FindFirstChild("ESP_Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = transparency or 0.2
    highlight.OutlineTransparency = transparency or 0.2
    highlight.Adornee = part
    highlight.Parent = part
    
    DebugLog("Highlight criado para: %s", part:GetFullName())
    return highlight
end

-- Função para encontrar o melhor objeto para o highlight
local function GetBestPart(character)
    if not character then return nil end
    
    -- Tenta encontrar partes principais
    local parts = {
        character:FindFirstChild("HumanoidRootPart"),
        character:FindFirstChild("UpperTorso"),
        character:FindFirstChild("LowerTorso"),
        character:FindFirstChild("Torso"),
        character:FindFirstChild("Head")
    }
    
    for _, part in ipairs(parts) do
        if part and part:IsA("BasePart") then
            return part
        end
    end
    
    -- Se não encontrou, procura qualquer BasePart
    for _, descendant in ipairs(character:GetDescendants()) do
        if descendant:IsA("BasePart") then
            return descendant
        end
    end
    
    return nil
end

-- Função para verificar se um modelo é um jogador válido
local function IsValidPlayerCharacter(model)
    if not model or not model:IsA("Model") then return false end
    if not model:FindFirstChild("Humanoid") then return false end
    
    -- Verifica se pertence a um jogador
    local player = Players:GetPlayerFromCharacter(model)
    if not player then return false end
    
    -- Não mostrar o próprio jogador
    if player == LocalPlayer then return false end
    
    return true
end

-- Atualiza ESP com método mais robusto
local function UpdateESP()
    local currentTime = tick()
    if currentTime - ESPManager.LastUpdate < ESPManager.UpdateInterval then
        return
    end
    ESPManager.LastUpdate = currentTime
    
    -- Limpa highlights antigos
    local cleanupCount = 0
    for i = #ESPManager.ActiveHighlights, 1, -1 do
        local highlight = ESPManager.ActiveHighlights[i]
        if not highlight or not highlight.Parent then
            table.remove(ESPManager.ActiveHighlights, i)
            cleanupCount = cleanupCount + 1
        end
    end
    if cleanupCount > 0 then
        DebugLog("Limpeza: %d highlights removidos", cleanupCount)
    end
    
    -- Se desativado, remove todos
    if not Config.ESP.Enabled then
        for _, highlight in ipairs(ESPManager.ActiveHighlights) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
        end
        ESPManager.ActiveHighlights = {}
        return
    end
    
    -- Verifica todos os players
    local players = Players:GetPlayers()
    local foundPlayers = 0
    
    for _, player in ipairs(players) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                local part = GetBestPart(character)
                if part then
                    -- Verifica se já tem highlight
                    local hasHighlight = false
                    for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                        if highlight and highlight.Adornee == part then
                            hasHighlight = true
                            break
                        end
                    end
                    
                    if not hasHighlight then
                        local highlight = CreateHighlight(part, Config.ESP.Color, Config.ESP.Transparency)
                        if highlight then
                            table.insert(ESPManager.ActiveHighlights, highlight)
                            foundPlayers = foundPlayers + 1
                            DebugLog("ESP ativado para: %s", player.Name)
                        end
                    end
                else
                    DebugLog("Nenhuma parte encontrada para: %s", player.Name)
                end
            end
        end
    end
    
    if foundPlayers > 0 then
        DebugLog("ESP ativado para %d jogadores", foundPlayers)
    end
end

-- Loop otimizado com detecção automática de novos players
local function ESPLoop()
    while true do
        task.wait(ESPManager.UpdateInterval)
        
        -- Verifica por novos players automaticamente
        if Config.ESP.Enabled then
            UpdateESP()
        end
    end
end

-- ============================================
-- SISTEMA DE UI
-- ============================================

local UI = {
    IsDragging = false,
    DragInput = nil,
    DragStart = nil,
    StartPos = nil,
    Minimized = false,
    ScreenGui = nil,
    MainFrame = nil,
    MinimizedButton = nil
}

local function CreateUI()
    -- ScreenGui principal
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AutomationFramework"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    UI.ScreenGui = screenGui
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 340, 0, 180)
    mainFrame.Position = UDim2.new(0.5, -170, 0.5, -90)
    mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.BorderSizePixel = 1
    mainFrame.BorderColor3 = Color3.fromRGB(50, 50, 70)
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1, 12, 1, 12)
    shadow.Position = UDim2.new(0, -6, 0, -6)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.6
    shadow.BorderSizePixel = 0
    shadow.ZIndex = -1
    shadow.Parent = mainFrame
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 32)
    titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    titleBar.BackgroundTransparency = 0.3
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    -- Título
    local titleText = Instance.new("TextLabel")
    titleText.Size = UDim2.new(0.6, 0, 1, 0)
    titleText.Position = UDim2.new(0, 10, 0, 0)
    titleText.BackgroundTransparency = 1
    titleText.Text = "⚡ ESP Framework"
    titleText.TextColor3 = Color3.fromRGB(0, 255, 180)
    titleText.TextScaled = true
    titleText.TextXAlignment = Enum.TextXAlignment.Left
    titleText.Font = Enum.Font.GothamBold
    titleText.Parent = titleBar
    
    -- Botão Minimizar
    local minButton = Instance.new("TextButton")
    minButton.Size = UDim2.new(0, 30, 1, 0)
    minButton.Position = UDim2.new(0.88, 0, 0, 0)
    minButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    minButton.BackgroundTransparency = 0.3
    minButton.Text = "−"
    minButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    minButton.TextScaled = true
    minButton.Font = Enum.Font.GothamBold
    minButton.BorderSizePixel = 0
    minButton.Parent = titleBar
    
    -- Botão Fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 1, 0)
    closeButton.Position = UDim2.new(0.94, 0, 0, 0)
    closeButton.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
    closeButton.BackgroundTransparency = 0.3
    closeButton.Text = "✕"
    closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    closeButton.TextScaled = true
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BorderSizePixel = 0
    closeButton.Parent = titleBar
    
    -- Conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Size = UDim2.new(1, -20, 1, -45)
    contentFrame.Position = UDim2.new(0, 10, 0, 38)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Label ESP
    local espLabel = Instance.new("TextLabel")
    espLabel.Size = UDim2.new(0.35, 0, 0, 30)
    espLabel.Position = UDim2.new(0, 0, 0.05, 0)
    espLabel.BackgroundTransparency = 1
    espLabel.Text = "ESP Players"
    espLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    espLabel.TextScaled = true
    espLabel.TextXAlignment = Enum.TextXAlignment.Left
    espLabel.Font = Enum.Font.Gotham
    espLabel.Parent = contentFrame
    
    -- Toggle ESP
    local espToggle = Instance.new("TextButton")
    espToggle.Size = UDim2.new(0, 55, 0, 28)
    espToggle.Position = UDim2.new(0.42, 0, 0.05, 0)
    espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    espToggle.Text = "OFF"
    espToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
    espToggle.TextScaled = true
    espToggle.Font = Enum.Font.GothamBold
    espToggle.BorderSizePixel = 0
    espToggle.Parent = contentFrame
    
    -- Color Picker Button
    local colorPickerBtn = Instance.new("TextButton")
    colorPickerBtn.Size = UDim2.new(0, 35, 0, 28)
    colorPickerBtn.Position = UDim2.new(0.62, 0, 0.05, 0)
    colorPickerBtn.BackgroundColor3 = Config.ESP.Color
    colorPickerBtn.Text = ""
    colorPickerBtn.BorderSizePixel = 2
    colorPickerBtn.BorderColor3 = Color3.fromRGB(255, 255, 255)
    colorPickerBtn.Parent = contentFrame
    
    -- Info label
    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(1, 0, 0, 25)
    infoLabel.Position = UDim2.new(0, 0, 0.5, 0)
    infoLabel.BackgroundTransparency = 1
    infoLabel.Text = "💡 Ative o ESP para ver jogadores"
    infoLabel.TextColor3 = Color3.fromRGB(150, 150, 180)
    infoLabel.TextScaled = true
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.Parent = contentFrame
    
    -- Status label
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 20)
    statusLabel.Position = UDim2.new(0, 0, 0.75, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Status: 💤 Aguardando"
    statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    statusLabel.TextScaled = true
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.Parent = contentFrame
    
    -- ============================================
    -- BOTÃO MINIMIZADO
    -- ============================================
    local minimizedButton = Instance.new("TextButton")
    minimizedButton.Size = UDim2.new(0, 50, 0, 50)
    minimizedButton.Position = UDim2.new(0, 10, 0, 10)
    minimizedButton.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
    minimizedButton.BackgroundTransparency = 0.1
    minimizedButton.Text = "Z"
    minimizedButton.TextColor3 = Color3.fromRGB(0, 255, 180)
    minimizedButton.TextScaled = true
    minimizedButton.Font = Enum.Font.GothamBold
    minimizedButton.BorderSizePixel = 1
    minimizedButton.BorderColor3 = Color3.fromRGB(50, 50, 70)
    minimizedButton.Visible = false
    minimizedButton.Parent = screenGui
    UI.MinimizedButton = minimizedButton
    
    -- ============================================
    -- LÓGICA DA UI
    -- ============================================
    
    local function MinimizeUI()
        UI.Minimized = true
        mainFrame.Visible = false
        minimizedButton.Visible = true
    end
    
    local function RestoreUI()
        UI.Minimized = false
        mainFrame.Visible = true
        minimizedButton.Visible = false
    end
    
    minButton.MouseButton1Click:Connect(MinimizeUI)
    minimizedButton.MouseButton1Click:Connect(RestoreUI)
    
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    -- ============================================
    -- SISTEMA DE ARRASTAR
    -- ============================================
    
    local function StartDrag(input)
        UI.IsDragging = true
        UI.DragStart = input.Position
        UI.StartPos = mainFrame.Position
        
        UI.DragInput = UserInputService.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                local delta = input.Position - UI.DragStart
                mainFrame.Position = UDim2.new(
                    UI.StartPos.X.Scale,
                    UI.StartPos.X.Offset + delta.X,
                    UI.StartPos.Y.Scale,
                    UI.StartPos.Y.Offset + delta.Y
                )
            end
        end)
    end
    
    local function EndDrag()
        if UI.DragInput then
            UI.DragInput:Disconnect()
            UI.DragInput = nil
        end
        UI.IsDragging = false
    end
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            StartDrag(input)
        end
    end)
    
    titleBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            EndDrag()
        end
    end)
    
    -- ============================================
    -- LÓGICA DOS BOTÕES
    -- ============================================
    
    espToggle.MouseButton1Click:Connect(function()
        Config.ESP.Enabled = not Config.ESP.Enabled
        
        if Config.ESP.Enabled then
            espToggle.Text = "ON"
            espToggle.TextColor3 = Color3.fromRGB(80, 255, 80)
            espToggle.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
            statusLabel.Text = "Status: 🟢 Ativo - Procurando jogadores..."
            statusLabel.TextColor3 = Color3.fromRGB(80, 255, 80)
            infoLabel.Text = "👀 ESP ativo! Procurando jogadores..."
            DebugLog("ESP ativado pelo usuário")
            
            -- Atualiza imediatamente
            UpdateESP()
        else
            espToggle.Text = "OFF"
            espToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
            espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            statusLabel.Text = "Status: 💤 Desativado"
            statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
            infoLabel.Text = "💡 Ative o ESP para ver jogadores"
            DebugLog("ESP desativado pelo usuário")
            
            -- Limpa tudo
            for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                if highlight and highlight.Parent then
                    highlight:Destroy()
                end
            end
            ESPManager.ActiveHighlights = {}
        end
    end)
    
    -- Color Picker
    colorPickerBtn.MouseButton1Click:Connect(function()
        local pickerFrame = Instance.new("Frame")
        pickerFrame.Size = UDim2.new(0, 280, 0, 220)
        pickerFrame.Position = UDim2.new(0.5, -140, 0.5, -110)
        pickerFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
        pickerFrame.BackgroundTransparency = 0.1
        pickerFrame.BorderSizePixel = 1
        pickerFrame.BorderColor3 = Color3.fromRGB(50, 50, 70)
        pickerFrame.Parent = screenGui
        
        local pickerTitle = Instance.new("TextLabel")
        pickerTitle.Size = UDim2.new(1, 0, 0, 35)
        pickerTitle.BackgroundTransparency = 1
        pickerTitle.Text = "🎨 Selecionar Cor"
        pickerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        pickerTitle.TextScaled = true
        pickerTitle.Font = Enum.Font.GothamBold
        pickerTitle.Parent = pickerFrame
        
        local gridFrame = Instance.new("Frame")
        gridFrame.Size = UDim2.new(1, -20, 1, -75)
        gridFrame.Position = UDim2.new(0, 10, 0, 40)
        gridFrame.BackgroundTransparency = 1
        gridFrame.Parent = pickerFrame
        
        local colors = {
            Color3.fromRGB(255, 0, 0), Color3.fromRGB(0, 255, 0),
            Color3.fromRGB(0, 0, 255), Color3.fromRGB(255, 255, 0),
            Color3.fromRGB(255, 0, 255), Color3.fromRGB(0, 255, 255),
            Color3.fromRGB(255, 128, 0), Color3.fromRGB(128, 255, 0),
            Color3.fromRGB(0, 128, 255), Color3.fromRGB(255, 0, 128),
            Color3.fromRGB(128, 0, 255), Color3.fromRGB(0, 255, 128),
            Color3.fromRGB(255, 255, 255), Color3.fromRGB(100, 100, 100),
            Color3.fromRGB(255, 200, 0), Color3.fromRGB(200, 0, 255),
        }
        
        local buttonSize = 35
        local spacing = 5
        local perRow = 7
        
        for i, color in ipairs(colors) do
            local btn = Instance.new("TextButton")
            local row = math.floor((i-1) / perRow)
            local col = (i-1) % perRow
            
            btn.Size = UDim2.new(0, buttonSize, 0, buttonSize)
            btn.Position = UDim2.new(0, col * (buttonSize + spacing), 0, row * (buttonSize + spacing))
            btn.BackgroundColor3 = color
            btn.Text = ""
            btn.BorderSizePixel = 1
            btn.BorderColor3 = Color3.fromRGB(255, 255, 255)
            btn.Parent = gridFrame
            
            btn.MouseButton1Click:Connect(function()
                Config.ESP.Color = color
                colorPickerBtn.BackgroundColor3 = color
                pickerFrame:Destroy()
                
                -- Atualiza ESP se ativo
                if Config.ESP.Enabled then
                    -- Remove highlights antigos
                    for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                        if highlight and highlight.Parent then
                            highlight:Destroy()
                        end
                    end
                    ESPManager.ActiveHighlights = {}
                    DebugLog("Cor alterada para: %s", tostring(color))
                    UpdateESP()
                end
            end)
        end
        
        local closePicker = Instance.new("TextButton")
        closePicker.Size = UDim2.new(0, 100, 0, 30)
        closePicker.Position = UDim2.new(0.5, -50, 0, 185)
        closePicker.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        closePicker.Text = "Fechar"
        closePicker.TextColor3 = Color3.fromRGB(255, 255, 255)
        closePicker.TextScaled = true
        closePicker.Font = Enum.Font.Gotham
        closePicker.BorderSizePixel = 0
        closePicker.Parent = pickerFrame
        
        closePicker.MouseButton1Click:Connect(function()
            pickerFrame:Destroy()
        end)
    end)
    
    return {
        MainFrame = mainFrame,
        MinimizedButton = minimizedButton,
        ScreenGui = screenGui
    }
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================

-- Função de diagnóstico
local function DiagnoseEnvironment()
    DebugLog("=== Diagnóstico do Ambiente ===")
    DebugLog("Jogador: %s", LocalPlayer and LocalPlayer.Name or "N/A")
    DebugLog("Players conectados: %d", #Players:GetPlayers())
    DebugLog("Workspace: %s", Workspace and "Disponível" or "N/A")
    DebugLog("Camera: %s", Camera and "Disponível" or "N/A")
    
    -- Lista jogadores
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local char = player.Character
            DebugLog("Jogador: %s - Personagem: %s", player.Name, char and "Sim" or "Não")
            if char then
                local parts = char:GetDescendants()
                local baseParts = 0
                for _, obj in ipairs(parts) do
                    if obj:IsA("BasePart") then
                        baseParts = baseParts + 1
                    end
                end
                DebugLog("  Partes Base: %d", baseParts)
            end
        end
    end
    DebugLog("================================")
end

-- Criar UI
local ui = CreateUI()

-- Iniciar diagnóstico
DiagnoseEnvironment()

-- Iniciar loop de ESP
coroutine.wrap(ESPLoop)()

-- ============================================
-- SISTEMA DE MONITORAMENTO
-- ============================================

-- Monitora novos players entrando
Players.PlayerAdded:Connect(function(player)
    DebugLog("Novo jogador entrou: %s", player.Name)
    if Config.ESP.Enabled then
        task.wait(1) -- Aguarda o personagem carregar
        UpdateESP()
    end
end)

-- Monitora quando um personagem carrega
Players.PlayerCharacterAdded:Connect(function(player)
    if player == LocalPlayer then return end
    DebugLog("Personagem carregado para: %s", player.Name)
    if Config.ESP.Enabled then
        task.wait(0.5)
        UpdateESP()
    end
end)

-- Atualização automática quando jogadores entram/saem
Players.PlayerRemoving:Connect(function(player)
    DebugLog("Jogador saiu: %s", player.Name)
    if Config.ESP.Enabled then
        UpdateESP()
    end
end)

-- Limpeza automática
LocalPlayer.AncestryChanged:Connect(function()
    if not LocalPlayer.Parent then
        if ui.ScreenGui then
            ui.ScreenGui:Destroy()
        end
        for _, highlight in ipairs(ESPManager.ActiveHighlights) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
        end
        ESPManager.ActiveHighlights = {}
    end
end)

print("⚡ ESP Framework v2.0 carregado com sucesso!")
print("📌 Pressione 'Z' para restaurar a UI se minimizada")
print("🔍 Verifique o console para mensagens de debug")
