--[[
    Interface #BACKROOMS SCRIPT
    Baseado no layout da imagem de referência.
    Desenvolvido para Roblox com Luau.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- =============== VARIÁVEIS GLOBAIS ===============
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local HeaderFrame = Instance.new("Frame")
local HeaderText = Instance.new("TextLabel")
local SidebarFrame = Instance.new("Frame")
local DividerLine = Instance.new("Frame")
local ContentFrame = Instance.new("Frame")

-- Elementos da sidebar
local EspButton = Instance.new("TextButton")
local StaminaButton = Instance.new("TextButton")

-- Elementos do conteúdo ESP
local EspContent = Instance.new("Frame")
local ItemsToggle = Instance.new("Frame")
local EntitiesToggle = Instance.new("Frame")
local PlayersToggle = Instance.new("Frame")

-- Elementos do conteúdo Stamina
local StaminaContent = Instance.new("Frame")
local InfiniteStaminaToggle = Instance.new("Frame")

-- Configurações
local Settings = {
    Esp = {
        Items = { Enabled = false, Color = Color3.fromRGB(13, 191, 37) },
        Entities = { Enabled = false, Color = Color3.fromRGB(255, 17, 0) },
        Players = { Enabled = false, Color = Color3.fromRGB(196, 82, 196) }
    },
    Stamina = {
        Infinite = false
    }
}

-- =============== FUNÇÕES AUXILIARES ===============
local function CreateToggle(parent, label, defaultColor)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 50)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local labelText = Instance.new("TextLabel")
    labelText.Size = UDim2.new(0.7, 0, 1, 0)
    labelText.BackgroundTransparency = 1
    labelText.Text = label
    labelText.TextColor3 = Color3.fromRGB(220, 220, 220)
    labelText.TextSize = 18
    labelText.TextXAlignment = Enum.TextXAlignment.Left
    labelText.Font = Enum.Font.Gotham
    labelText.Parent = frame

    local toggleContainer = Instance.new("Frame")
    toggleContainer.Size = UDim2.new(0, 100, 0, 34)
    toggleContainer.Position = UDim2.new(0.75, 0, 0.5, -17)
    toggleContainer.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    toggleContainer.BackgroundTransparency = 0.3
    toggleContainer.BorderSizePixel = 0
    toggleContainer.ClipsDescendants = true
    toggleContainer.Parent = frame

    local toggleBg = Instance.new("Frame")
    toggleBg.Size = UDim2.new(1, 0, 1, 0)
    toggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    toggleBg.BorderSizePixel = 0
    toggleBg.Parent = toggleContainer

    local toggleCircle = Instance.new("Frame")
    toggleCircle.Size = UDim2.new(0, 28, 0, 28)
    toggleCircle.Position = UDim2.new(0, 3, 0.5, -14)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    toggleCircle.BorderSizePixel = 0
    toggleCircle.Parent = toggleContainer

    local colorPicker = Instance.new("ImageButton")
    colorPicker.Size = UDim2.new(0, 28, 0, 28)
    colorPicker.Position = UDim2.new(0.92, 0, 0.5, -14)
    colorPicker.BackgroundColor3 = defaultColor or Color3.fromRGB(13, 191, 37)
    colorPicker.BorderSizePixel = 1
    colorPicker.BorderColor3 = Color3.fromRGB(255, 255, 255)
    colorPicker.Image = "rbxassetid://10505080859"
    colorPicker.Parent = frame

    return {
        Frame = frame,
        ToggleContainer = toggleContainer,
        ToggleBg = toggleBg,
        ToggleCircle = toggleCircle,
        ColorPicker = colorPicker,
        Label = labelText,
        State = false
    }
end

local function UpdateToggleVisual(toggle, state)
    toggle.State = state
    if state then
        toggle.ToggleBg.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
        toggle.ToggleCircle.Position = UDim2.new(0, 69, 0.5, -14)
    else
        toggle.ToggleBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        toggle.ToggleCircle.Position = UDim2.new(0, 3, 0.5, -14)
    end
end

local function TweenToggle(toggle, state)
    toggle.State = state
    local targetColor = state and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(60, 60, 60)
    local targetPos = state and UDim2.new(0, 69, 0.5, -14) or UDim2.new(0, 3, 0.5, -14)
    
    TweenService:Create(toggle.ToggleBg, TweenInfo.new(0.15), { BackgroundColor3 = targetColor }):Play()
    TweenService:Create(toggle.ToggleCircle, TweenInfo.new(0.15), { Position = targetPos }):Play()
end

-- =============== CONSTRUÇÃO DA INTERFACE ===============
ScreenGui.Name = "BackroomsScriptGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
MainFrame.Size = UDim2.new(0, 800, 0, 500)
MainFrame.Position = UDim2.new(0.5, -400, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(60, 55, 50)
MainFrame.BorderSizePixel = 3
MainFrame.BorderColor3 = Color3.fromRGB(255, 215, 0)
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Cabeçalho
HeaderFrame.Size = UDim2.new(1, 0, 0, 45)
HeaderFrame.BackgroundColor3 = Color3.fromRGB(45, 40, 35)
HeaderFrame.BorderSizePixel = 0
HeaderFrame.Parent = MainFrame

HeaderText.Size = UDim2.new(1, 0, 1, 0)
HeaderText.BackgroundTransparency = 1
HeaderText.Text = "#BACKROOMS SCRIPT"
HeaderText.TextColor3 = Color3.fromRGB(255, 215, 0)
HeaderText.TextSize = 22
HeaderText.Font = Enum.Font.GothamBold
HeaderText.TextXAlignment = Enum.TextXAlignment.Center
HeaderText.Parent = HeaderFrame

-- Linha divisória horizontal abaixo do cabeçalho
local topDivider = Instance.new("Frame")
topDivider.Size = UDim2.new(1, 0, 0, 2)
topDivider.Position = UDim2.new(0, 0, 0, 45)
topDivider.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
topDivider.BorderSizePixel = 0
topDivider.Parent = MainFrame

-- Sidebar
SidebarFrame.Size = UDim2.new(0, 220, 1, 0)
SidebarFrame.Position = UDim2.new(0, 0, 0, 47)
SidebarFrame.BackgroundColor3 = Color3.fromRGB(50, 45, 40)
SidebarFrame.BorderSizePixel = 0
SidebarFrame.Parent = MainFrame

-- Divisória vertical
DividerLine.Size = UDim2.new(0, 2, 1, 0)
DividerLine.Position = UDim2.new(0, 220, 0, 0)
DividerLine.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
DividerLine.BorderSizePixel = 0
DividerLine.Parent = MainFrame

-- Conteúdo principal
ContentFrame.Size = UDim2.new(1, -225, 1, -50)
ContentFrame.Position = UDim2.new(0, 225, 0, 50)
ContentFrame.BackgroundColor3 = Color3.fromRGB(40, 38, 35)
ContentFrame.BorderSizePixel = 0
ContentFrame.Parent = MainFrame

-- =============== SIDEBAR ===============
local function CreateSidebarButton(parent, text, position)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 45)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(50, 45, 40)
    button.BackgroundTransparency = 0.5
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(200, 200, 200)
    button.TextSize = 18
    button.Font = Enum.Font.Gotham
    button.TextXAlignment = Enum.TextXAlignment.Center
    button.Parent = parent
    return button
end

EspButton = CreateSidebarButton(SidebarFrame, "Esp", UDim2.new(0, 0, 0, 10))
StaminaButton = CreateSidebarButton(SidebarFrame, "Stamina", UDim2.new(0, 0, 0, 60))

-- Destaque inicial (Esp)
local EspHighlight = Instance.new("Frame")
EspHighlight.Size = UDim2.new(1, 0, 0, 45)
EspHighlight.Position = UDim2.new(0, 0, 0, 10)
EspHighlight.BackgroundColor3 = Color3.fromRGB(30, 28, 25)
EspHighlight.BorderSizePixel = 0
EspHighlight.Parent = SidebarFrame

-- =============== CONTEÚDO ESP ===============
EspContent.Size = UDim2.new(1, -20, 1, -20)
EspContent.Position = UDim2.new(0, 10, 0, 10)
EspContent.BackgroundTransparency = 1
EspContent.Parent = ContentFrame

local EspTitle = Instance.new("TextLabel")
EspTitle.Size = UDim2.new(1, 0, 0, 40)
EspTitle.BackgroundTransparency = 1
EspTitle.Text = "Visualizar"
EspTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
EspTitle.TextSize = 22
EspTitle.Font = Enum.Font.GothamBold
EspTitle.TextXAlignment = Enum.TextXAlignment.Left
EspTitle.Parent = EspContent

-- Item: Visualizar Itens
local ItemsToggleData = CreateToggle(EspContent, "Visualizar itens", Color3.fromRGB(13, 191, 37))
ItemsToggleData.Frame.Position = UDim2.new(0, 0, 0, 50)

ItemsToggleData.ToggleContainer.MouseButton1Click:Connect(function()
    local newState = not ItemsToggleData.State
    Settings.Esp.Items.Enabled = newState
    TweenToggle(ItemsToggleData, newState)
    
    -- Aplicar ESP de itens
    if newState then
        -- Lógica para destacar itens no mapa
        -- Exemplo: loop em partes com atributo "Item"
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part:GetAttribute("Item") then
                -- Aplicar Chams com transparência
                local highlight = Instance.new("Highlight")
                highlight.Adornee = part
                highlight.FillColor = Settings.Esp.Items.Color
                highlight.FillTransparency = 0.8
                highlight.OutlineTransparency = 0.5
                highlight.Parent = part
            end
        end
    else
        -- Remover highlights
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part:GetAttribute("Item") then
                local highlight = part:FindFirstChild("Highlight")
                if highlight then highlight:Destroy() end
            end
        end
    end
end)

ItemsToggleData.ColorPicker.MouseButton1Click:Connect(function()
    -- Abrir seletor de cor (simplificado)
    local color = Color3.fromRGB(13, 191, 37)
    -- Em implementação real, usar ColorPicker
    Settings.Esp.Items.Color = color
    ItemsToggleData.ColorPicker.BackgroundColor3 = color
end)

-- Item: Visualizar Entidades
local EntitiesToggleData = CreateToggle(EspContent, "Visualizar entidades", Color3.fromRGB(255, 17, 0))
EntitiesToggleData.Frame.Position = UDim2.new(0, 0, 0, 110)

EntitiesToggleData.ToggleContainer.MouseButton1Click:Connect(function()
    local newState = not EntitiesToggleData.State
    Settings.Esp.Entities.Enabled = newState
    TweenToggle(EntitiesToggleData, newState)
    
    if newState then
        -- Lógica para destacar entidades hostis
        for _, entity in pairs(workspace:GetDescendants()) do
            if entity:IsA("Model") and entity:GetAttribute("Hostile") then
                local highlight = Instance.new("Highlight")
                highlight.Adornee = entity
                highlight.FillColor = Settings.Esp.Entities.Color
                highlight.FillTransparency = 0.8
                highlight.OutlineTransparency = 0.5
                highlight.Parent = entity
            end
        end
    else
        for _, entity in pairs(workspace:GetDescendants()) do
            if entity:IsA("Model") and entity:GetAttribute("Hostile") then
                local highlight = entity:FindFirstChild("Highlight")
                if highlight then highlight:Destroy() end
            end
        end
    end
end)

EntitiesToggleData.ColorPicker.MouseButton1Click:Connect(function()
    Settings.Esp.Entities.Color = Color3.fromRGB(255, 17, 0)
    EntitiesToggleData.ColorPicker.BackgroundColor3 = Settings.Esp.Entities.Color
end)

-- Item: Visualizar Players
local PlayersToggleData = CreateToggle(EspContent, "Visualizar players", Color3.fromRGB(196, 82, 196))
PlayersToggleData.Frame.Position = UDim2.new(0, 0, 0, 170)

PlayersToggleData.ToggleContainer.MouseButton1Click:Connect(function()
    local newState = not PlayersToggleData.State
    Settings.Esp.Players.Enabled = newState
    TweenToggle(PlayersToggleData, newState)
    
    if newState then
        -- Lógica para destacar outros jogadores
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = Instance.new("Highlight")
                highlight.Adornee = player.Character
                highlight.FillColor = Settings.Esp.Players.Color
                highlight.FillTransparency = 0.8
                highlight.OutlineTransparency = 0.5
                highlight.Parent = player.Character
            end
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local highlight = player.Character:FindFirstChild("Highlight")
                if highlight then highlight:Destroy() end
            end
        end
    end
end)

PlayersToggleData.ColorPicker.MouseButton1Click:Connect(function()
    Settings.Esp.Players.Color = Color3.fromRGB(196, 82, 196)
    PlayersToggleData.ColorPicker.BackgroundColor3 = Settings.Esp.Players.Color
end)

-- =============== CONTEÚDO STAMINA ===============
StaminaContent.Size = UDim2.new(1, -20, 1, -20)
StaminaContent.Position = UDim2.new(0, 10, 0, 10)
StaminaContent.BackgroundTransparency = 1
StaminaContent.Visible = false
StaminaContent.Parent = ContentFrame

local StaminaTitle = Instance.new("TextLabel")
StaminaTitle.Size = UDim2.new(1, 0, 0, 40)
StaminaTitle.BackgroundTransparency = 1
StaminaTitle.Text = "Stamina"
StaminaTitle.TextColor3 = Color3.fromRGB(200, 200, 200)
StaminaTitle.TextSize = 22
StaminaTitle.Font = Enum.Font.GothamBold
StaminaTitle.TextXAlignment = Enum.TextXAlignment.Left
StaminaTitle.Parent = StaminaContent

-- Item: Stamina Infinita
local InfiniteStaminaData = CreateToggle(StaminaContent, "Stamina infinita", Color3.fromRGB(0, 200, 0))
InfiniteStaminaData.Frame.Position = UDim2.new(0, 0, 0, 50)
InfiniteStaminaData.ColorPicker.Visible = false

InfiniteStaminaData.ToggleContainer.MouseButton1Click:Connect(function()
    local newState = not InfiniteStaminaData.State
    Settings.Stamina.Infinite = newState
    TweenToggle(InfiniteStaminaData, newState)
    
    if newState then
        -- Lógica para stamina infinita
        RunService.Heartbeat:Connect(function()
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                local humanoid = LocalPlayer.Character.Humanoid
                humanoid.Stamina = humanoid.MaxStamina
            end
        end)
    end
end)

-- =============== NAVEGAÇÃO ===============
local function SwitchCategory(category)
    if category == "Esp" then
        EspContent.Visible = true
        StaminaContent.Visible = false
        EspHighlight.Position = UDim2.new(0, 0, 0, 10)
        EspButton.TextColor3 = Color3.fromRGB(255, 215, 0)
        StaminaButton.TextColor3 = Color3.fromRGB(200, 200, 200)
    else
        EspContent.Visible = false
        StaminaContent.Visible = true
        EspHighlight.Position = UDim2.new(0, 0, 0, 60)
        EspButton.TextColor3 = Color3.fromRGB(200, 200, 200)
        StaminaButton.TextColor3 = Color3.fromRGB(255, 215, 0)
    end
end

EspButton.MouseButton1Click:Connect(function()
    SwitchCategory("Esp")
end)

StaminaButton.MouseButton1Click:Connect(function()
    SwitchCategory("Stamina")
end)

-- =============== INICIALIZAÇÃO ===============
-- Definir estados iniciais
UpdateToggleVisual(ItemsToggleData, false)
UpdateToggleVisual(EntitiesToggleData, false)
UpdateToggleVisual(PlayersToggleData, false)
UpdateToggleVisual(InfiniteStaminaData, false)

-- Categoria inicial
SwitchCategory("Esp")

-- =============== REMOVER GUI (DEBUG) ===============
--[[
    Para remover a GUI durante desenvolvimento:
    ScreenGui:Destroy()
]]

print("[#BACKROOMS SCRIPT] Interface carregada com sucesso!")

-- =============== SISTEMA DE PERSISTÊNCIA (OPCIONAL) ===============
--[[
    Para salvar configurações entre sessões, use:
    local DataStore = game:GetService("DataStoreService"):GetDataStore("BackroomsScript")
    
    -- Salvar
    DataStore:SetAsync(LocalPlayer.UserId .. "_Settings", Settings)
    
    -- Carregar
    local saved = DataStore:GetAsync(LocalPlayer.UserId .. "_Settings")
    if saved then
        Settings = saved
        -- Aplicar configurações salvas
    end
]]
