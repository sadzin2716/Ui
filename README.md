--[[
	PurpleUI - Biblioteca de Interface para Roblox (Luau)
	Tema: Dark + Roxo | Moderna | Animações suaves | Suporte PC & Mobile

	COMO USAR (exemplo no fim do arquivo, dentro do bloco comentado "EXEMPLO DE USO")

	Recursos:
	- Sidebar com abas (Tabs)
	- Toggle (ativar/desativar)
	- Slider (valores numéricos)
	- Dropdown (seleção de lista)
	- Button (ação de uso único)
	- Botão de Minimizar (vira um quadrado pequeno com bordas arredondadas)
	- Botão de Fechar (destrói a UI por completo)
	- Arrastável (Drag) com suporte a mouse e toque (mobile)
]]

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--====================================================
-- TEMA
--====================================================
local Theme = {
	Background      = Color3.fromRGB(18, 16, 24),
	Sidebar         = Color3.fromRGB(14, 12, 20),
	Surface         = Color3.fromRGB(26, 22, 34),
	SurfaceLight    = Color3.fromRGB(34, 28, 46),
	Purple          = Color3.fromRGB(150, 80, 255),
	PurpleDark      = Color3.fromRGB(105, 55, 190),
	PurpleGlow      = Color3.fromRGB(180, 120, 255),
	Text            = Color3.fromRGB(235, 230, 245),
	SubText         = Color3.fromRGB(150, 145, 165),
	Stroke          = Color3.fromRGB(55, 48, 70),
	Success         = Color3.fromRGB(120, 220, 150),
	Font            = Enum.Font.GothamBold,
	FontRegular     = Enum.Font.Gotham,
}

local TWEEN_FAST = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local TWEEN_MED  = TweenInfo.new(0.28, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
local TWEEN_SPRING = TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out)

--====================================================
-- HELPERS
--====================================================
local function tween(obj, info, props)
	local t = TweenService:Create(obj, info, props)
	t:Play()
	return t
end

local function new(class, props, children)
	local inst = Instance.new(class)
	for k, v in pairs(props or {}) do
		inst[k] = v
	end
	for _, c in ipairs(children or {}) do
		c.Parent = inst
	end
	return inst
end

local function corner(radius)
	return new("UICorner", { CornerRadius = UDim.new(0, radius or 10) })
end

local function stroke(color, thickness, transparency)
	return new("UIStroke", {
		Color = color or Theme.Stroke,
		Thickness = thickness or 1,
		Transparency = transparency or 0,
	})
end

local function padding(l, t, r, b)
	return new("UIPadding", {
		PaddingLeft = UDim.new(0, l or 0),
		PaddingTop = UDim.new(0, t or 0),
		PaddingRight = UDim.new(0, r or 0),
		PaddingBottom = UDim.new(0, b or 0),
	})
end

local function isMobile()
	return UserInputService.TouchEnabled and not UserInputService.MouseEnabled
end

-- Torna um frame arrastável (mouse + touch)
local function makeDraggable(dragHandle, target)
	local dragging = false
	local dragStart, startPos

	local function update(input)
		local delta = input.Position - dragStart
		target.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y
		)
	end

	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = target.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	dragHandle.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			update(input)
		end
	end)
end

--====================================================
-- LIBRARY
--====================================================
local Library = {}
Library.__index = Library

function Library.new(config)
	config = config or {}
	local self = setmetatable({}, Library)

	self.Title = config.Title or "Purple UI"
	self.SubTitle = config.SubTitle or "v1.0"
	self.Tabs = {}
	self.ActiveTab = nil

	local mobile = isMobile()

	----------------------------------------------------
	-- ScreenGui
	----------------------------------------------------
	local screenGui = new("ScreenGui", {
		Name = "PurpleUI",
		ResetOnSpawn = false,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
		DisplayOrder = 999,
		IgnoreGuiInset = true,
	})

	local ok = pcall(function()
		screenGui.Parent = game:GetService("CoreGui")
	end)
	if not ok then
		screenGui.Parent = PlayerGui
	end
	self.ScreenGui = screenGui

	----------------------------------------------------
	-- Janela principal
	----------------------------------------------------
	local windowW, windowH = 620, 400
	if mobile then
		windowW, windowH = 340, 460
	end

	local main = new("Frame", {
		Name = "Main",
		BackgroundColor3 = Theme.Background,
		Size = UDim2.fromOffset(windowW, windowH),
		Position = UDim2.new(0.5, -windowW/2, 0.5, -windowH/2),
		ClipsDescendants = true,
		Parent = screenGui,
	})
	corner(14).Parent = main
	stroke(Theme.Stroke, 1).Parent = main

	-- sombra suave (usando ImageLabel padrão de sombra)
	local shadow = new("ImageLabel", {
		Name = "Shadow",
		BackgroundTransparency = 1,
		Image = "rbxassetid://1316045217",
		ImageColor3 = Color3.fromRGB(0,0,0),
		ImageTransparency = 0.5,
		ScaleType = Enum.ScaleType.Slice,
		SliceCenter = Rect.new(10,10,118,118),
		Size = UDim2.new(1, 60, 1, 60),
		Position = UDim2.new(0, -30, 0, -30),
		ZIndex = -1,
		Parent = main,
	})

	self.Main = main

	----------------------------------------------------
	-- Topbar
	----------------------------------------------------
	local topbar = new("Frame", {
		Name = "Topbar",
		BackgroundColor3 = Theme.Sidebar,
		Size = UDim2.new(1, 0, 0, 42),
		Parent = main,
	})
	corner(14).Parent = topbar
	-- corrige o corner só em cima
	local topbarFix = new("Frame", {
		BackgroundColor3 = Theme.Sidebar,
		Size = UDim2.new(1, 0, 0, 14),
		Position = UDim2.new(0, 0, 1, -14),
		BorderSizePixel = 0,
		Parent = topbar,
	})

	local titleLabel = new("TextLabel", {
		BackgroundTransparency = 1,
		Text = self.Title,
		Font = Theme.Font,
		TextSize = 15,
		TextColor3 = Theme.Text,
		TextXAlignment = Enum.TextXAlignment.Left,
		Position = UDim2.new(0, 14, 0, 0),
		Size = UDim2.new(0.5, 0, 1, 0),
		Parent = topbar,
	})

	local subLabel = new("TextLabel", {
		BackgroundTransparency = 1,
		Text = self.SubTitle,
		Font = Theme.FontRegular,
		TextSize = 12,
		TextColor3 = Theme.SubText,
		TextXAlignment = Enum.TextXAlignment.Left,
		Position = UDim2.new(0, 14, 0, 14),
		Size = UDim2.new(0.5, 0, 1, 0),
		Parent = topbar,
	})
	subLabel.Position = UDim2.new(0, 14, 0.5, 0)
	titleLabel.Size = UDim2.new(0.5, 0, 0.5, 0)
	titleLabel.Position = UDim2.new(0, 14, 0, 2)

	-- Botão Fechar
	local closeBtn = new("TextButton", {
		Name = "CloseButton",
		BackgroundColor3 = Theme.Surface,
		Size = UDim2.fromOffset(28, 28),
		Position = UDim2.new(1, -38, 0.5, -14),
		Text = "X",
		Font = Theme.Font,
		TextSize = 14,
		TextColor3 = Theme.SubText,
		AutoButtonColor = false,
		Parent = topbar,
	})
	corner(8).Parent = closeBtn

	-- Botão Minimizar
	local minBtn = new("TextButton", {
		Name = "MinimizeButton",
		BackgroundColor3 = Theme.Surface,
		Size = UDim2.fromOffset(28, 28),
		Position = UDim2.new(1, -74, 0.5, -14),
		Text = "-",
		Font = Theme.Font,
		TextSize = 18,
		TextColor3 = Theme.SubText,
		AutoButtonColor = false,
		Parent = topbar,
	})
	corner(8).Parent = minBtn

	for _, btn in ipairs({closeBtn, minBtn}) do
		btn.MouseEnter:Connect(function()
			tween(btn, TWEEN_FAST, {BackgroundColor3 = Theme.PurpleDark, TextColor3 = Theme.Text})
		end)
		btn.MouseLeave:Connect(function()
			tween(btn, TWEEN_FAST, {BackgroundColor3 = Theme.Surface, TextColor3 = Theme.SubText})
		end)
	end

	----------------------------------------------------
	-- Sidebar (abas)
	----------------------------------------------------
	local sidebarW = mobile and 90 or 150

	local sidebar = new("Frame", {
		Name = "Sidebar",
		BackgroundColor3 = Theme.Sidebar,
		Size = UDim2.new(0, sidebarW, 1, -42),
		Position = UDim2.new(0, 0, 0, 42),
		Parent = main,
	})

	local sidebarList = new("UIListLayout", {
		Padding = UDim.new(0, 6),
		SortOrder = Enum.SortOrder.LayoutOrder,
	})
	sidebarList.Parent = sidebar
	padding(8, 10, 8, 8).Parent = sidebar

	----------------------------------------------------
	-- Container de conteúdo
	----------------------------------------------------
	local content = new("Frame", {
		Name = "Content",
		BackgroundTransparency = 1,
		Size = UDim2.new(1, -sidebarW, 1, -42),
		Position = UDim2.new(0, sidebarW, 0, 42),
		Parent = main,
	})
	self.Content = content
	self.Sidebar = sidebar

	----------------------------------------------------
	-- Botão minimizado (pequeno quadrado)
	----------------------------------------------------
	local minimizedBtn = new("TextButton", {
		Name = "MinimizedIcon",
		BackgroundColor3 = Theme.Purple,
		Size = UDim2.fromOffset(46, 46),
		Position = main.Position,
		Text = "",
		Visible = false,
		AutoButtonColor = false,
		Parent = screenGui,
	})
	corner(14).Parent = minimizedBtn
	stroke(Theme.PurpleGlow, 1.5, 0.3).Parent = minimizedBtn

	local minIconLabel = new("TextLabel", {
		BackgroundTransparency = 1,
		Text = "◆",
		Font = Theme.Font,
		TextSize = 18,
		TextColor3 = Theme.Text,
		Size = UDim2.fromScale(1,1),
		Parent = minimizedBtn,
	})

	makeDraggable(minimizedBtn, minimizedBtn)
	makeDraggable(topbar, main)

	local minimized = false
	local function setMinimized(state)
		minimized = state
		if state then
			minimizedBtn.Position = main.Position
			minimizedBtn.Visible = true
			minimizedBtn.Size = UDim2.fromOffset(0,0)
			tween(main, TWEEN_MED, {Size = UDim2.fromOffset(0,0)})
			tween(minimizedBtn, TWEEN_SPRING, {Size = UDim2.fromOffset(46,46)})
			task.delay(0.28, function()
				main.Visible = false
			end)
		else
			main.Visible = true
			main.Position = minimizedBtn.Position
			main.Size = UDim2.fromOffset(0,0)
			tween(minimizedBtn, TWEEN_FAST, {Size = UDim2.fromOffset(0,0)})
			tween(main, TWEEN_SPRING, {Size = UDim2.fromOffset(windowW, windowH)})
			task.delay(0.2, function()
				minimizedBtn.Visible = false
			end)
		end
	end

	minBtn.MouseButton1Click:Connect(function()
		setMinimized(true)
	end)
	minimizedBtn.MouseButton1Click:Connect(function()
		setMinimized(false)
	end)

	closeBtn.MouseButton1Click:Connect(function()
		tween(main, TWEEN_FAST, {Size = UDim2.fromOffset(0,0), BackgroundTransparency = 1})
		task.delay(0.2, function()
			screenGui:Destroy()
		end)
	end)

	----------------------------------------------------
	-- Intro animation
	----------------------------------------------------
	main.Size = UDim2.fromOffset(0,0)
	main.Position = UDim2.new(0.5, 0, 0.5, 0)
	tween(main, TWEEN_SPRING, {
		Size = UDim2.fromOffset(windowW, windowH),
		Position = UDim2.new(0.5, -windowW/2, 0.5, -windowH/2),
	})

	self._windowSize = Vector2.new(windowW, windowH)
	self._mobile = mobile

	return self
end

--====================================================
-- TABS
--====================================================
function Library:AddTab(name, icon)
	local tabButton = new("TextButton", {
		Name = name,
		BackgroundColor3 = Theme.Surface,
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 0, 36),
		Text = "",
		AutoButtonColor = false,
		Parent = self.Sidebar,
	})
	corner(8).Parent = tabButton

	local indicator = new("Frame", {
		BackgroundColor3 = Theme.Purple,
		Size = UDim2.new(0, 3, 0, 0),
		Position = UDim2.new(0, 0, 0.5, 0),
		AnchorPoint = Vector2.new(0, 0.5),
		Parent = tabButton,
	})
	corner(2).Parent = indicator

	local label = new("TextLabel", {
		BackgroundTransparency = 1,
		Text = (icon and (icon.." ") or "") .. name,
		Font = Theme.FontRegular,
		TextSize = 13,
		TextColor3 = Theme.SubText,
		Size = UDim2.new(1, -16, 1, 0),
		Position = UDim2.new(0, 12, 0, 0),
		TextXAlignment = Enum.TextXAlignment.Left,
		TextTruncate = Enum.TextTruncate.AtEnd,
		Parent = tabButton,
	})

	local page = new("ScrollingFrame", {
		Name = name.."Page",
		BackgroundTransparency = 1,
		Size = UDim2.fromScale(1,1),
		CanvasSize = UDim2.new(0,0,0,0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
		ScrollBarThickness = 3,
		ScrollBarImageColor3 = Theme.Purple,
		Visible = false,
		Parent = self.Content,
	})
	padding(14, 14, 14, 14).Parent = page
	local pageList = new("UIListLayout", {
		Padding = UDim.new(0, 10),
		SortOrder = Enum.SortOrder.LayoutOrder,
	})
	pageList.Parent = page

	local tabObj = {
		Button = tabButton,
		Page = page,
		Name = name,
	}

	local function selectTab()
		if self.ActiveTab == tabObj then return end

		for _, t in ipairs(self.Tabs) do
			t.Page.Visible = false
			tween(t.Indicator or t.Button:FindFirstChild("Frame"), TWEEN_FAST, {Size = UDim2.new(0,3,0,0)})
			tween(t.Label or t.Button:FindFirstChild("TextLabel"), TWEEN_FAST, {TextColor3 = Theme.SubText})
			tween(t.Button, TWEEN_FAST, {BackgroundTransparency = 1})
		end

		page.Visible = true
		tween(indicator, TWEEN_MED, {Size = UDim2.new(0, 3, 0, 20)})
		tween(label, TWEEN_FAST, {TextColor3 = Theme.Text})
		tween(tabButton, TWEEN_FAST, {BackgroundTransparency = 0.85})

		self.ActiveTab = tabObj
	end

	tabObj.Indicator = indicator
	tabObj.Label = label
	tabObj.Select = selectTab

	tabButton.MouseButton1Click:Connect(selectTab)
	tabButton.MouseEnter:Connect(function()
		if self.ActiveTab ~= tabObj then
			tween(tabButton, TWEEN_FAST, {BackgroundTransparency = 0.92})
		end
	end)
	tabButton.MouseLeave:Connect(function()
		if self.ActiveTab ~= tabObj then
			tween(tabButton, TWEEN_FAST, {BackgroundTransparency = 1})
		end
	end)

	table.insert(self.Tabs, tabObj)

	if #self.Tabs == 1 then
		selectTab()
	end

	----------------------------------------------------
	-- API do conteúdo da aba (elementos)
	----------------------------------------------------
	local TabAPI = {}

	-- Card base usado por todos os elementos
	local function baseCard(height)
		local card = new("Frame", {
			BackgroundColor3 = Theme.Surface,
			Size = UDim2.new(1, 0, 0, height or 44),
			Parent = page,
		})
		corner(10).Parent = card
		stroke(Theme.Stroke, 1).Parent = card
		return card
	end

	-- SECTION LABEL
	function TabAPI:AddLabel(text)
		local lbl = new("TextLabel", {
			BackgroundTransparency = 1,
			Text = text,
			Font = Theme.Font,
			TextSize = 12,
			TextColor3 = Theme.SubText,
			Size = UDim2.new(1, 0, 0, 20),
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = page,
		})
		return lbl
	end

	-- BUTTON (uso único)
	function TabAPI:AddButton(config)
		config = config or {}
		local card = baseCard(42)
		local btn = new("TextButton", {
			BackgroundTransparency = 1,
			Size = UDim2.fromScale(1,1),
			Text = "",
			AutoButtonColor = false,
			Parent = card,
		})

		new("TextLabel", {
			BackgroundTransparency = 1,
			Text = config.Text or "Botão",
			Font = Theme.FontRegular,
			TextSize = 13,
			TextColor3 = Theme.Text,
			Size = UDim2.new(1, -24, 1, 0),
			Position = UDim2.new(0, 14, 0, 0),
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = card,
		})

		local arrow = new("TextLabel", {
			BackgroundTransparency = 1,
			Text = "›",
			Font = Theme.Font,
			TextSize = 16,
			TextColor3 = Theme.Purple,
			Size = UDim2.fromOffset(20, 20),
			Position = UDim2.new(1, -30, 0.5, -10),
			Parent = card,
		})

		btn.MouseEnter:Connect(function()
			tween(card, TWEEN_FAST, {BackgroundColor3 = Theme.SurfaceLight})
		end)
		btn.MouseLeave:Connect(function()
			tween(card, TWEEN_FAST, {BackgroundColor3 = Theme.Surface})
		end)
		btn.MouseButton1Down:Connect(function()
			tween(card, TWEEN_FAST, {BackgroundColor3 = Theme.PurpleDark})
		end)
		btn.MouseButton1Up:Connect(function()
			tween(card, TWEEN_FAST, {BackgroundColor3 = Theme.SurfaceLight})
		end)

		btn.MouseButton1Click:Connect(function()
			if config.Callback then
				task.spawn(config.Callback)
			end
		end)

		return card
	end

	-- TOGGLE
	function TabAPI:AddToggle(config)
		config = config or {}
		local state = config.Default or false

		local card = baseCard(42)

		new("TextLabel", {
			BackgroundTransparency = 1,
			Text = config.Text or "Toggle",
			Font = Theme.FontRegular,
			TextSize = 13,
			TextColor3 = Theme.Text,
			Size = UDim2.new(1, -70, 1, 0),
			Position = UDim2.new(0, 14, 0, 0),
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = card,
		})

		local switchBG = new("TextButton", {
			BackgroundColor3 = state and Theme.Purple or Theme.SurfaceLight,
			Size = UDim2.fromOffset(40, 22),
			Position = UDim2.new(1, -54, 0.5, -11),
			Text = "",
			AutoButtonColor = false,
			Parent = card,
		})
		corner(11).Parent = switchBG
		stroke(Theme.Stroke, 1).Parent = switchBG

		local knob = new("Frame", {
			BackgroundColor3 = Theme.Text,
			Size = UDim2.fromOffset(16, 16),
			Position = state and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
			Parent = switchBG,
		})
		corner(8).Parent = knob

		local function setState(newState, fireCallback)
			state = newState
			tween(switchBG, TWEEN_FAST, {BackgroundColor3 = state and Theme.Purple or Theme.SurfaceLight})
			tween(knob, TWEEN_SPRING, {Position = state and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)})
			if fireCallback ~= false and config.Callback then
				task.spawn(config.Callback, state)
			end
		end

		switchBG.MouseButton1Click:Connect(function()
			setState(not state)
		end)

		if config.Default then
			task.defer(function() setState(state, false) end)
		end

		return {
			Set = function(_, v) setState(v) end,
			Get = function() return state end,
		}
	end

	-- SLIDER
	function TabAPI:AddSlider(config)
		config = config or {}
		local min = config.Min or 0
		local max = config.Max or 100
		local default = math.clamp(config.Default or min, min, max)
		local decimals = config.Decimals or 0

		local card = baseCard(52)

		new("TextLabel", {
			BackgroundTransparency = 1,
			Text = config.Text or "Slider",
			Font = Theme.FontRegular,
			TextSize = 13,
			TextColor3 = Theme.Text,
			Size = UDim2.new(1, -80, 0, 20),
			Position = UDim2.new(0, 14, 0, 6),
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = card,
		})

		local valueLabel = new("TextLabel", {
			BackgroundTransparency = 1,
			Text = tostring(default),
			Font = Theme.Font,
			TextSize = 13,
			TextColor3 = Theme.Purple,
			Size = UDim2.new(0, 60, 0, 20),
			Position = UDim2.new(1, -74, 0, 6),
			TextXAlignment = Enum.TextXAlignment.Right,
			Parent = card,
		})

		local track = new("Frame", {
			BackgroundColor3 = Theme.SurfaceLight,
			Size = UDim2.new(1, -28, 0, 6),
			Position = UDim2.new(0, 14, 1, -16),
			Parent = card,
		})
		corner(3).Parent = track

		local fill = new("Frame", {
			BackgroundColor3 = Theme.Purple,
			Size = UDim2.new((default - min) / (max - min), 0, 1, 0),
			Parent = track,
		})
		corner(3).Parent = fill

		local knob = new("Frame", {
			BackgroundColor3 = Theme.Text,
			Size = UDim2.fromOffset(14, 14),
			AnchorPoint = Vector2.new(0.5, 0.5),
			Position = UDim2.new((default - min) / (max - min), 0, 0.5, 0),
			ZIndex = 2,
			Parent = track,
		})
		corner(7).Parent = knob
		stroke(Theme.Purple, 2).Parent = knob

		local dragging = false
		local value = default

		local function updateFromInputPos(xPos)
			local rel = math.clamp((xPos - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
			value = min + (max - min) * rel
			if decimals <= 0 then
				value = math.floor(value + 0.5)
			else
				local mult = 10 ^ decimals
				value = math.floor(value * mult + 0.5) / mult
			end
			local frac = (value - min) / (max - min)
			fill.Size = UDim2.new(frac, 0, 1, 0)
			knob.Position = UDim2.new(frac, 0, 0.5, 0)
			valueLabel.Text = tostring(value)
			if config.Callback then
				task.spawn(config.Callback, value)
			end
		end

		track.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				dragging = true
				updateFromInputPos(input.Position.X)
			end
		end)

		UserInputService.InputChanged:Connect(function(input)
			if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
				updateFromInputPos(input.Position.X)
			end
		end)

		UserInputService.InputEnded:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				dragging = false
			end
		end)

		return {
			Set = function(_, v)
				updateFromInputPos(track.AbsolutePosition.X + (math.clamp(v, min, max) - min)/(max-min) * track.AbsoluteSize.X)
			end,
			Get = function() return value end,
		}
	end

	-- DROPDOWN
	function TabAPI:AddDropdown(config)
		config = config or {}
		local options = config.Options or {}
		local current = config.Default or options[1]
		local open = false

		local card = baseCard(42)
		card.ClipsDescendants = false

		new("TextLabel", {
			BackgroundTransparency = 1,
			Text = config.Text or "Dropdown",
			Font = Theme.FontRegular,
			TextSize = 13,
			TextColor3 = Theme.Text,
			Size = UDim2.new(0.45, 0, 1, 0),
			Position = UDim2.new(0, 14, 0, 0),
			TextXAlignment = Enum.TextXAlignment.Left,
			Parent = card,
		})

		local selector = new("TextButton", {
			BackgroundColor3 = Theme.SurfaceLight,
			Size = UDim2.new(0.48, 0, 0, 30),
			Position = UDim2.new(1, -0.48*card.AbsoluteSize.X - 12, 0.5, -15),
			AnchorPoint = Vector2.new(0,0),
			Text = "",
			AutoButtonColor = false,
			Parent = card,
		})
		selector.Position = UDim2.new(1, -12, 0.5, -15)
		selector.AnchorPoint = Vector2.new(1, 0)
		corner(7).Parent = selector
		stroke(Theme.Stroke, 1).Parent = selector

		local selectedLabel = new("TextLabel", {
			BackgroundTransparency = 1,
			Text = tostring(current or "Selecionar"),
			Font = Theme.FontRegular,
			TextSize = 12,
			TextColor3 = Theme.SubText,
			Size = UDim2.new(1, -26, 1, 0),
			Position = UDim2.new(0, 10, 0, 0),
			TextXAlignment = Enum.TextXAlignment.Left,
			TextTruncate = Enum.TextTruncate.AtEnd,
			Parent = selector,
		})

		local arrowIcon = new("TextLabel", {
			BackgroundTransparency = 1,
			Text = "▾",
			Font = Theme.Font,
			TextSize = 12,
			TextColor3 = Theme.Purple,
			Size = UDim2.fromOffset(20, 20),
			Position = UDim2.new(1, -22, 0.5, -10),
			Parent = selector,
		})

		local listFrame = new("Frame", {
			BackgroundColor3 = Theme.SurfaceLight,
			Size = UDim2.new(1, 0, 0, 0),
			Position = UDim2.new(0, 0, 1, 6),
			ClipsDescendants = true,
			ZIndex = 10,
			Visible = false,
			Parent = selector,
		})
		corner(8).Parent = listFrame
		stroke(Theme.Stroke, 1).Parent = listFrame

		local listLayout = new("UIListLayout", {
			SortOrder = Enum.SortOrder.LayoutOrder,
		})
		listLayout.Parent = listFrame

		local function closeList()
			open = false
			tween(listFrame, TWEEN_FAST, {Size = UDim2.new(1, 0, 0, 0)})
			tween(arrowIcon, TWEEN_FAST, {Rotation = 0})
			task.delay(0.18, function()
				if not open then listFrame.Visible = false end
			end)
		end

		local function openList()
			open = true
			listFrame.Visible = true
			local h = math.min(#options * 28, 140)
			tween(listFrame, TWEEN_MED, {Size = UDim2.new(1, 0, 0, h)})
			tween(arrowIcon, TWEEN_FAST, {Rotation = 180})
		end

		for i, opt in ipairs(options) do
			local optBtn = new("TextButton", {
				BackgroundColor3 = Theme.SurfaceLight,
				BackgroundTransparency = 1,
				Size = UDim2.new(1, 0, 0, 28),
				Text = "",
				AutoButtonColor = false,
				LayoutOrder = i,
				Parent = listFrame,
			})
			new("TextLabel", {
				BackgroundTransparency = 1,
				Text = tostring(opt),
				Font = Theme.FontRegular,
				TextSize = 12,
				TextColor3 = Theme.Text,
				Size = UDim2.new(1, -16, 1, 0),
				Position = UDim2.new(0, 10, 0, 0),
				TextXAlignment = Enum.TextXAlignment.Left,
				Parent = optBtn,
			})
			optBtn.MouseEnter:Connect(function()
				tween(optBtn, TWEEN_FAST, {BackgroundTransparency = 0.6, BackgroundColor3 = Theme.Purple})
			end)
			optBtn.MouseLeave:Connect(function()
				tween(optBtn, TWEEN_FAST, {BackgroundTransparency = 1})
			end)
			optBtn.MouseButton1Click:Connect(function()
				current = opt
				selectedLabel.Text = tostring(opt)
				closeList()
				if config.Callback then
					task.spawn(config.Callback, opt)
				end
			end)
		end

		selector.MouseButton1Click:Connect(function()
			if open then closeList() else openList() end
		end)

		return {
			Set = function(_, v)
				current = v
				selectedLabel.Text = tostring(v)
			end,
			Get = function() return current end,
		}
	end

	return TabAPI
end

return Library

--[[
=========================================================
EXEMPLO DE USO (cole em um LocalScript separado):
=========================================================

local PurpleUI = loadstring(game:HttpGet("URL_DO_SEU_SCRIPT_AQUI"))()
-- ou: local PurpleUI = require(caminho.para.este.ModuleScript)

local Window = PurpleUI.new({
	Title = "Minha UI",
	SubTitle = "by VocÊ | v1.0",
})

local TabHome = Window:AddTab("Início", "🏠")
local TabCombat = Window:AddTab("Combate", "⚔️")
local TabConfig = Window:AddTab("Config", "⚙️")

TabHome:AddLabel("Geral")

TabHome:AddButton({
	Text = "Executar ação única",
	Callback = function()
		print("Botão pressionado!")
	end,
})

TabHome:AddToggle({
	Text = "Ativar Voo",
	Default = false,
	Callback = function(state)
		print("Voo:", state)
	end,
})

TabCombat:AddSlider({
	Text = "Velocidade",
	Min = 16,
	Max = 200,
	Default = 16,
	Callback = function(value)
		print("Velocidade:", value)
	end,
})

TabConfig:AddDropdown({
	Text = "Modo",
	Options = {"Fácil", "Médio", "Difícil"},
	Default = "Fácil",
	Callback = function(selected)
		print("Modo selecionado:", selected)
	end,
})

=========================================================
]]
