--// Murder Nice HUD - Murder Mystery 2
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = player.PlayerGui

-- Config ESP
local ESP_All = false
local ESP_Roles = false

local Color_All = Color3.fromHex("#00bf63")
local Color_Murder = Color3.fromHex("#c90e0e")
local Color_Sheriff = Color3.fromHex("#5696e3")

local highlights = {}

local function ClearHighlights()
	for _, hl in ipairs(highlights) do
		pcall(function() hl:Destroy() end)
	end
	highlights = {}
end

local function CreateHighlight(char, color)
	if not char or char:FindFirstChild("MM2ESP") then return end
	
	local hl = Instance.new("Highlight")
	hl.Name = "MM2ESP"
	hl.Adornee = char
	hl.FillColor = color
	hl.OutlineColor = color
	hl.FillTransparency = 0.72
	hl.OutlineTransparency = 0.25
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Parent = char
	
	table.insert(highlights, hl)
end

local function UpdateESP()
	ClearHighlights()
	
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr == player then continue end
		if not plr.Character then continue end
		
		local char = plr.Character
		local role = "Innocent"
		
		-- Detecção específica do Murder Mystery 2
		local leaderstats = plr:FindFirstChild("leaderstats")
		if leaderstats then
			local roleVal = leaderstats:FindFirstChild("Role") or leaderstats:FindFirstChild("role")
			if roleVal then
				local r = roleVal.Value
				if r == "Murderer" or r == "murderer" then
					role = "Murderer"
				elseif r == "Sheriff" or r == "sheriff" then
					role = "Sheriff"
				end
			end
		end
		
		-- Aplicar ESP
		if ESP_All then
			CreateHighlight(char, Color_All)
		end
		
		if ESP_Roles then
			if role == "Murderer" then
				CreateHighlight(char, Color_Murder)
			elseif role == "Sheriff" then
				CreateHighlight(char, Color_Sheriff)
			end
		end
	end
end

-- ==================== UI ====================
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 450, 0, 360)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -180)
MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 19)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 14)

-- Top Bar
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1,0,0,55)
TopBar.BackgroundColor3 = Color3.fromRGB(13,13,14)
TopBar.Parent = MainFrame
Instance.new("UICorner", TopBar).CornerRadius = UDim.new(0, 14)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -140, 1, 0)
Title.Position = UDim2.new(0, 20, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "# Murder nice"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 21
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

-- Close / Minimize
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0,36,0,36)
CloseBtn.Position = UDim2.new(1,-48,0.5,-18)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
CloseBtn.TextSize = 24
CloseBtn.Parent = TopBar

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0,36,0,36)
MinBtn.Position = UDim2.new(1,-88,0.5,-18)
MinBtn.BackgroundTransparency = 1
MinBtn.Text = "−"
MinBtn.TextColor3 = Color3.fromRGB(170,170,170)
MinBtn.TextSize = 28
MinBtn.Parent = TopBar

-- Sidebar
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 150, 1, -55)
Sidebar.Position = UDim2.new(0, 0, 0, 55)
Sidebar.BackgroundColor3 = Color3.fromRGB(21, 21, 22)
Sidebar.Parent = MainFrame

local MainTab = Instance.new("TextButton")
MainTab.Size = UDim2.new(1,0,0,70)
MainTab.BackgroundColor3 = Color3.fromRGB(30,30,33)
MainTab.Text = "  Main"
MainTab.TextColor3 = Color3.new(1,1,1)
MainTab.TextXAlignment = Enum.TextXAlignment.Left
MainTab.Font = Enum.Font.GothamSemibold
MainTab.TextSize = 18
MainTab.Parent = Sidebar

local RedLine = Instance.new("Frame")
RedLine.Size = UDim2.new(0,6,1,0)
RedLine.BackgroundColor3 = Color3.fromRGB(255, 45, 75)
RedLine.Parent = MainTab

-- Content
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -150, 1, -55)
Content.Position = UDim2.new(0, 150, 0, 55)
Content.BackgroundColor3 = Color3.fromRGB(25,25,27)
Content.Parent = MainFrame

-- Toggles
local function CreateToggle(y, text, defaultColor, callback)
	local Frame = Instance.new("Frame")
	Frame.Size = UDim2.new(0.9,0,0,58)
	Frame.Position = UDim2.new(0.05,0,0,y)
	Frame.BackgroundColor3 = Color3.fromRGB(32,32,36)
	Frame.Parent = Content
	Instance.new("UICorner", Frame).CornerRadius = UDim.new(0,12)

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(0.65,0,1,0)
	Label.BackgroundTransparency = 1
	Label.Text = "  " .. text
	Label.TextColor3 = Color3.new(1,1,1)
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.Font = Enum.Font.GothamSemibold
	Label.TextSize = 17
	Label.Parent = Frame

	local Toggle = Instance.new("TextButton")
	Toggle.Size = UDim2.new(0,55,0,30)
	Toggle.Position = UDim2.new(1,-70,0.5,-15)
	Toggle.BackgroundColor3 = Color3.fromRGB(55,55,60)
	Toggle.Text = ""
	Toggle.Parent = Frame
	Instance.new("UICorner", Toggle).CornerRadius = UDim.new(1,0)

	local enabled = false
	Toggle.MouseButton1Click:Connect(function()
		enabled = not enabled
		Toggle.BackgroundColor3 = enabled and Color3.fromRGB(0, 200, 100) or Color3.fromRGB(55,55,60)
		callback(enabled)
	end)
end

CreateToggle(50, "Jogadores", Color_All, function(v) ESP_All = v end)
CreateToggle(125, "Murder & Xerife", Color_Murder, function(v) ESP_Roles = v end)

-- ESP Loop
RunService.RenderStepped:Connect(function()
	if ESP_All or ESP_Roles then
		UpdateESP()
	else
		ClearHighlights()
	end
end)

CloseBtn.MouseButton1Click:Connect(function()
	ScreenGui:Destroy()
end)

print("✅ HUD para Murder Mystery 2 carregado!")
