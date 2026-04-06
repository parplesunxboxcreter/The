local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("ReplicatedStorage")
local TS = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")

local stopSpawning = false
local multiZoneActive = false
local selectedSpawns = {}

local parentGui = CoreGui
local Player = Players.LocalPlayer
if Player and Player:FindFirstChild("PlayerGui") then
	parentGui = Player.PlayerGui
end

local EventsFolder = RS:FindFirstChild("Events") or Instance.new("Folder", RS)
local RemoteFunction = EventsFolder:FindFirstChild("RemoteFunction") or Instance.new("Folder", EventsFolder)
local SandboxSettings = RemoteFunction:FindFirstChild("SandboxSettings")
local PlayerSpawn = RemoteFunction:FindFirstChild("PlayerSpawn")
local GiveUpEvent = EventsFolder:FindFirstChild("RemoteEvents") and EventsFolder.RemoteEvents:FindFirstChild("GiveUp")
local SandboxSpawn = RemoteFunction:FindFirstChild("SandboxSpawn")

print("Sandbox Spawner - Remote status:")
print("SandboxSettings:", SandboxSettings and "✅" or "❌")
print("PlayerSpawn:", PlayerSpawn and "✅" or "❌")
print("GiveUpEvent:", GiveUpEvent and "✅" or "❌")
print("SandboxSpawn:", SandboxSpawn and "✅" or "❌")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SandboxSpawner"
ScreenGui.Parent = parentGui
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local function setTextScaled(btn)
	btn.TextScaled = true
	btn.TextSize = 20
end

-- UIStroke helper: black for text elements, other colors for frames (we'll specify)
local function addTextStroke(obj, thickness)
	local stroke = Instance.new("UIStroke", obj)
	stroke.Color = Color3.new(0,0,0)  -- black
	stroke.Thickness = thickness or 1
	stroke.Transparency = 0.3
end

local function addFrameStroke(obj, color, thickness)
	local stroke = Instance.new("UIStroke", obj)
	stroke.Color = color
	stroke.Thickness = thickness or 2
	stroke.Transparency = 0
end

local function flashButton(btn)
	local orig = btn.BackgroundColor3
	btn.BackgroundColor3 = Color3.new(1,1,1)
	task.wait(0.1)
	btn.BackgroundColor3 = orig
end

local function waitWithStop(seconds)
	if seconds <= 0 then
		RunService.Heartbeat:Wait()
		return
	end
	local start = os.clock()
	while os.clock() - start < seconds and not stopSpawning do
		RunService.Heartbeat:Wait()
	end
end

-- Main frame (purple)
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 180, 0, 250)
Main.Position = UDim2.new(0.5, -90, 0.5, -125)
Main.BackgroundColor3 = Color3.fromRGB(25, 10, 40)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

Instance.new("UICorner", Main)
addFrameStroke(Main, Color3.fromRGB(150, 50, 255), 2)

-- Glowing circles
local function createGlowCircle(radius, positionOffset, color)
	local circle = Instance.new("ImageLabel")
	circle.Size = UDim2.new(0, radius*2, 0, radius*2)
	circle.Position = positionOffset
	circle.Image = "rbxassetid://266660794"
	circle.ImageColor3 = color
	circle.ImageTransparency = 0.5
	circle.BackgroundTransparency = 1
	circle.ZIndex = 1
	circle.Parent = Main
	return circle
end

local glowCircles = {
	createGlowCircle(60, UDim2.new(0, -30, 0, -30), Color3.fromRGB(150, 50, 255)),
	createGlowCircle(60, UDim2.new(1, -30, 0, -30), Color3.fromRGB(150, 50, 255)),
	createGlowCircle(60, UDim2.new(0, -30, 1, -30), Color3.fromRGB(150, 50, 255)),
	createGlowCircle(60, UDim2.new(1, -30, 1, -30), Color3.fromRGB(150, 50, 255)),
	createGlowCircle(80, UDim2.new(0.5, -40, 0.5, -40), Color3.fromRGB(200, 100, 255)),
}

task.spawn(function()
	while true do
		for _, circle in ipairs(glowCircles) do
			TS:Create(circle, TweenInfo.new(0.3), {ImageTransparency = 1}):Play()
			task.wait(0.3 + 0.5)
			TS:Create(circle, TweenInfo.new(0.2), {ImageTransparency = 0.5}):Play()
		end
		task.wait(0.8)
	end
end)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -110, 0, 30)
Title.Position = UDim2.new(0, 35, 0, 5)
Title.BackgroundTransparency = 1
Title.Text = "parple sundbox"
Title.TextColor3 = Color3.fromRGB(200, 150, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextScaled = true
Title.TextSize = 20
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Main
addTextStroke(Title, 1)

-- Left arrow (blue)
local LeftArrow = Instance.new("TextButton")
LeftArrow.Size = UDim2.new(0, 25, 0, 25)
LeftArrow.Position = UDim2.new(0, 5, 0, 5)
LeftArrow.BackgroundColor3 = Color3.fromRGB(40, 100, 180)
LeftArrow.Text = "<"
LeftArrow.TextColor3 = Color3.new(1,1,1)
LeftArrow.Font = Enum.Font.SourceSansBold
LeftArrow.TextScaled = true
LeftArrow.Parent = Main
Instance.new("UICorner", LeftArrow)
addTextStroke(LeftArrow, 1)

-- Multi‑zone toggle button (purple, not orange)
local MultiToggle = Instance.new("TextButton")
MultiToggle.Size = UDim2.new(0, 25, 0, 25)
MultiToggle.Position = UDim2.new(1, -90, 0, 5)
MultiToggle.BackgroundColor3 = Color3.fromRGB(100, 40, 180)
MultiToggle.Text = "🔁"
MultiToggle.TextColor3 = Color3.new(1,1,1)
MultiToggle.Font = Enum.Font.SourceSansBold
MultiToggle.TextScaled = true
MultiToggle.Parent = Main
Instance.new("UICorner", MultiToggle)
addTextStroke(MultiToggle, 1)

-- Right arrow (red)
local RightArrow = Instance.new("TextButton")
RightArrow.Size = UDim2.new(0, 25, 0, 25)
RightArrow.Position = UDim2.new(1, -60, 0, 5)
RightArrow.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
RightArrow.Text = ">"
RightArrow.TextColor3 = Color3.new(1,1,1)
RightArrow.Font = Enum.Font.SourceSansBold
RightArrow.TextScaled = true
RightArrow.Parent = Main
Instance.new("UICorner", RightArrow)
addTextStroke(RightArrow, 1)

local HelpBtn = Instance.new("TextButton")
HelpBtn.Size = UDim2.new(0, 25, 0, 25)
HelpBtn.Position = UDim2.new(1, -30, 0, 5)
HelpBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 180)
HelpBtn.Text = "❔"
HelpBtn.TextColor3 = Color3.new(1,1,1)
HelpBtn.Font = Enum.Font.SourceSansBold
HelpBtn.TextScaled = true
HelpBtn.Parent = Main
Instance.new("UICorner", HelpBtn)
addTextStroke(HelpBtn, 1)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 35)
CloseBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 0)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextScaled = true
CloseBtn.Parent = Main
Instance.new("UICorner", CloseBtn)
addTextStroke(CloseBtn, 1)

-- Warning popup
local WarnPopup = Instance.new("Frame")
WarnPopup.Size = UDim2.new(0, 260, 0, 140)
WarnPopup.Position = UDim2.new(0.5, -130, 0.5, -70)
WarnPopup.BackgroundColor3 = Color3.fromRGB(30, 10, 50)
WarnPopup.Visible = true
WarnPopup.ZIndex = 10
WarnPopup.Parent = ScreenGui
Instance.new("UICorner", WarnPopup)
addFrameStroke(WarnPopup, Color3.fromRGB(180, 50, 255), 2)

local WarnText = Instance.new("TextLabel")
WarnText.Size = UDim2.new(1, -20, 1, -40)
WarnText.Position = UDim2.new(0, 10, 0, 5)
WarnText.BackgroundTransparency = 1
WarnText.Text = "Must unlock battlers to do it, you're stupid if you try to spawn chronos without beating/unlock it"
WarnText.TextColor3 = Color3.fromRGB(255, 100, 100)
WarnText.TextWrapped = true
WarnText.Font = Enum.Font.SourceSansBold
WarnText.TextScaled = true
WarnText.Parent = WarnPopup
addTextStroke(WarnText, 1)

local WarnClose = Instance.new("TextButton")
WarnClose.Size = UDim2.new(0, 100, 0, 30)
WarnClose.Position = UDim2.new(0.5, -50, 1, -35)
WarnClose.BackgroundColor3 = Color3.fromRGB(120, 30, 180)
WarnClose.Text = "Close"
WarnClose.TextColor3 = Color3.new(1,1,1)
WarnClose.Font = Enum.Font.SourceSansBold
WarnClose.TextScaled = true
WarnClose.Parent = WarnPopup
Instance.new("UICorner", WarnClose)
addTextStroke(WarnClose, 1)
WarnClose.MouseButton1Click:Connect(function() WarnPopup.Visible = false end)

-- Help popup
local HelpPopup = Instance.new("Frame")
HelpPopup.Size = UDim2.new(0, 300, 0, 180)
HelpPopup.Position = UDim2.new(0.5, -150, 0.5, -90)
HelpPopup.BackgroundColor3 = Color3.fromRGB(30, 10, 50)
HelpPopup.Visible = false
HelpPopup.ZIndex = 10
HelpPopup.Parent = ScreenGui
Instance.new("UICorner", HelpPopup)
addFrameStroke(HelpPopup, Color3.fromRGB(255, 200, 100), 2)

local HelpText = Instance.new("TextLabel")
HelpText.Size = UDim2.new(1, -20, 1, -40)
HelpText.Position = UDim2.new(0, 10, 0, 5)
HelpText.BackgroundTransparency = 1
HelpText.Text = "Click on the spawn Id you desire... have fun in sandbox"
HelpText.TextColor3 = Color3.fromRGB(255, 255, 200)
HelpText.TextWrapped = true
HelpText.Font = Enum.Font.SourceSans
HelpText.TextScaled = true
HelpText.Parent = HelpPopup
addTextStroke(HelpText, 1)

local HelpOk = Instance.new("TextButton")
HelpOk.Size = UDim2.new(0, 100, 0, 30)
HelpOk.Position = UDim2.new(0.5, -50, 1, -35)
HelpOk.BackgroundColor3 = Color3.fromRGB(120, 30, 180)
HelpOk.Text = "Ok mate"
HelpOk.TextColor3 = Color3.new(1,1,1)
HelpOk.Font = Enum.Font.SourceSansBold
HelpOk.TextScaled = true
HelpOk.Parent = HelpPopup
Instance.new("UICorner", HelpOk)
addTextStroke(HelpOk, 1)
HelpOk.MouseButton1Click:Connect(function() HelpPopup.Visible = false end)

HelpBtn.MouseButton1Click:Connect(function()
	flashButton(HelpBtn)
	HelpPopup.Visible = true
end)

-- Left panel (Y=0, height 250)
local LeftPanel = Instance.new("Frame")
LeftPanel.Size = UDim2.new(0, 140, 0, 250)
LeftPanel.Position = UDim2.new(0, 10, 0, 0)
LeftPanel.BackgroundColor3 = Color3.fromRGB(15, 30, 55)
LeftPanel.BorderSizePixel = 0
LeftPanel.Visible = false
LeftPanel.ZIndex = 1
LeftPanel.Parent = Main
Instance.new("UICorner", LeftPanel)
addFrameStroke(LeftPanel, Color3.fromRGB(80, 150, 255), 2)

local LeftScroll = Instance.new("ScrollingFrame")
LeftScroll.Size = UDim2.new(1, -10, 1, -10)
LeftScroll.Position = UDim2.new(0, 5, 0, 5)
LeftScroll.BackgroundTransparency = 1
LeftScroll.BorderSizePixel = 0
LeftScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
LeftScroll.ScrollBarThickness = 4
LeftScroll.Parent = LeftPanel

local LeftLayout = Instance.new("UIListLayout", LeftScroll)
LeftLayout.Padding = UDim.new(0, 5)
LeftLayout.SortOrder = Enum.SortOrder.LayoutOrder

local function updateLeftCanvas()
	LeftScroll.CanvasSize = UDim2.new(0, 0, 0, LeftLayout.AbsoluteContentSize.Y + 10)
end
LeftLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateLeftCanvas)

local function createToggle(text, setting, initial)
	local state = initial
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, 30)
	btn.BackgroundColor3 = state and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
	btn.Text = text .. (state and " [ON]" or " [OFF]")
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextScaled = true
	btn.Parent = LeftScroll
	Instance.new("UICorner", btn)
	addTextStroke(btn, 1)
	btn.MouseButton1Click:Connect(function()
		flashButton(btn)
		state = not state
		btn.BackgroundColor3 = state and Color3.fromRGB(0,200,0) or Color3.fromRGB(200,0,0)
		btn.Text = text .. (state and " [ON]" or " [OFF]")
		if SandboxSettings then
			pcall(function() SandboxSettings:InvokeServer(setting, state) end)
		end
	end)
end

local function createAction(text, setting)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, 30)
	btn.BackgroundColor3 = Color3.fromRGB(180,40,40)
	btn.Text = text
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextScaled = true
	btn.Parent = LeftScroll
	Instance.new("UICorner", btn)
	addTextStroke(btn, 1)
	btn.MouseButton1Click:Connect(function()
		flashButton(btn)
		if SandboxSettings then
			pcall(function() SandboxSettings:InvokeServer(setting) end)
		end
	end)
end

-- Give Up
local GiveUp = Instance.new("TextButton")
GiveUp.Size = UDim2.new(1, 0, 0, 30)
GiveUp.BackgroundColor3 = Color3.fromRGB(180,40,40)
GiveUp.Text = "Give Up"
GiveUp.TextColor3 = Color3.new(1,1,1)
GiveUp.Font = Enum.Font.SourceSansBold
GiveUp.TextScaled = true
GiveUp.Parent = LeftScroll
Instance.new("UICorner", GiveUp)
addTextStroke(GiveUp, 1)
GiveUp.MouseButton1Click:Connect(function()
	flashButton(GiveUp)
	if GiveUpEvent then GiveUpEvent:FireServer() end
end)

-- Fast Lose label
local FastLoseLabel = Instance.new("TextLabel", LeftScroll)
FastLoseLabel.Size = UDim2.new(1,0,0,20)
FastLoseLabel.BackgroundTransparency = 1
FastLoseLabel.Text = "--- Fast Lose ---"
FastLoseLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
FastLoseLabel.Font = Enum.Font.SourceSansBold
FastLoseLabel.TextScaled = true
addTextStroke(FastLoseLabel, 1)

-- Toggles separator
local sep1 = Instance.new("TextLabel", LeftScroll)
sep1.Size = UDim2.new(1,0,0,20)
sep1.BackgroundTransparency = 1
sep1.Text = "--- Toggles ---"
sep1.TextColor3 = Color3.fromRGB(200,200,255)
sep1.Font = Enum.Font.SourceSansBold
sep1.TextScaled = true
addTextStroke(sep1, 1)

createToggle("SpawnTeamsSwap", "SpawnTeamsSwap", false)
createToggle("Pause Friendly", "PauseFriendlyBattlers", false)
createToggle("Pause Enemy", "PauseEnemyBattlers", false)
createToggle("Invul Enemy Base", "InvulnerableEnemyBase", false)
createToggle("Pause Enemy Spawning", "PauseEnemySpawning", false)

-- Actions separator
local sep2 = Instance.new("TextLabel", LeftScroll)
sep2.Size = UDim2.new(1,0,0,20)
sep2.BackgroundTransparency = 1
sep2.Text = "--- Actions ---"
sep2.TextColor3 = Color3.fromRGB(200,200,255)
sep2.Font = Enum.Font.SourceSansBold
sep2.TextScaled = true
addTextStroke(sep2, 1)

createAction("Kill Enemy Battlers", "KillEnemyBattlers")
createAction("Kill Friendly Battlers", "KillFriendlyBattlers")

-- Stop Spawning button moved into Actions
local StopBtn = Instance.new("TextButton")
StopBtn.Size = UDim2.new(1, 0, 0, 30)
StopBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
StopBtn.Text = "STOP SPAWNING"
StopBtn.TextColor3 = Color3.new(1,1,1)
StopBtn.Font = Enum.Font.SourceSansBold
StopBtn.TextScaled = true
StopBtn.Parent = LeftScroll
Instance.new("UICorner", StopBtn)
addTextStroke(StopBtn, 1)
StopBtn.MouseButton1Click:Connect(function()
	flashButton(StopBtn)
	stopSpawning = true
end)

-- Cannon Controls separator
local sep3 = Instance.new("TextLabel", LeftScroll)
sep3.Size = UDim2.new(1,0,0,20)
sep3.BackgroundTransparency = 1
sep3.Text = "--- Cannon Controls ---"
sep3.TextColor3 = Color3.fromRGB(200,200,255)
sep3.Font = Enum.Font.SourceSansBold
sep3.TextScaled = true
addTextStroke(sep3, 1)

local CannonFrame = Instance.new("Frame", LeftScroll)
CannonFrame.Size = UDim2.new(1,0,0,110)
CannonFrame.BackgroundTransparency = 1

local AmtLabel = Instance.new("TextLabel", CannonFrame)
AmtLabel.Size = UDim2.new(1,0,0,20)
AmtLabel.BackgroundTransparency = 1
AmtLabel.Text = "Amount:"
AmtLabel.TextColor3 = Color3.new(1,1,1)
AmtLabel.Font = Enum.Font.SourceSansBold
AmtLabel.TextScaled = true
AmtLabel.TextXAlignment = Enum.TextXAlignment.Left
addTextStroke(AmtLabel, 1)

local CannonAmount = Instance.new("TextBox", CannonFrame)
CannonAmount.Size = UDim2.new(1,0,0,25)
CannonAmount.Position = UDim2.new(0,0,0,20)
CannonAmount.BackgroundColor3 = Color3.fromRGB(60,30,90)
CannonAmount.Text = "1"
CannonAmount.PlaceholderText = "Number"
CannonAmount.TextColor3 = Color3.new(1,1,1)
CannonAmount.Font = Enum.Font.SourceSans
CannonAmount.TextScaled = true
Instance.new("UICorner", CannonAmount)
addTextStroke(CannonAmount, 1)

local WaitLabel = Instance.new("TextLabel", CannonFrame)
WaitLabel.Size = UDim2.new(1,0,0,20)
WaitLabel.Position = UDim2.new(0,0,0,50)
WaitLabel.BackgroundTransparency = 1
WaitLabel.Text = "Wait (s):"
WaitLabel.TextColor3 = Color3.new(1,1,1)
WaitLabel.Font = Enum.Font.SourceSansBold
WaitLabel.TextScaled = true
WaitLabel.TextXAlignment = Enum.TextXAlignment.Left
addTextStroke(WaitLabel, 1)

local CannonWait = Instance.new("TextBox", CannonFrame)
CannonWait.Size = UDim2.new(1,0,0,25)
CannonWait.Position = UDim2.new(0,0,0,70)
CannonWait.BackgroundColor3 = Color3.fromRGB(60,30,90)
CannonWait.Text = "0"
CannonWait.PlaceholderText = "Delay"
CannonWait.TextColor3 = Color3.new(1,1,1)
CannonWait.Font = Enum.Font.SourceSans
CannonWait.TextScaled = true
Instance.new("UICorner", CannonWait)
addTextStroke(CannonWait, 1)

local ShootBtn = Instance.new("TextButton", CannonFrame)
ShootBtn.Size = UDim2.new(1,0,0,30)
ShootBtn.Position = UDim2.new(0,0,0,100)
ShootBtn.BackgroundColor3 = Color3.fromRGB(40,100,180)
ShootBtn.Text = "Shoot Cannon(s)"
ShootBtn.TextColor3 = Color3.new(1,1,1)
ShootBtn.Font = Enum.Font.SourceSansBold
ShootBtn.TextScaled = true
Instance.new("UICorner", ShootBtn)
addTextStroke(ShootBtn, 1)
ShootBtn.MouseButton1Click:Connect(function()
	flashButton(ShootBtn)
	stopSpawning = false
	local amt = tonumber(CannonAmount.Text) or 1
	local wait = tonumber(CannonWait.Text) or 0
	amt = math.max(1, amt)
	wait = math.max(0, wait)
	task.spawn(function()
		for i = 1, amt do
			if stopSpawning then break end
			if PlayerSpawn then
				pcall(function() PlayerSpawn:InvokeServer("Cannon") end)
			end
			if i < amt then waitWithStop(wait) end
		end
	end)
end)

-- Blood Altar section (separate, below Cannon)
local sep4 = Instance.new("TextLabel", LeftScroll)
sep4.Size = UDim2.new(1,0,0,20)
sep4.BackgroundTransparency = 1
sep4.Text = "--- Blood Altar ---"
sep4.TextColor3 = Color3.fromRGB(200,200,255)
sep4.Font = Enum.Font.SourceSansBold
sep4.TextScaled = true
addTextStroke(sep4, 1)

local BloodFrame = Instance.new("Frame", LeftScroll)
BloodFrame.Size = UDim2.new(1,0,0,70)
BloodFrame.BackgroundTransparency = 1

local BloodAmtLabel = Instance.new("TextLabel", BloodFrame)
BloodAmtLabel.Size = UDim2.new(1,0,0,20)
BloodAmtLabel.BackgroundTransparency = 1
BloodAmtLabel.Text = "Amount:"
BloodAmtLabel.TextColor3 = Color3.new(1,1,1)
BloodAmtLabel.Font = Enum.Font.SourceSansBold
BloodAmtLabel.TextScaled = true
BloodAmtLabel.TextXAlignment = Enum.TextXAlignment.Left
addTextStroke(BloodAmtLabel, 1)

local BloodAmount = Instance.new("TextBox", BloodFrame)
BloodAmount.Size = UDim2.new(1,0,0,25)
BloodAmount.Position = UDim2.new(0,0,0,20)
BloodAmount.BackgroundColor3 = Color3.fromRGB(60,30,90)
BloodAmount.Text = "1"
BloodAmount.PlaceholderText = "Number"
BloodAmount.TextColor3 = Color3.new(1,1,1)
BloodAmount.Font = Enum.Font.SourceSans
BloodAmount.TextScaled = true
Instance.new("UICorner", BloodAmount)
addTextStroke(BloodAmount, 1)

local BloodWaitLabel = Instance.new("TextLabel", BloodFrame)
BloodWaitLabel.Size = UDim2.new(1,0,0,20)
BloodWaitLabel.Position = UDim2.new(0,0,0,50)
BloodWaitLabel.BackgroundTransparency = 1
BloodWaitLabel.Text = "Wait (s):"
BloodWaitLabel.TextColor3 = Color3.new(1,1,1)
BloodWaitLabel.Font = Enum.Font.SourceSansBold
BloodWaitLabel.TextScaled = true
BloodWaitLabel.TextXAlignment = Enum.TextXAlignment.Left
addTextStroke(BloodWaitLabel, 1)

local BloodWait = Instance.new("TextBox", BloodFrame)
BloodWait.Size = UDim2.new(1,0,0,25)
BloodWait.Position = UDim2.new(0,0,0,70)
BloodWait.BackgroundColor3 = Color3.fromRGB(60,30,90)
BloodWait.Text = "0"
BloodWait.PlaceholderText = "Delay"
BloodWait.TextColor3 = Color3.new(1,1,1)
BloodWait.Font = Enum.Font.SourceSans
BloodWait.TextScaled = true
Instance.new("UICorner", BloodWait)
addTextStroke(BloodWait, 1)

local bloodCooldowns = {}
local function createBloodBtn(text, spell)
	local btn = Instance.new("TextButton", LeftScroll)
	btn.Size = UDim2.new(1,0,0,30)
	btn.BackgroundColor3 = Color3.fromRGB(140,30,140)
	btn.Text = text
	btn.TextColor3 = Color3.new(1,1,1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextScaled = true
	Instance.new("UICorner", btn)
	addTextStroke(btn, 1)
	btn.MouseButton1Click:Connect(function()
		if bloodCooldowns[spell] and bloodCooldowns[spell] > tick() then
			btn.Text = "Cooldown..."
			task.wait(1)
			btn.Text = text
			return
		end
		flashButton(btn)
		stopSpawning = false
		local amt = tonumber(BloodAmount.Text) or 1
		local wait = tonumber(BloodWait.Text) or 0
		amt = math.max(1, amt)
		wait = math.max(0, wait)
		local remote = Player and Player:FindFirstChild("PlayerGui") and Player.PlayerGui:FindFirstChild("BloodAltarUI")
		if remote then remote = remote:FindFirstChild("RemoteFunction") end
		if remote then
			bloodCooldowns[spell] = tick() + 2
			btn.Text = "Cooldown..."
			task.spawn(function()
				for i = 1, amt do
					if stopSpawning then break end
					pcall(function() remote:InvokeServer(spell) end)
					if i < amt then waitWithStop(wait) end
				end
				task.wait(2)
				btn.Text = text
			end)
		end
	end)
	return btn
end

createBloodBtn("BloodStone", "BloodStone")
createBloodBtn("CrystalCast", "CrystalCast")
createBloodBtn("BloodSmite", "BloodSmite")

-- Name Changer section
local sep6 = Instance.new("TextLabel", LeftScroll)
sep6.Size = UDim2.new(1,0,0,20)
sep6.BackgroundTransparency = 1
sep6.Text = "--- Name Changer ---"
sep6.TextColor3 = Color3.fromRGB(200,200,255)
sep6.Font = Enum.Font.SourceSansBold
sep6.TextScaled = true
addTextStroke(sep6, 1)

local function applyNPCVisuals(model, text, textColor, strokeThickness, auraColor, auraSize, auraBrightness)
	if not model or not model:IsA("Model") then return end
	
	local existingTag = model:FindFirstChild("NameTag")
	if existingTag then existingTag:Destroy() end
	local existingAura = model:FindFirstChild("AuraGlow")
	if existingAura then existingAura:Destroy() end
	
	if not text or text == "" then return end
	
	local tag = Instance.new("BillboardGui")
	tag.Name = "NameTag"
	tag.Adornee = model
	tag.Size = UDim2.new(0, 200, 0, 40)
	tag.StudsOffset = Vector3.new(0, 3, 0)
	tag.AlwaysOnTop = true
	
	local label = Instance.new("TextLabel", tag)
	label.Size = UDim2.new(1,0,1,0)
	label.BackgroundTransparency = 1
	label.Text = text
	label.TextColor3 = textColor or Color3.new(1,1,1)
	label.TextScaled = true
	label.Font = Enum.Font.SourceSansBold
	if strokeThickness and strokeThickness > 0 then
		label.TextStrokeTransparency = 0
		label.TextStrokeColor3 = Color3.new(0,0,0) -- black outline
		label.TextStrokeThickness = strokeThickness
	else
		label.TextStrokeTransparency = 1
	end
	tag.Parent = model
	
	if auraEnabled and auraColor then
		local aura = Instance.new("BillboardGui")
		aura.Name = "AuraGlow"
		aura.Adornee = model
		aura.Size = UDim2.new(0, auraSize or 120, 0, auraSize or 120)
		aura.StudsOffset = Vector3.new(0, 0.5, 0)
		aura.AlwaysOnTop = true
		aura.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
		
		local frame = Instance.new("Frame", aura)
		frame.Size = UDim2.new(1,0,1,0)
		frame.BackgroundColor3 = auraColor
		frame.BackgroundTransparency = 1 - (auraBrightness or 0.4)
		frame.BorderSizePixel = 0
		Instance.new("UICorner", frame).CornerRadius = UDim.new(1,0)
		
		local inner = Instance.new("Frame", frame)
		inner.Size = UDim2.new(0.7,0,0.7,0)
		inner.Position = UDim2.new(0.15,0,0.15,0)
		inner.BackgroundColor3 = auraColor
		inner.BackgroundTransparency = 1 - ((auraBrightness or 0.4) * 0.7)
		inner.BorderSizePixel = 0
		Instance.new("UICorner", inner).CornerRadius = UDim.new(1,0)
		
		aura.Parent = model
	end
end

local function openNameChanger(folderName)
	local folder = workspace:FindFirstChild("NPCFolders") and workspace.NPCFolders:FindFirstChild(folderName)
	if not folder then
		warn("Folder not found: NPCFolders." .. folderName)
		return
	end
	
	local gui = Instance.new("ScreenGui")
	gui.Name = folderName .. "NameChanger"
	gui.Parent = parentGui
	gui.ResetOnSpawn = false
	
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 400, 0, 600)
	frame.Position = UDim2.new(0.5, -200, 0.5, -300)
	frame.BackgroundColor3 = Color3.fromRGB(25, 10, 40)
	frame.BorderSizePixel = 0
	frame.Active = true
	frame.Draggable = true
	frame.Parent = gui
	Instance.new("UICorner", frame)
	addFrameStroke(frame, Color3.fromRGB(150, 50, 255), 2)
	
	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, -10, 0, 30)
	title.Position = UDim2.new(0, 5, 0, 5)
	title.BackgroundTransparency = 1
	title.Text = folderName .. " Name Changer"
	title.TextColor3 = Color3.fromRGB(200,150,255)
	title.Font = Enum.Font.SourceSansBold
	title.TextScaled = true
	addTextStroke(title, 1)
	
	local close = Instance.new("TextButton", frame)
	close.Size = UDim2.new(0, 25, 0, 25)
	close.Position = UDim2.new(1, -30, 0, 5)
	close.BackgroundColor3 = Color3.fromRGB(80,0,0)
	close.Text = "X"
	close.TextColor3 = Color3.new(1,1,1)
	close.Font = Enum.Font.SourceSansBold
	close.TextScaled = true
	Instance.new("UICorner", close)
	addTextStroke(close, 1)
	close.MouseButton1Click:Connect(function() gui:Destroy() end)
	
	local refreshBtn = Instance.new("TextButton", frame)
	refreshBtn.Size = UDim2.new(0, 60, 0, 25)
	refreshBtn.Position = UDim2.new(1, -100, 0, 5)
	refreshBtn.BackgroundColor3 = Color3.fromRGB(60,60,120)
	refreshBtn.Text = "Refresh"
	refreshBtn.TextColor3 = Color3.new(1,1,1)
	refreshBtn.Font = Enum.Font.SourceSansBold
	refreshBtn.TextScaled = true
	Instance.new("UICorner", refreshBtn)
	addTextStroke(refreshBtn, 1)
	
	local colorFrame = Instance.new("Frame", frame)
	colorFrame.Size = UDim2.new(1, -10, 0, 100)
	colorFrame.Position = UDim2.new(0, 5, 0, 40)
	colorFrame.BackgroundColor3 = Color3.fromRGB(20,10,35)
	Instance.new("UICorner", colorFrame)
	
	local colorLabel = Instance.new("TextLabel", colorFrame)
	colorLabel.Size = UDim2.new(0.2, 0, 1, 0)
	colorLabel.BackgroundTransparency = 1
	colorLabel.Text = "Color:"
	colorLabel.TextColor3 = Color3.new(1,1,1)
	colorLabel.Font = Enum.Font.SourceSansBold
	colorLabel.TextScaled = true
	addTextStroke(colorLabel, 1)
	
	local currentColor = Color3.new(1,1,1)
	local currentStroke = 1
	local currentAuraSize = 120
	local currentAuraBrightness = 0.4
	local currentAuraEnabled = false
	local rainbowActive = false
	local rainbowThread = nil
	
	local colorButtons = {
		{name="Red", color=Color3.new(1,0,0)},
		{name="Green", color=Color3.new(0,1,0)},
		{name="Blue", color=Color3.new(0,0,1)},
		{name="Yellow", color=Color3.new(1,1,0)},
		{name="Purple", color=Color3.new(0.6,0,1)},
		{name="White", color=Color3.new(1,1,1)},
	}
	
	local function stopRainbow()
		if rainbowThread then
			task.cancel(rainbowThread)
			rainbowThread = nil
		end
		rainbowActive = false
	end
	
	local function startRainbow()
		stopRainbow()
		rainbowActive = true
		rainbowThread = task.spawn(function()
			local hue = 0
			while rainbowActive do
				currentColor = Color3.fromHSV(hue, 1, 1)
				hue = (hue + 0.0167) % 1
				task.wait(0.1)
			end
		end)
	end
	
	for i, data in ipairs(colorButtons) do
		local btn = Instance.new("TextButton", colorFrame)
		btn.Size = UDim2.new(0.1, 0, 0.4, -5)
		btn.Position = UDim2.new(0.2 + (i-1)*0.1, 5, 0, 2.5)
		btn.BackgroundColor3 = data.color
		btn.Text = ""
		Instance.new("UICorner", btn)
		addTextStroke(btn, 1)
		btn.MouseButton1Click:Connect(function()
			stopRainbow()
			currentColor = data.color
			flashButton(btn)
		end)
	end
	
	local rainbowBtn = Instance.new("TextButton", colorFrame)
	rainbowBtn.Size = UDim2.new(0.1, 0, 0.4, -5)
	rainbowBtn.Position = UDim2.new(0.2 + #colorButtons*0.1, 5, 0, 2.5)
	rainbowBtn.BackgroundColor3 = Color3.fromRGB(255,128,0)
	rainbowBtn.Text = "🌈"
	rainbowBtn.TextColor3 = Color3.new(1,1,1)
	rainbowBtn.Font = Enum.Font.SourceSansBold
	rainbowBtn.TextScaled = true
	Instance.new("UICorner", rainbowBtn)
	addTextStroke(rainbowBtn, 1)
	rainbowBtn.MouseButton1Click:Connect(function()
		if rainbowActive then
			stopRainbow()
			currentColor = Color3.new(1,1,1)
			flashButton(rainbowBtn)
		else
			startRainbow()
			flashButton(rainbowBtn)
		end
	end)
	
	local strokeLabel = Instance.new("TextLabel", colorFrame)
	strokeLabel.Size = UDim2.new(0.2, 0, 0.3, 0)
	strokeLabel.Position = UDim2.new(0, 5, 0.4, 5)
	strokeLabel.BackgroundTransparency = 1
	strokeLabel.Text = "Stroke:"
	strokeLabel.TextColor3 = Color3.new(1,1,1)
	strokeLabel.Font = Enum.Font.SourceSansBold
	strokeLabel.TextScaled = true
	addTextStroke(strokeLabel, 1)
	
	local strokeSlider = Instance.new("TextBox", colorFrame)
	strokeSlider.Size = UDim2.new(0.3, 0, 0.3, 0)
	strokeSlider.Position = UDim2.new(0.2, 5, 0.4, 5)
	strokeSlider.BackgroundColor3 = Color3.fromRGB(60,30,90)
	strokeSlider.Text = "1"
	strokeSlider.PlaceholderText = "Thickness"
	strokeSlider.TextColor3 = Color3.new(1,1,1)
	strokeSlider.Font = Enum.Font.SourceSans
	strokeSlider.TextScaled = true
	Instance.new("UICorner", strokeSlider)
	addTextStroke(strokeSlider, 1)
	strokeSlider:GetPropertyChangedSignal("Text"):Connect(function()
		local val = tonumber(strokeSlider.Text) or 1
		currentStroke = math.clamp(val, 0, 5)
	end)
	
	local sizeLabel = Instance.new("TextLabel", colorFrame)
	sizeLabel.Size = UDim2.new(0.2, 0, 0.3, 0)
	sizeLabel.Position = UDim2.new(0, 5, 0.7, 5)
	sizeLabel.BackgroundTransparency = 1
	sizeLabel.Text = "Aura Size:"
	sizeLabel.TextColor3 = Color3.new(1,1,1)
	sizeLabel.Font = Enum.Font.SourceSansBold
	sizeLabel.TextScaled = true
	addTextStroke(sizeLabel, 1)
	
	local sizeSlider = Instance.new("TextBox", colorFrame)
	sizeSlider.Size = UDim2.new(0.3, 0, 0.3, 0)
	sizeSlider.Position = UDim2.new(0.2, 5, 0.7, 5)
	sizeSlider.BackgroundColor3 = Color3.fromRGB(60,30,90)
	sizeSlider.Text = "120"
	sizeSlider.PlaceholderText = "Size"
	sizeSlider.TextColor3 = Color3.new(1,1,1)
	sizeSlider.Font = Enum.Font.SourceSans
	sizeSlider.TextScaled = true
	Instance.new("UICorner", sizeSlider)
	addTextStroke(sizeSlider, 1)
	sizeSlider:GetPropertyChangedSignal("Text"):Connect(function()
		local val = tonumber(sizeSlider.Text) or 120
		currentAuraSize = math.clamp(val, 50, 300)
	end)
	
	local brightLabel = Instance.new("TextLabel", colorFrame)
	brightLabel.Size = UDim2.new(0.2, 0, 0.3, 0)
	brightLabel.Position = UDim2.new(0.5, 5, 0.4, 5)
	brightLabel.BackgroundTransparency = 1
	brightLabel.Text = "Brightness:"
	brightLabel.TextColor3 = Color3.new(1,1,1)
	brightLabel.Font = Enum.Font.SourceSansBold
	brightLabel.TextScaled = true
	addTextStroke(brightLabel, 1)
	
	local brightSlider = Instance.new("TextBox", colorFrame)
	brightSlider.Size = UDim2.new(0.3, 0, 0.3, 0)
	brightSlider.Position = UDim2.new(0.7, 5, 0.4, 5)
	brightSlider.BackgroundColor3 = Color3.fromRGB(60,30,90)
	brightSlider.Text = "0.4"
	brightSlider.PlaceholderText = "Brightness"
	brightSlider.TextColor3 = Color3.new(1,1,1)
	brightSlider.Font = Enum.Font.SourceSans
	brightSlider.TextScaled = true
	Instance.new("UICorner", brightSlider)
	addTextStroke(brightSlider, 1)
	brightSlider:GetPropertyChangedSignal("Text"):Connect(function()
		local val = tonumber(brightSlider.Text) or 0.4
		currentAuraBrightness = math.clamp(val, 0.1, 1)
	end)
	
	local auraToggle = Instance.new("TextButton", colorFrame)
	auraToggle.Size = UDim2.new(0.15, 0, 0.3, 0)
	auraToggle.Position = UDim2.new(0.85, 5, 0.7, 5)
	auraToggle.BackgroundColor3 = Color3.fromRGB(80,80,120)
	auraToggle.Text = "Aura"
	auraToggle.TextColor3 = Color3.new(1,1,1)
	auraToggle.Font = Enum.Font.SourceSansBold
	auraToggle.TextScaled = true
	Instance.new("UICorner", auraToggle)
	addTextStroke(auraToggle, 1)
	auraToggle.MouseButton1Click:Connect(function()
		currentAuraEnabled = not currentAuraEnabled
		auraToggle.BackgroundColor3 = currentAuraEnabled and Color3.fromRGB(0,200,0) or Color3.fromRGB(80,80,120)
		flashButton(auraToggle)
	end)
	
	local scroll = Instance.new("ScrollingFrame", frame)
	scroll.Size = UDim2.new(1, -10, 1, -150)
	scroll.Position = UDim2.new(0, 5, 0, 150)
	scroll.BackgroundTransparency = 1
	scroll.BorderSizePixel = 0
	scroll.CanvasSize = UDim2.new(0,0,0,0)
	scroll.ScrollBarThickness = 4
	
	local layout = Instance.new("UIListLayout", scroll)
	layout.Padding = UDim.new(0, 5)
	
	local function refreshList()
		for _, child in ipairs(scroll:GetChildren()) do
			if child:IsA("Frame") then child:Destroy() end
		end
		local npcs = {}
		for _, child in ipairs(folder:GetChildren()) do
			if child:IsA("Model") then table.insert(npcs, child) end
		end
		for _, npc in ipairs(npcs) do
			local row = Instance.new("Frame", scroll)
			row.Size = UDim2.new(1, 0, 0, 45)
			row.BackgroundColor3 = Color3.fromRGB(40,20,60)
			Instance.new("UICorner", row)
			
			local nameLabel = Instance.new("TextLabel", row)
			nameLabel.Size = UDim2.new(0.4, -5, 1, 0)
			nameLabel.Position = UDim2.new(0, 5, 0, 0)
			nameLabel.BackgroundTransparency = 1
			nameLabel.Text = npc.Name
			nameLabel.TextColor3 = Color3.new(1,1,1)
			nameLabel.Font = Enum.Font.SourceSansBold
			nameLabel.TextScaled = true
			nameLabel.TextXAlignment = Enum.TextXAlignment.Left
			addTextStroke(nameLabel, 1)
			
			local input = Instance.new("TextBox", row)
			input.Size = UDim2.new(0.4, -5, 1, -10)
			input.Position = UDim2.new(0.4, 5, 0.5, -15)
			input.BackgroundColor3 = Color3.fromRGB(60,30,90)
			input.PlaceholderText = "Name tag text"
			input.TextColor3 = Color3.new(1,1,1)
			input.Font = Enum.Font.SourceSans
			input.TextScaled = true
			Instance.new("UICorner", input)
			addTextStroke(input, 1)
			
			local apply = Instance.new("TextButton", row)
			apply.Size = UDim2.new(0.2, -5, 1, -10)
			apply.Position = UDim2.new(0.8, 5, 0.5, -15)
			apply.BackgroundColor3 = Color3.fromRGB(40,100,180)
			apply.Text = "Apply"
			apply.TextColor3 = Color3.new(1,1,1)
			apply.Font = Enum.Font.SourceSansBold
			apply.TextScaled = true
			Instance.new("UICorner", apply)
			addTextStroke(apply, 1)
			apply.MouseButton1Click:Connect(function()
				local newText = input.Text
				if newText ~= "" then
					for _, other in ipairs(npcs) do
						if other.Name == npc.Name then
							applyNPCVisuals(other, newText, currentColor, currentStroke, currentColor, currentAuraSize, currentAuraBrightness)
						end
					end
				else
					for _, other in ipairs(npcs) do
						if other.Name == npc.Name then
							local tag = other:FindFirstChild("NameTag")
							if tag then tag:Destroy() end
							local aura = other:FindFirstChild("AuraGlow")
							if aura then aura:Destroy() end
						end
					end
				end
				flashButton(apply)
			end)
			
			local tag = npc:FindFirstChild("NameTag")
			if tag and tag:IsA("BillboardGui") then
				local label = tag:FindFirstChild("TextLabel")
				if label then input.Text = label.Text end
			end
		end
		scroll.CanvasSize = UDim2.new(0,0,0, #npcs * 50)
	end
	
	refreshBtn.MouseButton1Click:Connect(function()
		flashButton(refreshBtn)
		refreshList()
	end)
	
	refreshList()
end

local enemyNameBtn = Instance.new("TextButton", LeftScroll)
enemyNameBtn.Size = UDim2.new(1,0,0,30)
enemyNameBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 80)
enemyNameBtn.Text = "Enemy Name Tags"
enemyNameBtn.TextColor3 = Color3.new(1,1,1)
enemyNameBtn.Font = Enum.Font.SourceSansBold
enemyNameBtn.TextScaled = true
Instance.new("UICorner", enemyNameBtn)
addTextStroke(enemyNameBtn, 1)
enemyNameBtn.MouseButton1Click:Connect(function()
	flashButton(enemyNameBtn)
	openNameChanger("EnemyFolder")
end)

local friendlyNameBtn = Instance.new("TextButton", LeftScroll)
friendlyNameBtn.Size = UDim2.new(1,0,0,30)
friendlyNameBtn.BackgroundColor3 = Color3.fromRGB(40, 180, 80)
friendlyNameBtn.Text = "Friendly Name Tags"
friendlyNameBtn.TextColor3 = Color3.new(1,1,1)
friendlyNameBtn.Font = Enum.Font.SourceSansBold
friendlyNameBtn.TextScaled = true
Instance.new("UICorner", friendlyNameBtn)
addTextStroke(friendlyNameBtn, 1)
friendlyNameBtn.MouseButton1Click:Connect(function()
	flashButton(friendlyNameBtn)
	openNameChanger("FriendlyFolder")
end)

-- Texture Editor section
local sep7 = Instance.new("TextLabel", LeftScroll)
sep7.Size = UDim2.new(1,0,0,20)
sep7.BackgroundTransparency = 1
sep7.Text = "--- Texture Editor ---"
sep7.TextColor3 = Color3.fromRGB(200,200,255)
sep7.Font = Enum.Font.SourceSansBold
sep7.TextScaled = true
addTextStroke(sep7, 1)

local function formatTextureId(input)
	if not input or input == "" then return "" end
	input = input:gsub("^%s+", ""):gsub("%s+$", "")
	if input:match("^%d+$") then
		return "rbxassetid://" .. input
	elseif input:match("^rbxassetid://") or input:match("^http://") or input:match("^https://") then
		return input
	else
		return "rbxassetid://" .. input
	end
end

local function getAllTexturedParts(model)
	local parts = {}
	local function scan(obj)
		if obj:IsA("MeshPart") then
			table.insert(parts, {obj = obj, type = "MeshPart", texture = obj.TextureID, name = obj.Name})
		end
		if obj:IsA("SpecialMesh") and obj.Parent and obj.Parent:IsA("BasePart") then
			table.insert(parts, {obj = obj, type = "SpecialMesh", texture = obj.TextureId, name = obj.Parent.Name})
		end
		if obj:IsA("Shirt") or obj:IsA("Pants") or obj:IsA("Accessory") then
			table.insert(parts, {obj = obj, type = obj.ClassName, texture = obj.TextureId, name = obj.Name})
		end
		if obj:IsA("BasePart") then
			table.insert(parts, {obj = obj, type = "BasePart", color = obj.Color, name = obj.Name})
		end
		for _, child in ipairs(obj:GetChildren()) do
			scan(child)
		end
	end
	scan(model)
	return parts
end

local function copyTexturesToSame(sourceModel, folder)
	local name = sourceModel.Name
	local targetModels = {}
	for _, child in ipairs(folder:GetChildren()) do
		if child:IsA("Model") and child.Name == name and child ~= sourceModel then
			table.insert(targetModels, child)
		end
	end
	if #targetModels == 0 then
		warn("No other NPCs with the same name found.")
		return
	end
	
	local sourceTextures = {}
	for _, partInfo in ipairs(getAllTexturedParts(sourceModel)) do
		if partInfo.type ~= "BasePart" then
			local key = partInfo.name .. "|" .. partInfo.type
			sourceTextures[key] = partInfo.texture
		end
	end
	
	for _, target in ipairs(targetModels) do
		local targetParts = getAllTexturedParts(target)
		for _, partInfo in ipairs(targetParts) do
			if partInfo.type ~= "BasePart" then
				local key = partInfo.name .. "|" .. partInfo.type
				local tex = sourceTextures[key]
				if tex then
					if partInfo.type == "MeshPart" then
						partInfo.obj.TextureID = tex
					elseif partInfo.type == "SpecialMesh" then
						partInfo.obj.TextureId = tex
					elseif partInfo.type == "Shirt" or partInfo.type == "Pants" or partInfo.type == "Accessory" then
						partInfo.obj.TextureId = tex
					end
				end
			end
		end
	end
end

local function openColorPicker(callback)
	local pickerGui = Instance.new("ScreenGui")
	pickerGui.Name = "ColorPicker"
	pickerGui.Parent = parentGui
	pickerGui.ResetOnSpawn = false

	local frame = Instance.new("Frame", pickerGui)
	frame.Size = UDim2.new(0, 300, 0, 350)
	frame.Position = UDim2.new(0.5, -150, 0.5, -175)
	frame.BackgroundColor3 = Color3.fromRGB(30,10,50)
	frame.BorderSizePixel = 0
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)
	addFrameStroke(frame, Color3.fromRGB(150,50,255), 2)

	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, -10, 0, 30)
	title.Position = UDim2.new(0, 5, 0, 5)
	title.BackgroundTransparency = 1
	title.Text = "Pick a Color"
	title.TextColor3 = Color3.new(1,1,1)
	title.Font = Enum.Font.SourceSansBold
	title.TextScaled = true
	addTextStroke(title, 1)

	local close = Instance.new("TextButton", frame)
	close.Size = UDim2.new(0, 25, 0, 25)
	close.Position = UDim2.new(1, -30, 0, 5)
	close.BackgroundColor3 = Color3.fromRGB(80,0,0)
	close.Text = "X"
	close.TextColor3 = Color3.new(1,1,1)
	close.Font = Enum.Font.SourceSansBold
	close.TextScaled = true
	Instance.new("UICorner", close)
	addTextStroke(close, 1)
	close.MouseButton1Click:Connect(function() pickerGui:Destroy() end)

	local grid = Instance.new("Frame", frame)
	grid.Size = UDim2.new(1, -10, 0, 200)
	grid.Position = UDim2.new(0, 5, 0, 40)
	grid.BackgroundTransparency = 1

	local colors = {}
	for i = 0, 5 do
		for j = 0, 5 do
			local hue = i / 6
			local sat = 1
			local val = j / 5 + 0.2
			table.insert(colors, Color3.fromHSV(hue, sat, val))
		end
	end
	local extras = {Color3.new(1,1,1), Color3.new(0.9,0.9,0.9), Color3.new(0.8,0.8,0.8), Color3.new(0.5,0.5,0.5), Color3.new(0.2,0.2,0.2), Color3.new(0,0,0)}
	for _,c in ipairs(extras) do table.insert(colors, c) end

	local btnSize = 35
	local spacing = 5
	local perRow = math.floor((grid.AbsoluteSize.X - spacing) / (btnSize + spacing))
	local x = 0
	local y = 0
	for idx, col in ipairs(colors) do
		local btn = Instance.new("TextButton", grid)
		btn.Size = UDim2.new(0, btnSize, 0, btnSize)
		btn.Position = UDim2.new(0, x * (btnSize + spacing) + spacing, 0, y * (btnSize + spacing) + spacing)
		btn.BackgroundColor3 = col
		btn.Text = ""
		Instance.new("UICorner", btn)
		addTextStroke(btn, 1)
		btn.MouseButton1Click:Connect(function()
			callback(col)
			pickerGui:Destroy()
		end)
		x = x + 1
		if x >= perRow then
			x = 0
			y = y + 1
		end
	end
	grid.CanvasSize = UDim2.new(0, 0, 0, (y+1)*(btnSize+spacing) + spacing)

	local customFrame = Instance.new("Frame", frame)
	customFrame.Size = UDim2.new(1, -10, 0, 100)
	customFrame.Position = UDim2.new(0, 5, 0, 250)
	customFrame.BackgroundColor3 = Color3.fromRGB(20,10,35)
	Instance.new("UICorner", customFrame)

	local rSlider = Instance.new("TextButton", customFrame)
	rSlider.Size = UDim2.new(0.9, 0, 0, 25)
	rSlider.Position = UDim2.new(0, 5, 0, 5)
	rSlider.BackgroundColor3 = Color3.new(1,0,0)
	rSlider.Text = "Red"
	rSlider.TextColor3 = Color3.new(1,1,1)
	rSlider.TextScaled = true
	rSlider.AutoButtonColor = false
	addTextStroke(rSlider, 1)

	local gSlider = Instance.new("TextButton", customFrame)
	gSlider.Size = UDim2.new(0.9, 0, 0, 25)
	gSlider.Position = UDim2.new(0, 5, 0, 35)
	gSlider.BackgroundColor3 = Color3.new(0,1,0)
	gSlider.Text = "Green"
	gSlider.TextColor3 = Color3.new(1,1,1)
	gSlider.TextScaled = true
	gSlider.AutoButtonColor = false
	addTextStroke(gSlider, 1)

	local bSlider = Instance.new("TextButton", customFrame)
	bSlider.Size = UDim2.new(0.9, 0, 0, 25)
	bSlider.Position = UDim2.new(0, 5, 0, 65)
	bSlider.BackgroundColor3 = Color3.new(0,0,1)
	bSlider.Text = "Blue"
	bSlider.TextColor3 = Color3.new(1,1,1)
	bSlider.TextScaled = true
	bSlider.AutoButtonColor = false
	addTextStroke(bSlider, 1)

	local rValue = 1
	local gValue = 1
	local bValue = 1
	local function updateCustomColor()
		local newColor = Color3.new(rValue, gValue, bValue)
		callback(newColor)
		pickerGui:Destroy()
	end

	rSlider.MouseButton1Click:Connect(function()
		local newR = rValue + 0.1
		if newR > 1 then newR = 0 end
		rValue = newR
		updateCustomColor()
	end)
	gSlider.MouseButton1Click:Connect(function()
		local newG = gValue + 0.1
		if newG > 1 then newG = 0 end
		gValue = newG
		updateCustomColor()
	end)
	bSlider.MouseButton1Click:Connect(function()
		local newB = bValue + 0.1
		if newB > 1 then newB = 0 end
		bValue = newB
		updateCustomColor()
	end)
end

local faces = {
	["Left"] = Enum.NormalId.Left,
	["Right"] = Enum.NormalId.Right,
	["Bottom"] = Enum.NormalId.Bottom,
	["Top"] = Enum.NormalId.Top,
	["Front"] = Enum.NormalId.Front,
	["Back"] = Enum.NormalId.Back,
}

local function addDecal(part, face, textureId)
	if not part:IsA("BasePart") then return end
	local decal = Instance.new("Decal")
	decal.Texture = textureId
	decal.Face = face
	decal.Parent = part
end

local function removeDecal(decal)
	decal:Destroy()
end

local function openTextureEditor(folderName)
	local folder = workspace:FindFirstChild("NPCFolders") and workspace.NPCFolders:FindFirstChild(folderName)
	if not folder then
		warn("Folder not found: NPCFolders." .. folderName)
		return
	end

	local gui = Instance.new("ScreenGui")
	gui.Name = folderName .. "TextureEditor"
	gui.Parent = parentGui
	gui.ResetOnSpawn = false

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 500, 0, 600)
	frame.Position = UDim2.new(0.5, -250, 0.5, -300)
	frame.BackgroundColor3 = Color3.fromRGB(25, 10, 40)
	frame.BorderSizePixel = 0
	frame.Active = true
	frame.Draggable = true
	frame.Parent = gui
	Instance.new("UICorner", frame)
	addFrameStroke(frame, Color3.fromRGB(150, 50, 255), 2)

	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, -10, 0, 30)
	title.Position = UDim2.new(0, 5, 0, 5)
	title.BackgroundTransparency = 1
	title.Text = folderName .. " Texture & Color Editor"
	title.TextColor3 = Color3.fromRGB(200,150,255)
	title.Font = Enum.Font.SourceSansBold
	title.TextScaled = true
	addTextStroke(title, 1)

	local close = Instance.new("TextButton", frame)
	close.Size = UDim2.new(0, 25, 0, 25)
	close.Position = UDim2.new(1, -30, 0, 5)
	close.BackgroundColor3 = Color3.fromRGB(80,0,0)
	close.Text = "X"
	close.TextColor3 = Color3.new(1,1,1)
	close.Font = Enum.Font.SourceSansBold
	close.TextScaled = true
	Instance.new("UICorner", close)
	addTextStroke(close, 1)
	close.MouseButton1Click:Connect(function() gui:Destroy() end)

	local refreshBtn = Instance.new("TextButton", frame)
	refreshBtn.Size = UDim2.new(0, 60, 0, 25)
	refreshBtn.Position = UDim2.new(1, -100, 0, 5)
	refreshBtn.BackgroundColor3 = Color3.fromRGB(60,60,120)
	refreshBtn.Text = "Refresh"
	refreshBtn.TextColor3 = Color3.new(1,1,1)
	refreshBtn.Font = Enum.Font.SourceSansBold
	refreshBtn.TextScaled = true
	Instance.new("UICorner", refreshBtn)
	addTextStroke(refreshBtn, 1)

	local globalFrame = Instance.new("Frame", frame)
	globalFrame.Size = UDim2.new(1, -10, 0, 70)
	globalFrame.Position = UDim2.new(0, 5, 0, 40)
	globalFrame.BackgroundColor3 = Color3.fromRGB(20,10,35)
	Instance.new("UICorner", globalFrame)

	local textureIdBox = Instance.new("TextBox", globalFrame)
	textureIdBox.Size = UDim2.new(0.6, -10, 0, 30)
	textureIdBox.Position = UDim2.new(0, 5, 0, 5)
	textureIdBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
	textureIdBox.PlaceholderText = "Texture ID (rbxassetid://... or number)"
	textureIdBox.TextColor3 = Color3.new(1,1,1)
	textureIdBox.Font = Enum.Font.SourceSans
	textureIdBox.TextScaled = true
	Instance.new("UICorner", textureIdBox)
	addTextStroke(textureIdBox, 1)

	local applyTextureAll = Instance.new("TextButton", globalFrame)
	applyTextureAll.Size = UDim2.new(0.35, -5, 0, 30)
	applyTextureAll.Position = UDim2.new(0.65, 5, 0, 5)
	applyTextureAll.BackgroundColor3 = Color3.fromRGB(40,100,180)
	applyTextureAll.Text = "Apply Texture to ALL"
	applyTextureAll.TextColor3 = Color3.new(1,1,1)
	applyTextureAll.Font = Enum.Font.SourceSansBold
	applyTextureAll.TextScaled = true
	Instance.new("UICorner", applyTextureAll)
	addTextStroke(applyTextureAll, 1)

	local colorPickerBtn = Instance.new("TextButton", globalFrame)
	colorPickerBtn.Size = UDim2.new(0.2, 0, 0, 25)
	colorPickerBtn.Position = UDim2.new(0, 5, 0, 40)
	colorPickerBtn.BackgroundColor3 = Color3.new(1,1,1)
	colorPickerBtn.Text = "Pick Color"
	colorPickerBtn.TextColor3 = Color3.new(0,0,0)
	colorPickerBtn.Font = Enum.Font.SourceSansBold
	colorPickerBtn.TextScaled = true
	Instance.new("UICorner", colorPickerBtn)
	addTextStroke(colorPickerBtn, 1)

	local colorDisplay = Instance.new("TextLabel", globalFrame)
	colorDisplay.Size = UDim2.new(0.2, 0, 0, 25)
	colorDisplay.Position = UDim2.new(0.22, 5, 0, 40)
	colorDisplay.BackgroundColor3 = Color3.new(1,1,1)
	colorDisplay.Text = "Selected Color"
	colorDisplay.TextColor3 = Color3.new(0,0,0)
	colorDisplay.Font = Enum.Font.SourceSansBold
	colorDisplay.TextScaled = true
	Instance.new("UICorner", colorDisplay)
	addTextStroke(colorDisplay, 1)

	local applyColorAll = Instance.new("TextButton", globalFrame)
	applyColorAll.Size = UDim2.new(0.35, -5, 0, 25)
	applyColorAll.Position = UDim2.new(0.45, 5, 0, 40)
	applyColorAll.BackgroundColor3 = Color3.fromRGB(40,100,180)
	applyColorAll.Text = "Apply Color to ALL"
	applyColorAll.TextColor3 = Color3.new(1,1,1)
	applyColorAll.Font = Enum.Font.SourceSansBold
	applyColorAll.TextScaled = true
	Instance.new("UICorner", applyColorAll)
	addTextStroke(applyColorAll, 1)

	local selectedColor = Color3.new(1,1,1)
	colorDisplay.BackgroundColor3 = selectedColor
	colorPickerBtn.MouseButton1Click:Connect(function()
		openColorPicker(function(col)
			selectedColor = col
			colorDisplay.BackgroundColor3 = col
		end)
	end)

	local scroll = Instance.new("ScrollingFrame", frame)
	scroll.Size = UDim2.new(1, -10, 1, -130)
	scroll.Position = UDim2.new(0, 5, 0, 115)
	scroll.BackgroundTransparency = 1
	scroll.BorderSizePixel = 0
	scroll.CanvasSize = UDim2.new(0,0,0,0)
	scroll.ScrollBarThickness = 4

	local layout = Instance.new("UIListLayout", scroll)
	layout.Padding = UDim.new(0, 5)

	local function refreshList()
		for _, child in ipairs(scroll:GetChildren()) do
			if child:IsA("Frame") then child:Destroy() end
		end
		local npcs = {}
		for _, child in ipairs(folder:GetChildren()) do
			if child:IsA("Model") then table.insert(npcs, child) end
		end
		local totalHeight = 0
		for _, npc in ipairs(npcs) do
			local parts = getAllTexturedParts(npc)
			local npcFrame = Instance.new("Frame", scroll)
			npcFrame.Size = UDim2.new(1, 0, 0, 60 + #parts * 70)
			npcFrame.BackgroundColor3 = Color3.fromRGB(40,20,60)
			Instance.new("UICorner", npcFrame)

			local titleLabel = Instance.new("TextLabel", npcFrame)
			titleLabel.Size = UDim2.new(0.6, -5, 0, 30)
			titleLabel.Position = UDim2.new(0, 5, 0, 0)
			titleLabel.BackgroundTransparency = 1
			titleLabel.Text = npc.Name
			titleLabel.TextColor3 = Color3.new(1,1,1)
			titleLabel.Font = Enum.Font.SourceSansBold
			titleLabel.TextScaled = true
			titleLabel.TextXAlignment = Enum.TextXAlignment.Left
			addTextStroke(titleLabel, 1)

			local copyBtn = Instance.new("TextButton", npcFrame)
			copyBtn.Size = UDim2.new(0.3, -5, 0, 30)
			copyBtn.Position = UDim2.new(0.7, 5, 0, 0)
			copyBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 180)
			copyBtn.Text = "Copy to All Same"
			copyBtn.TextColor3 = Color3.new(1,1,1)
			copyBtn.Font = Enum.Font.SourceSansBold
			copyBtn.TextScaled = true
			Instance.new("UICorner", copyBtn)
			addTextStroke(copyBtn, 1)
			copyBtn.MouseButton1Click:Connect(function()
				copyTexturesToSame(npc, folder)
				flashButton(copyBtn)
				refreshList()
			end)

			local y = 35
			for _, partInfo in ipairs(parts) do
				local partFrame = Instance.new("Frame", npcFrame)
				partFrame.Size = UDim2.new(1, -10, 0, 30)
				partFrame.Position = UDim2.new(0, 5, 0, y)
				partFrame.BackgroundColor3 = Color3.fromRGB(30,15,45)
				Instance.new("UICorner", partFrame)

				local nameLabel = Instance.new("TextLabel", partFrame)
				nameLabel.Size = UDim2.new(0.35, -5, 1, 0)
				nameLabel.Position = UDim2.new(0, 5, 0, 0)
				nameLabel.BackgroundTransparency = 1
				nameLabel.Text = partInfo.name
				nameLabel.TextColor3 = Color3.new(0.8,0.8,1)
				nameLabel.Font = Enum.Font.SourceSansBold
				nameLabel.TextScaled = true
				nameLabel.TextXAlignment = Enum.TextXAlignment.Left
				addTextStroke(nameLabel, 1)

				if partInfo.type == "BasePart" then
					local colorInput = Instance.new("TextBox", partFrame)
					colorInput.Size = UDim2.new(0.25, -5, 1, -5)
					colorInput.Position = UDim2.new(0.35, 5, 0, 2.5)
					colorInput.BackgroundColor3 = Color3.fromRGB(60,30,90)
					colorInput.PlaceholderText = "Color (R,G,B)"
					colorInput.Text = string.format("%.2f,%.2f,%.2f", partInfo.obj.Color.R, partInfo.obj.Color.G, partInfo.obj.Color.B)
					colorInput.TextColor3 = Color3.new(1,1,1)
					colorInput.Font = Enum.Font.SourceSans
					colorInput.TextScaled = true
					Instance.new("UICorner", colorInput)
					addTextStroke(colorInput, 1)

					local applyColorBtn = Instance.new("TextButton", partFrame)
					applyColorBtn.Size = UDim2.new(0.2, -5, 1, -5)
					applyColorBtn.Position = UDim2.new(0.65, 5, 0, 2.5)
					applyColorBtn.BackgroundColor3 = Color3.fromRGB(40,100,180)
					applyColorBtn.Text = "Apply Color"
					applyColorBtn.TextColor3 = Color3.new(1,1,1)
					applyColorBtn.Font = Enum.Font.SourceSansBold
					applyColorBtn.TextScaled = true
					Instance.new("UICorner", applyColorBtn)
					addTextStroke(applyColorBtn, 1)
					applyColorBtn.MouseButton1Click:Connect(function()
						local nums = {string.match(colorInput.Text, "(%d+%.?%d*)")}
						if #nums >= 3 then
							local r = tonumber(nums[1]) or 0
							local g = tonumber(nums[2]) or 0
							local b = tonumber(nums[3]) or 0
							partInfo.obj.Color = Color3.new(r,g,b)
						end
						flashButton(applyColorBtn)
					end)

					local decalFrame = Instance.new("Frame", partFrame)
					decalFrame.Size = UDim2.new(1, 0, 0, 30)
					decalFrame.Position = UDim2.new(0, 0, 0, 35)
					decalFrame.BackgroundTransparency = 1

					local btnX = 0
					for faceName, faceId in pairs(faces) do
						local btn = Instance.new("TextButton", decalFrame)
						btn.Size = UDim2.new(0, 35, 0, 25)
						btn.Position = UDim2.new(0, btnX, 0, 0)
						btn.BackgroundColor3 = Color3.fromRGB(60,60,120)
						btn.Text = faceName
						btn.TextColor3 = Color3.new(1,1,1)
						btn.Font = Enum.Font.SourceSansBold
						btn.TextScaled = true
						Instance.new("UICorner", btn)
						addTextStroke(btn, 1)
						btn.MouseButton1Click:Connect(function()
							local tex = textureIdBox.Text
							if tex == "" then return end
							tex = formatTextureId(tex)
							addDecal(partInfo.obj, faceId, tex)
							flashButton(btn)
							refreshList()
						end)
						btnX = btnX + 40
					end

					local texAllBtn = Instance.new("TextButton", decalFrame)
					texAllBtn.Size = UDim2.new(0, 35, 0, 25)
					texAllBtn.Position = UDim2.new(0, btnX, 0, 0)
					texAllBtn.BackgroundColor3 = Color3.fromRGB(80,80,180)
					texAllBtn.Text = "Tex"
					texAllBtn.TextColor3 = Color3.new(1,1,1)
					texAllBtn.Font = Enum.Font.SourceSansBold
					texAllBtn.TextScaled = true
					Instance.new("UICorner", texAllBtn)
					addTextStroke(texAllBtn, 1)
					texAllBtn.MouseButton1Click:Connect(function()
						local tex = textureIdBox.Text
						if tex == "" then return end
						tex = formatTextureId(tex)
						for _, f in pairs(faces) do
							addDecal(partInfo.obj, f, tex)
						end
						flashButton(texAllBtn)
						refreshList()
					end)

					local allBtn = Instance.new("TextButton", decalFrame)
					allBtn.Size = UDim2.new(0, 45, 0, 25)
					allBtn.Position = UDim2.new(0, btnX + 40, 0, 0)
					allBtn.BackgroundColor3 = Color3.fromRGB(80,80,180)
					allBtn.Text = "All Faces"
					allBtn.TextColor3 = Color3.new(1,1,1)
					allBtn.Font = Enum.Font.SourceSansBold
					allBtn.TextScaled = true
					Instance.new("UICorner", allBtn)
					addTextStroke(allBtn, 1)
					allBtn.MouseButton1Click:Connect(function()
						local tex = textureIdBox.Text
						if tex == "" then return end
						tex = formatTextureId(tex)
						for _, f in pairs(faces) do
							addDecal(partInfo.obj, f, tex)
						end
						flashButton(allBtn)
						refreshList()
					end)

					local decals = {}
					for _, child in ipairs(partInfo.obj:GetChildren()) do
						if child:IsA("Decal") then
							table.insert(decals, child)
						end
					end
					local decalListFrame = Instance.new("Frame", partFrame)
					decalListFrame.Size = UDim2.new(1, 0, 0, #decals * 25)
					decalListFrame.Position = UDim2.new(0, 0, 0, 70)
					decalListFrame.BackgroundTransparency = 1
					local dy = 0
					for _, d in ipairs(decals) do
						local dRow = Instance.new("Frame", decalListFrame)
						dRow.Size = UDim2.new(1, 0, 0, 20)
						dRow.Position = UDim2.new(0, 0, 0, dy)
						dRow.BackgroundColor3 = Color3.fromRGB(40,30,60)
						Instance.new("UICorner", dRow)

						local faceText = Instance.new("TextLabel", dRow)
						faceText.Size = UDim2.new(0.6, -5, 1, 0)
						faceText.Position = UDim2.new(0, 5, 0, 0)
						faceText.BackgroundTransparency = 1
						faceText.Text = string.format("%s: %s", d.Face.Name, d.Texture)
						faceText.TextColor3 = Color3.new(0.9,0.9,1)
						faceText.Font = Enum.Font.SourceSans
						faceText.TextScaled = true
						faceText.TextXAlignment = Enum.TextXAlignment.Left
						addTextStroke(faceText, 1)

						local removeBtn = Instance.new("TextButton", dRow)
						removeBtn.Size = UDim2.new(0.15, -5, 1, -5)
						removeBtn.Position = UDim2.new(0.85, 5, 0, 2.5)
						removeBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
						removeBtn.Text = "Remove"
						removeBtn.TextColor3 = Color3.new(1,1,1)
						removeBtn.Font = Enum.Font.SourceSansBold
						removeBtn.TextScaled = true
						Instance.new("UICorner", removeBtn)
						addTextStroke(removeBtn, 1)
						removeBtn.MouseButton1Click:Connect(function()
							removeDecal(d)
							flashButton(removeBtn)
							refreshList()
						end)
						dy = dy + 22
					end
					if #decals > 0 then
						decalListFrame.Size = UDim2.new(1, 0, 0, dy)
					else
						decalListFrame:Destroy()
					end

					local finalHeight = 30
					if decalListFrame then
						finalHeight = finalHeight + 35 + decalListFrame.Size.Y.Offset
					else
						finalHeight = finalHeight + 35
					end
					partFrame.Size = UDim2.new(1, -10, 0, finalHeight)
				else
					local textureInput = Instance.new("TextBox", partFrame)
					textureInput.Size = UDim2.new(0.55, -5, 1, -5)
					textureInput.Position = UDim2.new(0.35, 5, 0, 2.5)
					textureInput.BackgroundColor3 = Color3.fromRGB(60,30,90)
					textureInput.PlaceholderText = "Texture ID"
					textureInput.Text = partInfo.texture or ""
					textureInput.TextColor3 = Color3.new(1,1,1)
					textureInput.Font = Enum.Font.SourceSans
					textureInput.TextScaled = true
					Instance.new("UICorner", textureInput)
					addTextStroke(textureInput, 1)

					local applyBtn = Instance.new("TextButton", partFrame)
					applyBtn.Size = UDim2.new(0.25, -5, 1, -5)
					applyBtn.Position = UDim2.new(0.95, -30, 0, 2.5)
					applyBtn.BackgroundColor3 = Color3.fromRGB(40,100,180)
					applyBtn.Text = "Apply"
					applyBtn.TextColor3 = Color3.new(1,1,1)
					applyBtn.Font = Enum.Font.SourceSansBold
					applyBtn.TextScaled = true
					Instance.new("UICorner", applyBtn)
					addTextStroke(applyBtn, 1)
					applyBtn.MouseButton1Click:Connect(function()
						local newTex = textureInput.Text
						if newTex ~= "" then
							newTex = formatTextureId(newTex)
							if partInfo.type == "MeshPart" then
								partInfo.obj.TextureID = newTex
							elseif partInfo.type == "SpecialMesh" then
								partInfo.obj.TextureId = newTex
							elseif partInfo.type == "Shirt" or partInfo.type == "Pants" or partInfo.type == "Accessory" then
								partInfo.obj.TextureId = newTex
							end
						end
						flashButton(applyBtn)
					end)
					partFrame.Size = UDim2.new(1, -10, 0, 30)
				end
				y = y + partFrame.Size.Y.Offset
			end
			npcFrame.Size = UDim2.new(1, 0, 0, y)
			totalHeight = totalHeight + y
		end
		scroll.CanvasSize = UDim2.new(0,0,0, totalHeight + 10)
	end

	refreshBtn.MouseButton1Click:Connect(function()
		flashButton(refreshBtn)
		refreshList()
	end)

	applyTextureAll.MouseButton1Click:Connect(function()
		local tex = textureIdBox.Text
		if tex == "" then return end
		tex = formatTextureId(tex)
		for _, npc in ipairs(folder:GetChildren()) do
			if npc:IsA("Model") then
				local parts = getAllTexturedParts(npc)
				for _, partInfo in ipairs(parts) do
					if partInfo.type ~= "BasePart" then
						if partInfo.type == "MeshPart" then
							partInfo.obj.TextureID = tex
						elseif partInfo.type == "SpecialMesh" then
							partInfo.obj.TextureId = tex
						elseif partInfo.type == "Shirt" or partInfo.type == "Pants" or partInfo.type == "Accessory" then
							partInfo.obj.TextureId = tex
						end
					end
				end
			end
		end
		flashButton(applyTextureAll)
		refreshList()
	end)

	applyColorAll.MouseButton1Click:Connect(function()
		for _, npc in ipairs(folder:GetChildren()) do
			if npc:IsA("Model") then
				local parts = getAllTexturedParts(npc)
				for _, partInfo in ipairs(parts) do
					if partInfo.type == "BasePart" then
						partInfo.obj.Color = selectedColor
					end
				end
			end
		end
		flashButton(applyColorAll)
		refreshList()
	end)

	refreshList()
end

local enemyTexBtn = Instance.new("TextButton", LeftScroll)
enemyTexBtn.Size = UDim2.new(1,0,0,30)
enemyTexBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 80)
enemyTexBtn.Text = "Enemy Texture Editor"
enemyTexBtn.TextColor3 = Color3.new(1,1,1)
enemyTexBtn.Font = Enum.Font.SourceSansBold
enemyTexBtn.TextScaled = true
Instance.new("UICorner", enemyTexBtn)
addTextStroke(enemyTexBtn, 1)
enemyTexBtn.MouseButton1Click:Connect(function()
	flashButton(enemyTexBtn)
	openTextureEditor("EnemyFolder")
end)

local friendlyTexBtn = Instance.new("TextButton", LeftScroll)
friendlyTexBtn.Size = UDim2.new(1,0,0,30)
friendlyTexBtn.BackgroundColor3 = Color3.fromRGB(40, 180, 80)
friendlyTexBtn.Text = "Friendly Texture Editor"
friendlyTexBtn.TextColor3 = Color3.new(1,1,1)
friendlyTexBtn.Font = Enum.Font.SourceSansBold
friendlyTexBtn.TextScaled = true
Instance.new("UICorner", friendlyTexBtn)
addTextStroke(friendlyTexBtn, 1)
friendlyTexBtn.MouseButton1Click:Connect(function()
	flashButton(friendlyTexBtn)
	openTextureEditor("FriendlyFolder")
end)

updateLeftCanvas()

-- Right panel (Y=0, height 250)
local RightPanel = Instance.new("Frame")
RightPanel.Size = UDim2.new(0, 140, 0, 250)
RightPanel.Position = UDim2.new(1, -10, 0, 0)
RightPanel.BackgroundColor3 = Color3.fromRGB(80, 20, 20)
RightPanel.BorderSizePixel = 0
RightPanel.Visible = false
RightPanel.ZIndex = 1
RightPanel.Parent = Main
Instance.new("UICorner", RightPanel)
addFrameStroke(RightPanel, Color3.fromRGB(255, 100, 100), 2)

local RightTitle = Instance.new("TextLabel", RightPanel)
RightTitle.Size = UDim2.new(1,0,0,25)
RightTitle.BackgroundTransparency = 1
RightTitle.Text = "Spawner Settings"
RightTitle.TextColor3 = Color3.new(1,1,1)
RightTitle.Font = Enum.Font.SourceSansBold
RightTitle.TextScaled = true
addTextStroke(RightTitle, 1)

local InputFrame = Instance.new("Frame", RightPanel)
InputFrame.Size = UDim2.new(1,-10,0,130)
InputFrame.Position = UDim2.new(0,5,0,30)
InputFrame.BackgroundTransparency = 1

local TypeBox = Instance.new("TextBox", InputFrame)
TypeBox.Size = UDim2.new(1,0,0,25)
TypeBox.Position = UDim2.new(0,0,0,0)
TypeBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
TypeBox.Text = "A"
TypeBox.PlaceholderText = "Type (A/B)"
TypeBox.TextColor3 = Color3.new(1,1,1)
TypeBox.Font = Enum.Font.SourceSansBold
TypeBox.TextScaled = true
Instance.new("UICorner", TypeBox)
addTextStroke(TypeBox, 1)

local AmountBox = Instance.new("TextBox", InputFrame)
AmountBox.Size = UDim2.new(1,-35,0,25)
AmountBox.Position = UDim2.new(0,0,0,30)
AmountBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
AmountBox.Text = "1"
AmountBox.PlaceholderText = "Amount"
AmountBox.TextColor3 = Color3.new(1,1,1)
AmountBox.Font = Enum.Font.SourceSansBold
AmountBox.TextScaled = true
Instance.new("UICorner", AmountBox)
addTextStroke(AmountBox, 1)

local WarnBtn = Instance.new("TextButton", InputFrame)
WarnBtn.Size = UDim2.new(0,30,0,25)
WarnBtn.Position = UDim2.new(1,-30,0,30)
WarnBtn.BackgroundColor3 = Color3.fromRGB(160,30,30)
WarnBtn.Text = "!"
WarnBtn.TextColor3 = Color3.new(1,1,1)
WarnBtn.Font = Enum.Font.SourceSansBold
WarnBtn.TextScaled = true
Instance.new("UICorner", WarnBtn)
addTextStroke(WarnBtn, 1)
WarnBtn.MouseButton1Click:Connect(function() WarnPopup.Visible = true end)

local TeamBox = Instance.new("TextBox", InputFrame)
TeamBox.Size = UDim2.new(1,0,0,25)
TeamBox.Position = UDim2.new(0,0,0,60)
TeamBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
TeamBox.Text = "Friendly"
TeamBox.PlaceholderText = "Friendly/Enemy"
TeamBox.TextColor3 = Color3.new(1,1,1)
TeamBox.Font = Enum.Font.SourceSansBold
TeamBox.TextScaled = true
Instance.new("UICorner", TeamBox)
addTextStroke(TeamBox, 1)

local WaitBox = Instance.new("TextBox", InputFrame)
WaitBox.Size = UDim2.new(1,0,0,25)
WaitBox.Position = UDim2.new(0,0,0,90)
WaitBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
WaitBox.Text = "0"
WaitBox.PlaceholderText = "Wait Time"
WaitBox.TextColor3 = Color3.new(1,1,1)
WaitBox.Font = Enum.Font.SourceSansBold
WaitBox.TextScaled = true
Instance.new("UICorner", WaitBox)
addTextStroke(WaitBox, 1)

local SpawnAll = Instance.new("TextButton", RightPanel)
SpawnAll.Size = UDim2.new(1,-10,0,25)
SpawnAll.Position = UDim2.new(0,5,1,-30)
SpawnAll.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
SpawnAll.Text = "Spawn All"
SpawnAll.TextColor3 = Color3.new(1,1,1)
SpawnAll.Font = Enum.Font.SourceSansBold
SpawnAll.TextScaled = true
Instance.new("UICorner", SpawnAll)
addTextStroke(SpawnAll, 1)

-- Multi‑zone panel (unchanged)
local MultiPanel = Instance.new("Frame")
MultiPanel.Size = UDim2.new(1, -20, 0, 150)
MultiPanel.Position = UDim2.new(0, 10, 1, -160)
MultiPanel.BackgroundColor3 = Color3.fromRGB(35, 15, 55)
MultiPanel.BorderSizePixel = 0
MultiPanel.Visible = false
MultiPanel.ZIndex = 4
MultiPanel.Parent = Main
Instance.new("UICorner", MultiPanel)
addFrameStroke(MultiPanel, Color3.fromRGB(255, 150, 0), 2)

local MultiTitle = Instance.new("TextLabel", MultiPanel)
MultiTitle.Size = UDim2.new(1, 0, 0, 20)
MultiTitle.BackgroundTransparency = 1
MultiTitle.Text = "MultiZone"
MultiTitle.TextColor3 = Color3.fromRGB(255, 200, 100)
MultiTitle.Font = Enum.Font.SourceSansBold
MultiTitle.TextScaled = true
addTextStroke(MultiTitle, 1)

local MultiScroll = Instance.new("ScrollingFrame", MultiPanel)
MultiScroll.Size = UDim2.new(1, -10, 1, -60)
MultiScroll.Position = UDim2.new(0, 5, 0, 25)
MultiScroll.BackgroundTransparency = 1
MultiScroll.BorderSizePixel = 0
MultiScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
MultiScroll.ScrollBarThickness = 4

local MultiLayout = Instance.new("UIListLayout", MultiScroll)
MultiLayout.Padding = UDim.new(0, 5)
MultiLayout.SortOrder = Enum.SortOrder.LayoutOrder

local BottomFrame = Instance.new("Frame", MultiPanel)
BottomFrame.Size = UDim2.new(1, -10, 0, 30)
BottomFrame.Position = UDim2.new(0, 5, 1, -35)
BottomFrame.BackgroundTransparency = 1

local SpawnMultiBtn = Instance.new("TextButton", BottomFrame)
SpawnMultiBtn.Size = UDim2.new(0.5, -3, 1, 0)
SpawnMultiBtn.Position = UDim2.new(0, 0, 0, 0)
SpawnMultiBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
SpawnMultiBtn.Text = "Spawn!"
SpawnMultiBtn.TextColor3 = Color3.new(1,1,1)
SpawnMultiBtn.Font = Enum.Font.SourceSansBold
SpawnMultiBtn.TextScaled = true
Instance.new("UICorner", SpawnMultiBtn)
addTextStroke(SpawnMultiBtn, 1)

local CancelAllBtn = Instance.new("TextButton", BottomFrame)
CancelAllBtn.Size = UDim2.new(0.5, -3, 1, 0)
CancelAllBtn.Position = UDim2.new(0.5, 3, 0, 0)
CancelAllBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
CancelAllBtn.Text = "Cancel All"
CancelAllBtn.TextColor3 = Color3.new(1,1,1)
CancelAllBtn.Font = Enum.Font.SourceSansBold
CancelAllBtn.TextScaled = true
Instance.new("UICorner", CancelAllBtn)
addTextStroke(CancelAllBtn, 1)

local function refreshMultiList()
	for _, child in ipairs(MultiScroll:GetChildren()) do
		if child:IsA("Frame") then child:Destroy() end
	end
	for idx, entry in ipairs(selectedSpawns) do
		local frame = Instance.new("Frame", MultiScroll)
		frame.Size = UDim2.new(1, 0, 0, 60)
		frame.BackgroundColor3 = Color3.fromRGB(50, 20, 80)
		Instance.new("UICorner", frame)

		local idLabel = Instance.new("TextLabel", frame)
		idLabel.Size = UDim2.new(0.15, 0, 1, 0)
		idLabel.BackgroundTransparency = 1
		idLabel.Text = "ID "..entry.id
		idLabel.TextColor3 = Color3.new(1,1,1)
		idLabel.Font = Enum.Font.SourceSansBold
		idLabel.TextScaled = true
		addTextStroke(idLabel, 1)

		local typeBox = Instance.new("TextBox", frame)
		typeBox.Size = UDim2.new(0.15, -5, 1, -10)
		typeBox.Position = UDim2.new(0.15, 5, 0.5, -15)
		typeBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
		typeBox.Text = entry.type or TypeBox.Text
		typeBox.PlaceholderText = "A/B"
		typeBox.TextColor3 = Color3.new(1,1,1)
		typeBox.Font = Enum.Font.SourceSans
		typeBox.TextScaled = true
		Instance.new("UICorner", typeBox)
		addTextStroke(typeBox, 1)
		typeBox:GetPropertyChangedSignal("Text"):Connect(function()
			entry.type = typeBox.Text
		end)

		local amtBox = Instance.new("TextBox", frame)
		amtBox.Size = UDim2.new(0.2, -5, 1, -10)
		amtBox.Position = UDim2.new(0.3, 5, 0.5, -15)
		amtBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
		amtBox.Text = tostring(entry.amount or 1)
		amtBox.PlaceholderText = "Amt"
		amtBox.TextColor3 = Color3.new(1,1,1)
		amtBox.Font = Enum.Font.SourceSans
		amtBox.TextScaled = true
		Instance.new("UICorner", amtBox)
		addTextStroke(amtBox, 1)
		amtBox:GetPropertyChangedSignal("Text"):Connect(function()
			entry.amount = tonumber(amtBox.Text) or 1
		end)

		local waitBox = Instance.new("TextBox", frame)
		waitBox.Size = UDim2.new(0.2, -5, 1, -10)
		waitBox.Position = UDim2.new(0.5, 5, 0.5, -15)
		waitBox.BackgroundColor3 = Color3.fromRGB(60,30,90)
		waitBox.Text = tostring(entry.wait or 0)
		waitBox.PlaceholderText = "Wait"
		waitBox.TextColor3 = Color3.new(1,1,1)
		waitBox.Font = Enum.Font.SourceSans
		waitBox.TextScaled = true
		Instance.new("UICorner", waitBox)
		addTextStroke(waitBox, 1)
		waitBox:GetPropertyChangedSignal("Text"):Connect(function()
			entry.wait = tonumber(waitBox.Text) or 0
		end)

		local cancelBtn = Instance.new("TextButton", frame)
		cancelBtn.Size = UDim2.new(0.15, -5, 1, -10)
		cancelBtn.Position = UDim2.new(0.7, 5, 0.5, -15)
		cancelBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
		cancelBtn.Text = "X"
		cancelBtn.TextColor3 = Color3.new(1,1,1)
		cancelBtn.Font = Enum.Font.SourceSansBold
		cancelBtn.TextScaled = true
		Instance.new("UICorner", cancelBtn)
		addTextStroke(cancelBtn, 1)
		cancelBtn.MouseButton1Click:Connect(function()
			table.remove(selectedSpawns, idx)
			refreshMultiList()
		end)
	end
	MultiScroll.CanvasSize = UDim2.new(0,0,0, #selectedSpawns * 65)
end

MultiToggle.MouseButton1Click:Connect(function()
	flashButton(MultiToggle)
	multiZoneActive = not multiZoneActive
	if multiZoneActive then
		MultiToggle.BackgroundColor3 = Color3.fromRGB(255, 150, 0)
		MultiPanel.Visible = true
	else
		MultiToggle.BackgroundColor3 = Color3.fromRGB(100, 40, 180)
		MultiPanel.Visible = false
	end
end)

CancelAllBtn.MouseButton1Click:Connect(function()
	flashButton(CancelAllBtn)
	selectedSpawns = {}
	refreshMultiList()
end)

SpawnMultiBtn.MouseButton1Click:Connect(function()
	flashButton(SpawnMultiBtn)
	stopSpawning = false
	for _, entry in ipairs(selectedSpawns) do
		if stopSpawning then break end
		local typ = string.upper(entry.type or TypeBox.Text)
		local team = TeamBox.Text ~= "" and TeamBox.Text or "Friendly"
		task.spawn(function()
			for c = 1, entry.amount do
				if stopSpawning then break end
				if SandboxSpawn then
					pcall(function() SandboxSpawn:InvokeServer(team, entry.id, typ) end)
				end
				if c < entry.amount then waitWithStop(entry.wait) end
			end
		end)
	end
end)

local Scroll = Instance.new("ScrollingFrame", Main)
Scroll.Size = UDim2.new(1, -20, 1, -90)
Scroll.Position = UDim2.new(0, 10, 0, 55)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 4
Scroll.ScrollBarImageColor3 = Color3.fromRGB(150,50,255)

local Layout = Instance.new("UIListLayout", Scroll)
Layout.Padding = UDim.new(0, 5)

local spawnButtons = {}
local currentMax = 94

local function rebuildList(maxID)
	for _,b in ipairs(spawnButtons) do b:Destroy() end
	spawnButtons = {}
	currentMax = maxID
	for i = 1, maxID do
		local btn = Instance.new("TextButton", Scroll)
		btn.Size = UDim2.new(1,-5,0,25)
		btn.BackgroundColor3 = Color3.fromRGB(50,20,80)
		btn.Text = "Spawn "..i
		btn.TextColor3 = Color3.new(1,1,1)
		btn.Font = Enum.Font.SourceSans
		btn.TextScaled = true
		Instance.new("UICorner", btn)
		addTextStroke(btn, 1)
		btn.MouseButton1Click:Connect(function()
			flashButton(btn)
			if multiZoneActive then
				local amt = tonumber(AmountBox.Text) or 1
				local wait = tonumber(WaitBox.Text) or 0
				local typ = TypeBox.Text
				table.insert(selectedSpawns, {id = i, amount = amt, wait = wait, type = typ})
				refreshMultiList()
			else
				stopSpawning = false
				local typ = string.upper(TypeBox.Text)
				local amt = tonumber(AmountBox.Text) or 1
				local team = TeamBox.Text ~= "" and TeamBox.Text or "Friendly"
				local wait = tonumber(WaitBox.Text) or 0
				task.spawn(function()
					for c = 1, amt do
						if stopSpawning then break end
						if SandboxSpawn then
							pcall(function() SandboxSpawn:InvokeServer(team, i, typ) end)
						end
						if c < amt then waitWithStop(wait) end
					end
				end)
			end
		end)
		table.insert(spawnButtons, btn)
	end
	Scroll.CanvasSize = UDim2.new(0,0,0, maxID * 30)
end

rebuildList(94)

TeamBox:GetPropertyChangedSignal("Text"):Connect(function()
	local t = TeamBox.Text:lower()
	if t:find("enemy") then
		rebuildList(350)
	else
		rebuildList(94)
	end
end)

SpawnAll.MouseButton1Click:Connect(function()
	flashButton(SpawnAll)
	stopSpawning = false
	local typ = string.upper(TypeBox.Text)
	local amt = tonumber(AmountBox.Text) or 1
	local team = TeamBox.Text ~= "" and TeamBox.Text or "Friendly"
	local wait = tonumber(WaitBox.Text) or 0
	task.spawn(function()
		for i = 1, currentMax do
			for c = 1, amt do
				if stopSpawning then break end
				if SandboxSpawn then
					pcall(function() SandboxSpawn:InvokeServer(team, i, typ) end)
				end
				if c < amt then waitWithStop(wait) end
			end
			if stopSpawning then break end
		end
	end)
end)

local rightOpen = false
RightArrow.MouseButton1Click:Connect(function()
	flashButton(RightArrow)
	rightOpen = not rightOpen
	if rightOpen then
		RightPanel.Visible = true
		RightPanel.Position = UDim2.new(1, -10, 0, 0)
		TS:Create(RightPanel, TweenInfo.new(0.4), {Position = UDim2.new(1, 10, 0, 0)}):Play()
		RightArrow.Text = "<"
	else
		local t = TS:Create(RightPanel, TweenInfo.new(0.3), {Position = UDim2.new(1, -10, 0, 0)})
		t:Play()
		t.Completed:Connect(function() if not rightOpen then RightPanel.Visible = false end end)
		RightArrow.Text = ">"
	end
end)

local leftOpen = false
LeftArrow.MouseButton1Click:Connect(function()
	flashButton(LeftArrow)
	leftOpen = not leftOpen
	if leftOpen then
		LeftPanel.Visible = true
		LeftPanel.Position = UDim2.new(0, 10, 0, 0)
		TS:Create(LeftPanel, TweenInfo.new(0.4), {Position = UDim2.new(0, -140, 0, 0)}):Play()
		LeftArrow.Text = ">"
	else
		local t = TS:Create(LeftPanel, TweenInfo.new(0.3), {Position = UDim2.new(0, 10, 0, 0)})
		t:Play()
		t.Completed:Connect(function() if not leftOpen then LeftPanel.Visible = false end end)
		LeftArrow.Text = "<"
	end
end)

CloseBtn.MouseButton1Click:Connect(function()
	flashButton(CloseBtn)
	ScreenGui:Destroy()
end)

refreshMultiList()

print("Sandbox Spawner loaded. All UIStrokes are black (text elements). Name tags have black outline.")