local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("ReplicatedStorage")
local TS = game:GetService("TweenService")

local RemoteSandbox = RS:WaitForChild("Events"):WaitForChild("RemoteFunction"):WaitForChild("SandboxSpawn")
local GiveUpRemote = RS:WaitForChild("Events"):WaitForChild("RemoteEvents"):WaitForChild("GiveUp")
local PlayerSpawnRemote = RS:WaitForChild("Events"):WaitForChild("RemoteFunction"):WaitForChild("PlayerSpawn")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PurpleSandboxSystem"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

-- MAIN FRAME
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 460, 0, 340)
Main.Position = UDim2.new(0.5, -230, 0.5, -170)
Main.BackgroundColor3 = Color3.fromRGB(25, 10, 40)
Main.BorderSizePixel = 0
Main.Active = true
Main.ClipsDescendants = false
Main.Parent = ScreenGui

Instance.new("UICorner", Main)
local MainStroke = Instance.new("UIStroke", Main)
MainStroke.Color = Color3.fromRGB(150, 50, 255)
MainStroke.Thickness = 2

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(0.5, 0, 0, 30)
Title.Position = UDim2.new(0, 12, 0, 8)
Title.BackgroundTransparency = 1
Title.Text = "Sandbox Gui Spawner"
Title.TextColor3 = Color3.fromRGB(200, 150, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 16
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Main

local SearchBox = Instance.new("TextBox")
SearchBox.Size = UDim2.new(0.38, 0, 0, 28)
SearchBox.Position = UDim2.new(0.52, 0, 0, 8)
SearchBox.BackgroundColor3 = Color3.fromRGB(45, 20, 75)
SearchBox.PlaceholderText = "🔎 Search units..."
SearchBox.Text = ""
SearchBox.TextColor3 = Color3.new(1, 1, 1)
SearchBox.Font = Enum.Font.SourceSans
SearchBox.TextSize = 14
SearchBox.ClearTextOnFocus = false
SearchBox.Parent = Main
Instance.new("UICorner", SearchBox)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -32, 0, 8)
CloseBtn.BackgroundColor3 = Color3.fromRGB(90, 10, 10)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextSize = 16
CloseBtn.Parent = Main
Instance.new("UICorner", CloseBtn)

local RightArrowBtn = Instance.new("TextButton")
RightArrowBtn.Size = UDim2.new(0, 26, 0, 26)
RightArrowBtn.Position = UDim2.new(1, -64, 0, 8)
RightArrowBtn.BackgroundColor3 = Color3.fromRGB(100, 40, 180)
RightArrowBtn.Text = ">"
RightArrowBtn.TextColor3 = Color3.new(1, 1, 1)
RightArrowBtn.Font = Enum.Font.SourceSansBold
RightArrowBtn.TextSize = 16
RightArrowBtn.Parent = Main
Instance.new("UICorner", RightArrowBtn)

local LeftArrowBtn = Instance.new("TextButton")
LeftArrowBtn.Size = UDim2.new(0, 26, 0, 26)
LeftArrowBtn.Position = UDim2.new(0, 8, 0, 8)
LeftArrowBtn.BackgroundColor3 = Color3.fromRGB(40, 80, 180)
LeftArrowBtn.Text = "<"
LeftArrowBtn.TextColor3 = Color3.new(1, 1, 1)
LeftArrowBtn.Font = Enum.Font.SourceSansBold
LeftArrowBtn.TextSize = 16
LeftArrowBtn.Parent = Main
Instance.new("UICorner", LeftArrowBtn)

-- WARNING
local WarnPopup = Instance.new("Frame")
WarnPopup.Size = UDim2.new(0, 280, 0, 160)
WarnPopup.Position = UDim2.new(0.5, -140, 0.5, -80)
WarnPopup.BackgroundColor3 = Color3.fromRGB(30, 10, 50)
WarnPopup.Visible = true
WarnPopup.ZIndex = 20
WarnPopup.Parent = ScreenGui

Instance.new("UICorner", WarnPopup)
local WarnStroke = Instance.new("UIStroke", WarnPopup)
WarnStroke.Color = Color3.fromRGB(180, 50, 255)
WarnStroke.Thickness = 2

local WarnText = Instance.new("TextLabel")
WarnText.Size = UDim2.new(1, -20, 1, -55)
WarnText.Position = UDim2.new(0, 10, 0, 10)
WarnText.BackgroundTransparency = 1
WarnText.Text = "Warning! If you spawn more, your game will crash or lag.\nDon't blame me!"
WarnText.TextColor3 = Color3.fromRGB(255, 90, 90)
WarnText.TextWrapped = true
WarnText.Font = Enum.Font.SourceSansBold
WarnText.TextSize = 16
WarnText.ZIndex = 21
WarnText.Parent = WarnPopup

local WarnClose = Instance.new("TextButton")
WarnClose.Size = UDim2.new(0, 110, 0, 32)
WarnClose.Position = UDim2.new(0.5, -55, 1, -40)
WarnClose.BackgroundColor3 = Color3.fromRGB(130, 30, 190)
WarnClose.Text = "I Understand"
WarnClose.TextColor3 = Color3.new(1, 1, 1)
WarnClose.Font = Enum.Font.SourceSansBold
WarnClose.TextSize = 14
WarnClose.ZIndex = 21
WarnClose.Parent = WarnPopup
Instance.new("UICorner", WarnClose)

WarnClose.MouseButton1Click:Connect(function()
	WarnPopup.Visible = false
end)

-- RIGHT PANEL
local SpawnerPanel = Instance.new("Frame")
SpawnerPanel.Size = UDim2.new(0, 155, 0, 235)
SpawnerPanel.Position = UDim2.new(1, -155, 0, 45)
SpawnerPanel.BackgroundColor3 = Color3.fromRGB(35, 15, 55)
SpawnerPanel.Visible = false
SpawnerPanel.Parent = Main

Instance.new("UICorner", SpawnerPanel)
local SpawnerStroke = Instance.new("UIStroke", SpawnerPanel)
SpawnerStroke.Color = Color3.fromRGB(170, 80, 255)
SpawnerStroke.Thickness = 2

local SpawnerTitle = Instance.new("TextLabel")
SpawnerTitle.Size = UDim2.new(1, 0, 0, 28)
SpawnerTitle.BackgroundTransparency = 1
SpawnerTitle.Text = "Spawn Settings"
SpawnerTitle.TextColor3 = Color3.new(1, 1, 1)
SpawnerTitle.Font = Enum.Font.SourceSansBold
SpawnerTitle.TextSize = 15
SpawnerTitle.Parent = SpawnerPanel

local TypeInput = Instance.new("TextBox")
TypeInput.Size = UDim2.new(1, -20, 0, 30)
TypeInput.Position = UDim2.new(0, 10, 0, 35)
TypeInput.BackgroundColor3 = Color3.fromRGB(55, 25, 85)
TypeInput.Text = "A"
TypeInput.PlaceholderText = "Type (A/B)"
TypeInput.TextColor3 = Color3.new(1, 1, 1)
TypeInput.Font = Enum.Font.SourceSans
TypeInput.Parent = SpawnerPanel
Instance.new("UICorner", TypeInput)

local AmountInput = Instance.new("TextBox")
AmountInput.Size = UDim2.new(1, -55, 0, 30)
AmountInput.Position = UDim2.new(0, 10, 0, 70)
AmountInput.BackgroundColor3 = Color3.fromRGB(55, 25, 85)
AmountInput.Text = "1"
AmountInput.PlaceholderText = "Times"
AmountInput.TextColor3 = Color3.new(1, 1, 1)
AmountInput.Font = Enum.Font.SourceSans
AmountInput.Parent = SpawnerPanel
Instance.new("UICorner", AmountInput)

local AmountWarnBtn = Instance.new("TextButton")
AmountWarnBtn.Size = UDim2.new(0, 35, 0, 30)
AmountWarnBtn.Position = UDim2.new(1, -45, 0, 70)
AmountWarnBtn.BackgroundColor3 = Color3.fromRGB(170, 30, 30)
AmountWarnBtn.Text = "!"
AmountWarnBtn.TextColor3 = Color3.new(1, 1, 1)
AmountWarnBtn.TextSize = 18
AmountWarnBtn.Font = Enum.Font.SourceSansBold
AmountWarnBtn.Parent = SpawnerPanel
Instance.new("UICorner", AmountWarnBtn)

AmountWarnBtn.MouseButton1Click:Connect(function()
	WarnText.Text = "Warning! If you spawn more, your game will crash or lag.\nDon't blame me!"
	WarnPopup.Visible = true
end)

local TeamInput = Instance.new("TextBox")
TeamInput.Size = UDim2.new(1, -20, 0, 30)
TeamInput.Position = UDim2.new(0, 10, 0, 105)
TeamInput.BackgroundColor3 = Color3.fromRGB(55, 25, 85)
TeamInput.Text = "Friendly"
TeamInput.PlaceholderText = "Friendly / Enemy"
TeamInput.TextColor3 = Color3.new(1, 1, 1)
TeamInput.Font = Enum.Font.SourceSans
TeamInput.Parent = SpawnerPanel
Instance.new("UICorner", TeamInput)

local WaitInput = Instance.new("TextBox")
WaitInput.Size = UDim2.new(1, -20, 0, 30)
WaitInput.Position = UDim2.new(0, 10, 0, 140)
WaitInput.BackgroundColor3 = Color3.fromRGB(55, 25, 85)
WaitInput.Text = "0"
WaitInput.PlaceholderText = "Delay (sec)"
WaitInput.TextColor3 = Color3.new(1, 1, 1)
WaitInput.Font = Enum.Font.SourceSans
WaitInput.Parent = SpawnerPanel
Instance.new("UICorner", WaitInput)

local SpawnAllBtn = Instance.new("TextButton")
SpawnAllBtn.Size = UDim2.new(1, -20, 0, 38)
SpawnAllBtn.Position = UDim2.new(0, 10, 0, 185)
SpawnAllBtn.BackgroundColor3 = Color3.fromRGB(160, 20, 80)
SpawnAllBtn.Text = "SPAWN ALL UNITS"
SpawnAllBtn.TextColor3 = Color3.new(1, 1, 1)
SpawnAllBtn.Font = Enum.Font.SourceSansBold
SpawnAllBtn.TextSize = 14
SpawnAllBtn.Parent = SpawnerPanel
Instance.new("UICorner", SpawnAllBtn)

-- LEFT PANEL
local OthersPanel = Instance.new("Frame")
OthersPanel.Size = UDim2.new(0, 160, 0, 320)
OthersPanel.Position = UDim2.new(0, -320, 0, 45)
OthersPanel.BackgroundColor3 = Color3.fromRGB(15, 30, 65)
OthersPanel.Visible = false
OthersPanel.Parent = Main

Instance.new("UICorner", OthersPanel)
local OthersStroke = Instance.new("UIStroke", OthersPanel)
OthersStroke.Color = Color3.fromRGB(80, 170, 255)
OthersStroke.Thickness = 2

local OthersTitle = Instance.new("TextLabel")
OthersTitle.Size = UDim2.new(1, 0, 0, 26)
OthersTitle.BackgroundTransparency = 1
OthersTitle.Text = "Unit Settings"
OthersTitle.TextColor3 = Color3.new(1, 1, 1)
OthersTitle.Font = Enum.Font.SourceSansBold
OthersTitle.TextSize = 15
OthersTitle.Parent = OthersPanel

local MultiLabel = Instance.new("TextLabel")
MultiLabel.Size = UDim2.new(1, -16, 0, 18)
MultiLabel.Position = UDim2.new(0, 8, 0, 28)
MultiLabel.BackgroundTransparency = 1
MultiLabel.Text = "multi 100"
MultiLabel.TextColor3 = Color3.fromRGB(150, 200, 255)
MultiLabel.Font = Enum.Font.SourceSansBold
MultiLabel.TextSize = 13
MultiLabel.TextXAlignment = Enum.TextXAlignment.Left
MultiLabel.Parent = OthersPanel

local NumberInput = Instance.new("TextBox")
NumberInput.Size = UDim2.new(1, -16, 0, 26)
NumberInput.Position = UDim2.new(0, 8, 0, 46)
NumberInput.BackgroundColor3 = Color3.fromRGB(20, 60, 120)
NumberInput.Text = "100"
NumberInput.PlaceholderText = "multi 100"
NumberInput.TextColor3 = Color3.new(1, 1, 1)
NumberInput.Font = Enum.Font.SourceSans
NumberInput.Parent = OthersPanel
Instance.new("UICorner", NumberInput)

local function createStat(name, default, y)
	local box = Instance.new("TextBox")
	box.Size = UDim2.new(1, -16, 0, 24)
	box.Position = UDim2.new(0, 8, 0, y)
	box.BackgroundColor3 = Color3.fromRGB(20, 60, 120)
	box.Text = tostring(default)
	box.PlaceholderText = name
	box.TextColor3 = Color3.new(1, 1, 1)
	box.Font = Enum.Font.SourceSans
	box.TextSize = 13
	box.Parent = OthersPanel
	Instance.new("UICorner", box)
	return box
end

local DamageInput     = createStat("Damage", 600, 78)
local HealthInput     = createStat("Health", 6, 106)
local RangeInput      = createStat("Range", 5, 134)
local ArmorInput      = createStat("Armor", 1, 162)
local ResistanceInput = createStat("Resistance", 1, 190)
local WalkSpeedInput  = createStat("WalkSpeed", 10, 218)
local AttackRateInput = createStat("AttackRate", 0, 246)

local GiveUpBtn = Instance.new("TextButton")
GiveUpBtn.Size = UDim2.new(1, -16, 0, 28)
GiveUpBtn.Position = UDim2.new(0, 8, 0, 278)
GiveUpBtn.BackgroundColor3 = Color3.fromRGB(140, 20, 20)
GiveUpBtn.Text = "Give up"
GiveUpBtn.TextColor3 = Color3.new(1, 1, 1)
GiveUpBtn.Font = Enum.Font.SourceSansBold
GiveUpBtn.TextSize = 14
GiveUpBtn.Parent = OthersPanel
Instance.new("UICorner", GiveUpBtn)

GiveUpBtn.MouseButton1Click:Connect(function()
	GiveUpRemote:FireServer()
end)

local ShootBtn = Instance.new("TextButton")
ShootBtn.Size = UDim2.new(1, -16, 0, 26)
ShootBtn.Position = UDim2.new(0, 8, 0, 310)
ShootBtn.BackgroundColor3 = Color3.fromRGB(20, 100, 200)
ShootBtn.Text = "Shoot Cannon"
ShootBtn.TextColor3 = Color3.new(1, 1, 1)
ShootBtn.Font = Enum.Font.SourceSansBold
ShootBtn.TextSize = 13
ShootBtn.Parent = OthersPanel
Instance.new("UICorner", ShootBtn)

ShootBtn.MouseButton1Click:Connect(function()
	PlayerSpawnRemote:InvokeServer("Cannon")
end)

local function getStatsTable()
	return {
		Damage = tonumber(DamageInput.Text) or 600,
		Health = tonumber(HealthInput.Text) or 6,
		Range = tonumber(RangeInput.Text) or 5,
		Armor = tonumber(ArmorInput.Text) or 1,
		Resistance = tonumber(ResistanceInput.Text) or 1,
		WalkSpeed = tonumber(WalkSpeedInput.Text) or 10,
		AttackRate = tonumber(AttackRateInput.Text) or 0
	}
end

-- SCROLL
local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1, -22, 1, -58)
Scroll.Position = UDim2.new(0, 11, 0, 48)
Scroll.BackgroundTransparency = 1
Scroll.ScrollBarThickness = 4
Scroll.ScrollBarImageColor3 = Color3.fromRGB(160, 60, 255)
Scroll.Parent = Main

local Layout = Instance.new("UIListLayout", Scroll)
Layout.Padding = UDim.new(0, 6)
Layout.SortOrder = Enum.SortOrder.LayoutOrder

-- FRIENDLY UNITS (previous list)
local friendlyUnits = {
	{1, "Battler"}, {2, "Trowel Battler"}, {3, "Sword Battler"}, {4, "Slinger Battler"},
	{5, "Baller Battler"}, {6, "Rocket Battler"}, {7, "Bomb Battler"}, {8, "Titan Battler"},
	{9, "Paintball Battler"}, {10, "Speed Battler"}, {11, "Regen Battler"}, {12, "Gravity Battler"},
	{13, "Grunt"}, {14, "Grenade Battler"}, {15, "Hammer Battler"}, {16, "Lil' Doombringer"},
	{17, "Lil' Turking"}, {18, "Lil' EXEC"}, {19, "Lil' Cesus"}, {20, "Lil' Lichen"},
	{21, "Lil' Stem"}, {22, "Lil' Root"}, {23, "Tumored Battler"}, {24, "Tumored Trowel"},
	{25, "Tumored Sword"}, {26, "Tumored Slinger"}, {27, "Tumored Baller"}, {28, "Sailor Battler"},
	{29, "Bugle Battler"}, {30, "Spartan Battler"}, {31, "Birthday Battler"}, {32, "Butcher Battler"},
	{33, "Jester Battler"}, {34, "Cloak Battler"}, {35, "Protest Battler"}, {36, "Mage Battler"},
	{37, "Stagger Battler"}, {38, "Teapot Battler"}, {39, "Bean Battler"}, {40, "Duck Battler"},
	{41, "Spambox Battler"}, {42, "Papaler"}, {43, "Golden Hammer"}, {44, "RAIG"},
	{45, "FRIEND"}, {46, "Gruntkeeper"}, {47, "Ninja Battler"}, {48, "Healer Battler"},
	{49, "Slateskin Battler"}, {50, "Cola Battler"}, {51, "Brew Battler"}, {52, "Goala Battler"},
	{53, "Hobo Battler"}, {54, "Boxer Battler"}, {55, "Flashlight Battler"}, {56, "Archer Battler"},
	{57, "Fencer Battler"}, {58, "Miner Battler"}, {59, "Trainee Battler"}, {60, "Spiked Battler"},
	{61, "Batter Battler"}, {62, "Sleepy Battler"}, {63, "Cowboy Battler"}, {64, "Volleyball Battler"},
	{65, "Officer Battler"}, {66, "Gamble Battler"}, {67, "Prankster Battler"}, {68, "Megaphone Battler"},
	{69, "Coil Battler"}, {70, "Expo Battler"}, {71, "Fireball Battler"}, {72, "Iceball Battler"},
	{73, "Hurtorb Battler"}, {74, "Motorbike Battler"}, {75, "Retro Battler"}, {76, "Farmer Battler"},
	{77, "Glass Battler"}, {78, "Winged Battler"}, {79, "Remote Battler"}, {80, "Werewolf Battler"},
	{81, "Moai Battler"}, {82, "Prototype Battler"}, {83, "Parasol Battler"}, {84, "Hockey Battler"},
	{85, "Glizzy Battler"}, {86, "Steampunk Battler"}, {87, "Chairman"}, {88, "Tix Collector"},
	{89, "Wes"}, {90, "Balloon"}, {91, "Mu"}, {92, "Conjuror"}, {93, "Dark Reaper"},
	{94, "Intern Elf"}, {95, "Gift"}, {96, "Psi"}, {97, "Operator"}, {98, "Iota"},
	{99, "Bluesteel"}, {100, "Gamma"}, {101, "Blaze"},
	{102, "Blonde Battler"}, {103, "Builder Battler"}, {104, "Brigand Battler"}, {105, "Stunner Battler"},
	{106, "Soccer Battler"}, {107, "Crocket Battler"}, {108, "Kamikaze Battler"}, {109, "Telamon Battler"},
	{110, "Sniper Battler"}, {111, "Dual Speed Battler"}, {112, "Dual Regen Battler"}, {113, "Dual Gravity Battler"},
	{114, "Mafia Grunt"}, {115, "Arson Battler"}, {116, "Banhammer Battler"}, {117, "Lil' Deathbringer"},
	{118, "Lil' Infernus"}, {119, "Lil' X-TREME"}, {120, "Lil' Cross"}, {121, "Lil' Chronos"},
	{122, "Lil' Mortis"}, {123, "Tarnished Battler"}, {124, "Tarnished Trowel"}, {125, "Tarnished Sword"},
	{126, "Tarnished Slinger"}, {127, "Tarnished Baller"}, {128, "Pirate Battler"}, {129, "Drummer Battler"},
	{130, "Barbaric Battler"}, {131, "Celebration Battler"}, {132, "Chainsaw Battler"}, {133, "Terror Battler"},
	{134, "Cyber Battler"}, {135, "Outcry Battler"}, {136, "Dryad Battler"}, {137, "Slasher Battler"},
	{138, "Newell Battler"}, {139, "Burrito Battler"}, {140, "Epikduck Battler"}, {141, "Sorcus Battler"},
	{142, "Ol' Papaler"}, {143, "Golden Banhammer"}, {144, "RAIG COLOSSAL"}, {145, "OMEGA FRIEND"},
	{146, "Gruntlord"}, {147, "Assassin Battler"}, {148, "Medic Battler"}, {149, "Ceramic Battler"},
	{150, "Binge Cola Battler"}, {151, "Binge Brew Battler"}, {152, "Binge Goala Battler"}, {153, "Hoarder Battler"},
	{154, "Shanker Battler"}, {155, "Lantern Battler"}, {156, "Crossbow Battler"}, {157, "Scherma Battler"},
	{158, "Prospect Battler"}, {159, "Drafted Battler"}, {160, "Barbed Battler"}, {161, "Striker Battler"},
	{162, "Nighty Battler"}, {163, "Gunslinger Battler"}, {164, "Basketball Battler"}, {165, "Sheriff Battler"},
	{166, "Luckroll Battler"}, {167, "Tomfoolery Battler"}, {168, "Micspammer Battler"}, {169, "Rail Battler"},
	{170, "Vulcan Battler"}, {171, "Ignis Battler"}, {172, "Frigus Battler"}, {173, "Healorb Battler"},
	{174, "Hyperbike Battler"}, {175, "Atari Battler"}, {176, "Rancher Battler"}, {177, "Payload Battler"},
	{178, "Jetpack Battler"}, {179, "Occult Battler"}, {180, "Beastly Battler"}, {181, "Tiki Battler"},
	{182, "Archetype Battler"}, {183, "Umbrella Battler"}, {184, "Jason Battler"}, {185, "Gymbro Battler"},
	{186, "Telepunk Battler"}, {187, "Raigquitter"}, {188, "B. Money"}, {189, "Maxwell"},
	{190, "Shade"}, {191, "Crimson"}, {192, "Trebuchet"}, {193, "Lord Skelly"}, {194, "Bonehead"},
	{195, "Bonefire"}, {196, "Mall Santa"}, {197, "Present"}, {198, "Noir"}, {199, "Commander"},
	{200, "Tank"}, {201, "Ivory"}, {202, "Goldaurum"}, {203, "Goldue"}, {204, "Chartreuse"},
	{205, "Micro"}
}

-- ENEMY UNITS (new list you just gave)
local enemyUnits = {
	{1, "Battler"}, {2, "Trowel Battler"}, {3, "Sword Battler"}, {4, "Slinger Battler"},
	{5, "Baller Battler"}, {6, "Paintball Battler"}, {7, "Paintmaster"}, {8, "Speed Battler"},
	{9, "Regen Battler"}, {10, "Gravity Battler"}, {11, "Rocket Battler"}, {12, "Bomb Battler"},
	{13, "Jetrock"}, {14, "Hammer Battler"}, {15, "Grenade Battler"}, {16, "Titan Battler"},
	{17, "Doombringer"}, {18, "Firebrand"}, {19, "Arch"}, {20, "Gallant"},
	{21, "Sailor Battler"}, {22, "Bucket Tagger"}, {23, "Cannonary"}, {24, "Captain Red"},
	{25, "Bugle Battler"}, {26, "Guitar Trainee"}, {27, "Cymbal Boy"}, {28, "Roadbumper"},
	{29, "DJ Radical"}, {30, "Radmobile"}, {31, "Goon"}, {32, "Spartan Battler"},
	{33, "Gargant"}, {34, "Turking"}, {35, "Telamon SuperFan"}, {36, "Darkheart"},
	{37, "Outlaw"}, {38, "Penny"}, {39, "Patrol"}, {40, "Butcher Battler"},
	{41, "Amputator"}, {42, "Mad Scientist"}, {43, "Doctor Despair"}, {44, "Jester Battler"},
	{45, "Blinder"}, {46, "Monkey Bread"}, {47, "Monkey"}, {48, "Littleman"},
	{49, "Pianist"}, {50, "Insecure"}, {51, "Deleter"}, {52, "Cloak Battler"},
	{53, "Sapper"}, {54, "Fatboy"}, {55, "EXEC"}, {56, "Windforce"},
	{57, "Flight"}, {58, "Knight"}, {59, "Airhead"}, {60, "Mage Battler"},
	{61, "Viking"}, {62, "Henchman"}, {63, "Monsalt"}, {64, "Rook"},
	{65, "Bishop"}, {66, "King"}, {67, "Stagger Battler"}, {68, "Ire"},
	{69, "Grappler"}, {70, "Singer"}, {71, "Judgement"}, {72, "McBurn"},
	{73, "Martyr"}, {74, "Killbot"}, {75, "Beacon"}, {76, "Teapot Battler"},
	{77, "Father"}, {78, "Cesus"}, {79, "Zombie"}, {80, "Venomshank"},
	{81, "Chuck"}, {82, "Cheez"}, {83, "Bean Battler"}, {84, "Ghostwalker"},
	{85, "Sign Buddy"}, {86, "Hoboe"}, {87, "Pizza Pal"}, {88, "Chef Sarge"},
	{89, "Bouncer"}, {90, "Klown"}, {91, "Duck Battler"}, {92, "Swinburne"},
	{93, "Drakocage"}, {94, "Drako"}, {95, "FunK1d"}, {96, "BulkK1d"},
	{97, "Fraud"}, {98, "Dopamine"}, {99, "Spambox Battler"}, {100, "Flower"},
	{101, "Oscar"}, {102, "Lichen"}, {103, "Stem"}, {104, "Root"},
	{105, "Battler (Classic)"}, {106, "Shield Battler"}, {107, "Sword Battler (Classic)"}, {108, "Rusher Battler"},
	{109, "Bomber Battler"}, {110, "Speedster"}, {111, "Regenerator"}, {112, "Gravitator"},
	{113, "Fusecoil"}, {114, "FireGrind"}, {115, "SideToxic"}, {116, "EuroFrost"},
	{117, "Cake Battler"}, {118, "Baker Battler"}, {119, "Mystery Battler"}, {120, "Birthday Battler"},
	{121, "Icedagger Battler"}, {122, "Snowtrooper Battler"}, {123, "Freeze Battler"}, {124, "Breaker Battler"},
	{125, "Gale Battler"}, {126, "Icecaster Battler"}, {127, "Frostburn Battler"}, {128, "Stalagmight"},
	{129, "Bedridden"}, {130, "Muffin"}, {131, "Sprinter Muffin"}, {132, "Recruit Muffin"},
	{133, "Armored Muffin"}, {134, "Cupcake"}, {135, "Bazooka Muffin"}, {136, "Specialist Muffin"},
	{137, "Cultist Muffin"}, {138, "Tank Muffin"}, {139, "Catalog Muffin"}, {140, "Brand"},
	{141, "Birdshot"}, {142, "Artifact"}, {143, "Boxtrot"}, {144, "Princess"},
	{145, "Burnout"}, {146, "Boffin"}, {147, "Clifford"}, {148, "Razor"},
	{149, "Shader"}, {150, "Mobster"}, {151, "CRIME"}, {152, "Slenderhead"},
	{153, "Jeff"}, {154, "Gamblecore"}, {155, "Zipbomb"}, {156, "Warrior"},
	{157, "Hunter"}, {158, "Damned"}, {159, "Blood Crystal"}, {160, "Caster"},
	{161, "Carcass"}, {162, "Admirer"}, {163, "Apostle"}, {164, "Mystery II Battler"},
	{165, "McDonald Battler"}, {166, "American Fatboy"}, {167, "Cyborg Rocket Battle"}, {168, "Grimace Battler"},
	{169, "Grimace"}, {170, "Patriot Warhead"}, {171, "Cyborg Crocket Battle"}, {172, "Enigma II Battler"},

	-- Alternative version (B)
	{173, "Blonde Battler"}, {174, "Builder Battler"}, {175, "Brigand Battler"}, {176, "Stunner Battler"},
	{177, "Soccer Battler"}, {178, "Sniper Battler"}, {179, "Terminator"}, {180, "Dual Speed Battler"},
	{181, "Dual Regen Battler"}, {182, "Dual Gravity Battler"}, {183, "Crocket Battler"}, {184, "Kamikaze Battler"},
	{185, "Hellfighter"}, {186, "Hell Jet"}, {187, "Banhammer Battler"}, {188, "Arson Battler"},
	{189, "Telamon Battler"}, {190, "Deathbringer"}, {191, "Dual Firebrand"}, {192, "Nemesis"},
	{193, "Shieldman"}, {194, "Pirate Battler"}, {195, "Treasure Seller"}, {196, "Mercenary"},
	{197, "Scarlet Storm"}, {198, "Drummer Battler"}, {199, "Guitar Hero"}, {200, "Saxophone Man"},
	{201, "Speeddemon"}, {202, "Rockstar Radical"}, {203, "Bombcopter"}, {204, "Supergoon"},
	{205, "Punkslayer"}, {206, "Techninja"}, {207, "Barbaric Battler"}, {208, "Gehenna"},
	{209, "Infernus"}, {210, "Dual Darkheart"}, {211, "Vigilante"}, {212, "Betty"},
	{213, "Principal"}, {214, "Chainsaw Battler"}, {215, "Necromancer"}, {216, "Skull"},
	{217, "Crazy Chemist"}, {218, "Mister Mancer"}, {219, "Skeleton"}, {220, "Terror Battler"},
	{221, "Confuddler"}, {222, "Gorilla Barrel"}, {223, "Gorilla"}, {224, "Bloodhorn"},
	{225, "Grandmaster"}, {226, "Blackmail"}, {227, "Melvin"}, {228, "Expunger"},
	{229, "Cyber Battler"}, {230, "Stunlock"}, {231, "Warhead"}, {232, "X-TREME"},
	{233, "Dual Windforce"}, {234, "Jolly"}, {235, "Pwnsalot"}, {236, "Skittle"},
	{237, "Dryad Battler"}, {238, "Norman"}, {239, "Trollage"}, {240, "Monolith"},
	{241, "Pawn"}, {242, "Slasher Battler"}, {243, "Fury"}, {244, "Crusader"},
	{245, "Vocalist"}, {246, "Punishment"}, {247, "McBlast"}, {248, "Reinbo"},
	{249, "Atomic"}, {250, "Ultimate"}, {251, "Newell Battler"}, {252, "Crusher"},
	{253, "Chronos"}, {254, "Cross"}, {255, "Teabot"}, {256, "Dual Venomshank"},
	{257, "Cartil"}, {258, "Chardy"}, {259, "Burrito Battler"}, {260, "Dual Ghostwalker"},
	{261, "Fish Buddy"}, {262, "Fishe"}, {263, "Delivery Pal"}, {264, "Sargent"},
	{265, "Scrap Sarge"}, {266, "Cyclist"}, {267, "Kreep"}, {268, "Epikduck Battler"},
	{269, "Algernon"}, {270, "Monfined"}, {271, "Monsuer"}, {272, "GameK1d"},
	{273, "RetroK1d"}, {274, "Graft"}, {275, "Dendritic"}, {276, "Sorcus Battler"},
	{277, "Depriver"}, {278, "Edmond"}, {279, "Mortis"}, {280, "Server"},
	{281, "Byte"}, {282, "Dark Battler"}, {283, "Dark Shield Battler"}, {284, "Dark Sword Battler"},
	{285, "Dark Rusher Battler"}, {286, "Dark Bomber Battler"}, {287, "Ragester"}, {288, "Magerator"},
	{289, "Spiritator"}, {290, "Soulfuse"}, {291, "Ultimate Coil"}, {292, "Ultigamation"},
	{293, "Soul of Speed"}, {294, "Soul of Regen"}, {295, "Soul of Gravity"}, {296, "Puke Battler"},
	{297, "Fudge Battler"}, {298, "Enigma Battler"}, {299, "Celebration Battler"}, {300, "Giant Noob"},
	{301, "Dual Icedagger Battler"}, {302, "Icegunner Battler"}, {303, "Icicle Battler"}, {304, "Breacher Battler"},
	{305, "Mistral Battler"}, {306, "Fowlcaster Battler"}, {307, "Ice Fowl"}, {308, "Okami Battler"},
	{309, "Stalagcythe"}, {310, "Treachstone"}, {311, "Fat Muffin"}, {312, "Exploder Muffin"},
	{313, "Soldier Muffin"}, {314, "Shielded Muffin"}, {315, "Cake"}, {316, "Slice"},
	{317, "Robotic Muffin"}, {318, "Anarchist Muffin"}, {319, "Illuminist Muffin"}, {320, "Prism Tank Muffin"},
	{321, "Doomsday Muffin"}, {322, "Scorch"}, {323, "Bigshot"}, {324, "Braverush"},
	{325, "Shankcut"}, {326, "Sweetheart"}, {327, "Bottled"}, {328, "Brainiac"},
	{329, "Sunbeam"}, {330, "Edgelord"}, {331, "Sparkle"}, {332, "Kingpin"},
	{333, "PRIME"}, {334, "Screecherhead"}, {335, "Omor"}, {336, "Ransomware"},
	{337, "Redzone"}, {338, "Champion"}, {339, "Arbalest"}, {340, "Tormented"},
	{341, "Blight Crystal"}, {342, "Bloodcaster"}, {343, "Cadaver"}, {344, "Acolyte"},
	{345, "Outcast"}, {346, "Blight"}, {347, "Enigma II Battler"}, {348, "Grimace Battler"},
	{349, "Grimace"}, {350, "Patriot Warhead"}, {351, "Cyborg Crocket Battle"}
}

local currentUnits = friendlyUnits
local buttons = {}

local function clearButtons()
	for _, btn in ipairs(buttons) do
		btn:Destroy()
	end
	buttons = {}
end

local function createButtons(unitList)
	clearButtons()
	for _, data in ipairs(unitList) do
		local id, name = data[1], data[2]
		local Btn = Instance.new("TextButton")
		Btn.Size = UDim2.new(1, -8, 0, 38)
		Btn.BackgroundColor3 = Color3.fromRGB(48, 18, 75)
		Btn.Text = id .. ": " .. name
		Btn.TextColor3 = Color3.new(1, 1, 1)
		Btn.Font = Enum.Font.SourceSans
		Btn.TextSize = 13
		Btn.TextXAlignment = Enum.TextXAlignment.Left
		Btn.TextTruncate = Enum.TextTruncate.AtEnd
		Btn.Parent = Scroll
		Instance.new("UICorner", Btn)
		
		table.insert(buttons, Btn)
		
		Btn.MouseButton1Click:Connect(function()
			local sType = string.upper(TypeInput.Text or "A")
			local times = math.clamp(tonumber(AmountInput.Text) or 1, 1, 50)
			local teamStr = TeamInput.Text \~= "" and TeamInput.Text or "Friendly"
			local waitTime = tonumber(WaitInput.Text) or 0
			local multi100 = tonumber(NumberInput.Text) or 100
			local stats = getStatsTable()
			
			task.spawn(function()
				for _ = 1, times do
					RemoteSandbox:InvokeServer(teamStr, id, sType, multi100, stats)
					if waitTime > 0 then task.wait(waitTime) end
				end
			end)
		end)
	end
	Scroll.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 10)
end

-- Initial load
createButtons(friendlyUnits)

-- Switch list when Team changes
TeamInput:GetPropertyChangedSignal("Text"):Connect(function()
	local team = string.lower(TeamInput.Text or "")
	if team == "enemy" then
		currentUnits = enemyUnits
		createButtons(enemyUnits)
	else
		currentUnits = friendlyUnits
		createButtons(friendlyUnits)
	end
end)

SearchBox:GetPropertyChangedSignal("Text"):Connect(function()
	local query = string.lower(SearchBox.Text)
	for _, btn in ipairs(buttons) do
		btn.Visible = (query == "") or string.find(string.lower(btn.Text), query, 1, true) \~= nil
	end
	Scroll.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 10)
end)

local function spawnAllUnits()
	local sType = string.upper(TypeInput.Text or "A")
	local times = math.clamp(tonumber(AmountInput.Text) or 1, 1, 20)
	local teamStr = TeamInput.Text \~= "" and TeamInput.Text or "Friendly"
	local waitTime = tonumber(WaitInput.Text) or 0.05
	local multi100 = tonumber(NumberInput.Text) or 100
	local stats = getStatsTable()
	
	task.spawn(function()
		for _, data in ipairs(currentUnits) do
			local id = data[1]
			for _ = 1, times do
				RemoteSandbox:InvokeServer(teamStr, id, sType, multi100, stats)
				task.wait(waitTime)
			end
			task.wait(0.08)
		end
	end)
end

SpawnAllBtn.MouseButton1Click:Connect(function()
	WarnText.Text = "Warning. You will crash or lag if you put too much.\n\nBe extremely careful with SPAWN ALL."
	WarnPopup.Visible = true
	
	local tempConn
	tempConn = WarnClose.MouseButton1Click:Connect(function()
		WarnPopup.Visible = false
		tempConn:Disconnect()
		WarnText.Text = "Warning! If you spawn more, your game will crash or lag.\nDon't blame me!"
		spawnAllUnits()
	end)
end)

-- PANEL ANIMATIONS
local rightOpen = false
RightArrowBtn.MouseButton1Click:Connect(function()
	rightOpen = not rightOpen
	if rightOpen then
		SpawnerPanel.Visible = true
		TS:Create(SpawnerPanel, TweenInfo.new(0.35, Enum.EasingStyle.Quart), {Position = UDim2.new(1, 12, 0, 45)}):Play()
		RightArrowBtn.Text = "<"
	else
		local t = TS:Create(SpawnerPanel, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {Position = UDim2.new(1, -155, 0, 45)})
		t:Play()
		t.Completed:Connect(function() if not rightOpen then SpawnerPanel.Visible = false end end)
		RightArrowBtn.Text = ">"
	end
end)

local leftOpen = false
LeftArrowBtn.MouseButton1Click:Connect(function()
	leftOpen = not leftOpen
	if leftOpen then
		OthersPanel.Visible = true
		TS:Create(OthersPanel, TweenInfo.new(0.35, Enum.EasingStyle.Quart), {Position = UDim2.new(0, -170, 0, 45)}):Play()
		LeftArrowBtn.Text = ">"
	else
		local t = TS:Create(OthersPanel, TweenInfo.new(0.3, Enum.EasingStyle.Quart), {Position = UDim2.new(0, -320, 0, 45)})
		t:Play()
		t.Completed:Connect(function() if not leftOpen then OthersPanel.Visible = false end end)
		LeftArrowBtn.Text = "<"
	end
end)

CloseBtn.MouseButton1Click:Connect(function()
	ScreenGui:Destroy()
end)

-- Dragging
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = Main.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)

Main.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UIS.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

print("✅ Sandbox Gui Spawner (Friendly + Enemy lists) loaded")