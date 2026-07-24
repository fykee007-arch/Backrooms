local function LoadScript()
    local success, err = pcall(function()
        local Players = game:GetService("Players")
        local RunService = game:GetService("RunService")
        local UserInputService = game:GetService("UserInputService")
        local Workspace = game:GetService("Workspace")
        local LocalPlayer = Players.LocalPlayer

        local Config = {
            ESP = {
                Enabled = false,
                Color = Color3.fromRGB(0, 255, 0),
                Transparency = 0.2
            }
        }

        local ESPManager = {
            ActiveHighlights = {},
            LastUpdate = 0,
            UpdateInterval = 0.1
        }

        local function CreateHighlight(part, color, transparency)
            if not part or not part.Parent then return nil end
            local existing = part:FindFirstChild("ESP_Highlight")
            if existing then existing:Destroy() end
            local highlight = Instance.new("Highlight")
            highlight.Name = "ESP_Highlight"
            highlight.FillColor = color
            highlight.OutlineColor = color
            highlight.FillTransparency = transparency or 0.2
            highlight.OutlineTransparency = transparency or 0.2
            highlight.Adornee = part
            highlight.Parent = part
            return highlight
        end

        local function GetBestPart(character)
            if not character then return nil end
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
            for _, descendant in ipairs(character:GetDescendants()) do
                if descendant:IsA("BasePart") then
                    return descendant
                end
            end
            return nil
        end

        local function UpdateESP()
            local currentTime = tick()
            if currentTime - ESPManager.LastUpdate < ESPManager.UpdateInterval then
                return
            end
            ESPManager.LastUpdate = currentTime
            
            for i = #ESPManager.ActiveHighlights, 1, -1 do
                local highlight = ESPManager.ActiveHighlights[i]
                if not highlight or not highlight.Parent then
                    table.remove(ESPManager.ActiveHighlights, i)
                end
            end
            
            if not Config.ESP.Enabled then
                for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                    if highlight and highlight.Parent then
                        highlight:Destroy()
                    end
                end
                ESPManager.ActiveHighlights = {}
                return
            end
            
            local players = Players:GetPlayers()
            for _, player in ipairs(players) do
                if player ~= LocalPlayer then
                    local character = player.Character
                    if character then
                        local part = GetBestPart(character)
                        if part then
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
                                end
                            end
                        end
                    end
                end
            end
        end

        local function ESPLoop()
            while true do
                task.wait(ESPManager.UpdateInterval)
                UpdateESP()
            end
        end

        local function CreateUI()
            local screenGui = Instance.new("ScreenGui")
            screenGui.Name = "ESP_Framework"
            screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
            
            local mainFrame = Instance.new("Frame")
            mainFrame.Size = UDim2.new(0, 340, 0, 180)
            mainFrame.Position = UDim2.new(0.5, -170, 0.5, -90)
            mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
            mainFrame.BackgroundTransparency = 0.1
            mainFrame.BorderSizePixel = 1
            mainFrame.BorderColor3 = Color3.fromRGB(50, 50, 70)
            mainFrame.ClipsDescendants = true
            mainFrame.Parent = screenGui
            
            local titleBar = Instance.new("Frame")
            titleBar.Size = UDim2.new(1, 0, 0, 32)
            titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
            titleBar.BackgroundTransparency = 0.3
            titleBar.BorderSizePixel = 0
            titleBar.Parent = mainFrame
            
            local titleText = Instance.new("TextLabel")
            titleText.Size = UDim2.new(0.6, 0, 1, 0)
            titleText.Position = UDim2.new(0, 10, 0, 0)
            titleText.BackgroundTransparency = 1
            titleText.Text = "ESP Framework"
            titleText.TextColor3 = Color3.fromRGB(0, 255, 180)
            titleText.TextScaled = true
            titleText.TextXAlignment = Enum.TextXAlignment.Left
            titleText.Font = Enum.Font.GothamBold
            titleText.Parent = titleBar
            
            local minButton = Instance.new("TextButton")
            minButton.Size = UDim2.new(0, 30, 1, 0)
            minButton.Position = UDim2.new(0.88, 0, 0, 0)
            minButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            minButton.BackgroundTransparency = 0.3
            minButton.Text = "-"
            minButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            minButton.TextScaled = true
            minButton.Font = Enum.Font.GothamBold
            minButton.BorderSizePixel = 0
            minButton.Parent = titleBar
            
            local closeButton = Instance.new("TextButton")
            closeButton.Size = UDim2.new(0, 30, 1, 0)
            closeButton.Position = UDim2.new(0.94, 0, 0, 0)
            closeButton.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
            closeButton.BackgroundTransparency = 0.3
            closeButton.Text = "X"
            closeButton.TextColor3 = Color3.fromRGB(255, 100, 100)
            closeButton.TextScaled = true
            closeButton.Font = Enum.Font.GothamBold
            closeButton.BorderSizePixel = 0
            closeButton.Parent = titleBar
            
            local contentFrame = Instance.new("Frame")
            contentFrame.Size = UDim2.new(1, -20, 1, -45)
            contentFrame.Position = UDim2.new(0, 10, 0, 38)
            contentFrame.BackgroundTransparency = 1
            contentFrame.Parent = mainFrame
            
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
            
            local colorPickerBtn = Instance.new("TextButton")
            colorPickerBtn.Size = UDim2.new(0, 35, 0, 28)
            colorPickerBtn.Position = UDim2.new(0.62, 0, 0.05, 0)
            colorPickerBtn.BackgroundColor3 = Config.ESP.Color
            colorPickerBtn.Text = ""
            colorPickerBtn.BorderSizePixel = 2
            colorPickerBtn.BorderColor3 = Color3.fromRGB(255, 255, 255)
            colorPickerBtn.Parent = contentFrame
            
            local statusLabel = Instance.new("TextLabel")
            statusLabel.Size = UDim2.new(1, 0, 0, 20)
            statusLabel.Position = UDim2.new(0, 0, 0.75, 0)
            statusLabel.BackgroundTransparency = 1
            statusLabel.Text = "Status: Desativado"
            statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
            statusLabel.TextScaled = true
            statusLabel.Font = Enum.Font.Gotham
            statusLabel.Parent = contentFrame
            
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
            
            local IsDragging = false
            local DragInput = nil
            local DragStart = nil
            local StartPos = nil
            local Minimized = false
            
            local function MinimizeUI()
                Minimized = true
                mainFrame.Visible = false
                minimizedButton.Visible = true
            end
            
            local function RestoreUI()
                Minimized = false
                mainFrame.Visible = true
                minimizedButton.Visible = false
            end
            
            minButton.MouseButton1Click:Connect(MinimizeUI)
            minimizedButton.MouseButton1Click:Connect(RestoreUI)
            
            closeButton.MouseButton1Click:Connect(function()
                screenGui:Destroy()
            end)
            
            local function StartDrag(input)
                IsDragging = true
                DragStart = input.Position
                StartPos = mainFrame.Position
                DragInput = UserInputService.InputChanged:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseMovement then
                        local delta = input.Position - DragStart
                        mainFrame.Position = UDim2.new(
                            StartPos.X.Scale,
                            StartPos.X.Offset + delta.X,
                            StartPos.Y.Scale,
                            StartPos.Y.Offset + delta.Y
                        )
                    end
                end)
            end
            
            local function EndDrag()
                if DragInput then
                    DragInput:Disconnect()
                    DragInput = nil
                end
                IsDragging = false
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
            
            espToggle.MouseButton1Click:Connect(function()
                Config.ESP.Enabled = not Config.ESP.Enabled
                if Config.ESP.Enabled then
                    espToggle.Text = "ON"
                    espToggle.TextColor3 = Color3.fromRGB(80, 255, 80)
                    espToggle.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
                    statusLabel.Text = "Status: Ativo"
                    statusLabel.TextColor3 = Color3.fromRGB(80, 255, 80)
                    UpdateESP()
                else
                    espToggle.Text = "OFF"
                    espToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
                    espToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
                    statusLabel.Text = "Status: Desativado"
                    statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
                    for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                        if highlight and highlight.Parent then
                            highlight:Destroy()
                        end
                    end
                    ESPManager.ActiveHighlights = {}
                end
            end)
            
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
                pickerTitle.Text = "Selecionar Cor"
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
                    Color3.fromRGB(255, 200, 0), Color3.fromRGB(200, 0, 255)
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
                        if Config.ESP.Enabled then
                            for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                                if highlight and highlight.Parent then
                                    highlight:Destroy()
                                end
                            end
                            ESPManager.ActiveHighlights = {}
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
            
            return { MainFrame = mainFrame, MinimizedButton = minimizedButton, ScreenGui = screenGui }
        end

        local ui = CreateUI()
        coroutine.wrap(ESPLoop)()
        
        Players.PlayerAdded:Connect(function()
            if Config.ESP.Enabled then
                task.wait(1)
                UpdateESP()
            end
        end)
        
        Players.PlayerCharacterAdded:Connect(function(player)
            if player ~= LocalPlayer and Config.ESP.Enabled then
                task.wait(0.5)
                UpdateESP()
            end
        end)
        
        Players.PlayerRemoving:Connect(function()
            if Config.ESP.Enabled then
                UpdateESP()
            end
        end)
        
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
        
        print("ESP Framework carregado com sucesso!")
    end)
    
    if not success then
        warn("Erro ao carregar ESP Framework: " .. tostring(err))
    end
end

LoadScript()
