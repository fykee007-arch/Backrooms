--// Murder Nice HUD - Visual Melhorado (Funções intactas)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MurderNiceHUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player.PlayerGui

-- Configurações ESP (mantidas)
local ESP_All = false
local ESP_Roles = false

local Color_All = Color3.fromHex("#00bf63")
local Color_Murder = Color3.fromHex("#c90e0e")
local Color_Sheriff = Color3.fromHex("#5696e3")

local highlights = {}

local function ClearHighlights()
	for _, hl in ipairs(highlights) do pcall(function() hl:Destroy() end) end
	highlights = {}
end

local function CreateHighlight(char, color)
	if not char or char:FindFirstChild("MM2ESP") then return end
	local hl = Instance.new("Highlight")
	hl.Name = "MM2ESP"
	hl.Adornee = char
	hl.FillColor = color
	hl.OutlineColor = color
	hl.FillTransparency = 0.68
	hl.OutlineTransparency = 0.25
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = char
	table.insert(highlights, hl)
end

local function GetRole(plr)
	local ls = plr:FindFirstChild("leaderstats")
	if ls then
		local role = ls:FindFirstChild("Role") or ls:FindFirstChild("role")
		if role then
			local r = tostring(role.Value):lower()
			if r:find("murder") then return "Murderer" end
			if r:find("sheriff") or r:find("xerife") then return "Sheriff" end
		end
	end
	return "Innocent"
end

local function UpdateESP()
	ClearHighlights()
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr == player or not plr.Character then continue end
		local role = GetRole(plr)
		
		if ESP_All then CreateHighlight(plr.Character, Color_All) end
		if ESP_Roles then
			if role == "Murderer" then CreateHighlight(plr.Character, Color_Murder)
			elseif role == "Sheriff" then CreateHighlight(plr.Character, Color_Sheriff) end
		end
	end
end

RunService.RenderStepped:Connect(function()
	if ESP_All or ESP_Roles then UpdateESP() else ClearHighlights() end
end)

-- ==================== HUD MELHORADO ====================
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 460, 0, 355)
MainFrame.Position = UDim2.new(0.5, -230, 0.5, -180)
MainFrame.BackgroundColor3 = Color3.fromRGB(16, 16, 18)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 16)

-- Top Bar
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1,0,0,52)
TopBar.BackgroundColor3 = Color3.fromRGB(11, 11, 13)
TopBar.Parent = MainFrame
Instance.new("UICorner", TopBar).CornerRadius = UDim.new(0, 16)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.65,0,1,0)
Title.Position = UDim2.new(0, 22, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "# Murder nice"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 21
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

-- Close and Minimize
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0,36,0,36)
CloseBtn.Position = UDim2.new(1,-48,0.5,-18)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 85, 85)
CloseBtn.TextSize = 23
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Parent = TopBar

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0,36,0,36)
MinBtn.Position = UDim2.new(1,-88,0.5,-18)
MinBtn.BackgroundTransparency = 1
MinBtn.Text = "−"
MinBtn.TextColor3 = Color3.fromRGB(160,160,160)
MinBtn.TextSize = 28
MinBtn.Parent = TopBar

-- Sidebar
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 155, 1, -52)
Sidebar.Position = UDim2.new(0, 0, 0, 52)
Sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 22)
Sidebar.Parent = MainFrame

local MainTab = Instance.new("TextButton")
MainTab.Size = UDim2.new(1,0,0,68)
MainTab.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
MainTab.Text = "  Main"
MainTab.TextColor3 = Color3.new(1,1,1)
MainTab.TextXAlignment = Enum.TextXAlignment.Left
MainTab.TextSize = 18
MainTab.Font = Enum.Font.GothamSemibold
MainTab.Parent = Sidebar

local RedAccent = Instance.new("Frame")
RedAccent.Size = UDim2.new(0, 5, 1, 0)
RedAccent.BackgroundColor3 = Color3.fromRGB(255, 40, 70)
RedAccent.Parent = MainTab

local VersionTab = Instance.new("TextButton")
VersionTab.Size = UDim2.new(1,0,0,68)
VersionTab.Position = UDim2.new(0,0,0,68)
VersionTab.BackgroundColor3 = Color3.fromRGB(20,20,22)
VersionTab.Text = "  Versão"
VersionTab.TextColor3 = Color3.fromRGB(170,170,170)
VersionTab.TextXAlignment = Enum.TextXAlignment.Left
VersionTab.TextSize = 18
VersionTab.Font = Enum.Font.GothamSemibold
VersionTab.Parent = Sidebar

-- Content Area
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -155, 1, -52)
Content.Position = UDim2.new(0, 155, 0, 52)
Content.BackgroundColor3 = Color3.fromRGB(24, 24, 26)
Content.Parent = MainFrame

-- Toggles
local function CreateToggle(yPos, text, callback)
	local f = Instance.new("Frame")
	f.Size = UDim2.new(0.92,0,0,56)
	f.Position = UDim2.new(0.04,0,0,yPos)
	f.BackgroundColor3 = Color3.fromRGB(30,30,34)
	f.Parent = Content
	Instance.new("UICorner", f).CornerRadius = UDim.new(0, 12)

	local lbl = Instance.new("TextLabel")
	lbl.Size = UDim2.new(0.6,0,1,0)
	lbl.BackgroundTransparency = 1
	lbl.Text = "   " .. text
	lbl.TextColor3 = Color3.new(1,1,1)
	lbl.TextXAlignment = Enum.TextXAlignment.Left
	lbl.Font = Enum.Font.GothamSemibold
	lbl.TextSize = 17
	lbl.Parent = f

	local tog = Instance.new("TextButton")
	tog.Size = UDim2.new(0,54,0,30)
	tog.Position = UDim2.new(1,-68,0.5,-15)
	tog.BackgroundColor3 = Color3.fromRGB(50,50,55)
	tog.Text = ""
	tog.Parent = f
	Instance.new("UICorner", tog).CornerRadius = UDim.new(1,0)

	local enabled = false
	tog.MouseButton1Click:Connect(function()
		enabled = not enabled
		tog.BackgroundColor3 = enabled and Color3.fromRGB(0, 195, 105) or Color3.fromRGB(50,50,55)
		callback(enabled)
	end)
end

CreateToggle(45, "Jogadores", function(v) ESP_All = v end)
CreateToggle(115, "Murder & Xerife", function(v) ESP_Roles = v end)

-- Fechar
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

print("✅ HUD melhorado carregado! (Funções mantidas)")
