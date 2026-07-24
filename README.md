-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ----------------------------------------------------
-- CREATION OF SCREEN GUI
-- ----------------------------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MurderNiceHUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

-- ----------------------------------------------------
-- MAIN HUD FRAME
-- ----------------------------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 520, 0, 310)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -155)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Top Bar / Header
local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 15, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "# Murder nice"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 18
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TopBar

-- Window Controls (Minimize & Close)
local ControlsFrame = Instance.new("Frame")
ControlsFrame.Name = "ControlsFrame"
ControlsFrame.Size = UDim2.new(0, 60, 1, 0)
ControlsFrame.Position = UDim2.new(1, -65, 0, 0)
ControlsFrame.BackgroundTransparency = 1
ControlsFrame.Parent = TopBar

local UIListControls = Instance.new("UIListLayout")
UIListControls.FillDirection = Enum.FillDirection.Horizontal
UIListControls.HorizontalAlignment = Enum.HorizontalAlignment.Right
UIListControls.VerticalAlignment = Enum.VerticalAlignment.Center
UIListControls.Padding = UDim.new(0, 8)
UIListControls.Parent = ControlsFrame

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 24, 0, 24)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Color3.fromRGB(180, 180, 180)
MinimizeBtn.TextSize = 16
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Parent = ControlsFrame

local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 24, 0, 24)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(180, 180, 180)
CloseBtn.TextSize = 16
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.Parent = ControlsFrame

-- ----------------------------------------------------
-- SIDEBAR (NAVIGATION)
-- ----------------------------------------------------
local SideBar = Instance.new("Frame")
SideBar.Name = "SideBar"
SideBar.Size = UDim2.new(0, 140, 1, -40)
SideBar.Position = UDim2.new(0, 0, 0, 40)
SideBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
SideBar.BorderSizePixel = 0
SideBar.Parent = MainFrame

local SideBarList = Instance.new("UIListLayout")
SideBarList.SortOrder = Enum.SortOrder.LayoutOrder
SideBarList.Padding = UDim.new(0, 6)
SideBarList.Parent = SideBar

local SideBarPadding = Instance.new("UIPadding")
SideBarPadding.PaddingTop = UDim.new(0, 10)
SideBarPadding.PaddingLeft = UDim.new(0, 10)
SideBarPadding.PaddingRight = UDim.new(0, 10)
SideBarPadding.Parent = SideBar

-- Function to create sidebar tab buttons
local function createTabButton(name, text, order)
	local btn = Instance.new("TextButton")
	btn.Name = name .. "TabBtn"
	btn.Size = UDim2.new(1, 0, 0, 36)
	btn.BackgroundColor3 = Color3.fromRGB(42, 42, 42)
	btn.BorderSizePixel = 0
	btn.Text = "      " .. text
	btn.TextColor3 = Color3.fromRGB(180, 180, 180)
	btn.TextSize = 14
	btn.Font = Enum.Font.GothamMedium
	btn.TextXAlignment = Enum.TextXAlignment.Left
	btn.LayoutOrder = order
	btn.Parent = SideBar

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 6)
	btnCorner.Parent = btn

	local redBar = Instance.new("Frame")
	redBar.Name = "RedBar"
	redBar.Size = UDim2.new(0, 3, 0, 18)
	redBar.Position = UDim2.new(0, 0, 0.5, -9)
	redBar.BackgroundColor3 = Color3.fromRGB(235, 45, 45)
	redBar.BorderSizePixel = 0
	redBar.Visible = false
	redBar.Parent = btn

	local barCorner = Instance.new("UICorner")
	barCorner.CornerRadius = UDim.new(0, 2)
	barCorner.Parent = redBar

	return btn, redBar
end

local MainTabBtn, MainRedBar = createTabButton("Main", "Main", 1)
local VersaoTabBtn, VersaoRedBar = createTabButton("Versao", "Versão", 2)

-- ----------------------------------------------------
-- CONTENT PAGES
-- ----------------------------------------------------
local ContentContainer = Instance.new("Frame")
ContentContainer.Name = "ContentContainer"
ContentContainer.Size = UDim2.new(1, -140, 1, -40)
ContentContainer.Position = UDim2.new(0, 140, 0, 40)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

local ContentPadding = Instance.new("UIPadding")
ContentPadding.PaddingTop = UDim.new(0, 15)
ContentPadding.PaddingLeft = UDim.new(0, 20)
ContentPadding.PaddingRight = UDim.new(0, 20)
ContentPadding.Parent = ContentContainer

-- PAGE 1: MAIN
local MainPage = Instance.new("Frame")
MainPage.Name = "MainPage"
MainPage.Size = UDim2.new(1, 0, 1, 0)
MainPage.BackgroundTransparency = 1
MainPage.Visible = true
MainPage.Parent = ContentContainer

local MainHeader = Instance.new("TextLabel")
MainHeader.Size = UDim2.new(1, 0, 0, 25)
MainHeader.BackgroundTransparency = 1
MainHeader.Text = "ESP"
MainHeader.TextColor3 = Color3.fromRGB(255, 255, 255)
MainHeader.TextSize = 16
MainHeader.Font = Enum.Font.GothamBold
MainHeader.TextXAlignment = Enum.TextXAlignment.Left
MainHeader.Parent = MainPage

local MainList = Instance.new("UIListLayout")
MainList.SortOrder = Enum.SortOrder.LayoutOrder
MainList.Padding = UDim.new(0, 10)
MainList.Parent = MainPage

-- Toggle State Registry
local activeToggles = {}

-- Function to create Toggle Options
local function createToggleRow(name, labelText, order, hasColorPicker)
	local row = Instance.new("Frame")
	row.Name = name .. "Row"
	row.Size = UDim2.new(1, 0, 0, 42)
	row.BackgroundColor3 = Color3.fromRGB(48, 48, 48)
	row.BorderSizePixel = 0
	row.LayoutOrder = order
	row.Parent = MainPage

	local rowCorner = Instance.new("UICorner")
	rowCorner.CornerRadius = UDim.new(0, 8)
	rowCorner.Parent = row

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(0.6, 0, 1, 0)
	title.Position = UDim2.new(0, 12, 0, 0)
	title.BackgroundTransparency = 1
	title.Text = labelText
	title.TextColor3 = Color3.fromRGB(220, 220, 220)
	title.TextSize = 14
	title.Font = Enum.Font.GothamMedium
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = row

	-- Toggle Switch
	local toggleBg = Instance.new("TextButton")
	toggleBg.Name = "ToggleBg"
	toggleBg.Size = UDim2.new(0, 44, 0, 22)
	toggleBg.Position = UDim2.new(1, (hasColorPicker and -78 or -52), 0.5, -11)
	toggleBg.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
	toggleBg.AutoButtonColor = false
	toggleBg.Text = ""
	toggleBg.Parent = row

	local toggleCorner = Instance.new("UICorner")
	toggleCorner.CornerRadius = UDim.new(1, 0)
	toggleCorner.Parent = toggleBg

	local toggleCircle = Instance.new("Frame")
	toggleCircle.Name = "Circle"
	toggleCircle.Size = UDim2.new(0, 16, 0, 16)
	toggleCircle.Position = UDim2.new(0, 3, 0.5, -8)
	toggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	toggleCircle.BorderSizePixel = 0
	toggleCircle.Parent = toggleBg

	local circleCorner = Instance.new("UICorner")
	circleCorner.CornerRadius = UDim.new(1, 0)
	circleCorner.Parent = toggleCircle

	local state = false

	local function setToggleState(newState)
		state = newState
		activeToggles[name] = state

		if state then
			TweenService:Create(toggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(46, 204, 113)}):Play()
			TweenService:Create(toggleCircle, TweenInfo.new(0.2), {Position = UDim2.new(1, -19, 0.5, -8)}):Play()
		else
			TweenService:Create(toggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(100, 100, 100)}):Play()
			TweenService:Create(toggleCircle, TweenInfo.new(0.2), {Position = UDim2.new(0, 3, 0.5, -8)}):Play()
		end
	end

	toggleBg.MouseButton1Click:Connect(function()
		setToggleState(not state)
	end)

	if hasColorPicker then
		local colorBox = Instance.new("Frame")
		colorBox.Name = "ColorPicker"
		colorBox.Size = UDim2.new(0, 22, 0, 22)
		colorBox.Position = UDim2.new(1, -30, 0.5, -11)
		colorBox.BackgroundColor3 = Color3.fromRGB(46, 204, 113)
		colorBox.BorderSizePixel = 0
		colorBox.Parent = row

		local boxCorner = Instance.new("UICorner")
		boxCorner.CornerRadius = UDim.new(0, 6)
		boxCorner.Parent = colorBox
	end

	return {
		Reset = function() setToggleState(false) end,
		GetState = function() return state end
	}
end
