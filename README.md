--[[
    Backrooms Company Script
    Funcionalidades:
    - ESP para Itens, Entidades e Jogadores com seleção de cor.
    - Stamina Infinita.
    - Atalhos: P para alternar ESP de Itens, I para abrir/fechar a UI.
]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Configurações
local ESP = {
    Items = { Enabled = false, Color = Color3.fromHex("#0dbf25") },
    Entities = { Enabled = false, Color = Color3.fromHex("#ff1100") },
    Players = { Enabled = false, Color = Color3.fromHex("#c452c4") }
}
local InfiniteStamina = { Enabled = false }
local UI = { Visible = true }

-- Objetos ESP ativos
local ESPObjects = { Items = {}, Entities = {}, Players = {} }

-- Funções auxiliares para ESP
local function ClearESP(type)
    for _, obj in ipairs(ESPObjects[type] or {}) do
        if obj and obj.Parent then obj:Destroy() end
    end
    ESPObjects[type] = {}
end

local function CreateChams(part, color)
    if not part or not part.Parent then return nil end
    local highlight = Instance.new("Highlight")
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 0.2
    highlight.OutlineTransparency = 0.2
    highlight.Adornee = part
    highlight.Parent = part
    return highlight
end

-- Criação da UI (idêntica à imagem)
local function CreateUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "BackroomScriptGUI"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local function createMainFrame(visible)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0, 350, 0, 200)
        frame.Position = UDim2.new(0.5, -175, 0.5, -100)
        frame.BackgroundColor3 = Color3.fromHex("#1a1a2e")
        frame.BorderSizePixel = 0
        frame.BackgroundTransparency = 0.1
        frame.Visible = visible
        frame.Parent = screenGui
        return frame
    end

    local mainFrame = createMainFrame(true)
    local staminaFrame = createMainFrame(false)

    -- Título principal (comum a ambos)
    local function createTitle(parent)
        local title = Instance.new("TextLabel")
        title.Size = UDim2.new(1, 0, 0, 30)
        title.Position = UDim2.new(0, 0, 0, 5)
        title.BackgroundTransparency = 1
        title.Text = "#BACKROOMSCRIPT"
        title.TextColor3 = Color3.fromHex("#00ff88")
        title.TextScaled = true
        title.Font = Enum.Font.Code
        title.Parent = parent
        return title
    end
    createTitle(mainFrame)
    createTitle(staminaFrame)

    -- Aba "Esp"
    local function createESPSection(parent, yPos)
        local section = Instance.new("Frame")
        section.Size = UDim2.new(1, 0, 0, 90)
        section.Position = UDim2.new(0, 0, 0, yPos)
        section.BackgroundTransparency = 1
        section.Parent = parent

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 0, 25)
        label.BackgroundTransparency = 1
        label.Text = "Esp"
        label.TextColor3 = Color3.fromHex("#ffffff")
        label.TextScaled = true
        label.Font = Enum.Font.Code
        label.Parent = section

        local function createOption(parent, yOffset, text, defaultColor, espKey)
            local optFrame = Instance.new("Frame")
            optFrame.Size = UDim2.new(1, 0, 0, 30)
            optFrame.Position = UDim2.new(0, 0, 0, yOffset)
            optFrame.BackgroundTransparency = 1
            optFrame.Parent = parent

            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(0.5, 0, 1, 0)
            label.Position = UDim2.new(0, 5, 0, 0)
            label.BackgroundTransparency = 1
            label.Text = text
            label.TextColor3 = Color3.fromHex("#ffffff")
            label.TextScaled = true
            label.TextXAlignment = Enum.TextXAlignment.Left
            label.Font = Enum.Font.Code
            label.Parent = optFrame

            local toggle = Instance.new("TextButton")
            toggle.Size = UDim2.new(0, 40, 0, 20)
            toggle.Position = UDim2.new(0.7, 0, 0.5, -10)
            toggle.BackgroundColor3 = Color3.fromHex("#333333")
            toggle.Text = "OFF"
            toggle.TextColor3 = Color3.fromHex("#ff0000")
            toggle.TextScaled = true
            toggle.Font = Enum.Font.Code
            toggle.Parent = optFrame

            local colorBtn = Instance.new("TextButton")
            colorBtn.Size = UDim2.new(0, 25, 0, 20)
            colorBtn.Position = UDim2.new(0.85, 0, 0.5, -10)
            colorBtn.BackgroundColor3 = defaultColor
            colorBtn.Text = ""
            colorBtn.BorderSizePixel = 1
            colorBtn.BorderColor3 = Color3.fromHex("#ffffff")
            colorBtn.Parent = optFrame

            -- Lógica do toggle
            toggle.MouseButton1Click:Connect(function()
                local isOn = toggle.Text == "ON"
                toggle.Text = isOn and "OFF" or "ON"
                toggle.TextColor3 = isOn and Color3.fromHex("#ff0000") or Color3.fromHex("#00ff00")
                toggle.BackgroundColor3 = isOn and Color3.fromHex("#333333") or Color3.fromHex("#00aa00")
                ESP[espKey].Enabled = not isOn
                if not isOn then ClearESP(espKey) end
            end)

            -- Seletor de cor
            colorBtn.MouseButton1Click:Connect(function()
                local picker = Instance.new("Frame")
                picker.Size = UDim2.new(0, 200, 0, 150)
                picker.Position = UDim2.new(0.5, -100, 0.5, -75)
                picker.BackgroundColor3 = Color3.fromHex("#1a1a2e")
                picker.BorderSizePixel = 0
                picker.Parent = screenGui

                local grid = Instance.new("Frame")
                grid.Size = UDim2.new(1, -10, 1, -40)
                grid.Position = UDim2.new(0, 5, 0, 5)
                grid.BackgroundTransparency = 1
                grid.Parent = picker

                local colors = {
                    Color3.fromHex("#ff0000"), Color3.fromHex("#00ff00"), Color3.fromHex("#0000ff"),
                    Color3.fromHex("#ffff00"), Color3.fromHex("#ff00ff"), Color3.fromHex("#00ffff"),
                    Color3.fromHex("#ffffff"), Color3.fromHex("#888888"), Color3.fromHex("#000000"),
                    Color3.fromHex("#ff8800"), Color3.fromHex("#88ff00"), Color3.fromHex("#0088ff")
                }

                for i, color in ipairs(colors) do
                    local btn = Instance.new("TextButton")
                    btn.Size = UDim2.new(0, 30, 0, 30)
                    btn.Position = UDim2.new(0, ((i-1) % 4) * 35 + 10, 0, math.floor((i-1) / 4) * 35 + 10)
                    btn.BackgroundColor3 = color
                    btn.Text = ""
                    btn.BorderSizePixel = 1
                    btn.BorderColor3 = Color3.fromHex("#ffffff")
                    btn.Parent = grid

                    btn.MouseButton1Click:Connect(function()
                        ESP[espKey].Color = color
                        colorBtn.BackgroundColor3 = color
                        picker:Destroy()
                        if ESP[espKey].Enabled then ClearESP(espKey) end
                    end)
                end

                local close = Instance.new("TextButton")
                close.Size = UDim2.new(0, 50, 0, 25)
                close.Position = UDim2.new(0.5, -25, 0, 120)
                close.BackgroundColor3 = Color3.fromHex("#ff0000")
                close.Text = "Fechar"
                close.TextColor3 = Color3.fromHex("#ffffff")
                close.TextScaled = true
                close.Font = Enum.Font.Code
                close.Parent = picker
                close.MouseButton1Click:Connect(function() picker:Destroy() end)
            end)

            return toggle
        end

        createOption(section, 25, "Visualizar itens", ESP.Items.Color, "Items")
        createOption(section, 55, "Visualizar entidades", ESP.Entities.Color, "Entities")
        createOption(section, 85, "Visualizar players", ESP.Players.Color, "Players")
    end
    createESPSection(mainFrame, 40)

    -- Aba "Stamina"
    local function createStaminaSection(parent)
        local section = Instance.new("Frame")
        section.Size = UDim2.new(1, 0, 0, 60)
        section.Position = UDim2.new(0, 0, 0, 40)
        section.BackgroundTransparency = 1
        section.Parent = parent

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 0, 25)
        label.BackgroundTransparency = 1
        label.Text = "Stamina"
        label.TextColor3 = Color3.fromHex("#ffffff")
        label.TextScaled = true
        label.Font = Enum.Font.Code
        label.Parent = section

        local optFrame = Instance.new("Frame")
        optFrame.Size = UDim2.new(1, 0, 0, 30)
        optFrame.Position = UDim2.new(0, 0, 0, 25)
        optFrame.BackgroundTransparency = 1
        optFrame.Parent = section

        local label2 = Instance.new("TextLabel")
        label2.Size = UDim2.new(0.5, 0, 1, 0)
        label2.Position = UDim2.new(0, 5, 0, 0)
        label2.BackgroundTransparency = 1
        label2.Text = "Stamina infinita"
        label2.TextColor3 = Color3.fromHex("#ffffff")
        label2.TextScaled = true
        label2.TextXAlignment = Enum.TextXAlignment.Left
        label2.Font = Enum.Font.Code
        label2.Parent = optFrame

        local toggle = Instance.new("TextButton")
        toggle.Size = UDim2.new(0, 40, 0, 20)
        toggle.Position = UDim2.new(0.7, 0, 0.5, -10)
        toggle.BackgroundColor3 = Color3.fromHex("#333333")
        toggle.Text = "OFF"
        toggle.TextColor3 = Color3.fromHex("#ff0000")
        toggle.TextScaled = true
        toggle.Font = Enum.Font.Code
        toggle.Parent = optFrame

        toggle.MouseButton1Click:Connect(function()
            local isOn = toggle.Text == "ON"
            toggle.Text = isOn and "OFF" or "ON"
            toggle.TextColor3 = isOn and Color3.fromHex("#ff0000") or Color3.fromHex("#00ff00")
            toggle.BackgroundColor3 = isOn and Color3.fromHex("#333333") or Color3.fromHex("#00aa00")
            InfiniteStamina.Enabled = not isOn
        end)
    end
    createStaminaSection(staminaFrame)

    -- Botões de navegação
    local function createNavButton(parent, text, xPos)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 80, 0, 25)
        btn.Position = UDim2.new(xPos, 0, 0, 5)
        btn.BackgroundColor3 = Color3.fromHex("#00ff88")
        btn.Text = text
        btn.TextColor3 = Color3.fromHex("#000000")
        btn.TextScaled = true
        btn.Font = Enum.Font.Code
        btn.Parent = parent
        return btn
    end

    local staminaNav = createNavButton(mainFrame, "Stamina", 0.8)
    local espNav = createNavButton(staminaFrame, "Esp", 0.8)

    staminaNav.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
        staminaFrame.Visible = true
    end)

    espNav.MouseButton1Click:Connect(function()
        staminaFrame.Visible = false
        mainFrame.Visible = true
    end)

    -- Atalho I para abrir/fechar UI
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.I then
            UI.Visible = not UI.Visible
            mainFrame.Visible = UI.Visible
            staminaFrame.Visible = false
        end
    end)

    return { mainFrame = mainFrame, staminaFrame = staminaFrame }
end

-- Inicializa a UI
CreateUI()

-- Loop principal para ESP e Stamina
local function MainLoop()
    while true do
        task.wait(0.5)

        -- Limpar ESPs desativados
        if not ESP.Items.Enabled then ClearESP("Items") end
        if not ESP.Entities.Enabled then ClearESP("Entities") end
        if not ESP.Players.Enabled then ClearESP("Players") end

        -- Processar ESPs
        if ESP.Items.Enabled then
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("BasePart") and obj.Parent and obj.Parent:IsA("Model") then
                    local model = obj.Parent
                    if model.Name:lower():match("item") or model.Name:lower():match("value") or 
                       model.Name:lower():match("collect") or model.Name:lower():match("pickup") then
                        local highlight = CreateChams(obj, ESP.Items.Color)
                        if highlight then table.insert(ESPObjects.Items, highlight) end
                    end
                end
            end
        end

        if ESP.Entities.Enabled then
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj:IsA("BasePart") and obj.Parent and obj.Parent:IsA("Model") then
                    local model = obj.Parent
                    if model:FindFirstChild("Humanoid") and model:FindFirstChild("HumanoidRootPart") then
                        if not Players:GetPlayerFromCharacter(model) then
                            local highlight = CreateChams(obj, ESP.Entities.Color)
                            if highlight then table.insert(ESPObjects.Entities, highlight) end
                        end
                    end
                end
            end
        end

        if ESP.Players.Enabled then
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local rootPart = player.Character.HumanoidRootPart
                    local highlight = CreateChams(rootPart, ESP.Players.Color)
                    if highlight then table.insert(ESPObjects.Players, highlight) end
                end
            end
        end

        -- Stamina Infinita
        if InfiniteStamina.Enabled then
            local char = LocalPlayer.Character
            if char then
                local humanoid = char:FindFirstChild("Humanoid")
                if humanoid then
                    if humanoid:FindFirstChild("Stamina") then
                        humanoid.Stamina.Value = 100
                    end
                    for _, obj in ipairs(char:GetDescendants()) do
                        if obj:IsA("NumberValue") and obj.Name:lower():match("stamina") then
                            obj.Value = 100
                        elseif obj:IsA("IntValue") and obj.Name:lower():match("stamina") then
                            obj.Value = 100
                        end
                    end
                end
            end
        end
    end
end

-- Inicia o loop em uma thread separada
coroutine.wrap(MainLoop)()
