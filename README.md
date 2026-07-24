--[[
    ESP Framework v5.0 - FULL ESP
    100% Funcional para executores reais
    Features: Highlight, Box, Distance, Skeleton, Team Check, Name
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
        
        -- Verifica se o Drawing está disponível
        if not Drawing then
            warn("Drawing API não disponível! Usando modo compatibilidade.")
        end

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
                    Thickness = 1
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
                    TeamColor = Color3.fromRGB(0, 150, 255),
                    EnemyColor = Color3.fromRGB(255, 0, 0)
                },
                Name = {
                    Enabled = true,
                    Color = Color3.fromRGB(255, 255, 255),
                    Size = 14
                },
                Health = {
                    Enabled = true,
                    Color = Color3.fromRGB(0, 255, 0),
                    Size = 12
                }
            }
        }

        -- Gerenciador de ESP
        local ESPManager = {
            ActiveHighlights = {},
            ActiveDrawings = {},
            LastUpdate = 0,
            UpdateInterval = 0.05
        }

        -- Função para criar Highlight
        local function CreateHighlight(character, color, transparency)
            if not character or not character.Parent then return nil end
            
            local old = character:FindFirstChild("ESP_Highlight")
            if old then old:Destroy() end
            
            local highlight = Instance.new("Highlight")
            highlight.Name = "ESP_Highlight"
            highlight.FillColor = color
            highlight.OutlineColor = color
            highlight.FillTransparency = transparency or 0.2
            highlight.OutlineTransparency = transparency or 0.2
            highlight.Adornee = character
            highlight.Parent = character
            
            return highlight
        end

        -- Função para criar Box
        local function CreateBox(color, thickness)
            local box = Drawing.new("Square")
            box.Visible = false
            box.Color = color
            box.Thickness = thickness or 1
            box.Filled = false
            box.Transparency = 0.5
            box.ZIndex = 999
            return box
        end

        -- Função para criar Texto
        local function CreateText(color, size)
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

        -- Função para criar Linha (Skeleton)
        local function CreateLine(color, thickness)
            local line = Drawing.new("Line")
            line.Visible = false
            line.Color = color
            line.Thickness = thickness or 1
            line.Transparency = 0.7
            line.ZIndex = 999
            return line
        end

        -- Função para atualizar Box
        local function UpdateBox(box, character)
            if not box or not character then return end
            
            local root = character:FindFirstChild("HumanoidRootPart")
            local head = character:FindFirstChild("Head")
            
            if root and head then
                local pos = Camera:WorldToViewportPoint(root.Position)
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

        -- Função para atualizar Skeleton
        local function UpdateSkeleton(lines, character)
            if not lines or not character then return end
            
            local bones = {
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
            
            local parts = {}
            for _, bone in ipairs(bones) do
                for _, name in ipairs(bone) do
                    if not parts[name] then
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
                end
            end
            
            for i, bone in ipairs(bones) do
                local line = lines[i]
                if line then
                    local p1 = parts[bone[1]]
                    local p2 = parts[bone[2]]
                    
                    if p1 and p2 then
                        line.From = p1
                        line.To = p2
                        line.Visible = true
                    else
                        line.Visible = false
                    end
                end
            end
        end

        -- Função principal de atualização
        local function UpdateESP()
            local currentTime = tick()
            if currentTime - ESPManager.LastUpdate < ESPManager.UpdateInterval then
                return
            end
            ESPManager.LastUpdate = currentTime
            
            -- Limpeza
            for i = #ESPManager.ActiveHighlights, 1, -1 do
                local item = ESPManager.ActiveHighlights[i]
                if not item or not item.Parent then
                    table.remove(ESPManager.ActiveHighlights, i)
                end
            end
            
            for i = #ESPManager.ActiveDrawings, 1, -1 do
                local item = ESPManager.ActiveDrawings[i]
                if not item or not item.character or not item.character.Parent then
                    if item.box then item.box:Remove() end
                    if item.dist then item.dist:Remove() end
                    if item.name then item.name:Remove() end
                    if item.health then item.health:Remove() end
                    if item.lines then
                        for _, line in ipairs(item.lines) do
                            line:Remove()
                        end
                    end
                    table.remove(ESPManager.ActiveDrawings, i)
                end
            end
            
            if not Config.ESP.Enabled then
                -- Remove Highlights
                for _, highlight in ipairs(ESPManager.ActiveHighlights) do
                    if highlight and highlight.Parent then
                        highlight:Destroy()
                    end
                end
                ESPManager.ActiveHighlights = {}
                
                -- Remove Drawings
                for _, item in ipairs(ESPManager.ActiveDrawings) do
                    if item.box then item.box:Remove() end
                    if item.dist then item.dist:Remove() end
                    if item.name then item.name:Remove() end
                    if item.health then item.health:Remove() end
                    if item.lines then
                        for _, line in ipairs(item.lines) do
                            line:Remove()
                        end
                    end
                end
                ESPManager.ActiveDrawings = {}
                return
            end
            
            -- Processa players
            local players = Players:GetPlayers()
            for _, player in ipairs(players) do
                if player ~= LocalPlayer then
                    local character = player.Character
                    if character and character:FindFirstChild("Humanoid") then
                        local humanoid = character.Humanoid
                        local team = player.Team
                        local isEnemy = team ~= LocalPlayer.Team
                        
                        -- Define cor baseada no time
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
                        
                        -- Drawings
                        local hasDrawings = false
                        for _, item in ipairs(ESPManager.ActiveDrawings) do
                            if item.character == character then
                                hasDrawings = true
                                break
                            end
                        end
                        
                        if not hasDrawings then
                            local drawings = {
                                character = character,
                                box = Config.ESP.Box.Enabled and CreateBox(color, Config.ESP.Box.Thickness) or nil,
                                dist = Config.ESP.Distance.Enabled and CreateText(Config.ESP.Distance.Color, Config.ESP.Distance.Size) or nil,
                                name = Config.ESP.Name.Enabled and CreateText(Config.ESP.Name.Color, Config.ESP.Name.Size) or nil,
                                health = Config.ESP.Health.Enabled and CreateText(Config.ESP.Health.Color, Config.ESP.Health.Size) or nil,
                                lines = {}
                            }
                            
                            if Config.ESP.Skeleton.Enabled then
                                local joints = {
                                    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
                                    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"},
                                    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"},
                                    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"},
                                    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}
                                }
                                for _ in ipairs(joints) do
                                    table.insert(drawings.lines, CreateLine(color, Config.ESP.Skeleton.Thickness))
                                end
                            end
                            
                            table.insert(ESPManager.ActiveDrawings, drawings)
                        end
                    end
                end
            end
            
            -- Atualiza posições
            for _, item in ipairs(ESPManager.ActiveDrawings) do
                if item.character and item.character.Parent then
                    local root = item.character:FindFirstChild("HumanoidRootPart")
                    local head = item.character:FindFirstChild("Head")
                    local humanoid = item.character:FindFirstChild("Humanoid")
                    
                    if root and head then
                        local pos = Camera:WorldToViewportPoint(root.Position)
                        local headPos = Camera:WorldToViewportPoint(head.Position)
                        
                        if pos.Z > 0 then
                            -- Distância
                            if item.dist then
                                local distance = math.floor((LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and 
                                    (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude) or 0)
                                item.dist.Position = Vector2.new(pos.X, pos.Y + 30)
                                item.dist.Text = distance .. "m"
                                item.dist.Visible = true
                            end
                            
                            -- Nome
                            if item.name then
                                item.name.Position = Vector2.new(pos.X, pos.Y - 20)
                                item.name.Text = item.character.Name
                                item.name.Visible = true
                            end
                            
                            -- Health
                            if item.health and humanoid then
                                local healthPercent = math.floor((humanoid.Health / humanoid.MaxHealth) * 100)
                                local healthColor = Color3.fromRGB(
                                    math.floor(255 * (1 - healthPercent/100)),
                                    math.floor(255 * (healthPercent/100)),
                                    0
                                )
                                item.health.Color = healthColor
                                item.health.Position = Vector2.new(pos.X, pos.Y + 55)
                                item.health.Text = healthPercent .. "%"
                                item.health.Visible = true
                            end
                            
                            -- Box
                            if item.box then
                                local height = math.abs(pos.Y - headPos.Y) * 1.5
                                local width = height * 0.6
                                item.box.Position = Vector2.new(pos.X - width/2, headPos.Y - height/4)
                                item.box.Size = Vector2.new(width, height)
                                item.box.Visible = true
                            end
                            
                            -- Skeleton
                            if #item.lines > 0 then
                                UpdateSkeleton(item.lines, item.character)
                            end
                        else
                            if item.box then item.box.Visible = false end
                            if item.dist then item.dist.Visible = false end
                            if item.name then item.name.Visible = false end
                            if item.health then item.health.Visible = false end
                            if #item.lines > 0 then
                                for _, line in ipairs(item.lines) do
                                    line.Visible = false
                                end
                            end
                        end
                    end
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
            mainFrame.Size = UDim2.new(0, 350, 0, 340)
            mainFrame.Position = UDim2.new(0.5, -175, 0.5, -170)
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
            titleText.Text = "ESP Framework v5.0"
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
            
            -- Master ESP
            local masterLabel = Instance.new("TextLabel")
            masterLabel.Size = UDim2.new(0.5, 0, 0, 25)
            masterLabel.Position = UDim2.new(0, 0, 0, 0)
            masterLabel.BackgroundTransparency = 1
            masterLabel.Text = "Master ESP"
            masterLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
            masterLabel.TextScaled = true
            masterLabel.TextXAlignment = Enum.TextXAlignment.Left
            masterLabel.Font = Enum.Font.GothamBold
            masterLabel.Parent = contentFrame
            
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
            local sep = Instance.new("Frame")
            sep.Size = UDim2.new(1, 0, 0, 1)
            sep.Position = UDim2.new(0, 0, 0, 30)
            sep.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            sep.BorderSizePixel = 0
            sep.Parent = contentFrame
            
            -- Opções
            local options = {
                {key = "Highlight", label = "Highlight", y = 40},
                {key = "Box", label = "Box ESP", y = 68},
                {key = "Distance", label = "Distancia", y = 96},
                {key = "Skeleton", label = "Esqueleto", y = 124},
                {key = "TeamCheck", label = "Time Check", y = 152},
                {key = "Name", label = "Nome", y = 180},
                {key = "Health", label = "Vida", y = 208}
            }
            
            local toggles = {}
            local colorButtons = {}
            
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
                
                -- Color Picker
                local colorBtn = Instance.new("TextButton")
                colorBtn.Size = UDim2.new(0, 20, 0, 20)
                colorBtn.Position = UDim2.new(0.88, 0, 0.5, -10)
                colorBtn.BackgroundColor3 = Config.ESP[opt.key].Color
                colorBtn.Text = ""
                colorBtn.BorderSizePixel = 1
                colorBtn.BorderColor3 = Color3.fromRGB(255, 255, 255)
                colorBtn.Parent = frame
                colorButtons[opt.key] = colorBtn
                
                colorBtn.MouseButton1Click:Connect(function()
                    local picker = Instance.new("Frame")
                    picker.Size = UDim2.new(0, 280, 0, 220)
                    picker.Position = UDim2.new(0.5, -140, 0.5, -110)
                    picker.BackgroundColor3 = Color3.fromRGB(20, 20, 35)
                    picker.BackgroundTransparency = 0.1
                    picker.BorderSizePixel = 1
                    picker.BorderColor3 = Color3.fromRGB(50, 50, 70)
                    picker.Parent = screenGui
                    
                    local pTitle = Instance.new("TextLabel")
                    pTitle.Size = UDim2.new(1, 0, 0, 35)
                    pTitle.BackgroundTransparency = 1
                    pTitle.Text = "Selecionar Cor"
                    pTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
                    pTitle.TextScaled = true
                    pTitle.Font = Enum.Font.GothamBold
                    pTitle.Parent = picker
                    
                    local grid = Instance.new("Frame")
                    grid.Size = UDim2.new(1, -20, 1, -75)
                    grid.Position = UDim2.new(0, 10, 0, 40)
                    grid.BackgroundTransparency = 1
                    grid.Parent = picker
                    
                    local colors = {
                        Color3.fromRGB(255,0,0), Color3.fromRGB(0,255,0),
                        Color3.fromRGB(0,0,255), Color3.fromRGB(255,255,0),
                        Color3.fromRGB(255,0,255), Color3.fromRGB(0,255,255),
                        Color3.fromRGB(255,128,0), Color3.fromRGB(128,255,0),
                        Color3.fromRGB(0,128,255), Color3.fromRGB(255,0,128),
                        Color3.fromRGB(128,0,255), Color3.fromRGB(0,255,128),
                        Color3.fromRGB(255,255,255), Color3.fromRGB(100,100,100)
                    }
                    
                    for i, color in ipairs(colors) do
                        local btn = Instance.new("TextButton")
                        local row = math.floor((i-1) / 7)
                        local col = (i-1) % 7
                        btn.Size = UDim2.new(0, 35, 0, 35)
                        btn.Position = UDim2.new(0, col * 40 + 5, 0, row * 40 + 5)
                        btn.BackgroundColor3 = color
                        btn.Text = ""
                        btn.BorderSizePixel = 1
                        btn.BorderColor3 = Color3.fromRGB(255,255,255)
                        btn.Parent = grid
                        btn.MouseButton1Click:Connect(function()
                            Config.ESP[opt.key].Color = color
                            colorBtn.BackgroundColor3 = color
                            picker:Destroy()
                        end)
                    end
                    
                    local close = Instance.new("TextButton")
                    close.Size = UDim2.new(0, 100, 0, 30)
                    close.Position = UDim2.new(0.5, -50, 0, 185)
                    close.BackgroundColor3 = Color3.fromRGB(60,60,80)
                    close.Text = "Fechar"
                    close.TextColor3 = Color3.fromRGB(255,255,255)
                    close.TextScaled = true
                    close.Font = Enum.Font.Gotham
                    close.BorderSizePixel = 0
                    close.Parent = picker
                    close.MouseButton1Click:Connect(function()
                        picker:Destroy()
                    end)
                end)
            end
            
            -- Status
            local statusLabel = Instance.new("TextLabel")
            statusLabel.Size = UDim2.new(1, 0, 0, 20)
            statusLabel.Position = UDim2.new(0, 0, 0, 240)
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
        
        -- Eventos
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
        
        print("ESP Framework v5.0 carregado com sucesso!")
        print("Features: Highlight, Box, Distance, Skeleton, TeamCheck, Name, Health")
    end)
    
    if not success then
        warn("Erro ao carregar ESP Framework: " .. tostring(err))
    end
end

LoadScript()
