--// Minimalist ESP + Stamina HUD (LocalScript)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- Configurações
local ESP = {
    Items =    {Enabled = false, Color = Color3.fromHex("#0dbf25"), Trans = 0.8},
    Entities = {Enabled = false, Color = Color3.fromHex("#ff1100"), Trans = 0.8},
    Players =  {Enabled = false, Color = Color3.fromHex("#c452c4"), Trans = 0.8}
}

local InfiniteStamina = false

--// Criação do HUD Minimalista
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MinimalESP_HUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player.PlayerGui

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 320, 0, 380)
Main.Position = UDim2.new(0.5, -160, 0.5, -190)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)

-- Título Minimal
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "ESP HUD"
Title.TextColor3 = Color3.fromRGB(220, 220, 230)
Title.TextSize = 16
Title.Font = Enum.Font.GothamBold
Title.Parent = Main

-- Tabs
local TabContainer = Instance.new("Frame")
TabContainer.Size = UDim2.new(1, -20, 0, 35)
TabContainer.Position = UDim2.new(0, 10, 0, 45)
TabContainer.BackgroundTransparency = 1
TabContainer.Parent = Main

local TabESP = Instance.new("TextButton")
TabESP.Size = UDim2.new(0.5, -5, 1, 0)
TabESP.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
TabESP.Text = "ESP"
TabESP.TextColor3 = Color3.new(1,1,1)
TabESP.Font = Enum.Font.GothamSemibold
TabESP.TextSize = 14
TabESP.Parent = TabContainer

local TabStamina = Instance.new("TextButton")
TabStamina.Size = UDim2.new(0.5, -5, 1, 0)
TabStamina.Position = UDim2.new(0.5, 5, 0, 0)
TabStamina.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
TabStamina.Text = "Stamina"
TabStamina.TextColor3 = Color3.new(1,1,1)
TabStamina.Font = Enum.Font.GothamSemibold
TabStamina.TextSize = 14
TabStamina.Parent = TabContainer

local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -20, 1, -95)
Content.Position = UDim2.new(0, 10, 0, 85)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 3
Content.ScrollBarImageColor3 = Color3.fromRGB(80, 80, 90)
Content.Parent = Main

local List = Instance.new("UIListLayout")
List.Padding = UDim.new(0, 6)
List.SortOrder = Enum.SortOrder.LayoutOrder
List.Parent = Content

-- Função Toggle Minimalista
local function NewToggle(text, default, callback)
    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(1, 0, 0, 42)
    Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
    Frame.Parent = Content
    Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 10)

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.new(1,1,1)
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Font = Enum.Font.GothamSemibold
    Label.TextSize = 15
    Label.Parent = Frame

    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(0, 48, 0, 26)
    Toggle.Position = UDim2.new(1, -58, 0.5, -13)
    Toggle.BackgroundColor3 = default and Color3.fromRGB(0, 200, 80) or Color3.fromRGB(70, 70, 80)
    Toggle.Text = ""
    Toggle.Parent = Frame
    Instance.new("UICorner", Toggle).CornerRadius = UDim.new(1, 0)

    local enabled = default

    Toggle.MouseButton1Click:Connect(function()
        enabled = not enabled
        TweenService:Create(Toggle, TweenInfo.new(0.2), {
            BackgroundColor3 = enabled and Color3.fromRGB(0, 200, 80) or Color3.fromRGB(70, 70, 80)
        }):Play()
        callback(enabled)
    end)

    return Frame
end

-- Função Color Picker (Clique para mudar)
local function NewColorPicker(defaultColor, onChange)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 70, 0, 28)
    Btn.BackgroundColor3 = defaultColor
    Btn.Text = "🎨"
    Btn.TextSize = 16
    Btn.Font = Enum.Font.Gotham
    Btn.Parent = nil -- será parentado depois
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 8)

    Btn.MouseButton1Click:Connect(function()
        -- Simples ciclo de cores bonitas (pode expandir)
        local colors = {
            Color3.fromHex("#0dbf25"), Color3.fromHex("#ff1100"),
            Color3.fromHex("#c452c4"), Color3.fromHex("#00d0ff"),
            Color3.fromHex("#ffd700"), Color3.fromHex("#ff44aa")
        }
        local current = table.find(colors, Btn.BackgroundColor3) or 1
        local nextColor = colors[current % #colors + 1]
        Btn.BackgroundColor3 = nextColor
        onChange(nextColor)
    end)
    return Btn
end

-- ESP Section
NewToggle("Visualizar Itens", false, function(v) ESP.Items.Enabled = v end)
local itemColor = NewColorPicker(ESP.Items.Color, function(c) ESP.Items.Color = c end)
itemColor.Parent = Content

NewToggle("Visualizar Entidades", false, function(v) ESP.Entities.Enabled = v end)
local entColor = NewColorPicker(ESP.Entities.Color, function(c) ESP.Entities.Color = c end)
entColor.Parent = Content

NewToggle("Visualizar Players", false, function(v) ESP.Players.Enabled = v end)
local playerColor = NewColorPicker(ESP.Players.Color, function(c) ESP.Players.Color = c end)
playerColor.Parent = Content

-- Stamina
NewToggle("Stamina Infinita", false, function(v) InfiniteStamina = v end)

--// ESP Logic
local function ApplyChams(obj, color, trans)
    if obj:FindFirstChild("MinimalHighlight") then return end
    local hl = Instance.new("Highlight")
    hl.Name = "MinimalHighlight"
    hl.FillColor = color
    hl.OutlineColor = color
    hl.FillTransparency = trans
    hl.OutlineTransparency = 0.4
    hl.Adornee = obj
    hl.Parent = obj
end

RunService.RenderStepped:Connect(function()
    -- Itens (ajuste conforme seu jogo)
    if ESP.Items.Enabled then
        for _, v in ipairs(workspace:GetDescendants()) do
            if (v.Name:find("Item") or v.Name:find("Chest") or v:FindFirstChild("Value")) and v:IsA("Model") or v:IsA("Part") then
                ApplyChams(v, ESP.Items.Color, ESP.Items.Trans)
            end
        end
    end

    -- Entidades
    if ESP.Entities.Enabled then
        for _, v in ipairs(workspace:GetDescendants()) do
            if v:FindFirstChild("Humanoid") and not Players:GetPlayerFromCharacter(v) then
                ApplyChams(v, ESP.Entities.Color, ESP.Entities.Trans)
            end
        end
    end

    -- Players
    if ESP.Players.Enabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                ApplyChams(plr.Character, ESP.Players.Color, ESP.Players.Trans)
            end
        end
    end
end)

-- Stamina Loop
task.spawn(function()
    while task.wait(0.15) do
        if InfiniteStamina and character and character:FindFirstChild("Humanoid") then
            local hum = character.Humanoid
            if hum:FindFirstChild("Stamina") then hum.Stamina.Value = 100 end
            if hum:FindFirstChild("Sprint") then hum.Sprint.Value = 100 end
        end
    end
end)

player.CharacterAdded:Connect(function(c) character = c end)

print("✅ HUD Minimalista carregado!")
