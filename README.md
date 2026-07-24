local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESP = {
    Items = {
        Enabled = false,
        Color = Color3.fromHex("#0dbf25")
    },
    Entities = {
        Enabled = false,
        Color = Color3.fromHex("#ff1100")
    },
    Players = {
        Enabled = false,
        Color = Color3.fromHex("#c452c4")
    }
}

local InfiniteStamina = {
    Enabled = false
}

local ESPObjects = {
    Items = {},
    Entities = {},
    Players = {}
}

local function CreateUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "BackroomScriptGUI"
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 350, 0, 200)
    mainFrame.Position = UDim2.new(0.5, -175, 0.5, -100)
    mainFrame.BackgroundColor3 = Color3.fromHex("#1a1a2e")
    mainFrame.BorderSizePixel = 0
    mainFrame.BackgroundTransparency = 0.1
    mainFrame.Parent = screenGui
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "#BACKROOMSCRIPT"
    title.TextColor3 = Color3.fromHex("#00ff88")
    title.TextScaled = true
    title.Font = Enum.Font.Code
    title.Parent = mainFrame
    
    local espLabel = Instance.new("TextLabel")
    espLabel.Size = UDim2.new(1, 0, 0, 25)
    espLabel.Position = UDim2.new(0, 0, 0, 40)
    espLabel.BackgroundTransparency = 1
    espLabel.Text = "Esp"
    espLabel.TextColor3 = Color3.fromHex("#ffffff")
    espLabel.TextScaled = true
    espLabel.Font = Enum.Font.Code
    espLabel.Parent = mainFrame
    
    local itemsFrame = Instance.new("Frame")
    itemsFrame.Size = UDim2.new(1, 0, 0, 30)
    itemsFrame.Position = UDim2.new(0, 0, 0, 70)
    itemsFrame.BackgroundTransparency = 1
    itemsFrame.Parent = mainFrame
    
    local itemsLabel = Instance.new("TextLabel")
    itemsLabel.Size = UDim2.new(0.5, 0, 1, 0)
    itemsLabel.Position = UDim2.new(0, 5, 0, 0)
    itemsLabel.BackgroundTransparency = 1
    itemsLabel.Text = "Visualizar itens"
    itemsLabel.TextColor3 = Color3.fromHex("#ffffff")
    itemsLabel.TextScaled = true
    itemsLabel.TextXAlignment = Enum.TextXAlignment.Left
    itemsLabel.Font = Enum.Font.Code
    itemsLabel.Parent = itemsFrame
    
    local itemsToggle = Instance.new("TextButton")
    itemsToggle.Size = UDim2.new(0, 40, 0, 20)
    itemsToggle.Position = UDim2.new(0.7, 0, 0.5, -10)
    itemsToggle.BackgroundColor3 = Color3.fromHex("#333333")
    itemsToggle.Text = "OFF"
    itemsToggle.TextColor3 = Color3.fromHex("#ff0000")
    itemsToggle.TextScaled = true
    itemsToggle.Font = Enum.Font.Code
    itemsToggle.Parent = itemsFrame
    
    local itemsColor = Instance.new("TextButton")
    itemsColor.Size = UDim2.new(0, 25, 0, 20)
    itemsColor.Position = UDim2.new(0.85, 0, 0.5, -10)
    itemsColor.BackgroundColor3 = Color3.fromHex("#0dbf25")
    itemsColor.Text = ""
    itemsColor.BorderSizePixel = 1
    itemsColor.BorderColor3 = Color3.fromHex("#ffffff")
    itemsColor.Parent = itemsFrame
    
    local entitiesFrame = Instance.new("Frame")
    entitiesFrame.Size = UDim2.new(1, 0, 0, 30)
    entitiesFrame.Position = UDim2.new(0, 0, 0, 105)
    entitiesFrame.BackgroundTransparency = 1
    entitiesFrame.Parent = mainFrame
    
    local entitiesLabel = Instance.new("TextLabel")
    entitiesLabel.Size = UDim2.new(0.5, 0, 1, 0)
    entitiesLabel.Position = UDim2.new(0, 5, 0, 0)
    entitiesLabel.BackgroundTransparency = 1
    entitiesLabel.Text = "Visualizar entidades"
    entitiesLabel.TextColor3 = Color3.fromHex("#ffffff")
    entitiesLabel.TextScaled = true
    entitiesLabel.TextXAlignment = Enum.TextXAlignment.Left
    entitiesLabel.Font = Enum.Font.Code
    entitiesLabel.Parent = entitiesFrame
    
    local entitiesToggle = Instance.new("TextButton")
    entitiesToggle.Size = UDim2.new(0, 40, 0, 20)
    entitiesToggle.Position = UDim2.new(0.7, 0, 0.5, -10)
    entitiesToggle.BackgroundColor3 = Color3.fromHex("#333333")
    entitiesToggle.Text = "OFF"
    entitiesToggle.TextColor3 = Color3.fromHex("#ff0000")
    entitiesToggle.TextScaled = true
    entitiesToggle.Font = Enum.Font.Code
    entitiesToggle.Parent = entitiesFrame
    
    local entitiesColor = Instance.new("TextButton")
    entitiesColor.Size = UDim2.new(0, 25, 0, 20)
    entitiesColor.Position = UDim2.new(0.85, 0, 0.5, -10)
    entitiesColor.BackgroundColor3 = Color3.fromHex("#ff1100")
    entitiesColor.Text = ""
    entitiesColor.BorderSizePixel = 1
    entitiesColor.BorderColor3 = Color3.fromHex("#ffffff")
    entitiesColor.Parent = entitiesFrame
    
    local playersFrame = Instance.new("Frame")
    playersFrame.Size = UDim2.new(1, 0, 0, 30)
    playersFrame.Position = UDim2.new(0, 0, 0, 140)
    playersFrame.BackgroundTransparency = 1
    playersFrame.Parent = mainFrame
    
    local playersLabel = Instance.new("TextLabel")
    playersLabel.Size = UDim2.new(0.5, 0, 1, 0)
    playersLabel.Position = UDim2.new(0, 5, 0, 0)
    playersLabel.BackgroundTransparency = 1
    playersLabel.Text = "Visualizar players"
    playersLabel.TextColor3 = Color3.fromHex("#ffffff")
    playersLabel.TextScaled = true
    playersLabel.TextXAlignment = Enum.TextXAlignment.Left
    playersLabel.Font = Enum.Font.Code
    playersLabel.Parent = playersFrame
    
    local playersToggle = Instance.new("TextButton")
    playersToggle.Size = UDim2.new(0, 40, 0, 20)
    playersToggle.Position = UDim2.new(0.7, 0, 0.5, -10)
    playersToggle.BackgroundColor3 = Color3.fromHex("#333333")
    playersToggle.Text = "OFF"
    playersToggle.TextColor3 = Color3.fromHex("#ff0000")
    playersToggle.TextScaled = true
    playersToggle.Font = Enum.Font.Code
    playersToggle.Parent = playersFrame
    
    local playersColor = Instance.new("TextButton")
    playersColor.Size = UDim2.new(0, 25, 0, 20)
    playersColor.Position = UDim2.new(0.85, 0, 0.5, -10)
    playersColor.BackgroundColor3 = Color3.fromHex("#c452c4")
    playersColor.Text = ""
    playersColor.BorderSizePixel = 1
    playersColor.BorderColor3 = Color3.fromHex("#ffffff")
    playersColor.Parent = playersFrame
    
    local staminaButton = Instance.new("TextButton")
    staminaButton.Size = UDim2.new(0, 80, 0, 25)
    staminaButton.Position = UDim2.new(0.8, 0, 0, 5)
    staminaButton.BackgroundColor3 = Color3.fromHex("#00ff88")
    staminaButton.Text = "Stamina"
    staminaButton.TextColor3 = Color3.fromHex("#000000")
    staminaButton.TextScaled = true
    staminaButton.Font = Enum.Font.Code
    staminaButton.Parent = mainFrame
    
    local staminaFrame = Instance.new("Frame")
    staminaFrame.Name = "StaminaFrame"
    staminaFrame.Size = UDim2.new(0, 350, 0, 200)
    staminaFrame.Position = UDim2.new(0.5, -175, 0.5, -100)
    staminaFrame.BackgroundColor3 = Color3.fromHex("#1a1a2e")
    staminaFrame.BorderSizePixel = 0
    staminaFrame.BackgroundTransparency = 0.1
    staminaFrame.Visible = false
    staminaFrame.Parent = screenGui
    
    local staminaTitle = Instance.new("TextLabel")
    staminaTitle.Size = UDim2.new(1, 0, 0, 30)
    staminaTitle.Position = UDim2.new(0, 0, 0, 5)
    staminaTitle.BackgroundTransparency = 1
    staminaTitle.Text = "#BACKROOMSCRIPT"
    staminaTitle.TextColor3 = Color3.fromHex("#00ff88")
    staminaTitle.TextScaled = true
    staminaTitle.Font = Enum.Font.Code
    staminaTitle.Parent = staminaFrame
    
    local staminaLabel = Instance.new("TextLabel")
    staminaLabel.Size = UDim2.new(1, 0, 0, 25)
    staminaLabel.Position = UDim2.new(0, 0, 0, 40)
    staminaLabel.BackgroundTransparency = 1
    staminaLabel.Text = "Stamina"
    staminaLabel.TextColor3 = Color3.fromHex("#ffffff")
    staminaLabel.TextScaled = true
    staminaLabel.Font = Enum.Font.Code
    staminaLabel.Parent = staminaFrame
    
    local infiniteStaminaFrame = Instance.new("Frame")
    infiniteStaminaFrame.Size = UDim2.new(1, 0, 0, 30)
    infiniteStaminaFrame.Position = UDim2.new(0, 0, 0, 70)
    infiniteStaminaFrame.BackgroundTransparency = 1
    infiniteStaminaFrame.Parent = staminaFrame
    
    local infiniteStaminaLabel = Instance.new("TextLabel")
    infiniteStaminaLabel.Size = UDim2.new(0.5, 0, 1, 0)
    infiniteStaminaLabel.Position = UDim2.new(0, 5, 0, 0)
    infiniteStaminaLabel.BackgroundTransparency = 1
    infiniteStaminaLabel.Text = "Stamina infinita"
    infiniteStaminaLabel.TextColor3 = Color3.fromHex("#ffffff")
    infiniteStaminaLabel.TextScaled = true
    infiniteStaminaLabel.TextXAlignment = Enum.TextXAlignment.Left
    infiniteStaminaLabel.Font = Enum.Font.Code
    infiniteStaminaLabel.Parent = infiniteStaminaFrame
    
    local infiniteStaminaToggle = Instance.new("TextButton")
    infiniteStaminaToggle.Size = UDim2.new(0, 40, 0, 20)
    infiniteStaminaToggle.Position = UDim2.new(0.7, 0, 0.5, -10)
    infiniteStaminaToggle.BackgroundColor3 = Color3.fromHex("#333333")
    infiniteStaminaToggle.Text = "OFF"
    infiniteStaminaToggle.TextColor3 = Color3.fromHex("#ff0000")
    infiniteStaminaToggle.TextScaled = true
    infiniteStaminaToggle.Font = Enum.Font.Code
    infiniteStaminaToggle.Parent = infiniteStaminaFrame
    
    local espButton = Instance.new("TextButton")
    espButton.Size = UDim2.new(0, 80, 0, 25)
    espButton.Position = UDim2.new(0.8, 0, 0, 5)
    espButton.BackgroundColor3 = Color3.fromHex("#00ff88")
    espButton.Text = "Esp"
    espButton.TextColor3 = Color3.fromHex("#000000")
    espButton.TextScaled = true
    espButton.Font = Enum.Font.Code
    espButton.Parent = staminaFrame
    
    local function ToggleESP(toggle, espType)
        local isOn = toggle.Text == "ON"
        toggle.Text = isOn and "OFF" or "ON"
        toggle.TextColor3 = isOn and Color3.fromHex("#ff0000") or Color3.fromHex("#00ff00")
        toggle.BackgroundColor3 = isOn and Color3.fromHex("#333333") or Color3.fromHex("#00aa00")
        ESP[espType].Enabled = not isOn
        
        if not isOn then
            ClearESP(espType)
        end
    end
    
    local function ToggleStamina()
        local isOn = infiniteStaminaToggle.Text == "ON"
        infiniteStaminaToggle.Text = isOn and "OFF" or "ON"
        infiniteStaminaToggle.TextColor3 = isOn and Color3.fromHex("#ff0000") or Color3.fromHex("#00ff00")
        infiniteStaminaToggle.BackgroundColor3 = isOn and Color3.fromHex("#333333") or Color3.fromHex("#00aa00")
        InfiniteStamina.Enabled = not isOn
    end
    
    itemsToggle.MouseButton1Click:Connect(function()
        ToggleESP(itemsToggle, "Items")
    end)
    
    entitiesToggle.MouseButton1Click:Connect(function()
        ToggleESP(entitiesToggle, "Entities")
    end)
    
    playersToggle.MouseButton1Click:Connect(function()
        ToggleESP(playersToggle, "Players")
    end)
    
    infiniteStaminaToggle.MouseButton1Click:Connect(function()
        ToggleStamina()
    end)
    
    local function SetupColorPicker(button, espType)
        button.MouseButton1Click:Connect(function()
            local colorPicker = Instance.new("Frame")
            colorPicker.Size = UDim2.new(0, 200, 0, 150)
            colorPicker.Position = UDim2.new(0.5, -100, 0.5, -75)
            colorPicker.BackgroundColor3 = Color3.fromHex("#1a1a2e")
            colorPicker.BorderSizePixel = 0
            colorPicker.Parent = screenGui
            
            local colorGrid = Instance.new("Frame")
            colorGrid.Size = UDim2.new(1, -10, 1, -40)
            colorGrid.Position = UDim2.new(0, 5, 0, 5)
            colorGrid.BackgroundTransparency = 1
            colorGrid.Parent = colorPicker
            
            local colors = {
                Color3.fromHex("#ff0000"), Color3.fromHex("#00ff00"), Color3.fromHex("#0000ff"),
                Color3.fromHex("#ffff00"), Color3.fromHex("#ff00ff"), Color3.fromHex("#00ffff"),
                Color3.fromHex("#ffffff"), Color3.fromHex("#888888"), Color3.fromHex("#000000"),
                Color3.fromHex("#ff8800"), Color3.fromHex("#88ff00"), Color3.fromHex("#0088ff")
            }
            
            for i, color in ipairs(colors) do
                local colorBtn = Instance.new("TextButton")
                colorBtn.Size = UDim2.new(0, 30, 0, 30)
                colorBtn.Position = UDim2.new(0, ((i-1) % 4) * 35 + 10, 0, math.floor((i-1) / 4) * 35 + 10)
                colorBtn.BackgroundColor3 = color
                colorBtn.Text = ""
                colorBtn.BorderSizePixel = 1
                colorBtn.BorderColor3 = Color3.fromHex("#ffffff")
                colorBtn.Parent = colorGrid
                
                colorBtn.MouseButton1Click:Connect(function()
                    ESP[espType].Color = color
                    button.BackgroundColor3 = color
                    colorPicker:Destroy()
                    
                    if ESP[espType].Enabled then
                        ClearESP(espType)
                    end
                end)
            end
            
            local closeBtn = Instance.new("TextButton")
            closeBtn.Size = UDim2.new(0, 50, 0, 25)
            closeBtn.Position = UDim2.new(0.5, -25, 0, 120)
            closeBtn.BackgroundColor3 = Color3.fromHex("#ff0000")
            closeBtn.Text = "Fechar"
            closeBtn.TextColor3 = Color3.fromHex("#ffffff")
            closeBtn.TextScaled = true
            closeBtn.Font = Enum.Font.Code
            closeBtn.Parent = colorPicker
            
            closeBtn.MouseButton1Click:Connect(function()
                colorPicker:Destroy()
            end)
        end)
    end
    
    SetupColorPicker(itemsColor, "Items")
    SetupColorPicker(entitiesColor, "Entities")
    SetupColorPicker(playersColor, "Players")
    
    staminaButton.MouseButton1Click:Connect(function()
        mainFrame.Visible = false
        staminaFrame.Visible = true
    end)
    
    espButton.MouseButton1Click:Connect(function()
        staminaFrame.Visible = false
        mainFrame.Visible = true
    end)
    
    return {
        MainFrame = mainFrame,
        StaminaFrame = staminaFrame,
        ItemsToggle = itemsToggle,
        EntitiesToggle = entitiesToggle,
        PlayersToggle = playersToggle,
        InfiniteStaminaToggle = infiniteStaminaToggle,
        ItemsColor = itemsColor,
        EntitiesColor = entitiesColor,
        PlayersColor = playersColor
    }
end

local function ClearESP(type)
    for _, obj in ipairs(ESPObjects[type]) do
        if obj and obj.Parent then
            obj:Destroy()
        end
    end
    ESPObjects[type] = {}
end

local function CreateChams(part, color)
    if not part or not part.Parent then return end
    
    local highlight = Instance.new("Highlight")
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 0.2
    highlight.OutlineTransparency = 0.2
    highlight.Adornee = part
    highlight.Parent = part
    return highlight
end

local function ProcessEntities()
    if ESP.Items.Enabled then
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Parent and obj.Parent:IsA("Model") then
                local model = obj.Parent
                if model.Name:lower():match("item") or model.Name:lower():match("value") or 
                   model.Name:lower():match("collect") or model.Name:lower():match("pickup") then
                    local highlight = CreateChams(obj, ESP.Items.Color)
                    if highlight then
                        table.insert(ESPObjects.Items, highlight)
                    end
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
                        if highlight then
                            table.insert(ESPObjects.Entities, highlight)
                        end
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
                if highlight then
                    table.insert(ESPObjects.Players, highlight)
                end
            end
        end
    end
end

local function HandleStamina()
    if not InfiniteStamina.Enabled then return end
    
    local char = LocalPlayer.Character
    if not char then return end
    
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    
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

local function MainLoop()
    while true do
        task.wait(0.5)
        
        if not ESP.Items.Enabled then ClearESP("Items") end
        if not ESP.Entities.Enabled then ClearESP("Entities") end
        if not ESP.Players.Enabled then ClearESP("Players") end
        
        ProcessEntities()
        HandleStamina()
    end
end

CreateUI()
coroutine.wrap(MainLoop)()
