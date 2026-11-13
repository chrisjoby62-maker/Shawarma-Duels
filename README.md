
--========================================================
-- SHAWARMA HUB | SHAWARMA  HUB
-- LocalScript: StarterPlayer > StarterPlayerScripts
--========================================================

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local previous = playerGui:FindFirstChild("ShawarmaHub")
if previous then previous:Destroy() end

local flying = false
local locked = false
local speed = 65

local character, humanoid, root
local attachment, linearVelocity
local flightConnection

local function getCharacter(char)
    character = char or player.Character or player.CharacterAdded:Wait()
    humanoid = character:WaitForChild("Humanoid")
    root = character:WaitForChild("HumanoidRootPart")
end

getCharacter()

-- PALETTE
local DARK = Color3.fromRGB(24, 17, 14)
local PANEL = Color3.fromRGB(39, 27, 21)
local BROWN = Color3.fromRGB(83, 49, 30)
local GOLD = Color3.fromRGB(218, 166, 91)
local CREAM = Color3.fromRGB(255, 231, 194)
local MUTED = Color3.fromRGB(178, 145, 112)
local GREEN = Color3.fromRGB(111, 190, 124)
local RED = Color3.fromRGB(191, 83, 65)

local function make(class, properties, parent)
    local obj = Instance.new(class)
    for key, value in pairs(properties) do
        obj[key] = value
    end
    obj.Parent = parent
    return obj
end

-- GUI
local gui = make("ScreenGui", {
    Name = "ShawarmaHub",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, playerGui)

local main = make("Frame", {
    Name = "Main",
    Size = UDim2.fromOffset(330, 270),
    Position = UDim2.new(0.5, -165, 0.5, -135),
    BackgroundColor3 = DARK,
    BorderSizePixel = 0
}, gui)

make("UICorner", {CornerRadius = UDim.new(0, 18)}, main)
make("UIStroke", {
    Color = GOLD,
    Thickness = 1.6,
    Transparency = 0.08
}, main)

make("UIGradient", {
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(48, 32, 23)),
        ColorSequenceKeypoint.new(1, DARK)
    }),
    Rotation = 35
}, main)

-- HEADER
local header = make("Frame", {
    Size = UDim2.new(1, 0, 0, 70),
    BackgroundColor3 = BROWN,
    BorderSizePixel = 0
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 18)}, header)

local headerBottom = make("Frame", {
    Size = UDim2.new(1, 0, 0, 16),
    Position = UDim2.new(0, 0, 1, -16),
    BackgroundColor3 = BROWN,
    BorderSizePixel = 0
}, header)

make("TextLabel", {
    Size = UDim2.new(1, -110, 0, 31),
    Position = UDim2.fromOffset(17, 8),
    BackgroundTransparency = 1,
    Text = "SHAWARMA",
    TextColor3 = CREAM,
    Font = Enum.Font.GothamBlack,
    TextSize = 20,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

make("TextLabel", {
    Size = UDim2.new(1, -110, 0, 17),
    Position = UDim2.fromOffset(18, 38),
    BackgroundTransparency = 1,
    Text = "✦  PREMIUM FLIGHT CLUB",
    TextColor3 = GOLD,
    Font = Enum.Font.GothamMedium,
    TextSize = 10,
    TextXAlignment = Enum.TextXAlignment.Left
}, header)

local function headerButton(text, position, color)
    local b = make("TextButton", {
        Size = UDim2.fromOffset(34, 32),
        Position = position,
        BackgroundColor3 = color,
        Text = text,
        TextColor3 = CREAM,
        Font = Enum.Font.GothamBold,
        TextSize = 15,
        AutoButtonColor = true
    }, header)
    make("UICorner", {CornerRadius = UDim.new(0, 9)}, b)
    make("UIStroke", {Color = GOLD, Transparency = 0.65}, b)
    return b
end

local lockButton = headerButton("🔓", UDim2.new(1, -82, 0, 18), PANEL)
local closeButton = headerButton("×", UDim2.new(1, -42, 0, 18), RED)

-- STATUS
make("TextLabel", {
    Size = UDim2.new(1, -34, 0, 20),
    Position = UDim2.fromOffset(17, 82),
    BackgroundTransparency = 1,
    Text = "FLIGHT STATUS",
    TextColor3 = MUTED,
    Font = Enum.Font.GothamBold,
    TextSize = 10,
    TextXAlignment = Enum.TextXAlignment.Left
}, main)

local status = make("TextLabel", {
    Size = UDim2.new(1, -34, 0, 25),
    Position = UDim2.fromOffset(17, 101),
    BackgroundTransparency = 1,
    Text = "●  FLIGHT OFF",
    TextColor3 = RED,
    Font = Enum.Font.GothamBold,
    TextSize = 15,
    TextXAlignment = Enum.TextXAlignment.Left
}, main)

-- FLIGHT TOGGLE
local toggle = make("TextButton", {
    Size = UDim2.new(1, -34, 0, 49),
    Position = UDim2.fromOffset(17, 139),
    BackgroundColor3 = GOLD,
    Text = "✈     ACTIVATE FLIGHT",
    TextColor3 = DARK,
    Font = Enum.Font.GothamBlack,
    TextSize = 15,
    AutoButtonColor = true
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 11)}, toggle)

-- SPEED
local speedButton = make("TextButton", {
    Size = UDim2.new(1, -34, 0, 38),
    Position = UDim2.fromOffset(17, 198),
    BackgroundColor3 = PANEL,
    Text = "FLIGHT SPEED     •     " .. speed,
    TextColor3 = CREAM,
    Font = Enum.Font.GothamBold,
    TextSize = 12
}, main)

make("UICorner", {CornerRadius = UDim.new(0, 9)}, speedButton)
make("UIStroke", {Color = GOLD, Transparency = 0.7}, speedButton)

make("TextLabel", {
    Size = UDim2.new(1, 0, 0, 18),
    Position = UDim2.fromOffset(0, 245),
    BackgroundTransparency = 1,
    Text = "SHAWARMA HUB  •  EST. 2026",
    TextColor3 = MUTED,
    Font = Enum.Font.GothamMedium,
    TextSize = 9
}, main)

-- MINIMISED BUTTON
local reopen = make("TextButton", {
    Size = UDim2.fromOffset(62, 62),
    Position = UDim2.new(0, 18, 0.5, -31),
    BackgroundColor3 = BROWN,
    Text = "🌯",
    TextSize = 29,
    Visible = false
}, gui)

make("UICorner", {CornerRadius = UDim.new(1, 0)}, reopen)
make("UIStroke", {Color = GOLD, Thickness = 2}, reopen)

closeButton.MouseButton1Click:Connect(function()
    main.Visible = false
    reopen.Visible = true
end)

reopen.MouseButton1Click:Connect(function()
    main.Visible = true
    reopen.Visible = false
end)

-- DRAGGING
local dragging = false
local dragStart, startPosition

header.InputBegan:Connect(function(input)
    if locked then return end

    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if not dragging or locked then return end

    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

lockButton.MouseButton1Click:Connect(function()
    locked = not locked
    lockButton.Text = locked and "🔒" or "🔓"
end)

-- FLIGHT CLEANUP
local function stopFlight()
    flying = false

    if flightConnection then
        flightConnection:Disconnect()
        flightConnection = nil
    end

    if linearVelocity then
        linearVelocity:Destroy()
        linearVelocity = nil
    end

    if attachment then
        attachment:Destroy()
        attachment = nil
    end

    if humanoid then
        humanoid.AutoRotate = true
    end

    toggle.Text = "✈     ACTIVATE FLIGHT"
    toggle.BackgroundColor3 = GOLD
    toggle.TextColor3 = DARK
    status.Text = "●  FLIGHT OFF"
    status.TextColor3 = RED
end

-- START FLIGHT
local function startFlight()
    getCharacter()

    if not root or not humanoid or humanoid.Health <= 0 then
        return
    end

    flying = true

    -- Attachment and LinearVelocity only.
    -- No BodyGyro, AlignOrientation or CFrame rotation.
    attachment = Instance.new("Attachment")
    attachment.Name = "ShawarmaFlightAttachment"
    attachment.Parent = root

    linearVelocity = Instance.new("LinearVelocity")
    linearVelocity.Name = "ShawarmaFlightVelocity"
    linearVelocity.Attachment0 = attachment
    linearVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
    linearVelocity.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
    linearVelocity.MaxForce = 1000000
    linearVelocity.VectorVelocity = Vector3.zero
    linearVelocity.Parent = root

    humanoid.AutoRotate = false

    toggle.Text = "■     STOP FLIGHT"
    toggle.BackgroundColor3 = RED
    toggle.TextColor3 = CREAM
    status.Text = "●  FLIGHT ACTIVE"
    status.TextColor3 = GREEN

    flightConnection = RunService.Heartbeat:Connect(function()
        if not flying or not root or not root.Parent
        or not humanoid or humanoid.Health <= 0 then
            stopFlight()
            return
        end

        local move = humanoid.MoveDirection
        local vertical = 0

        if UIS:IsKeyDown(Enum.KeyCode.Space) or humanoid.Jump then
            vertical += 1
        end

        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
            vertical -= 1
        end

        -- Joystick movement stays horizontal.
        -- Camera pitch does not rotate the character.
        local direction = Vector3.new(move.X, vertical, move.Z)

        if direction.Magnitude > 1 then
            direction = direction.Unit
        end

        linearVelocity.VectorVelocity = direction * speed
    end)
end

toggle.MouseButton1Click:Connect(function()
    if flying then
        stopFlight()
    else
        startFlight()
    end
end)

speedButton.MouseButton1Click:Connect(function()
    speed += 15
    if speed > 125 then
        speed = 35
    end
    speedButton.Text = "FLIGHT SPEED     •     " .. speed
end)

player.CharacterAdded:Connect(function(newCharacter)
    stopFlight()
    character = newCharacter
    humanoid = newCharacter:WaitForChild("Humanoid")
    root = newCharacter:WaitForChild("HumanoidRootPart")
end)

print("SHAWARMA HUB PREMIUM LOADED")
