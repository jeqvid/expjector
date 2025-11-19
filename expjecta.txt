-- === expjector 3008 2.80 – BHOP + WALL HOP (SPACE ONLY) + SPEED METER + WHITE LINE VELOCITY GRAPH ===
-- Insert = Menu | TP Players to Me in RAGE
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

-- === CONFIG ===
local ESP_MAX_DIST = 400
local ESP_UPDATE_RATE = 0.1
local SPIN_SPEED = 18
local FORWARD_ACCEL = 3.0
local MAX_BHOP_SPEED = 25
local FLY_SPEED = 80
-- THIRD PERSON CAMERA SETTINGS
local THIRD_PERSON_DISTANCE = 16
local THIRD_PERSON_HEIGHT = 0
local CAMERA_SENSITIVITY = 0.35
-- WALL HOP SETTINGS
local WALL_HOP_POWER = 45
local WALL_HOP_COOLDOWN = 0.15
local WALL_HOP_FORWARD_BOOST = 20
local WALL_CHECK_DISTANCE = 3
-- VELOCITY GRAPH SETTINGS
local GRAPH_WIDTH = 220
local GRAPH_HEIGHT = 60
local GRAPH_MAX_VEL = 60
local GRAPH_HISTORY = 60

-- === VARIABLES ===
local bhopEnabled = false
local speedMeterEnabled = false
local velocityGraphEnabled = false
local spinbotEnabled = false
local flyEnabled = false
local noclipEnabled = false
local thirdPersonEnabled = false
local fullBrightEnabled = false
local showCoordsEnabled = false
local wallHopEnabled = false
local spaceHeld = false
local spinning = false
local trackedObjects = {}
local lastEspUpdate = 0
local rootPart = nil
local character = nil
local humanoid = nil
local camera = Workspace.CurrentCamera
-- Fly & noclip
local bodyVelocity = nil
local flyActive = false
local noclipConnection = nil
-- Third Person
local thirdPersonConn = nil
local defaultCamType = nil
local defaultMouseBehavior = nil
local defaultFieldOfView = nil
-- 360° Free Camera
local camYaw = 0
local camPitch = 0
local mouseDeltaConn = nil
-- ESP Toggles
local espEmployees = false
local espPlayers = false
local espItems = false
-- Sliders
local flySpeedValue = FLY_SPEED
local flySpeedLabel = nil
local bhopSpeedValue = MAX_BHOP_SPEED
local bhopSpeedLabel = nil
local wallHopPowerValue = WALL_HOP_POWER
local wallHopPowerLabel = nil
-- Cooldowns
local lastWallHop = 0
-- VELOCITY GRAPH DATA
local velocityHistory = {}

-- === COORDINATES DISPLAY ===
local coordsGui = Instance.new("ScreenGui")
coordsGui.Name = "CoordsDisplay"
coordsGui.ResetOnSpawn = false
coordsGui.Parent = CoreGui
local coordsLabel = Instance.new("TextLabel")
coordsLabel.Size = UDim2.new(0, 220, 0, 40)
coordsLabel.Position = UDim2.new(0, 10, 1, -50)
coordsLabel.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
coordsLabel.BorderSizePixel = 0
coordsLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
coordsLabel.TextStrokeTransparency = 0.7
coordsLabel.TextStrokeColor3 = Color3.new(0,0,0)
coordsLabel.Font = Enum.Font.GothamBold
coordsLabel.TextScaled = true
coordsLabel.Text = "X: 0 | Y: 0 | Z: 0"
coordsLabel.Visible = false
coordsLabel.Parent = coordsGui
local coordsCorner = Instance.new("UICorner")
coordsCorner.CornerRadius = UDim.new(0, 8)
coordsCorner.Parent = coordsLabel

-- === FULL BRIGHT FUNCTION ===
local function applyFullBright(enabled)
    if enabled then
        Lighting.Brightness = 3
        Lighting.ClockTime = 12
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
        Lighting.GlobalShadows = false
        Lighting.ShadowSoftness = 0
        for _, v in pairs(Lighting:GetChildren()) do
            if v:IsA("PostEffect") then v.Enabled = false end
        end
    else
        Lighting.Brightness = 1
        Lighting.ClockTime = 14
        Lighting.FogEnd = 100000
        Lighting.FogStart = 0
        Lighting.GlobalShadows = true
        Lighting.ShadowSoftness = 0.2
    end
end

-- === CLEANUP FUNCTION ===
local function clearAllESP()
    for part, _ in pairs(trackedObjects) do
        removeESP(part)
    end
end

-- === GUI SETUP ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "expjector3008"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 400, 0, 650)
MainFrame.Position = UDim2.new(0.05, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = false
MainFrame.Parent = ScreenGui
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "expjector 3008"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextSize = 22
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Center
Title.Parent = MainFrame
spawn(function()
    local h = 0
    while Title.Parent do
        h = (h + 0.005) % 1
        Title.TextColor3 = Color3.fromHSV(h, 1, 1)
        task.wait()
    end
end)

-- Tabs
local Tabs = Instance.new("Frame")
Tabs.Size = UDim2.new(1, 0, 0, 40)
Tabs.Position = UDim2.new(0, 0, 0, 40)
Tabs.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
Tabs.Parent = MainFrame
local MainTab = Instance.new("TextButton")
MainTab.Size = UDim2.new(0.33, 0, 1, 0)
MainTab.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
MainTab.Text = "MAIN"
MainTab.TextColor3 = Color3.new(1,1,1)
MainTab.TextSize = 16
MainTab.Font = Enum.Font.GothamBold
MainTab.Parent = Tabs
local VisualTab = Instance.new("TextButton")
VisualTab.Size = UDim2.new(0.33, 0, 1, 0)
VisualTab.Position = UDim2.new(0.33, 0, 0, 0)
VisualTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
VisualTab.Text = "VISUAL"
VisualTab.TextColor3 = Color3.new(1,1,1)
VisualTab.TextSize = 16
VisualTab.Font = Enum.Font.GothamBold
VisualTab.Parent = Tabs
local RageTab = Instance.new("TextButton")
RageTab.Size = UDim2.new(0.34, 0, 1, 0)
RageTab.Position = UDim2.new(0.66, 0, 0, 0)
RageTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
RageTab.Text = "RAGE"
RageTab.TextColor3 = Color3.new(1,1,1)
RageTab.TextSize = 16
RageTab.Font = Enum.Font.GothamBold
RageTab.Parent = Tabs

-- Content Frames
local MainContent = Instance.new("Frame")
MainContent.Size = UDim2.new(1, -20, 1, -100)
MainContent.Position = UDim2.new(0, 10, 0, 85)
MainContent.BackgroundTransparency = 1
MainContent.Visible = true
MainContent.Parent = MainFrame
local VisualContent = Instance.new("Frame")
VisualContent.Size = UDim2.new(1, -20, 1, -100)
VisualContent.Position = UDim2.new(0, 10, 0, 85)
VisualContent.BackgroundTransparency = 1
VisualContent.Visible = false
VisualContent.Parent = MainFrame
local RageContent = Instance.new("Frame")
RageContent.Size = UDim2.new(1, -20, 1, -100)
RageContent.Position = UDim2.new(0, 10, 0, 85)
RageContent.BackgroundTransparency = 1
RageContent.Visible = false
RageContent.Parent = MainFrame

-- Close Button
local Close = Instance.new("TextButton")
Close.Size = UDim2.new(0, 30, 0, 30)
Close.Position = UDim2.new(1, -35, 0, 5)
Close.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
Close.Text = "X"
Close.TextColor3 = Color3.new(1,1,1)
Close.TextSize = 18
Close.Font = Enum.Font.GothamBold
Close.Parent = MainFrame
local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = Close

-- Toggle Creator
local toggleY = 0
local function createToggle(parent, text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.Position = UDim2.new(0, 0, 0, toggleY)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    btn.Text = ""
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -50, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text .. ": OFF"
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextColor3 = Color3.new(1,1,1)
    label.Parent = btn
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        label.Text = text .. ": " .. (state and "ON" or "OFF")
        btn.BackgroundColor3 = state and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(45, 45, 55)
        callback(state)
    end)
    toggleY = toggleY + 42
end

-- Slider Creator
local function createSlider(parent, text, min, max, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 50)
    frame.Position = UDim2.new(0, 0, 0, toggleY)
    frame.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    frame.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = frame
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -10, 0, 25)
    label.Position = UDim2.new(0, 5, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. default
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextColor3 = Color3.new(1,1,1)
    label.Parent = frame
    local slider = Instance.new("TextButton")
    slider.Size = UDim2.new(1, -20, 0, 20)
    slider.Position = UDim2.new(0, 10, 0, 25)
    slider.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    slider.Text = ""
    slider.Parent = frame
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    fill.Parent = slider
    local dragging = false
    slider.MouseButton1Down:Connect(function() dragging = true end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local ratio = math.clamp((i.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X, 0, 1)
            local value = math.floor(min + ratio * (max - min))
            fill.Size = UDim2.new(ratio, 0, 1, 0)
            label.Text = text .. ": " .. value
            callback(value)
        end
    end)
    toggleY = toggleY + 54
    return label
end

-- === MAIN TAB ===
toggleY = 0
createToggle(MainContent, "BHOP", function(v) bhopEnabled = v end)
bhopSpeedLabel = createSlider(MainContent, "Bhop Speed Limit", 20, 100, MAX_BHOP_SPEED, function(v) bhopSpeedValue = v end)
createToggle(MainContent, "Wall Hop", function(v) wallHopEnabled = v end)
wallHopPowerLabel = createSlider(MainContent, "Wall Hop Power", 30, 80, WALL_HOP_POWER, function(v) wallHopPowerValue = v end)
createToggle(MainContent, "Speed Meter", function(v) speedMeterEnabled = v end)
createToggle(MainContent, "Velocity Graph", function(v) velocityGraphEnabled = v end)
createToggle(MainContent, "Third Person", function(v)
    thirdPersonEnabled = v
    if v and character and rootPart then
        if not defaultCamType then
            defaultCamType = camera.CameraType
            defaultMouseBehavior = UserInputService.MouseBehavior
            defaultFieldOfView = camera.FieldOfView
        end
        camera.CameraType = Enum.CameraType.Scriptable
        UserInputService.MouseBehavior = Enum.MouseBehavior.Default
        UserInputService.MouseIconEnabled = true
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then part.LocalTransparencyModifier = 0
            elseif part:IsA("Decal") then part.Transparency = 0 end
        end
        camYaw = 0; camPitch = 0
        if thirdPersonConn then thirdPersonConn:Disconnect() end
        if mouseDeltaConn then mouseDeltaConn:Disconnect() end
        mouseDeltaConn = UserInputService.InputChanged:Connect(function(i)
            if not thirdPersonEnabled or i.UserInputType ~= Enum.UserInputType.MouseMovement then return end
            local d = i.Delta
            camYaw = camYaw - d.X * CAMERA_SENSITIVITY
            camPitch = math.clamp(camPitch - d.Y * CAMERA_SENSITIVITY, -80, 80)
        end)
        thirdPersonConn = RunService.RenderStepped:Connect(function()
            if not thirdPersonEnabled or not character or not rootPart then return end
            local head = character:FindFirstChild("Head")
            local focus = head and head.Position or (character:FindFirstChild("UpperTorso") or rootPart).Position + Vector3.new(0,1.5,0)
            local cf = CFrame.new(focus)
                * CFrame.Angles(0, math.rad(camYaw), 0)
                * CFrame.Angles(math.rad(camPitch), 0, 0)
                * CFrame.new(0, THIRD_PERSON_HEIGHT, THIRD_PERSON_DISTANCE)
            camera.CFrame = camera.CFrame:lerp(cf, 0.25)
            camera.FieldOfView = 70
        end)
    else
        if thirdPersonConn then thirdPersonConn:Disconnect(); thirdPersonConn = nil end
        if mouseDeltaConn then mouseDeltaConn:Disconnect(); mouseDeltaConn = nil end
        camera.CameraType = defaultCamType or Enum.CameraType.Custom
        UserInputService.MouseBehavior = defaultMouseBehavior or Enum.MouseBehavior.LockCenter
        camera.FieldOfView = defaultFieldOfView or 70
    end
end)
createToggle(MainContent, "Full Bright", function(v) fullBrightEnabled = v; applyFullBright(v) end)
createToggle(MainContent, "Show Coordinates", function(v) showCoordsEnabled = v; coordsLabel.Visible = v end)

-- === VISUAL TAB ===
toggleY = 0
createToggle(VisualContent, "Employee ESP", function(v) espEmployees = v end)
createToggle(VisualContent, "Player ESP", function(v) espPlayers = v end)
createToggle(VisualContent, "Item ESP", function(v) espItems = v end)

-- === RAGE TAB ===
toggleY = 0
createToggle(RageContent, "Spinbot", function(v)
    spinbotEnabled = v; spinning = v
    if humanoid then humanoid.AutoRotate = not v end
end)
createToggle(RageContent, "Fly", function(v)
    flyEnabled = v; flyActive = v
    if character and rootPart and humanoid then
        if v then
            if bodyVelocity then bodyVelocity:Destroy() end
            bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new()
            bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
            bodyVelocity.P = 1000
            bodyVelocity.Parent = rootPart
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown,false)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,false)
            humanoid:ChangeState(Enum.HumanoidStateType.Flying)
        else
            if bodyVelocity then bodyVelocity:Destroy(); bodyVelocity = nil end
            humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown,true)
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,true)
            humanoid:ChangeState(Enum.HumanoidStateType.Landed)
        end
    end
end)
createToggle(RageContent, "Noclip", function(v)
    noclipEnabled = v
    if v and character then
        if noclipConnection then noclipConnection:Disconnect() end
        noclipConnection = RunService.Stepped:Connect(function()
            if not noclipEnabled or not character then return end
            for _, p in ipairs(character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end)
    else
        if noclipConnection then noclipConnection:Disconnect(); noclipConnection = nil end
        if character then
            for _, p in ipairs(character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = true end
            end
        end
    end
end)

-- === TP TO RANDOM PLAYER BUTTON ===
local function createButton(parent, text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.Position = UDim2.new(0, 0, 0, toggleY)
    btn.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    btn.MouseButton1Click:Connect(callback)
    toggleY = toggleY + 42
end
createButton(RageContent, "TP to Random Player", function()
    if not rootPart then return end
    local alive = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChild("Humanoid")
            if hrp and hum and hum.Health > 0 then table.insert(alive, hrp) end
        end
    end
    if #alive > 0 then
        local target = alive[math.random(1, #alive)]
        rootPart.CFrame = target.CFrame + Vector3.new(0, 3, 0)
        print("Teleported to: " .. target.Parent.Name)
    else
        print("No alive players found!")
    end
end)
flySpeedLabel = createSlider(RageContent, "Fly Speed", 10, 200, FLY_SPEED, function(v) flySpeedValue = v end)

-- === TAB SWITCHING ===
MainTab.MouseButton1Click:Connect(function()
    MainContent.Visible = true; VisualContent.Visible = false; RageContent.Visible = false
    MainTab.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
    VisualTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    RageTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)
VisualTab.MouseButton1Click:Connect(function()
    MainContent.Visible = false; VisualContent.Visible = true; RageContent.Visible = false
    MainTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    VisualTab.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    RageTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)
RageTab.MouseButton1Click:Connect(function()
    MainContent.Visible = false; VisualContent.Visible = false; RageContent.Visible = true
    MainTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    VisualTab.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    RageTab.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
end)
Close.MouseButton1Click:Connect(function() MainFrame.Visible = false end)
UserInputService.InputBegan:Connect(function(i,gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.Insert then MainFrame.Visible = not MainFrame.Visible end
end)

-- === SPEED METER + VELOCITY GRAPH GUI ===
local speedGui = Instance.new("ScreenGui")
speedGui.Name = "SpeedDisplay"
speedGui.ResetOnSpawn = false
speedGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- SPEED METER (centered, bottom)
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 200, 0, 60)
speedLabel.Position = UDim2.new(0.5, -100, 1, -140)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(0, 255, 100)
speedLabel.TextStrokeTransparency = 0.5
speedLabel.TextStrokeColor3 = Color3.new(0,0,0)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextScaled = true
speedLabel.Text = "0.0"
speedLabel.Visible = false
speedLabel.Parent = speedGui

-- VELOCITY GRAPH (centered, a little higher than before, NO BACKGROUND)
local graphFrame = Instance.new("Frame")
graphFrame.Size = UDim2.new(0, GRAPH_WIDTH, 0, GRAPH_HEIGHT)
graphFrame.Position = UDim2.new(0.5, -GRAPH_WIDTH/2, 1, -200)  -- HIGHER: -200 (was -190)
graphFrame.BackgroundTransparency = 1  -- NO BACKGROUND
graphFrame.Visible = false
graphFrame.Parent = speedGui

local graphLine = Instance.new("Frame")
graphLine.Name = "Line"
graphLine.Size = UDim2.new(1, 0, 1, 0)
graphLine.BackgroundTransparency = 1
graphLine.Parent = graphFrame

-- === UPDATE VELOCITY GRAPH (WHITE LINE) ===
local function updateVelocityLine()
    if not rootPart then return end
    local speed = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z).Magnitude
    table.insert(velocityHistory, speed)
    if #velocityHistory > GRAPH_HISTORY then
        table.remove(velocityHistory, 1)
    end

    -- Clear old line
    for _, child in ipairs(graphLine:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    local points = {}
    local w = graphLine.AbsoluteSize.X
    local h = graphLine.AbsoluteSize.Y
    local step = w / (GRAPH_HISTORY - 1)

    for i = 1, #velocityHistory do
        local x = (i - 1) * step
        local y = h - (velocityHistory[i] / GRAPH_MAX_VEL) * h
        table.insert(points, Vector2.new(x, y))
    end

    -- Draw line segments (WHITE)
    for i = 1, #points - 1 do
        local p1 = points[i]
        local p2 = points[i + 1]
        local line = Instance.new("Frame")
        line.BorderSizePixel = 0
        line.BackgroundColor3 = Color3.fromRGB(255, 255, 255)  -- WHITE LINE
        local dx = p2.X - p1.X
        local dy = p2.Y - p1.Y
        local dist = math.sqrt(dx*dx + dy*dy)
        line.Size = UDim2.new(0, dist, 0, 2)
        line.Position = UDim2.new(0, p1.X, 0, p1.Y)
        line.Rotation = math.deg(math.atan2(dy, dx))
        line.Parent = graphLine
    end
end

-- === CHARACTER UPDATE ===
local function updateCharacter()
    character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart", 5)
    if thirdPersonEnabled then
        task.wait(0.1)
        for _, p in ipairs(character:GetDescendants()) do
            if p:IsA("BasePart") then p.LocalTransparencyModifier = 0 end
        end
    end
end
LocalPlayer.CharacterAdded:Connect(updateCharacter)
if LocalPlayer.Character then updateCharacter() end

-- === ESP PLACEHOLDER ===
local function createESP(part, color, txt) end
function removeESP(part) end

-- === WALL HOP LOGIC ===
local function performWallHop()
    if not wallHopEnabled or not spaceHeld or not rootPart or not humanoid then return end
    if tick() - lastWallHop < WALL_HOP_COOLDOWN then return end
    if humanoid:GetState() == Enum.HumanoidStateType.Dead then return end
    local rayOrigin = rootPart.Position
    local forwardDir = rootPart.CFrame.LookVector
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = {character}
    local result = workspace:Raycast(rayOrigin, forwardDir * WALL_CHECK_DISTANCE, params)
    if result and result.Instance and result.Instance.CanCollide then
        local normal = result.Normal
        local hopVelocity = normal * wallHopPowerValue + forwardDir * WALL_HOP_FORWARD_BOOST + Vector3.new(0, 25, 0)
        rootPart.Velocity = hopVelocity
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        lastWallHop = tick()
    end
end

-- === INPUT HANDLING ===
UserInputService.InputBegan:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.Space then
        spaceHeld = true
    end
end)
UserInputService.InputEnded:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.Space then
        spaceHeld = false
    end
end)

-- === BHOP & WALL HOP LOGIC ===
RunService.Heartbeat:Connect(function()
    performWallHop()
    if not bhopEnabled or not spaceHeld or not character or not rootPart then return end
    local hum = character:FindFirstChild("Humanoid")
    if not hum or hum:GetState() == Enum.HumanoidStateType.Dead then return end
    if hum.FloorMaterial ~= Enum.Material.Air then
        hum:ChangeState(Enum.HumanoidStateType.Jumping)
    end
    local moveDir = hum.MoveDirection
    if moveDir.Magnitude < 0.1 then return end
    local forwardDir = rootPart.CFrame.LookVector * Vector3.new(1,0,1)
    local move2D = Vector2.new(moveDir.X, moveDir.Z)
    local forward2D = Vector2.new(forwardDir.X, forwardDir.Z)
    if move2D:Dot(forward2D) < 0.7 then return end
    local currentSpeed = rootPart.Velocity * Vector3.new(1,0,1)
    local forwardSpeed = currentSpeed:Dot(forwardDir)
    if forwardSpeed < bhopSpeedValue then
        local boost = forwardDir * FORWARD_ACCEL
        rootPart.Velocity = rootPart.Velocity + boost
    end
end)

-- === FLY CONTROL ===
local function handleFly()
    if not flyActive or not bodyVelocity or not rootPart then return end
    local cam = workspace.CurrentCamera
    local move = Vector3.new()
    local UIS = UserInputService
    if UIS:IsKeyDown(Enum.KeyCode.W) then move = move + cam.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.S) then move = move - cam.CFrame.LookVector end
    if UIS:IsKeyDown(Enum.KeyCode.A) then move = move - cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.D) then move = move + cam.CFrame.RightVector end
    if UIS:IsKeyDown(Enum.KeyCode.Space) then move = move + Vector3.new(0,1,0) end
    if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then move = move + Vector3.new(0,-1,0) end
    if move.Magnitude > 0 then
        bodyVelocity.Velocity = move.Unit * flySpeedValue
    else
        bodyVelocity.Velocity = Vector3.new()
    end
end

-- === RENDER LOOP ===
RunService.RenderStepped:Connect(function()
    if rootPart then
        local pos = rootPart.Position
        coordsLabel.Text = string.format("X: %.1f | Y: %.1f | Z: %.1f", pos.X, pos.Y, pos.Z)
    end
    if spinning and rootPart then
        rootPart.CFrame = rootPart.CFrame * CFrame.Angles(0, math.rad(SPIN_SPEED), 0)
    end

    -- Speed Meter
    if speedMeterEnabled and rootPart then
        speedLabel.Visible = true
        local speed = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z).Magnitude
        speedLabel.Text = string.format("%.1f", speed)
    else
        speedLabel.Visible = false
    end

    -- Velocity Graph (WHITE LINE, HIGHER)
    if velocityGraphEnabled and rootPart then
        graphFrame.Visible = true
        updateVelocityLine()
    else
        graphFrame.Visible = false
    end

    handleFly()
end)

-- === RESPAWN HANDLING ===
LocalPlayer.CharacterAdded:Connect(function(newChar)
    task.wait(0.5)
    updateCharacter()
    if spinbotEnabled then
        spinning = true
        if humanoid then humanoid.AutoRotate = false end
    end
    if flyEnabled and rootPart then
        flyActive = true
        if bodyVelocity then bodyVelocity:Destroy() end
        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new()
        bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
        bodyVelocity.P = 1000
        bodyVelocity.Parent = rootPart
        humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown,false)
        humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,false)
        humanoid:ChangeState(Enum.HumanoidStateType.Flying)
    end
    if noclipEnabled then
        if noclipConnection then noclipConnection:Disconnect() end
        noclipConnection = RunService.Stepped:Connect(function()
            if not noclipEnabled or not character then return end
            for _, p in ipairs(character:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end)
    end
    if thirdPersonEnabled then
        camera.CameraType = Enum.CameraType.Scriptable
        UserInputService.MouseBehavior = Enum.MouseBehavior.Default
        UserInputService.MouseIconEnabled = true
        task.wait(0.1)
        for _, p in ipairs(character:GetDescendants()) do
            if p:IsA("BasePart") then p.LocalTransparencyModifier = 0 end
        end
        camYaw = 0; camPitch = 0
        if thirdPersonConn then thirdPersonConn:Disconnect() end
        if mouseDeltaConn then mouseDeltaConn:Disconnect() end
        mouseDeltaConn = UserInputService.InputChanged:Connect(function(i)
            if not thirdPersonEnabled or i.UserInputType ~= Enum.UserInputType.MouseMovement then return end
            local d = i.Delta
            camYaw = camYaw - d.X * CAMERA_SENSITIVITY
            camPitch = math.clamp(camPitch - d.Y * CAMERA_SENSITIVITY, -80, 80)
        end)
        thirdPersonConn = RunService.RenderStepped:Connect(function()
            if not thirdPersonEnabled or not character or not rootPart then return end
            local head = character:FindFirstChild("Head")
            local focus = head and head.Position or (character:FindFirstChild("UpperTorso") or rootPart).Position + Vector3.new(0,1.5,0)
            local cf = CFrame.new(focus)
                * CFrame.Angles(0, math.rad(camYaw), 0)
                * CFrame.Angles(math.rad(camPitch), 0, 0)
                * CFrame.new(0, THIRD_PERSON_HEIGHT, THIRD_PERSON_DISTANCE)
            camera.CFrame = camera.CFrame:lerp(cf, 0.25)
            camera.FieldOfView = 70
        end)
    end
end)

print("expjector 3008 2.80 – BHOP + WALL HOP + WHITE LINE VELOCITY GRAPH | LOADED")