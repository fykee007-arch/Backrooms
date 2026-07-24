--[[
    Advanced Roblox Automation Framework v1.0
    - ESP Chams para Players com Color Picker
    - UI Arrastável e Minimizável
    - Alto desempenho com otimizações
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local Config = {
    ESP = {
        Enabled = false,
        Color = Color3.fromRGB(0, 255, 0)
    }
}

-- Sistema de Otimização (Throttling)
local Throttle = {
    LastUpdate = 0,
    UpdateInterval = 0.1, -- Atualiza a cada 100ms
    ESPCache = {}
}

-- Gerenciador de ESP
local ESPManager = {
    ActiveHighlights = {},
    Enabled = false
}

-- Função para criar Highlight (Chams)
local function CreateHighlight(part, color)
    if not part or not part.Parent then return nil end
    
    local highlight = Instance.new("Highlight")
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 0.2 -- 80% transparente
    highlight.OutlineTransparency = 0.2
    highlight.Adornee = part
    highlight.Parent = part
    
    return highlight
end

-- Atualiza ESP para todos os players
local function UpdateESP()
    if not Config.ESP.Enabled then
        -- Limpa todos os highlights
        for _, highlight in ipairs(ESPManager.ActiveHighlights) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
        end
        ESPManager.ActiveHighlights = {}
        return
    end
    
    -- Remove highlights antigos
    for _, highlight in ipairs(ESPManager.ActiveHighlights) do
        if highlight and highlight.Parent then
            highlight:Destroy()
        end
    end
    ESPManager.ActiveHighlights = {}
    
    -- Aplica novos highlights
    local color = Config.ESP.Color
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local character = player.Character
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            local torso = character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso")
            
            local primaryPart = rootPart or torso
            if primaryPart then
                local highlight = CreateHighlight(primaryPart, color)
                if highlight then
                    table.insert(ESPManager.ActiveHighlights, highlight)
                end
            end
        end
    end
end

-- Loop otimizado para ESP
local function ESPLoop()
    while true do
        task.wait(Throttle.UpdateInterval)
        UpdateESP()
    end
end

-- ============================================
-- SISTEMA DE UI AVANÇADO
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
    
    -- Frame principal (arrastável)
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 320, 0, 150)
    mainFrame.Position = UDim2.new(0.5, -160, 0.5, -75)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    mainFrame.BackgroundTransparency = 0.15
    mainFrame.BorderSizePixel = 0
    mainFrame.ClipsDescendants = true
    mainFrame.Parent = screenGui
    
    -- Sombra
    local shadow = Instance.new("Frame")
    shadow.Size = UDim2.new(1, 10, 1, 10)
    shadow.Position = UDim2.new(0, -5, 0, -5)
    shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    shadow.BackgroundTransparency = 0.5
    shadow.BorderSizePixel = 0
    shadow.Parent = mainFrame
    
    -- Barra de título (para arrastar)
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    titleBar.BackgroundTransparency = 0.3
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    -- Título
    local titleText = Instance.new("TextLabel")
    titleText.Size = UDim2.new(0.6, 0, 1, 0)
    titleText.Position = UDim2.new(0, 10, 0, 0)
    titleText.BackgroundTransparency = 1
    titleText.Text = "⚡ Automation Framework"
    titleText.TextColor3 = Color3.fromRGB(0, 255, 180)
    titleText.TextScaled = true
    titleText.TextXAlignment = Enum.TextXAlignment.Left
    titleText.Font = Enum.Font.GothamBold
    titleText.Parent = titleBar
    
    -- Botão Minimizar
    local minButton = Instance.new("TextButton")
    minButton.Size = UDim2.new(0, 30, 1, 0)
    minButton.Position = UDim2.new(0.9, 0, 0, 0)
    minButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    minButton.BackgroundTransparency = 0.2
    minButton.Text = "−"
    minButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    minButton.TextScaled = true
    minButton.Font = Enum.Font.GothamBold
    minButton.BorderSizePixel = 0
    minButton.Parent = titleBar
    
    -- Botão Fechar (opcional)
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 1, 0)
    closeButton.Position = UDim2.new(0.95, 0, 0, 0)
    closeButton.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
    closeButton.BackgroundTransparency = 0.2
    closeButton.Text = "✕"
    closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
    closeButton.TextScaled = true
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BorderSizePixel = 0
    closeButton.Parent = titleBar
    
    -- Conteúdo da UI
    local contentFrame = Instance.new("Frame")
    contentFrame.Size = UDim2.new(1, -20, 1, -40)
    contentFrame.Position = UDim2.new(0, 10, 0, 35)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Label ESP
    local espLabel = Instance.new("TextLabel")
    espLabel.Size = UDim2.new(0.4, 0, 0, 30)
    espLabel.Position = UDim2.new(0, 0, 0.1, 0)
    espLabel.BackgroundTransparency = 1
    espLabel.Text = "ESP Players"
    espLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    espLabel.TextScaled = true
    espLabel.TextXAlignment = Enum.TextXAlignment.Left
    espLabel.Font = Enum.Font.Gotham
    espLabel.Parent = contentFrame
    
    -- Toggle ESP
    local espToggle = Instance.new("TextButton")
    espToggle.Size = UDim2.new(0, 50, 0, 25)
    espToggle.Position = UDim2.new(0.45, 0, 0.1, 0)
    espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    espToggle.Text = "OFF"
    espToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
    espToggle.TextScaled = true
    espToggle.Font = Enum.Font.GothamBold
    espToggle.BorderSizePixel = 0
    espToggle.Parent = contentFrame
    
    -- Color Picker Button
    local colorPickerBtn = Instance.new("TextButton")
    colorPickerBtn.Size = UDim2.new(0, 30, 0, 25)
    colorPickerBtn.Position = UDim2.new(0.65, 0, 0.1, 0)
    colorPickerBtn.BackgroundColor3 = Config.ESP.Color
    colorPickerBtn.Text = ""
    colorPickerBtn.BorderSizePixel = 2
    colorPickerBtn.BorderColor3 = Color3.fromRGB(255, 255, 255)
    colorPickerBtn.Parent = contentFrame
    
    -- Label de status
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 20)
    statusLabel.Position = UDim2.new(0, 0, 0.6, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Sistema: 💤 Inativo"
    statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
    statusLabel.TextScaled = true
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.Parent = contentFrame
    
    -- ============================================
    -- BOTÃO MINIMIZADO (aparece quando minimizado)
    -- ============================================
    local minimizedButton = Instance.new("TextButton")
    minimizedButton.Size = UDim2.new(0, 45, 0, 45)
    minimizedButton.Position = UDim2.new(0, 10, 0, 10)
    minimizedButton.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
    minimizedButton.BackgroundTransparency = 0.15
    minimizedButton.Text = "Z"
    minimizedButton.TextColor3 = Color3.fromRGB(0, 255, 180)
    minimizedButton.TextScaled = true
    minimizedButton.Font = Enum.Font.GothamBold
    minimizedButton.BorderSizePixel = 0
    minimizedButton.Visible = false
    minimizedButton.Parent = screenGui
    UI.MinimizedButton = minimizedButton
    
    -- ============================================
    -- LÓGICA DA UI
    -- ============================================
    
    -- Função para minimizar
    local function MinimizeUI()
        UI.Minimized = true
        mainFrame.Visible = false
        minimizedButton.Visible = true
        
        -- Tween para suavizar
        local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local tween = TweenService:Create(minimizedButton, tweenInfo, {
            BackgroundTransparency = 0.1,
            Size = UDim2.new(0, 50, 0, 50)
        })
        tween:Play()
    end
    
    -- Função para restaurar
    local function RestoreUI()
        UI.Minimized = false
        mainFrame.Visible = true
        minimizedButton.Visible = false
        
        local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local tween = TweenService:Create(mainFrame, tweenInfo, {
            BackgroundTransparency = 0.15
        })
        tween:Play()
    end
    
    -- Evento do botão minimizar
    minButton.MouseButton1Click:Connect(MinimizeUI)
    
    -- Evento do botão minimizado
    minimizedButton.MouseButton1Click:Connect(RestoreUI)
    
    -- Evento do botão fechar
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    -- ============================================
    -- SISTEMA DE ARRASTAR (com smoothing)
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
    
    -- Eventos de arrastar na barra de título
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
    
    -- Toggle ESP
    espToggle.MouseButton1Click:Connect(function()
        Config.ESP.Enabled = not Config.ESP.Enabled
        
        if Config.ESP.Enabled then
            espToggle.Text = "ON"
            espToggle.TextColor3 = Color3.fromRGB(80, 255, 80)
            espToggle.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
            statusLabel.Text = "Sistema: 🟢 Ativo"
            statusLabel.TextColor3 = Color3.fromRGB(80, 255, 80)
        else
            espToggle.Text = "OFF"
            espToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
            espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            statusLabel.Text = "Sistema: 💤 Inativo"
            statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
            -- Limpa ESP
            UpdateESP()
        end
    end)
    
    -- Color Picker
    colorPickerBtn.MouseButton1Click:Connect(function()
        -- Criar color picker
        local pickerFrame = Instance.new("Frame")
        pickerFrame.Size = UDim2.new(0, 250, 0, 200)
        pickerFrame.Position = UDim2.new(0.5, -125, 0.5, -100)
        pickerFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
        pickerFrame.BackgroundTransparency = 0.1
        pickerFrame.BorderSizePixel = 0
        pickerFrame.Parent = screenGui
        
        -- Título do picker
        local pickerTitle = Instance.new("TextLabel")
        pickerTitle.Size = UDim2.new(1, 0, 0, 30)
        pickerTitle.BackgroundTransparency = 1
        pickerTitle.Text = "Selecionar Cor"
        pickerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        pickerTitle.TextScaled = true
        pickerTitle.Font = Enum.Font.GothamBold
        pickerTitle.Parent = pickerFrame
        
        -- Grid de cores
        local gridFrame = Instance.new("Frame")
        gridFrame.Size = UDim2.new(1, -20, 1, -70)
        gridFrame.Position = UDim2.new(0, 10, 0, 35)
        gridFrame.BackgroundTransparency = 1
        gridFrame.Parent = pickerFrame
        
        -- Cores pré-definidas
        local colors = {
            Color3.fromRGB(255, 0, 0),
            Color3.fromRGB(0, 255, 0),
            Color3.fromRGB(0, 0, 255),
            Color3.fromRGB(255, 255, 0),
            Color3.fromRGB(255, 0, 255),
            Color3.fromRGB(0, 255, 255),
            Color3.fromRGB(255, 128, 0),
            Color3.fromRGB(128, 255, 0),
            Color3.fromRGB(0, 128, 255),
            Color3.fromRGB(255, 0, 128),
            Color3.fromRGB(128, 0, 255),
            Color3.fromRGB(0, 255, 128),
        }
        
        local buttonSize = 35
        local spacing = 5
        local perRow = 6
        
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
                
                -- Atualiza ESP imediatamente se estiver ativo
                if Config.ESP.Enabled then
                    UpdateESP()
                end
            end)
        end
        
        -- Botão fechar
        local closePicker = Instance.new("TextButton")
        closePicker.Size = UDim2.new(0, 80, 0, 25)
        closePicker.Position = UDim2.new(0.5, -40, 0, 170)
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

-- Criar UI
local ui = CreateUI()

-- Iniciar loop de ESP em thread separada
coroutine.wrap(ESPLoop)()

-- ============================================
-- SISTEMA DE SEGURANÇA E PERFORMANCE
-- ============================================

-- Limpeza automática quando o jogador sair
LocalPlayer.AncestryChanged:Connect(function()
    if not LocalPlayer.Parent then
        if ui.ScreenGui then
            ui.ScreenGui:Destroy()
        end
    end
end)

-- Prevenir memory leaks
game:GetService("CoreGui").ChildRemoved:Connect(function(child)
    if child.Name == "AutomationFramework" then
        -- Cleanup
        for _, highlight in ipairs(ESPManager.ActiveHighlights) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
        end
        ESPManager.ActiveHighlights = {}
    end
end)

print("⚡ Automation Framework carregado com sucesso!")
print("📌 Pressione 'Z' para restaurar a UI se minimizada")
