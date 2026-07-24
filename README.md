--// Murder Nice HUD - Exact UI Recreation (LocalScript)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MurderNiceHUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player:WaitForChild("PlayerGui")

-- Config
local ESP_Config = {
    AllPlayers = {Enabled = false, Color = Color3.fromHex("#00bf63")},
    MurderSheriff = {Enabled = false, MurderColor = Color3.fromHex("#c90e0e"), SheriffColor = Color3.fromHex("#5696e3")}
}

local minimized = false
local MainFrame, MiniIcon

--// Main Frame
MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 420, 0, 320)
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

-- Top Bar
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 45)
TopBar.BackgroundColor3 = Color3.fromRGB(15, 15, 17)
TopBar.Parent = MainFrame
Instance.new("UICorner", TopBar).CornerRadius = UDim.new(0, 10)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.6, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.Text = "# Murder nice"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Parent = TopBar

-- Close and Minimize
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -15)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.TextSize = 20
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Parent = TopBar

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 30, 0, 30)
MinBtn.Position = UDim2.new(1, -75, 0.5, -15)
MinBtn.BackgroundTransparency = 1
MinBtn.Text = "−"
MinBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
MinBtn.TextSize = 24
MinBtn.Font = Enum.Font.GothamBold
MinBtn.Parent = TopBar

-- Left Sidebar
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, -45)
Sidebar.Position = UDim2.new(0, 0, 0, 45)
Sidebar.BackgroundColor3 = Color3.fromRGB(18, 18, 20)
Sidebar.Parent = MainFrame

local MainTab = Instance.new("TextButton")
MainTab.Size = UDim2.new(1, 0, 0, 50)
MainTab.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
MainTab.Text = "Main"
MainTab.TextColor3 = Color3.new(1,1,1)
MainTab.Font = Enum.Font.GothamSemibold
MainTab.TextSize = 16
MainTab.Parent = Sidebar

local VersionTab = Instance.new("TextButton")
VersionTab.Size = UDim2.new(1, 0, 0, 50)
VersionTab.Position = UDim2.new(0, 0, 0, 50)
VersionTab.BackgroundColor3 = Color3.fromRGB(18, 18, 20)
VersionTab.Text = "Versão"
VersionTab.TextColor3 = Color3.new(1,1,1)
VersionTab.Font = Enum.Font.GothamSemibold
VersionTab.TextSize = 16
VersionTab.Parent = Sidebar

-- Content Area
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -130, 1, -45)
Content.Position = UDim2.new(0, 130, 0, 45)
Content.BackgroundColor3 = Color3.fromRGB(25, 25, 28)
Content.Parent = MainFrame

-- Main Tab Content
local MainContent = Instance.new("Frame")
MainContent.Size = UDim2.new(1,0,1,0)
MainContent.BackgroundTransparency = 1
MainContent.Visible = true
MainContent.Parent = Content

local ESPTitle = Instance.new("TextLabel")
ESPTitle.Size = UDim2.new(1,0,0,40)
ESPTitle.Text = "ESP"
ESPTitle.TextColor3 = Color3.new(1,1,1)
ESPTitle.BackgroundTransparency = 1
ESPTitle.Font = Enum.Font.GothamBold
ESPTitle.TextSize = 18
ESPTitle.Parent = MainContent

-- Toggle Function
local function CreateToggle(parent, text, default, color, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -30, 0, 50)
    frame.Position = UDim2.new(0, 15, 0, 50)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.6,0,1,0)
    label.Text = text
    label.TextColor3 = Color3.new(1,1,1)
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 16
    label.Parent = frame

    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0, 50, 0, 28)
    toggle.Position = UDim2.new(1, -60, 0.5, -14)
    toggle.BackgroundColor3 = default and Color3.fromRGB(0, 191, 99) or Color3.fromRGB(60,60,65)
    toggle.Text = ""
    toggle.Parent = frame
    Instance.new("UICorner", toggle).CornerRadius = UDim.new(1,0)

    local enabled = default
    toggle.MouseButton1Click:Connect(function()
        enabled = not enabled
        toggle.BackgroundColor3 = enabled and Color3.fromRGB(0, 191, 99) or Color3.fromRGB(60,60,65)
        callback(enabled)
    end)

    -- Color Picker
    local picker = Instance.new("TextButton")
    picker.Size = UDim2.new(0, 40, 0, 28)
    picker.Position = UDim2.new(1, -110, 0.5, -14)
    picker.BackgroundColor3 = color
    picker.Text = ""
    picker.Parent = frame
    Instance.new("UICorner", picker).CornerRadius = UDim.new(0,6)

    picker.MouseButton1Click:Connect(function()
        -- Simple color cycle
        local colors = {Color3.fromHex("#00bf63"), Color3.fromHex("#c90e0e"), Color3.fromHex("#5696e3"), Color3.fromHex("#00d0ff")}
        local idx = table.find(colors, picker.BackgroundColor3) or 1
        local newC = colors[idx % #colors + 1]
        picker.BackgroundColor3 = newC
        if text == "Jogadores" then ESP_Config.AllPlayers.Color = newC
        elseif text == "Murder & Xerife" then -- will handle separately
        end
    end)

    return frame
end

CreateToggle(MainContent, "Jogadores", false, Color3.fromHex("#00bf63"), function(v)
    ESP_Config.AllPlayers.Enabled = v
end)

CreateToggle(MainContent, "Murder & Xerife", false, Color3.fromHex("#c90e0e"), function(v)
    ESP_Config.MurderSheriff.Enabled = v
end)

-- Version Tab
local VersionContent = Instance.new("Frame")
VersionContent.Size = UDim2.new(1,0,1,0)
VersionContent.BackgroundTransparency = 1
VersionContent.Visible = false
VersionContent.Parent = Content

local VerText = Instance.new("TextLabel")
VerText.Size = UDim2.new(1,0,1,0)
VerText.BackgroundTransparency = 1
VerText.Text = "Versão: 1.0"
VerText.TextColor3 = Color3.new(1,1,1)
VerText.TextSize = 20
VerText.Font = Enum.Font.Gotham
VerText.Parent = VersionContent

-- Tab Switching
MainTab.MouseButton1Click:Connect(function()
    MainContent.Visible = true
    VersionContent.Visible = false
    MainTab.BackgroundColor3 = Color3.fromRGB(35,35,40)
    VersionTab.BackgroundColor3 = Color3.fromRGB(18,18,20)
end)

VersionTab.MouseButton1Click:Connect(function()
    MainContent.Visible = false
    VersionContent.Visible = true
    MainTab.BackgroundColor3 = Color3.fromRGB(18,18,20)
    VersionTab.BackgroundColor3 = Color3.fromRGB(35,35,40)
end)

--// Minimize System
local function Minimize()
    minimized = true
    MainFrame.Visible = false
    
    MiniIcon = Instance.new("ImageButton")
    MiniIcon.Size = UDim2.new(0, 70, 0, 70)
    MiniIcon.Position = UDim2.new(0.5, -35, 0.5, -35)
    MiniIcon.BackgroundColor3 = Color3.fromRGB(20,20,22)
    MiniIcon.Image = "rbxassetid://0" -- You can change to a real M logo
    MiniIcon.Parent = ScreenGui
    Instance.new("UICorner", MiniIcon).CornerRadius = UDim.new(0, 16)
    
    local MLabel = Instance.new("TextLabel")
    MLabel.Size = UDim2.new(1,0,1,0)
    MLabel.BackgroundTransparency = 1
    MLabel.Text = "M"
    MLabel.TextColor3 = Color3.new(1,1,1)
    MLabel.TextSize = 40
    MLabel.Font = Enum.Font.GothamBold
    MLabel.Parent = MiniIcon
    
    MiniIcon.Draggable = true
    MiniIcon.MouseButton1Click:Connect(function()
        minimized = false
        MiniIcon:Destroy()
        MainFrame.Visible = true
    end)
end

MinBtn.MouseButton1Click:Connect(Minimize)
CloseBtn.MouseButton1Click:Connect(function()
    ESP_Config.AllPlayers.Enabled = false
    ESP_Config.MurderSheriff.Enabled = false
    ScreenGui:Destroy()
end)

--// ESP System
local function ApplyHighlight(obj, color)
    if obj:FindFirstChild("MurderESP") then return end
    local hl = Instance.new("Highlight")
    hl.Name = "MurderESP"
    hl.FillColor = color
    hl.OutlineColor = color
    hl.FillTransparency = 0.75
    hl.OutlineTransparency = 0.3
    hl.Adornee = obj
    hl.Parent = obj
end

RunService.RenderStepped:Connect(function()
    if not ESP_Config.AllPlayers.Enabled and not ESP_Config.MurderSheriff.Enabled then return end

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local root = plr.Character
            local role = plr:FindFirstChild("leaderstats") and plr.leaderstats:FindFirstChild("Role") or nil

            if ESP_Config.AllPlayers.Enabled then
                ApplyHighlight(root, ESP_Config.AllPlayers.Color)
            end

            if ESP_Config.MurderSheriff.Enabled then
                if role and role.Value == "Murderer" then
                    ApplyHighlight(root, ESP_Config.MurderSheriff.MurderColor)
                elseif role and (role.Value == "Sheriff" or role.Value == "Xerife") then
                    ApplyHighlight(root, ESP_Config.MurderSheriff.SheriffColor)
                end
            end
        end
    end
end)

print("✅ Murder Nice HUD carregado com sucesso!")
