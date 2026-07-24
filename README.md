--[[
    ESP Framework v4.0 - Full ESP
    Recursos: Highlight, Box, Distância, Esqueleto, Time Check
    Carregar com: loadstring(game:HttpGet("URL_AQUI"))()
]]

local function LoadScript()
    local success, err = pcall(function()
        local Players = game:GetService("Players")
        local RunService = game:GetService("RunService")
        local UserInputService = game:GetService("UserInputService")
        local Workspace = game:GetService("Workspace")
        local LocalPlayer = Players.LocalPlayer
        local Camera = workspace.CurrentCamera

        -- Configurações
        local Config = {
            ESP = {
                Enabled = false,
                Highlight = {
                    Enabled = true,
                    Color = Color3.fromRGB(0, 255, 0),
                    Transparency = 0.2
                },
                Box = {
                    Enabled = true,
                    Color = Color3.fromRGB(0, 255, 0),
                    Thickness = 1,
                    Transparency = 0.5
                },
                Distance = {
                    Enabled = true,
                    Color = Color3.fromRGB(255, 255, 255),
                    Size = 14
                },
                Skeleton = {
                    Enabled = true,
                    Color = Color3.fromRGB(0, 255, 0),
                    Thickness = 1
                },
                TeamCheck = {
                    Enabled = true,
                    TeamColor = Color3.fromRGB(0, 0, 255),
                    EnemyColor = Color3.fromRGB(255, 0, 0)
                }
            }
        }

        -- Gerenciador de ESP
        local ESPManager = {
            ActiveHighlights = {},
            ActiveBoxes = {},
            ActiveDistances = {},
            ActiveSkeletons = {},
            LastUpdate = 0,
            UpdateInterval = 0.05
        }

        -- Função para criar Highlight
        local function CreateHighlight(character, color, transparency)
            if not character or not character.Parent then return nil end
            
            local highlight = Instance.new("Highlight")
            highlight.FillColor = color
            highlight.OutlineColor = color
            highlight.FillTransparency = transparency or 0.2
            highlight.OutlineTransparency = transparency or 0.2
            highlight.Adornee = character
            highlight.Parent = character
            highlight.Name = "ESP_Highlight"
            
            return highlight
        end

        -- Função para criar Drawing Box (usando Drawing API)
        local function CreateBox(character, color, thickness)
            if not character or not character.Parent then return nil end
            
            local box = Drawing.new("Square")
            box.Visible = false
            box.Color = color
            box.Thickness = thickness or 1
            box.Filled = false
            box.Transparency = 0.5
            box.ZIndex = 999
            
            return box
        end

        -- Função para criar texto de distância
        local function CreateDistanceText(color, size)
            local text = Drawing.new("Text")
            text.Visible = false
            text.Color = color
            text.Size = size or 14
            text.Center = true
            text.Outline = true
            text.OutlineColor = Color3.fromRGB(0, 0, 0)
            text.ZIndex = 999
            
            return text
        end

        -- Função para criar esqueleto (usando linhas)
        local function CreateSkeletonLines(character, color, thickness)
            if not character or not character.Parent then return {} end
            
            local lines = {}
            local joints = {
                {"Head", "UpperTorso"},
                {"UpperTorso", "LowerTorso"},
                {"UpperTorso", "LeftUpperArm"},
                {"LeftUpperArm", "LeftLowerArm"},
                {"UpperTorso", "RightUpperArm"},
                {"RightUpperArm", "RightLowerArm"},
                {"LowerTorso", "LeftUpperLeg"},
                {"LeftUpperLeg", "LeftLowerLeg"},
                {"LowerTorso", "RightUpperLeg"},
                {"RightUpperLeg", "RightLowerLeg"}
            }
            
            for _, joint in ipairs(joints) do
                local line = Drawing.new("Line")
                line.Visible = false
                line.Color = color
                line.Thickness = thickness or 1
                line.Transparency = 0.7
                line.ZIndex = 999
                table.insert(lines, line)
            end
            
            return lines
        end

        -- Função para atualizar a posição da box
        local function UpdateBox(box, character)
            if not box or not character then return end
            
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            local head = character:FindFirstChild("Head")
            
            if rootPart and head then
                local pos = Camera:WorldToViewportPoint(rootPart.Position)
                local headPos = Camera:WorldToViewportPoint(head.Position)
                
                if pos.Z > 0 then
                    local height = math.abs(pos.Y - headPos.Y) * 1.5
                    local width = height * 0.6
                    
                    box.Position = Vector2.new(pos.X - width/2, headPos.Y - height/4)
                    box.Size = Vector2.new(width, height)
                    box.Visible = true
                else
                    box.Visible = false
                end
            else
                box.Visible = false
            end
        end

        -- Função para atualizar texto de distância
        local function UpdateDistance(text, character)
            if not text or not character then return end
            
            local rootPart = character:FindFirstChild("HumanoidRootPart")
            if rootPart then
                local pos = Camera:WorldToViewportPoint(rootPart.Position)
                if pos.Z > 0 then
                    local distance = math.floor((LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and 
                        (LocalPlayer.Character.HumanoidRootPart.Position - rootPart.Position).Magnitude) or 0)
                    
                    text.Position = Vector2.new(pos.X, pos.Y - 40)
                    text.Text = distance .. "m"
                    text.Visible = true
                else
                    text.Visible = false
                end
            else
                text.Visible = false
            end
        end

        -- Função para atualizar esqueleto
        local function UpdateSkeleton(lines, character)
            if not lines or not character then return end
            
            local parts = {}
            local partNames = {"Head", "UpperTorso", "LowerTorso", "LeftUpperArm", "LeftLowerArm", 
                              "RightUpperArm", "RightLowerArm", "LeftUpperLeg", "LeftLowerLeg", 
                              "RightUpperLeg", "RightLowerLeg"}
            
            for _, name in ipairs(partNames) do
                local part = character:FindFirstChild(name)
                if part and part:IsA("BasePart") then
                    local pos = Camera:WorldToViewportPoint(part.Position)
                    if pos.Z > 0 then
                        parts[name] = Vector2.new(pos.X, pos.Y)
                    else
                        parts[name] = nil
                    end
                end
            end
            
            local joints = {
                {1, 2}, {2, 3}, {2, 4}, {4, 5}, {2, 6}, {6, 7},
                {3, 8}, {8, 9}, {3, 10}, {10, 11}
            }
            
            for i, joint in ipairs(joints) do
                local line = lines[i]
                if line then
                    local part1 = parts[partNames[joint[1]]]
                    local part2 = parts[partNames[joint[2]]]
                    
                    if part1 and part2 then
                        line.From = part1
                        line.To = part2
                        line.Visible = true
                    else
                        line.Visible = false
                    end
                end
            end
        end

        -- Função principal de atualização ESP
        local function UpdateESP()
            local currentTime = tick()
            if currentTime - ESPManager.LastUpdate < ESPManager.UpdateInterval then
                return
            end
            ESPManager.LastUpdate = currentTime
            
            -- Limpeza de objetos inválidos
            for i = #ESPManager.ActiveHighlights, 1, -1 do
                local item = ESPManager.ActiveHighlights[i]
                if not item or not item.Parent then
                    table.remove(ESPManager.ActiveHighlights, i)
                end
            end
            
            for i = #ESPManager.ActiveBoxes, 1, -1 do
                local item = ESPManager.ActiveBoxes[i]
                if not item or not item.character or not item.character.Parent then
                    if item.box then item.box:Remove() end
                    table.remove(ESPManager.ActiveBoxes, i)
                end
            end
            
            for i = #ESPManager.ActiveDistances, 1, -1 do
                local item = ESPManager.ActiveDistances[i]
                if not item or not item.character or not item.character.Parent then
                    if item.text then item.text:Remove() end
                    table.remove(ESPManager.ActiveDistances, i)
                end
            end
            
            for i = #ESPManager.ActiveSkeletons, 1, -1 do
                local item = ESPManager.ActiveSkeletons[i]
                if not item or not item.character or not item.character.Parent then
                    if item.lines then
                        for _, line in ipairs(item.lines) do
                            line:Remove()
                        end
                    end
                    table.remove(ESPManager.ActiveSkeletons, i)
                end
            end
            
            if not Config.ESP.Enabled then
                -- Remove todos os ESPs
                for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                    if highlight and highlight.Parent then
                        highlight:Destroy()
                    end
                end
                ESPManager.ActiveHighlights = {}
                
                for _, item in ipairs(ESPManager.ActiveBoxes) do
                    if item.box then item.box:Remove() end
                end
                ESPManager.ActiveBoxes = {}
                
                for _, item in ipairs(ESPManager.ActiveDistances) do
                    if item.text then item.text:Remove() end
                end
                ESPManager.ActiveDistances = {}
                
                for _, item in ipairs(ESPManager.ActiveSkeletons) do
                    if item.lines then
                        for _, line in ipairs(item.lines) do
                            line:Remove()
                        end
                    end
                end
                ESPManager.ActiveSkeletons = {}
                return
            end
            
            -- Processa todos os players
            local players = Players:GetPlayers()
            for _, player in ipairs(players) do
                if player ~= LocalPlayer then
                    local character = player.Character
                    if character and character:FindFirstChild("Humanoid") then
                        local team = player.Team
                        local isEnemy = team ~= LocalPlayer.Team
                        
                        -- Define cores baseado no time
                        local color
                        if Config.ESP.TeamCheck.Enabled then
                            color = isEnemy and Config.ESP.TeamCheck.EnemyColor or Config.ESP.TeamCheck.TeamColor
                        else
                            color = Config.ESP.Highlight.Color
                        end
                        
                        -- Highlight
                        if Config.ESP.Highlight.Enabled then
                            local hasHighlight = false
                            for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                                if highlight and highlight.Adornee == character then
                                    hasHighlight = true
                                    break
                                end
                            end
                            
                            if not hasHighlight then
                                local highlight = CreateHighlight(character, color, Config.ESP.Highlight.Transparency)
                                if highlight then
                                    table.insert(ESPManager.ActiveHighlights, highlight)
                                end
                            end
                        end
                        
                        -- Box
                        if Config.ESP.Box.Enabled then
                            local hasBox = false
                            for _, item in ipairs(ESPManager.ActiveBoxes) do
                                if item.character == character then
                                    hasBox = true
                                    break
                                end
                            end
                            
                            if not hasBox then
                                local box = CreateBox(character, color, Config.ESP.Box.Thickness)
                                if box then
                                    table.insert(ESPManager.ActiveBoxes, {
                                        character = character,
                                        box = box
                                    })
                                end
                            end
                        end
                        
                        -- Distância
                        if Config.ESP.Distance.Enabled then
                            local hasDistance = false
                            for _, item in ipairs(ESPManager.ActiveDistances) do
                                if item.character == character then
                                    hasDistance = true
                                    break
                                end
                            end
                            
                            if not hasDistance then
                                local text = CreateDistanceText(Config.ESP.Distance.Color, Config.ESP.Distance.Size)
                                if text then
                                    table.insert(ESPManager.ActiveDistances, {
                                        character = character,
                                        text = text
                                    })
                                end
                            end
                        end
                        
                        -- Esqueleto
                        if Config.ESP.Skeleton.Enabled then
                            local hasSkeleton = false
                            for _, item in ipairs(ESPManager.ActiveSkeletons) do
                                if item.character == character then
                                    hasSkeleton = true
                                    break
                                end
                            end
                            
                            if not hasSkeleton then
                                local lines = CreateSkeletonLines(character, color, Config.ESP.Skeleton.Thickness)
                                if lines and #lines > 0 then
                                    table.insert(ESPManager.ActiveSkeletons, {
                                        character = character,
                                        lines = lines
                                    })
                                end
                            end
                        end
                    end
                end
            end
            
            -- Atualiza posições dos elementos
            for _, item in ipairs(ESPManager.ActiveBoxes) do
                if item.character and item.character.Parent then
                    UpdateBox(item.box, item.character)
                end
            end
            
            for _, item in ipairs(ESPManager.ActiveDistances) do
                if item.character and item.character.Parent then
                    UpdateDistance(item.text, item.character)
                end
            end
            
            for _, item in ipairs(ESPManager.ActiveSkeletons) do
                if item.character and item.character.Parent then
                    UpdateSkeleton(item.lines, item.character)
                end
            end
        end

        -- Loop principal
        local function ESPLoop()
            while true do
                task.wait(ESPManager.UpdateInterval)
                UpdateESP()
            end
        end

        -- Criação da UI
        local function CreateUI()
            local screenGui = Instance.new("ScreenGui")
            screenGui.Name = "ESP_Framework"
            screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
            
            local mainFrame = Instance.new("Frame")
            mainFrame.Size = UDim2.new(0, 350, 0, 280)
            mainFrame.Position = UDim2.new(0.5, -175, 0.5, -140)
            mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
            mainFrame.BackgroundTransparency = 0.1
            mainFrame.BorderSizePixel = 1
            mainFrame.BorderColor3 = Color3.fromRGB(50, 50, 70)
            mainFrame.ClipsDescendants = true
            mainFrame.Parent = screenGui
            
            -- Barra de título
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
            titleText.Text = "ESP Framework v4.0"
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
            
            -- Conteúdo
            local contentFrame = Instance.new("Frame")
            contentFrame.Size = UDim2.new(1, -20, 1, -45)
            contentFrame.Position = UDim2.new(0, 10, 0, 38)
            contentFrame.BackgroundTransparency = 1
            contentFrame.Parent = mainFrame
            
            -- Label principal
            local espLabel = Instance.new("TextLabel")
            espLabel.Size = UDim2.new(0.5, 0, 0, 25)
            espLabel.Position = UDim2.new(0, 0, 0, 0)
            espLabel.BackgroundTransparency = 1
            espLabel.Text = "Master ESP"
            espLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            espLabel.TextScaled = true
            espLabel.TextXAlignment = Enum.TextXAlignment.Left
            espLabel.Font = Enum.Font.GothamBold
            espLabel.Parent = contentFrame
            
            -- Toggle Master ESP
            local masterToggle = Instance.new("TextButton")
            masterToggle.Size = UDim2.new(0, 55, 0, 25)
            masterToggle.Position = UDim2.new(0.75, 0, 0, 0)
            masterToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            masterToggle.Text = "OFF"
            masterToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
            masterToggle.TextScaled = true
            masterToggle.Font = Enum.Font.GothamBold
            masterToggle.BorderSizePixel = 0
            masterToggle.Parent = contentFrame
            
            -- Separador
            local separator = Instance.new("Frame")
            separator.Size = UDim2.new(1, 0, 0, 1)
            separator.Position = UDim2.new(0, 0, 0, 30)
            separator.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            separator.BorderSizePixel = 0
            separator.Parent = contentFrame
            
            -- Opções de ESP
            local options = {
                {name = "Highlight", label = "Highlight", y = 40, key = "Highlight"},
                {name = "Box", label = "Box ESP", y = 70, key = "Box"},
                {name = "Distance", label = "Distancia", y = 100, key = "Distance"},
                {name = "Skeleton", label = "Esqueleto", y = 130, key = "Skeleton"},
                {name = "TeamCheck", label = "Time Check", y = 160, key = "TeamCheck"}
            }
            
            local toggles = {}
            
            for _, opt in ipairs(options) do
                local frame = Instance.new("Frame")
                frame.Size = UDim2.new(1, 0, 0, 25)
                frame.Position = UDim2.new(0, 0, 0, opt.y)
                frame.BackgroundTransparency = 1
                frame.Parent = contentFrame
                
                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(0.5, 0, 1, 0)
                label.Position = UDim2.new(0, 0, 0, 0)
                label.BackgroundTransparency = 1
                label.Text = opt.label
                label.TextColor3 = Color3.fromRGB(200, 200, 200)
                label.TextScaled = true
                label.TextXAlignment = Enum.TextXAlignment.Left
                label.Font = Enum.Font.Gotham
                label.Parent = frame
                
                local toggle = Instance.new("TextButton")
                toggle.Size = UDim2.new(0, 45, 0, 20)
                toggle.Position = UDim2.new(0.75, 0, 0.5, -10)
                toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
                toggle.Text = "ON"
                toggle.TextColor3 = Color3.fromRGB(80, 255, 80)
                toggle.TextScaled = true
                toggle.Font = Enum.Font.GothamBold
                toggle.BorderSizePixel = 0
                toggle.Parent = frame
                
                toggles[opt.key] = toggle
                
                if opt.key ~= "TeamCheck" then
                    local colorBtn = Instance.new("TextButton")
                    colorBtn.Size = UDim2.new(0, 20, 0, 20)
                    colorBtn.Position = UDim2.new(0.88, 0, 0.5, -10)
                    colorBtn.BackgroundColor3 = Config.ESP[opt.key].Color
                    colorBtn.Text = ""
                    colorBtn.BorderSizePixel = 1
                    colorBtn.BorderColor3 = Color3.fromRGB(255, 255, 255)
                    colorBtn.Parent = frame
                    
                    colorBtn.MouseButton1Click:Connect(function()
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
                            Color3.fromRGB(255, 255, 255), Color3.fromRGB(100, 100, 100)
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
                                Config.ESP[opt.key].Color = color
                                colorBtn.BackgroundColor3 = color
                                pickerFrame:Destroy()
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
                end
            end
            
            -- Status
            local statusLabel = Instance.new("TextLabel")
            statusLabel.Size = UDim2.new(1, 0, 0, 20)
            statusLabel.Position = UDim2.new(0, 0, 0, 195)
            statusLabel.BackgroundTransparency = 1
            statusLabel.Text = "Status: Desativado"
            statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
            statusLabel.TextScaled = true
            statusLabel.Font = Enum.Font.Gotham
            statusLabel.Parent = contentFrame
            
            -- Botão minimizado
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
            
            -- Sistema de arrastar
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
            
            -- Lógica dos toggles
            masterToggle.MouseButton1Click:Connect(function()
                Config.ESP.Enabled = not Config.ESP.Enabled
                if Config.ESP.Enabled then
                    masterToggle.Text = "ON"
                    masterToggle.TextColor3 = Color3.fromRGB(80, 255, 80)
                    masterToggle.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
                    statusLabel.Text = "Status: Ativo"
                    statusLabel.TextColor3 = Color3.fromRGB(80, 255, 80)
                else
                    masterToggle.Text = "OFF"
                    masterToggle.TextColor3 = Color3.fromRGB(255, 80, 80)
                    masterToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
                    statusLabel.Text = "Status: Desativado"
                    statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
                end
            end)
            
            -- Toggles individuais
            for key, toggle in pairs(toggles) do
                toggle.MouseButton1Click:Connect(function()
                    Config.ESP[key].Enabled = not Config.ESP[key].Enabled
                    if Config.ESP[key].Enabled then
                        toggle.Text = "ON"
                        toggle.TextColor3 = Color3.fromRGB(80, 255, 80)
                        toggle.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
                    else
                        toggle.Text = "OFF"
                        toggle.TextColor3 = Color3.fromRGB(255, 80, 80)
                        toggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
                    end
                end)
            end
            
            return { MainFrame = mainFrame, MinimizedButton = minimizedButton, ScreenGui = screenGui }
        end

        -- Inicialização
        local ui = CreateUI()
        coroutine.wrap(ESPLoop)()
        
        -- Eventos automáticos
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
        
        print("ESP Framework v4.0 carregado com sucesso!")
        print("Recursos: Highlight, Box, Distancia, Esqueleto, Time Check")
    end)
    
    if not success then
        warn("Erro ao carregar ESP Framework: " .. tostring(err))
    end
end

LoadScript()
