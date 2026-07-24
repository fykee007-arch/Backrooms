local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
-- Se estiver usando em Studio, vai para o PlayerGui. Se for executor, tenta usar CoreGui para não bugar.
local targetParent = (pcall(function() return CoreGui.Name end) and CoreGui) or LocalPlayer:WaitForChild("PlayerGui")

-- Evita duplicatas
if targetParent:FindFirstChild("MurderNiceHUD") then
	targetParent.MurderNiceHUD:Destroy()
end

-- ==========================================
-- 1. BASE DA GUI
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MurderNiceHUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = targetParent

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 500, 0, 300)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 8)
MainCorner.Parent = MainFrame

-- Top Bar (Área de Arrastar e Título)
local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
TopBar.BorderSizePixel = 0
TopBar.Active = true
TopBar.Parent = MainFrame

local TopBarCorner = Instance.new("UICorner")
TopBarCorner.CornerRadius = UDim.new(0, 8)
TopBarCorner.Parent = TopBar

-- Tapa o canto inferior da TopBar para conectar com o resto do menu
local TopBarHider = Instance.new("Frame")
TopBarHider.Size = UDim2.new(1, 0, 0, 8)
TopBarHider.Position = UDim2.new(0, 0, 1, -8)
TopBarHider.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
TopBarHider.BorderSizePixel = 0
TopBarHider.Parent = TopBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0, 200, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "# Murder nice"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

-- Botões de Controle (- e X)
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Size = UDim2.new(0, 30, 1, 0)
MinimizeBtn.Position = UDim2.new(1, -60, 0, 0)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextSize = 18
MinimizeBtn.Parent = TopBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 1, 0)
CloseBtn.Position = UDim2.new(1, -30, 0, 0)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 16
CloseBtn.Parent = TopBar

-- ==========================================
-- 2. SIDEBAR E CONTEÚDO
-- ==========================================
local SideBar = Instance.new("Frame")
SideBar.Size = UDim2.new(0, 130, 1, -35)
SideBar.Position = UDim2.new(0, 0, 0, 35)
SideBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
SideBar.BorderSizePixel = 0
SideBar.Parent = MainFrame

local ContentArea = Instance.new("Frame")
ContentArea.Size = UDim2.new(1, -130, 1, -35)
ContentArea.Position = UDim2.new(0, 130, 0, 35)
ContentArea.BackgroundTransparency = 1
ContentArea.Parent = MainFrame

local TabList = Instance.new("UIListLayout")
TabList.SortOrder = Enum.SortOrder.LayoutOrder
TabList.Padding = UDim.new(0, 5)
TabList.Parent = SideBar

local SidePadding = Instance.new("UIPadding")
SidePadding.PaddingTop = UDim.new(0, 10)
SidePadding.PaddingLeft = UDim.new(0, 10)
SidePadding.PaddingRight = UDim.new(0, 10)
SidePadding.Parent = SideBar

-- Função para criar Abas
local Tabs = {}
local Pages = {}

local function createTab(name, defaultActive)
	local TabBtn = Instance.new("TextButton")
	TabBtn.Size = UDim2.new(1, 0, 0, 30)
	TabBtn.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
	TabBtn.BackgroundTransparency = defaultActive and 0 or 1
	TabBtn.BorderSizePixel = 0
	TabBtn.Text = name
	TabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	TabBtn.Font = Enum.Font.GothamSemibold
	TabBtn.TextSize = 14
	TabBtn.TextXAlignment = Enum.TextXAlignment.Left
	TabBtn.Parent = SideBar

	local TabPadding = Instance.new("UIPadding")
	TabPadding.PaddingLeft = UDim.new(0, 10)
	TabPadding.Parent = TabBtn

	local TabCorner = Instance.new("UICorner")
	TabCorner.CornerRadius = UDim.new(0, 6)
	TabCorner.Parent = TabBtn

	local RedLine = Instance.new("Frame")
	RedLine.Size = UDim2.new(0, 3, 0, 16)
	RedLine.Position = UDim2.new(0, -10, 0.5, -8)
	RedLine.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
	RedLine.BorderSizePixel = 0
	RedLine.Visible = defaultActive
	RedLine.Parent = TabBtn

	local Page = Instance.new("Frame")
	Page.Size = UDim2.new(1, 0, 1, 0)
	Page.BackgroundTransparency = 1
	Page.Visible = defaultActive
	Page.Parent = ContentArea
	
	local PagePadding = Instance.new("UIPadding")
	PagePadding.PaddingTop = UDim.new(0, 15)
	PagePadding.PaddingLeft = UDim.new(0, 15)
	PagePadding.PaddingRight = UDim.new(0, 15)
	PagePadding.Parent = Page

	Tabs[name] = {Btn = TabBtn, Line = RedLine, Page = Page}

	TabBtn.MouseButton1Click:Connect(function()
		for tName, data in pairs(Tabs) do
			data.Btn.BackgroundTransparency = 1
			data.Line.Visible = false
			data.Page.Visible = false
		end
		TabBtn.BackgroundTransparency = 0
		RedLine.Visible = true
		Page.Visible = true
	end)

	return Page
end

local MainPage = createTab("Main", true)
local VersaoPage = createTab("Versão", false)

-- ==========================================
-- 3. OPÇÕES DA ABA "MAIN"
-- ==========================================
local ESPLabel = Instance.new("TextLabel")
ESPLabel.Size = UDim2.new(1, 0, 0, 20)
ESPLabel.BackgroundTransparency = 1
ESPLabel.Text = "ESP"
ESPLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
ESPLabel.Font = Enum.Font.GothamBold
ESPLabel.TextSize = 16
ESPLabel.TextXAlignment = Enum.TextXAlignment.Left
ESPLabel.Parent = MainPage

local MainUIList = Instance.new("UIListLayout")
MainUIList.SortOrder = Enum.SortOrder.LayoutOrder
MainUIList.Padding = UDim.new(0, 8)
MainUIList.Parent = MainPage

ESPLabel.LayoutOrder = 1

local allToggles = {}

local function createToggle(page, name, order, hasColorPicker, isToggled)
	local Row = Instance.new("Frame")
	Row.Size = UDim2.new(1, 0, 0, 35)
	Row.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
	Row.BorderSizePixel = 0
	Row.LayoutOrder = order
	Row.Parent = page

	local RowCorner = Instance.new("UICorner")
	RowCorner.CornerRadius = UDim.new(0, 6)
	RowCorner.Parent = Row

	local Title = Instance.new("TextLabel")
	Title.Size = UDim2.new(0.5, 0, 1, 0)
	Title.Position = UDim2.new(0, 10, 0, 0)
	Title.BackgroundTransparency = 1
	Title.Text = name
	Title.TextColor3 = Color3.fromRGB(220, 220, 220)
	Title.Font = Enum.Font.GothamSemibold
	Title.TextSize = 14
	Title.TextXAlignment = Enum.TextXAlignment.Left
