-- Services
local Players = game:GetService("Players")
local CollectionService = game:GetService("CollectionService")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Player = Players.LocalPlayer

-- =======================
-- GUI
-- =======================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoFarmChestGUI"
ScreenGui.Parent = Player:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 250, 0, 250)
MainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(40,40,40)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local ToggleButton = Instance.new("TextButton")
local HopButton = Instance.new("TextButton")
local TeamDelayLabel = Instance.new("TextLabel")
local TeamDelayBox = Instance.new("TextBox")
local SwitchDelayLabel = Instance.new("TextLabel")
local SwitchDelayBox = Instance.new("TextBox")
local ChestDelayLabel = Instance.new("TextLabel")
local ChestDelayBox = Instance.new("TextBox")

-- Toggle AutoFarm
ToggleButton.Size = UDim2.new(0.9,0,0,40)
ToggleButton.Position = UDim2.new(0.05,0,0.05,0)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 20
ToggleButton.TextColor3 = Color3.fromRGB(255,255,255)
ToggleButton.Text = "AutoFarm Chest: ON"
ToggleButton.BackgroundColor3 = Color3.fromRGB(0,150,0)
ToggleButton.Parent = MainFrame

-- Toggle AutoHop
HopButton.Size = UDim2.new(0.9,0,0,30)
HopButton.Position = UDim2.new(0.05,0,0.93,0)
HopButton.Font = Enum.Font.SourceSansBold
HopButton.TextSize = 18
HopButton.TextColor3 = Color3.fromRGB(255,255,255)
HopButton.Text = "Auto Hop: ON"
HopButton.BackgroundColor3 = Color3.fromRGB(0,100,200)
HopButton.Parent = MainFrame

-- Labels + TextBox
TeamDelayLabel.Size = UDim2.new(0.9,0,0,20)
TeamDelayLabel.Position = UDim2.new(0.05,0,0.28,0)
TeamDelayLabel.BackgroundTransparency = 1
TeamDelayLabel.Text = "Thời gian đổi phe (giây):"
TeamDelayLabel.TextColor3 = Color3.fromRGB(255,255,255)
TeamDelayLabel.TextSize = 16
TeamDelayLabel.Font = Enum.Font.SourceSans
TeamDelayLabel.Parent = MainFrame

TeamDelayBox.Size = UDim2.new(0.9,0,0,25)
TeamDelayBox.Position = UDim2.new(0.05,0,0.38,0)
TeamDelayBox.BackgroundColor3 = Color3.fromRGB(60,60,60)
TeamDelayBox.TextColor3 = Color3.fromRGB(255,255,255)
TeamDelayBox.Font = Enum.Font.SourceSans
TeamDelayBox.TextSize = 18
TeamDelayBox.Text = "5"
TeamDelayBox.Parent = MainFrame

SwitchDelayLabel.Size = UDim2.new(0.9,0,0,20)
SwitchDelayLabel.Position = UDim2.new(0.05,0,0.52)
SwitchDelayLabel.BackgroundTransparency = 1
SwitchDelayLabel.Text = "Delay sau khi đổi phe (giây):"
SwitchDelayLabel.TextColor3 = Color3.fromRGB(255,255,255)
SwitchDelayLabel.TextSize = 16
SwitchDelayLabel.Font = Enum.Font.SourceSans
SwitchDelayLabel.Parent = MainFrame

SwitchDelayBox.Size = UDim2.new(0.9,0,0,25)
SwitchDelayBox.Position = UDim2.new(0.05,0,0.62,0)
SwitchDelayBox.BackgroundColor3 = Color3.fromRGB(60,60,60)
SwitchDelayBox.TextColor3 = Color3.fromRGB(255,255,255)
SwitchDelayBox.Font = Enum.Font.SourceSans
SwitchDelayBox.TextSize = 18
SwitchDelayBox.Text = "1"
SwitchDelayBox.Parent = MainFrame

ChestDelayLabel.Size = UDim2.new(0.9,0,0,20)
ChestDelayLabel.Position = UDim2.new(0.05,0,0.74,0)
ChestDelayLabel.BackgroundTransparency = 1
ChestDelayLabel.Text = "Delay Chest (giây):"
ChestDelayLabel.TextColor3 = Color3.fromRGB(255,255,255)
ChestDelayLabel.TextSize = 16
ChestDelayLabel.Font = Enum.Font.SourceSans
ChestDelayLabel.Parent = MainFrame

ChestDelayBox.Size = UDim2.new(0.9,0,0,25)
ChestDelayBox.Position = UDim2.new(0.05,0,0.84,0)
ChestDelayBox.BackgroundColor3 = Color3.fromRGB(60,60,60)
ChestDelayBox.TextColor3 = Color3.fromRGB(255,255,255)
ChestDelayBox.Font = Enum.Font.SourceSans
ChestDelayBox.TextSize = 18
ChestDelayBox.Text = "0"
ChestDelayBox.Parent = MainFrame

-- =======================
-- Biến toàn cục
-- =======================
getgenv().AutoFarmChest = true
getgenv().AutoHop = true
local PauseFarm = false
local SwitchMode = true -- true = đổi phe, false = reset
local CurrentTeam = "Pirates"
local HasJoinedMarines = false

-- =======================
-- Auto join Marines 1 lần
-- =======================
local function JoinMarines()
    if HasJoinedMarines then return end
    pcall(function()
        if ReplicatedStorage.Remotes.CommF_ then
            ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", "Marines")
        end
    end)
    HasJoinedMarines = true
end

task.spawn(function()
    JoinMarines()
end)

-- =======================
-- ESP Chest
-- =======================
local ChestESP = true

local function CreateChestESP(Chest)
    if Chest:FindFirstChild("ChestEspAttachment") then return end
    local attachment = Instance.new("Attachment")
    attachment.Name = "ChestEspAttachment"
    attachment.Parent = Chest
    attachment.Position = Vector3.new(0,3,0)
    local nameEsp = Instance.new("BillboardGui")
    nameEsp.Name = "NameEsp"
    nameEsp.Size = UDim2.new(0,200,0,30)
    nameEsp.Adornee = attachment
    nameEsp.ExtentsOffset = Vector3.new(0,1,0)
    nameEsp.AlwaysOnTop = true
    nameEsp.Parent = attachment
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Font = Enum.Font.Code
    nameLabel.TextSize = 14
    nameLabel.TextWrapped = true
    nameLabel.Size = UDim2.new(1,0,1,0)
    nameLabel.TextYAlignment = Enum.TextYAlignment.Top
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.TextColor3 = Color3.fromRGB(80,245,245)
    nameLabel.Parent = nameEsp
end

local function UpdateChestESP()
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local playerPos = Character:GetPivot().Position
    local Chests = CollectionService:GetTagged("_ChestTagged")
    for _, Chest in ipairs(Chests) do
        if not SelectedIsland or Chest:IsDescendantOf(SelectedIsland) then
            if not Chest:GetAttribute("IsDisabled") then
                CreateChestESP(Chest)
                local espAttachment = Chest:FindFirstChild("ChestEspAttachment")
                if espAttachment and espAttachment:FindFirstChild("NameEsp") then
                    local distanceMagnitude = (Chest:GetPivot().Position - playerPos).Magnitude
                    local displayDistance = math.floor(distanceMagnitude / 3)
                    local chestName = Chest.Name:gsub("Label","")
                    espAttachment.NameEsp.TextLabel.Text = string.format("[%s] %d M", chestName, displayDistance)
                end
            end
        end
    end
end

RunService.RenderStepped:Connect(function()
    if ChestESP then
        pcall(UpdateChestESP)
    end
end)

-- =======================
-- Fly tới chest
-- =======================
local function FlyToChest(targetCFrame)
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local HRP = Character:FindFirstChild("HumanoidRootPart")
    local Humanoid = Character:FindFirstChild("Humanoid")
    if not HRP or not Humanoid then return end
    Humanoid.PlatformStand = true

    local targetPos = targetCFrame.Position

    while (HRP.Position - targetPos).Magnitude > 1 and getgenv().AutoFarmChest and not PauseFarm do
        HRP.CFrame = CFrame.new(Vector3.new(targetPos.X,targetPos.Y,targetPos.Z))
        RunService.RenderStepped:Wait()
    end

    Humanoid.PlatformStand = false
end

-- =======================
-- Lấy chest gần nhất
-- =======================
local function GetNearestChest()
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local Position = Character:GetPivot().Position
    local Chests = CollectionService:GetTagged("_ChestTagged")
    local Distance, Nearest = math.huge, nil
    for _, Chest in ipairs(Chests) do
        if not SelectedIsland or Chest:IsDescendantOf(SelectedIsland) then
            if not Chest:GetAttribute("IsDisabled") then
                local Magnitude = (Chest:GetPivot().Position - Position).Magnitude
                if Magnitude < Distance then
                    Distance = Magnitude
                    Nearest = Chest
                end
            end
        end
    end
    return Nearest
end

-- =======================
-- Auto Hop Server
-- =======================
local function HopServer()
    local PlaceID = game.PlaceId
    local success, data = pcall(function()
        return game:HttpGet("https://games.roblox.com/v1/games/"..PlaceID.."/servers/Public?sortOrder=Asc&limit=100")
    end)
    if not success then return end
    local servers = HttpService:JSONDecode(data)
    for _,v in pairs(servers.data) do
        if v.playing < v.maxPlayers and v.id ~= game.JobId then
            TeleportService:TeleportToPlaceInstance(PlaceID, v.id, Player)
            break
        end
    end
end

-- =======================
-- AutoFarm Chest loop
-- =======================
task.spawn(function()
    while task.wait(tonumber(ChestDelayBox.Text) or 0) do
        if getgenv().AutoFarmChest and not PauseFarm then
            local chest = GetNearestChest()
            if chest then
                FlyToChest(chest:GetPivot())
            else
                if getgenv().AutoHop then
                    HopServer()
                    task.wait(5)
                end
            end
        end
    end
end)

-- =======================
-- Switch team + reset luân phiên
-- =======================
local function SwitchTeam(team)
    pcall(function()
        ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", team)
    end)
    PauseFarm = true
    task.delay(tonumber(SwitchDelayBox.Text) or 1, function()
        PauseFarm = false
    end)
end

local function ResetChar()
	local humanoid = Player.Character and Player.Character:FindFirstChild("Humanoid")
	if humanoid then humanoid.Health = 0 end
	Player.CharacterAdded:Wait()
	task.wait(1)
end

task.spawn(function()
	while true do
		if not getgenv().AutoFarmChest then
			task.wait(0.2)
			continue
		end

		local delayTime = tonumber(TeamDelayBox.Text) or 5
		local t = 0
		while t < delayTime and getgenv().AutoFarmChest do
			task.wait(0.1)
			t += 0.1
		end
		if not getgenv().AutoFarmChest then continue end

		if SwitchMode then
			if CurrentTeam == "Pirates" then
				SwitchTeam("Marines")
				CurrentTeam = "Marines"
			else
				SwitchTeam("Pirates")
				CurrentTeam = "Pirates"
			end
		else
			ResetChar()
		end

		SwitchMode = not SwitchMode
	end
end)

-- =======================
-- GUI Events
-- =======================
ToggleButton.MouseButton1Click:Connect(function()
	getgenv().AutoFarmChest = not getgenv().AutoFarmChest
	if getgenv().AutoFarmChest then
		ToggleButton.Text = "AutoFarm Chest: ON"
		ToggleButton.BackgroundColor3 = Color3.fromRGB(0,150,0)
	else
		ToggleButton.Text = "AutoFarm Chest: OFF"
		ToggleButton.BackgroundColor3 = Color3.fromRGB(150,0,0)
	end
end)

HopButton.MouseButton1Click:Connect(function()
	getgenv().AutoHop = not getgenv().AutoHop
	if getgenv().AutoHop then
		HopButton.Text = "Auto Hop: ON"
		HopButton.BackgroundColor3 = Color3.fromRGB(0,100,200)
	else
		HopButton.Text = "Auto Hop: OFF"
		HopButton.BackgroundColor3 = Color3.fromRGB(100,100,100)
	end
end)

TeamDelayBox.FocusLost:Connect(function()
	local val = tonumber(TeamDelayBox.Text)
	if val and val > 0 then
		TeamDelayBox.Text = tostring(val)
	else
		TeamDelayBox.Text = "5"
	end
end)

SwitchDelayBox.FocusLost:Connect(function()
	local val = tonumber(SwitchDelayBox.Text)
	if val and val >= 0 then
		SwitchDelayBox.Text = tostring(val)
	else
		SwitchDelayBox.Text = "1"
	end
end)

ChestDelayBox.FocusLost:Connect(function()
	local val = tonumber(ChestDelayBox.Text)
	if val and val >= 0 then
		ChestDelayBox.Text = tostring(val)
	else
		ChestDelayBox.Text = "0"
	end
end)
